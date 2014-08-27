
### 缘起 ###
上午计划为一个小站点，增加一个日志的Dashboard，找到了这篇文章：
> [Log Reporting Dashboard for ASP.NET MVC](http://www.codeproject.com/Articles/104112/Log-Reporting-Dashboard-for-ASP-NET-MVC#ConfiguringLog4Net)

文中讲解了如何用ASP.NET MVC实现一个Dashboard，为四种`日志源`提供检索和统计图表的功能。
![ASP.NET MVC Log Reporting Dashboard](http://www.codeproject.com/KB/trace/mvc_logging/mvc_logging2.png)

从图中也可以看出来，这四种`日志源`分别是：
- Elmah
- Health Monitoring
- log4net
- NLog

可以把它们归为三类日志工具：
1. Elmah，应用场景非常明确，用于搜集`ASP.NET应用程序`中`未捕获异常`及大量上下文信息，并提供了不错的UI。
2. log4net和NLog都是通用的日志库，专注在`记录`上，而记录哪些内容，给查看者提供UI，并不是他们的目标。
3. Health Monitoring，也是站在`ASP.NET应用程序`的角度，基于一套`事件`模型，提供（`Provider`）记录（或通知）机制。

因此，这三类工具，虽然从`记录`这个角度看有些交集，但各自侧重点不同。

眼前这个小站点，只使用log4net作为单纯的`记录`工具。对于应用程序级别的监控，还一片空白，遂想着看这方面的需求，能否利用Health Monitoring来满足一部分。

### Health Monitoring ###
对于Health Monitoring，以前几乎没有了解。甚至从来没有想过，为什么一个未捕获的异常，在`ASP.NET应用程序`中，会被记录到`Windows EventLog`中。

先是读了MSDN:
> [healthMonitoring Element (ASP.NET Settings Schema)](http://msdn.microsoft.com/en-us/library/vstudio/2fwh2ss9(v=vs.100\).aspx)

最吸引我的是：
- `healthMonitoring`的`heartbeatInterval`属性
- `bufferModes`
- `Profile`中的`minInterval`属性
从这里也可以看出，已经不是单纯的做记录了，而是侧重于`Monitoring`

在`%windir%\Microsoft.NET\Framework\v4.0.30319\Config\web.config`中定义了四种`bufferModes`（应该也可以很容易定义自己的bufferModes）：
1. Critical Notification
2. Notification
3. Analysis
4. Logging

	<bufferModes>
	    <add name="Critical Notification" maxBufferSize="100" maxFlushSize="20"
	        urgentFlushThreshold="1" regularFlushInterval="Infinite" urgentFlushInterval="00:01:00"
	        maxBufferThreads="1" />
	    <add name="Notification" maxBufferSize="300" maxFlushSize="20"
	        urgentFlushThreshold="1" regularFlushInterval="Infinite" urgentFlushInterval="00:01:00"
	        maxBufferThreads="1" />
	    <add name="Analysis" maxBufferSize="1000" maxFlushSize="100"
	        urgentFlushThreshold="100" regularFlushInterval="00:05:00"
	        urgentFlushInterval="00:01:00" maxBufferThreads="1" />
	    <add name="Logging" maxBufferSize="1000" maxFlushSize="200" urgentFlushThreshold="800"
	        regularFlushInterval="00:30:00" urgentFlushInterval="00:05:00"
	        maxBufferThreads="1" />
	</bufferModes>

由于这个机制最早是在`ASP.NET 2.0`中引入的，时隔多年，便有一点想去了解有没有变化的部分。找到的文章里，并没有发现这方面内容，但其中的示例，
很轻易的就实现了一个常见的需求：当某个特定异常发生时，发邮件通知干系人。
> [Health Monitoring in ASP.NET 3.5](http://www.dotnetcurry.com/ShowArticle.aspx?ID=283)

另外，受该文启发，有另一篇入门文章，其中其中有一个非常好的关于`Health Monitoring`的`quick reference`（见下图）
> [Re: Health Monitoring in ASP.NET 3.5](http://blog.andreloker.de/post/2009/03/12/Re-Health-Monitoring-in-ASPNET-35.aspx)
> 
![healthMonitoring](http://blog.andreloker.de/image.axd?picture=WindowsLiveWriter/ReHealthMonitoringinASP.NET3.5_985A/healthMonitoring_thumb.png)


Health Monitoring的其它相关资料：
> MSDN
> [ASP.NET Health Monitoring Overview](http://msdn.microsoft.com/en-us/library/bb398933(v=vs.100\).aspx)
>
> www.asp.net
> [Logging Error Details with ASP.NET Health Monitoring
(C#)](http://www.asp.net/web-forms/tutorials/deployment/deploying-web-site-projects/logging-error-details-with-asp-net-health-monitoring-cs)
>
> 两篇2007年的园子里的翻译文章
> [[翻译] ASP.NET 2.0中的健康监测系统（Health Monitoring）(1) - 基本应用](http://www.cnblogs.com/webabcd/archive/2007/05/20/753507.html)
> [[翻译]ASP.NET 2.0中的健康监测系统（Health Monitoring）(2) - 通过Email发送监测信息](http://www.cnblogs.com/webabcd/archive/2007/05/27/761821.html)
