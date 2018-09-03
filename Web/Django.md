# Django跨域请求问题
```python
# setting.py
INSTALLED_APPS = [
    ···
    'rest_framework',
    # 跨域
    'corsheaders', 
]

MIDDLEWARE_CLASSES = [
    ···
    # 支持跨域
    'corsheaders.middleware.CorsMiddleware',
    # 顺序不能颠倒
    'django.middleware.common.CommonMiddleware', 
]

# 我为了方便就直接写成了*
CORS_ORIGIN_WHITELIST = (
    '*', 
)
```
# Django ORM
## aggregate 和 annotate

通过 aggregate 和 annotate 可以使用 SQL 的聚合函数，例如 SUM、COUNT、MIN 等。aggregate: 针对所有记录调用聚合函数，返回一个 dict 对象，下面是使用示例：
```python
from django.db.models import Min
from django.db.models import Sum

result = Blog.objects.aggregate(Min('id'))
print result
# {u'id__min': 3L}
# SELECT MIN(orm_blog.id) AS id__min FROM orm_blog
# 自定义属性名
result = Blog.objects.aggregate(total=Sum('id'))
print result
# {'total': Decimal(‘657')}
# SELECT SUM(orm_blog.id) AS total FROM orm_blog
```
annotate 先使用 groupby 分组，然后对于每组再调用聚合函数，返回 QuerySet 对象。 annotate 默认按照 id 进行分组，如果需要按其他字段分组，要结合 values ／values_list 方法。下面是使用示例：
```python
#认按照 id 进行分组
blogs = Blog.objects.annotate(Count('title'))
for blog in blogs:
    print blog.title__count
# SELECT orm_blog.id,  orm_blog.author_id,  orm_blog.title,  orm_blog.content, 
# COUNT(orm_blog.title) AS title__count FROM orm_blog GROUP BY orm_blog.id ORDER BY NULL
# 使用 values 方法，会按照 values 中传入的属性分组
blogs = Blog.objects.values('title').annotate(Count('title'))
for blog in blogs:
   print blog['title__count']
# SELECT orm_blog.title,  COUNT(orm_blog.title) AS title__count FROM orm_blog
# GROUP BY orm_blog.title ORDER BY NULL
blogs = Blog.objects.values('title', 'content').annotate(Count('title'))
for blog in blogs:
   print blog['title__count']
# SELECT orm_blog.title, orm_blog.content,  COUNT(orm_blog.title) AS title__count
# FROM orm_blog GROUP BY orm_blog.title,  orm_blog.content ORDER BY NULL
```
## extra

如何一些查询比较复杂可以考虑使用 extra 方法。extra 能在 ORM 生成的 SQL 子句中注入 SQL 代码，语法格式如下：
```python
# 至少保证一个参数不为空
extra(select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
```

*   select：在 select 子句中插入 SQL 代码
    ```python
     blogs = Blog.objects.extra(select={ ‘is_one’: ‘id=5’ })
     # SELECT (id=5) AS is_one,  orm_blog.id. . . 
     authors = Author.objects.extra(select={
     'blog_count' : 'select count(*) from orm_blog where orm_blog.author_id = orm_author.id'})
     # SELECT (select count(*) from orm_blog where orm_blog.author_id = orm_author.id) AS blog_count . . .
    ```
*   select_params: 设置 select 参数
    ```python
    blogs = Blog.objects.extra(
    select={'a': '%s'}, select_params=(‘id=1',)
     # SELECT ('id=1') AS a . . .
    )
    ```
*   where: 在 where 子句中插入 SQL 代码
    ```python
    blogs = Blog.objects.extra(where=['id=4 or id=5'])
    # SELECT . . . . . . FROM orm_blog WHERE (id=4 or id=5)
    ```
*   params: 为 where 设置参数
    ```python
    blogs = Blog.objects.extra(where=['id=%s'],params=(1,))
    # SELECT . . . . . . FROM orm_blog WHERE (id=1)
    ```
*   tables: 在 FROM 子句中插入 table 名称
    ```python
    blogs = Blog.objects.extra(tables=['orm_author', 'auth_group'])
    # SELECT . . . . . . FROM orm_blog ,  orm_author ,  auth_group
    ```
*   order_by：在 order_by 子句中插入排序字段
    ```python
    # - 表示倒序
    blogs = Blog.objects.extra(order_by=['-id', 'title'])
    # SELECT . . . . . . FROM orm_blog ORDER BY orm_blog.id DESC,  orm_blog.title ASC
    ```
