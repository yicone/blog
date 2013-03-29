此文介绍的内容，无关语言和平台。本质上是职责的分离，将`ViewModel` 的创建过程分离到`ModelBuilder` 中。好处是：
1. 提高了Controller 的可读性
2. 

## 从Controller中分离出ModelBuilder ##
使用MVC时，在Controller的实现中，会包含很多用于创建ViewModel的代码。  
例如，
我们有一个用于呈现填写订单的页面(`~\Order\FT-BJS-95486`)，对应的Action 如下：

	public ActionResult Order(string productNo)
	{
		var p = ProductService.GetProduct(productNo);
		var model = new OrderModel{
			ProductNo = productNo,
			ProductName = p.ProductName,
		};

		return View(model);
	}

其中OrderModel 这个`ViewModel` 的定义为：

	public class OrderModel{
		public string ProductNo { get; set; }
		public string ProductName {get; set; }

		public int Count { get; set; }
		public string Address { get; set; }
	}

接下来，为了接收用户输入的订单信息，我们可能会有这样一个Action：

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

		var model2 = new OrderConfirmModel{
			OrderNo = OrderService.NewOrderNo(),
			ProductName = p.ProductName,
			Count = model.Count,
			Address = model.Address,
		};
		return View("OrderConfirm", model2);
	}

在这个例子中，我们可以归纳出两种ViewModel被创建的情形：
1. Create
2. Rebuild

相应地，我们将其抽象为IModelBuilder 接口的两个方法

    public interface IModelBuilder<TViewModel>
    {
        TViewModel Create();
        TViewModel Rebuild(TViewModel model);
    }

