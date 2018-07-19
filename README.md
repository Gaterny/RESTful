#### RESTful api相关
##### 1.HTTP方法
```
GET（SELECT）：从服务器取出资源（一项或多项）
POST（CREATE）：在服务器新建一个资源
PUT（UPDATE）：在服务器更新资源（客户端提供完整资源数据）
PATCH（UPDATE）：在服务器更新资源（客户端提供需要修改的资源数据）
DELETE（DELETE）：从服务器删除资源
```
##### 2.response包含的数据和状态码
```
当GET, PUT和PATCH请求成功时，要返回对应的数据，及状态码200，即SUCCESS
当POST创建数据成功时，要返回创建的数据，及状态码201，即CREATED
当DELETE删除数据成功时，不返回数据，状态码要返回204，即NO CONTENT
当GET 不到数据时，状态码要返回404，即NOT FOUND
任何时候，如果请求有问题，如校验请求数据时发现错误，要返回状态码 400，即BAD REQUEST
当API 请求需要用户认证时，如果request中的认证信息不正确，要返回状态码 401，即NOT AUTHORIZED
当API 请求需要验证用户权限时，如果当前用户无相应权限，要返回状态码 403，即FORBIDDEN
```
关于Request 和 Response，不要忽略了http header中的Content-Type,例如json的Content-Type=application/json

##### 3.序列化和反序列化
```
Serialization 和 Deserialization即序列化和反序列化.将native类型数据转为json即为序列化，将json转为native类型数据即为反序列化
```

##### 4.数据校验
Validation即数据校验，当客户端向服务器发出post, put或patch请求时，通常会同时给服务器发送json格式的相关数据，服务器在做数据处理之前，先做数据校验，是最合理和安全的前后端交互。如果客户端发送的数据不正确或不合理，服务器端经过校验后直接向客户端返回400错误及相应的数据错误信息即可.常见的校验方式：
```
数据类型校验，如字段类型如果是int，那么给字段赋字符串的值则报错
数据格式校验，如邮箱或密码，其赋值必须满足相应的正则表达式，才是正确的输入数据
数据逻辑校验，如数据包含出生日期和年龄两个字段，如果这两个字段的数据不一致，则数据校验失败
```

##### 5.用户认证和权限机制
```
Authentication指用户认证，Permission指权限机制。常用的认证机制是Basic Auth和OAuth
```

##### 6.跨域资源共享
```
CORS即Cross-origin resource sharing，在RESTful API开发中，主要是为js服务的，解决javascript 调用 RESTful API时的跨域问题。

由于固有的安全机制，js的跨域请求时是无法被服务器成功响应的。现在前后端分离日益成为web开发主流方式的大趋势下，后台逐渐趋向指提供API服务，为各客户端提供数据及相关操作，而网站的开发全部交给前端搞定，网站和API服务很少部署在同一台服务器上并使用相同的端口，js的跨域请求时普遍存在的，开发RESTful API时，通常都要考虑到CORS功能的实现，以便js能正常使用API。
```

##### 7.URL命名规则
7.1 规范的API应该包含版本信息，在RESTful API中，最简单的包含版本的方法是将版本信息放到url中，如：
```
/api/v1/posts/
/api/v2/posts/
```
7.2 RESTful API 中的url是指向资源的，而不是描述行为的，因此设计API时，应使用名词而非动词来描述语义，否则会引起混淆和语义不清。即：
```
# Bad APIs
/api/getArticle/1/
/api/updateArticle/1/
/api/deleteArticle/1/
上面四个url都是指向同一个资源的，虽然一个资源允许多个url指向它，但不同的url应该表达不同的语义，上面的API可以优化为：
# Good APIs
/api/Article/1/

article 资源的获取、更新和删除分别通过 GET, PUT 和 DELETE方法请求API即可。试想，如果url以动词来描述，用PUT方法请求 /api/deleteArticle/1/ 会感觉多么不舒服。
```
7.3 GET and HEAD should always be safe
RFC2616已经明确指出，GET和HEAD方法必须始终是安全的。例如，有这样一个不规范的API:
```
# The following api is used to delete articles
# [GET]
/api/deleteArticle?id=1
```
7.4 Nested resources routing
如果要获取一个资源子集，采用 nested routing 是一个优雅的方式，如，列出所有文章中属于Gevin编写的文章：
```
# List Gevin's articles
/api/authors/gevin/articles/
```
获取资源子集的另一种方式是基于filter（见下面章节），这两种方式都符合规范，但语义不同：如果语义上将资源子集看作一个独立的资源集合，则使用 nested routing 感觉更恰当，如果资源子集的获取是出于过滤的目的，则使用filter更恰当。

7.5 Filter
对于资源集合，可以通过url参数对资源进行过滤，如：
```
# List Gevin's articles
/api/articles?author=gevin
```

分页就是一种最典型的资源过滤。

7.6 Pagination
对于资源集合，分页获取是一种比较合理的方式。如果基于开发框架（如Django REST Framework），直接使用开发框架中的分页机制即可，如果是自己实现分页机制，策略是：

返回资源集合是，包含与分页有关的数据如下：
```
{
  "page": 1,            # 当前是第几页
  "pages": 3,           # 总共多少页
  "per_page": 10,       # 每页多少数据
  "has_next": true,     # 是否有下一页数据
  "has_prev": false,    # 是否有前一页数据
  "total": 27           # 总共多少数据
}
```
当想API请求资源集合时，可选的分页参数为：
```
参数	含义
page	当前是第几页，默认为1
per_page	每页多少条记录，默认为系统默认值
另外，系统内还设置一个per_page_max字段，用于标记系统允许的每页最大记录数，当per_page值大于 per_page_max 值时，每页记录条数为 per_page_max。
```
7.7 Url design tricks
（1）Url是区分大小写的，这点经常被忽略，即：
```
/Posts
/posts
上面这两个url是不同的两个url，可以指向不同的资源
```
（2）Back forward Slash (/)

目前比较流行的API设计方案，通常建议url以/作为结尾，如果API GET请求中，url不以/结尾，则重定向到以/结尾的API上去（这点现在的web框架基本都支持），因为有没有 /，也是两个url，即：
```
/posts/
/posts
这也是两个不同的url，可以对应不同的行为和资源
```
（3）连接符 - 和 下划线 _

RESTful API 应具备良好的可读性，当url中某一个片段（segment）由多个单词组成时，建议使用 - 来隔断单词，而不是使用 _，即：
```
# Good
/api/featured-post/

# Bad
/api/featured_post/
```
这主要是因为，浏览器中超链接显示的默认效果是，文字并附带下划线，如果API以_隔断单词，二者会重叠，影响可读性。
