此文介绍的内容，无关语言和平台。

## 从Controller中分离出ModelBuilder ##
将`ViewModel`的创建过程从`Controller`中分离到`ModelBuilder`中，本质上是为了职责的分离，也提高了Controller的可读性。

### 通常的情况 ###
使用MVC时，在Controller中，会包含很多用于创建ViewModel的代码。  
让我们从一个例子开始。  
假设我们有一个用于呈现填写订单的页面(`~\Order\FT-BJS-95486`)，对应的Action 如下：

	```c#
	public ActionResult Order(string productNo)
	{
		var p = ProductService.GetProduct(productNo);
		var model = new OrderModel
		{
			ProductNo = productNo,
			ProductName = p.ProductName,
		};

		return View(model);
	}
	```
其中OrderModel 这个`ViewModel`的定义为：

	public class OrderModel 
	{
		public string ProductNo { get; set; }
		public string ProductName { get; set; }
		public int Count { get; set; }
		public string Address { get; set; }
	}
接下来，为了接收用户输入的订单信息，我们可能会有这样一个`Action`：

	[HttpPost]
	public ActionResult Order(OrderModel model)
	{
		var p = ProductService.GetProduct(productNo);

		if(!ModelState.IsVaidate())
		{
			// 这里假设：
			// 1. ProductName并没用通过hidden input的方式post回来
			// 2. ProductNo 随同其它用户输入信息post回来
			model.ProductName = p.ProductName;
			return View(model);
		}

		var model2 = new OrderConfirmModel
		{
			ProductNo = model.ProductNo,
			ProductName = p.ProductName,
			Count = model.Count,
			Address = model.Address,
		};
		return View("OrderConfirm", model2);
	}
### 1. 问题 ###
1. 有些ViewModel的创建可能比较“复杂”，从阅读Controller的角度，不是“主控制流”
2. ViewModel会在不同阶段被创建出来，不同阶段的创建过程，往往出现一些重复的代码

### 2. 分析 ###
在上述的例子中，虽然OrderModel本身的结构及创建过程都非常简单，但我们依然可以识别出重复的部分：

	var p = ProductService.GetProduct(productNo);
	model.ProductName = product.ProductName;
我们可以归纳出这些重复的部分，分属于两个`ViewModel被创建的“阶段”`:

1. Create
2. Rebuild

我们据此抽象出`IModelBuilder`接口

    public interface IModelBuilder<TViewModel>
    {
        TViewModel Create();
        TViewModel Rebuild(TViewModel model);
    }
### 3. 改造 ###
我们来写一个OrderModelBuilder,实现IModelBuilder接口

	public class OrderModelBuilder : IModelBuilder<OrderModel>
	{
		private string _productNo;
		private string _productName;
		
		public OrderModelBuilder(string productNo, string productName)
		{
			_productNo = productNo;
			_productName = productName;
		}
		
		public OrderModel Create()
		{
			var model = new OrderModel
			{
				ProductNo = _productNo,
			};
			model = this.Rebuild(model);
			return model;
		}
		
		public OrderModel Rebuild(OrderModel model)
		{
			model.ProductName = _productName;
			return model;
		}
	}
用OrderModelBuilder来简化OrderController

	private OrderModelBuilder GetBuilder(string productNo)
	{
		var p = ProductService.GetProduct(productNo);
		var builder = new OrderModelBuilder(p);
		return builder;
	}

	public ActionResult Order(string productNo)
	{
		var builder = GetBuilder(productNo);
		var model = builder.Create();
		return View(model);
	}
	
	[HttpPost]
	public ActionResult Order(OrderModel model)
	{
		if(!model.IsVaidate())
		{
			var builder = GetBuilder(model.ProductNo);
			var model = builder.Rebuild(model);
			return View(model);
		}
		// 根据OrderModel创建OrderConfirmModel
		var builder2 = new OrderConfirmModelBuilder(model);
		var model2 = builder2.Create();
		return View("OrderConfirm", model2);
	}
这样，就解决了我们在第1节中提出的两个问题。