System.Web.Optimization是在ASP.NET 4.5中正式引入的。  

## 在MVC 3中如何使用Bundling and Minification机制 ##
个人建议的方法，是在VS中建一个MVC 4的项目，看看`Global.asax.cs`里多了什么，看看`App_Start`目录下多了什么。
>**也可参考**  
1. [内容已过期]http://joeyiodice.com/using-mvc4-minification-and-bundling-in-mvc3   
2. [http://forums.asp.net/t/1799992.aspx/1](http://forums.asp.net/t/1799992.aspx/1) 

## 重要特性总结 ##
(待补充)  
1. 对bundle的请求不受`OutputCache`影响。对`~/bundleVirtualPath` 的请求，总是会根据内存中`BundleTable.Bundles` 所定义的映射，定位物理文件，并响应到客户端。  
2. 对物理文件修改后，文件变化可以立刻输出。  
3. 文件内容相同的请求，`v`参数的值不变，因此可能会使用上浏览器缓存。  
4. bundle不能定义对非同域的文件的引用。能否定义同域的但路径为虚拟路径的文件的引用，未知。  

## 动态Bundle ##
<script type="text/javascript" src="https://gist.github.com/4201780.js?file=HtmlExtensions.cs"></script>

> 参考资料  
1. [**asp-net-mvc-4-use-bundles-beneficts-for-url-content**](http://stackoverflow.com/questions/13124218/asp-net-mvc-4-use-bundles-beneficts-for-url-content?rq=1)  
2. [runtime-bundling-and-minifying-in-mvc-4](http://stackoverflow.com/questions/10614441/runtime-bundling-and-minifying-in-mvc-4)  
3. [a-localized-scriptbundle-solution](http://stackoverflow.com/questions/10978979/a-localized-scriptbundle-solution)  
4. [在MVC 4中使用的官方教程](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification) 
