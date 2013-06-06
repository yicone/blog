原则：
- 避免使用硬编码的URL，如Url.Action("Product", new { productId = 1})
- 如果有多个Route处理同一类URL，则使用Url.RouteUrl，明确指定routeName；否则使用Url.Action

包括ajax需要用到的URL也要尽量避免因route规则改写，导致访问失败

通过UrlHelper的扩展方法提供
------------------

- 绝对地址  
如Url.AbsoluteAction、Url.AbsoluteContent、Url.AbosluteRouteUrl等
- 站内常用链接  
如Url.Index, Url.Product(productId)

Route相关
-------
url转小写，可以通过继承RouteBase重载GetVirtualPath方法来实现
http://www.cnblogs.com/John-Connor/archive/2012/05/03/2478821.html
但route中Url的非程序生成的部分，不在其影响范围内，需要手工处理为小写
如 todo

上文还提供了语义化的方法，但我不赞成在GetVirtualPath中来读取数据用以生成表达语义的部分，而应通过Url.Action的routeValue参数来传递，随需而做，既不浪费，也减少了不必要的耦合

nuget里
http://nuget.org/packages/LowercaseRoutesMVC/

.NET 4.5提供了一个[方式](http://msdn.microsoft.com/en-us/library/system.web.routing.routecollection.lowercaseurls.aspx)，但对于query string部分也有个 **注意事项**
> If a query string is included in the URL, that part of the URL is not converted to lower case

多个可选参数的处理
http://haacked.com/archive/2011/02/20/routing-regression-with-two-consecutive-optional-url-parameters.aspx

老route匹配的url, 301跳转到新route
http://haacked.com/archive/2011/02/02/redirecting-routes-to-maintain-persistent-urls.aspx


辅助编写Route的工具
------------
http://haacked.com/archive/2008/03/13/url-routing-debugger.aspx (学ASP.NET MVC 1的时代用的就是这个）
**Cobisi Routing Assistant** 神器
http://visualstudiogallery.msdn.microsoft.com/f0589156-a8e6-47db-8bac-90f01ca6b8a3

Route还有不少其他的扩展点 （同样来自Phil Haack）
http://haacked.com/archive/2011/01/30/introducing-routemagic.aspx

网站扫描工具
-------
主要是从SEO的角度，检查坏链、缺少描述的元素(a, img)

推荐：
http://www.iis.net/learn/extensions/iis-search-engine-optimization-toolkit
其他：
http://sourceforge.net/projects/paros/ （需要安装JRE） [介绍文章](http://blog.csdn.net/tzh2009/article/details/6427571)
http://www.totalvalidator.com
http://www.the-escape.co.uk/tools/pageanalyzer/