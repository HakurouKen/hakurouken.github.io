---
title: 打造自己的 django-inspectdb 命令
tags:
  - python
  - django
date: 2017-01-03 00:07:16
---


Django 提供了一个 inspectdb 的指令，可以通过内省现有的数据库来创建 django 的 ORM models。执行起来也很简单:
```bash
python manage.py inspectdb --database YOUR_DB
```
但是实际上，这个 inspectdb 并没有给太多定制化的选项，并不能满足所有的场景，举几个我在业务中遇到的问题：
1. 不能把 DB column 的注释直接输出到 help_text 字段
2. 没有把表注释转化为 Model 类的 docstring
3. Model 类名直接 capitalize 转化，而我们业务比较繁杂，表名大多比较长，用大驼峰更适合
4. 同样的道理，我们希望把列名规范成下划线连接的方式

鉴于以上原因，我决定基于原有根据业务改一个新的 inspectdb。
<!-- more -->

## inspectdb 工作过程
inspectdb 的源代码位于 `django/core/management/commands/inspectdb.py`下，可以在 [Github](https://github.com/django/django/blob/master/django/core/management/commands/inspectdb.py) 上看到其源码。对于 django-admin 的命令，执行入口在 handle 方法。如果对 django-command 还不了解的话，可以看官方文档的[Custom Command](https://docs.djangoproject.com/en/1.10/howto/custom-management-commands/)。粗略浏览一遍，代码的大致结构如下：
```python
class Command(BaseCommand):
    # 类变量声明
    def add_arguments(self, parser):
        # 解析参数
        pass

    def handle(self, **options):
        try:
            for line in self.handle_inspection(options):
                self.stdout.write("%s\n" % line)
        except NotImplementedError:
            raise CommandError("Database inspection isn't supported for the currently selected database backend.")

    def handle_inspection(self, options):
        # 主体的生成处理逻辑
        connection = connections[options['database']]
        table_name_filter = options.get('table_name_filter')

        # 表名转成 model 类名
        def table2model(table_name):
            return re.sub(r'[^a-zA-Z0-9]', '', table_name.title())

        # 取到对应链接的游标
        # 这里的 connection (django.db.connection)是 django.db.utils.ConnectionHandler 的实例
        # 默认使用 settings.DATABASES 初始化，源代码在
        #   https://github.com/django/django/blob/master/django/db/utils.py#L144
        # 如果需要自定义 DB 而不使用 settings.DATABASES 的配置，这里可以改为自己新建的实例
        with connection.cursor() as cursor:
            # 输出公共头信息
            known_models = []
            tables_to_introspect = options['table'] or connection.introspection.table_names(cursor)

            # 遍历所有(需要的) table
            for table_name in tables_to_introspect:
                # 获取 table 的信息，包括表名，列名，列信息等等
                # 输出每个表的头，即 `class Tablename(models.Models):`
                known_models.append(table2model(table_name))
                used_column_names = []
                column_to_field_name = {}

                # 遍历每一列
                for row in table_description:
                    # 对于每一列，标准化列名
                    att_name, params, notes = self.normalize_col_name(
                        column_name, used_column_names, is_relation)
                    # 对于每一列，判断输出不同的属性（最麻烦的部分之一）

                for meta_line in self.get_meta(table_name, constraints, column_to_field_name):
                    yield meta_line

    def normalize_col_name(self, col_name, used_column_names, is_relation):
        # 格式化数据库列名为 Python 的字段名
        return new_name, field_params, field_notes

    def get_field_type(self, connection, table_name, row):
        # 将 DB 的类型映射到不同的 class，比如 int -> IntegerField
        return field_type, field_params, field_notes

    def get_meta(self, table_name, constraints, column_to_field_name):
        # 根据指定的 APP/表的名称返回表的元信息代码 (Meta Class)
        return meta
```
知道了这些，再对源代码做修改，就容易很多了。

## 修改过程中遇到问题
inspectdb 执行过程中，不同的数据库用到的数据库检查代码也不同，分别位于 `django/db/backends/<db>/introspection.py` 中。由于我们的后台数据库全部是 mysql，因此这里讨论 DB 相关的部分也全部针对 mysql。其它 DB 相关信息可以去对应的 backends 中查找。

针对 Mysql 获取表/列字段信息基本都是使用 `information_schema` 这个大杀器。例如 django 内部获取表信息时用到的 `get_table_description`:
```python
def get_table_description(self, cursor, table_name):
    """
    Returns a description of the table, with the DB-API cursor.description interface."
    """
    cursor.execute("""
        SELECT column_name, data_type, character_maximum_length, numeric_precision,
               numeric_scale, extra, column_default
        FROM information_schema.columns
        WHERE table_name = %s AND table_schema = DATABASE()""", [table_name])
    field_info = {line[0]: InfoLine(*line) for line in cursor.fetchall()}

    cursor.execute("SELECT * FROM %s LIMIT 1" % self.connection.ops.quote_name(table_name))

    def to_int(i):
        return int(i) if i is not None else i

    fields = []
    for line in cursor.description:
        col_name = force_text(line[0])
        fields.append(
            FieldInfo(*(
                (col_name,) +
                line[1:3] +
                (
                    to_int(field_info[col_name].max_len) or line[3],
                    to_int(field_info[col_name].num_prec) or line[4],
                    to_int(field_info[col_name].num_scale) or line[5],
                    line[6],
                    field_info[col_name].extra,
                    field_info[col_name].column_default,
                )
            ))
        )
    return fields
```
现在我需要显示表注释，首先得查询得到表注释，需要对这个方法进行扩展。大概会有3种方案：
1. 直接改源码，SELECT 的时候加一列 `table_comment`，返回的 field 加上一列即可。
2. 写一个类继承 `mysql.introspection.DatabaseIntrospection`，然后把对应的方法重写。
3. 需要的时候，多查询一遍 DB。

直接修改库源码是愚蠢的行为，方案1 直接PASS。剩下就是在方案 2/3 中做一个抉择。方案2 似乎看上去很优雅，而且解决方案非常工程化，但是实施的时候又出现了问题：如果要只查询一次DB，我们只需要改一下 SELECT 相关部分即可。但是剩下的逻辑怎么办？似乎需要照抄。算下来，大概复制100行代码出来，后续如果升级，还要额外关注这部分代码有没有变动。最麻烦的是，这个相对底层的代码，django 官方并没有文档支持，这个自定义的 "db-engine" 表现会不会和其它的不一致，升级后 API 有没有产生变化，这些都要自己去判断。
鉴于这个问题，最终退而求其次选择了方案3，原因是逻辑简单（简单写两个函数即可），而且 inspectdb 并不是高频需求，实际执行起来，也没有明显变慢。简单的 hack 代码如下：
```python
def is_mysql_cursor(cursor):
    ''' 检查 cursor 对应的 backends 是否为 mysql '''
    return isinstance(cursor.db,backends.mysql.base.DatabaseWrapper)

def get_table_comment(cursor, table_name):
    ''' 获取表注释,仅支持 mysql '''
    if not is_mysql_cursor(cursor):
        return
    cursor.execute('''
        SELECT table_comment
        FROM information_schema.tables
        WHERE table_name = %s AND table_schema = DATABASE()
    ''',[table_name])
    tb = cursor.fetchone()
    return tb[0] if tb else None

def get_column_comment(cursor, table_name):
    ''' 查询并返回 table 对应的注释，对于非 mysql 的游标，返回空 dict '''
    if not is_mysql_cursor(cursor):
        return {}

    cursor.execute("""
        SELECT column_name, column_comment
        FROM information_schema.columns
        WHERE table_name = %s AND table_schema = DATABASE()""", [table_name])
    return {line[0]: line[1] for line in cursor.fetchall()}
```
后续的逻辑比较简单，比如添加我将表注释导出为 Class 的 docstring，只需在[列名输出之后](https://github.com/django/django/blob/master/django/core/management/commands/inspectdb.py#L91)添加：
```python
table_comment = get_table_comment(cursor,table_name)
if table_comment:
    yield '    """ %s """' % table_comment
```
想要把类名改为大驼峰，只需将 [table2model](https://github.com/django/django/blob/master/django/core/management/commands/inspectdb.py#L42) 改为类似这样:
```python
def table2model(table_name):
    ''' 表名转化为 model 名(去除特殊字符的大驼峰) '''
    snake = re.sub(r'\W','', table_name)
    # 这里不用 capitalize() 方法，（如果之前是驼峰，保持之前驼峰中大写的字母还是大写）
    return ''.join(word[:1].upper() + word[1:] for word in snake.split('_'))
```
其它定制化也类似，整体代码结构也很清晰易懂，不再赘述。
