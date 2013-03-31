`System.Web.Optimization`是在`ASP.NET 4.5`中正式引入的。  

## 1. 在`MVC 3`中如何使用Bundling and Minification机制 ##
个人建议的方法，是在VS中建一个`MVC 4`的项目，看看`Global.asax.cs`里多了什么，看看`App_Start`目录下多了什么。

**MvcApplication.Application_Start() in Global.ascx.cs**

	BundleTable.EnableOptimizations = true;  
	BundleConfig.RegisterBundles(BundleTable.Bundles);

> [内容已过期] http://joeyiodice.com/using-mvc4-minification-and-bundling-in-mvc3
> http://forums.asp.net/t/1799992.aspx/1
> [在MVC 4中使用的官方教程](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification) 


## 2. 重要特性总结(待补充) ##
1. `Bundle`使用了`OutputCache`。  
2. 对物理文件修改后，文件变化可以立刻输出。原理是使用了缓存依赖，源文件作为依赖项。  
3. 文件内容相同的请求，`v`参数的值不变，因此可能会使用上浏览器内的客户端缓存。  
4. `Bundle`不能定义对不同`domain`的文件的引用。能否定义同域的但路径为虚拟路径的文件的引用，未知。  

## 3. 动态Bundle ##
`Bundle`在`Application_Start`时被注册到`BundleTable.Bundles`中。这样做的好处是，（预定义的）`Bundle`可以被重用。  

但如果仅有这一条途径注册`Bundle`，有时候会觉得未免太繁琐：  
1）修改`BundelConfig.cs`  
2）重新编译项目  
3）忍受站点启动缓慢的过程。  

假如你遇到的情形是，某个页面上需要一个偶尔使用的一个或多个js，你想利用**Bundling and Minification机制**对其打包或启用压缩，有没有简单点的办法呢？  

看下别人的解决方案：
<script type="text/javascript" src="https://gist.github.com/4201780.js?file=HtmlExtensions.cs"></script>

前面的问题很好的解决了。现在试着想一个新问题，这样做是否就无法满足**重用Bundle**的需求呢？如果你了解`MVC 2`，可以试着用`MVC 2`中引入的`DisplayTemplates `机制解决这个问题。

## 4. Sample ##
静态Bundle

	// 注册
    public static void RegisterBundles(BundleCollection bundles)
    {
	    var styleBundle = new StyleBundle("~/Content/css").Include(
		    "~/css/reset.css",
		    "~/css/common.css",
		    "~/css/layout.css"
	    );
	    bundles.Add(styleBundle);
    }

	@*使用*@
	@Styles.Render("~/Content/css")

动态Bundle 


	// 定义
    public static IHtmlString DynamicScriptsBundle(this HtmlHelper htmlHelper, string nombre, params string[] urls)
    {
        string path = string.Format("{0}", nombre);
        if (BundleTable.Bundles.GetBundleFor(path) == null)
            BundleTable.Bundles.Add(new ScriptBundle(path).Include(urls));
        return Scripts.Render(path);
    }
> [asp-net-mvc-4-use-bundles-beneficts-for-url-content] (http://stackoverflow.com/questions/13124218/asp-net-mvc-4-use-bundles-beneficts-for-url-content?rq=1)
> [runtime-bundling-and-minifying-in-mvc-4](http://stackoverflow.com/questions/10614441/runtime-bundling-and-minifying-in-mvc-4)  

	@*使用*@
	@Html.DynamicScriptsBundle("~/Scripts/Common", "~/js/common.js")

### 延伸阅读 ###
> 
> [a-localized-scriptbundle-solution](http://stackoverflow.com/questions/10978979/a-localized-scriptbundle-solution)  