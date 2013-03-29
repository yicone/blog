## 将ViewModel分成两部分来看待 ##
    
IPostModel

    public interface IPostModel
    {
        void Validate(System.Web.Mvc.ModelStateDictionary modelState);
    }

## 使用Object Mapper 技术辅助Rebuild ##


使用示例（用AutoMapper演示)

	if (!ModelState.IsValid)
	{
		var builder = new OrderModelBuilder();
		Mapper.CreateMap<OrderPostModel, OrderModel>();
		OrderModel model = Mapper.Map<OrderPostModel, OrderModel>(postModel);
		model = builder.Rebuild(model);

	    return View(model);
	}
