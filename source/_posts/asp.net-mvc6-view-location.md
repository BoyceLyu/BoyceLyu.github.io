title: "asp.net mvc6 中修改View路径"
date: 2015-05-25 23:16:55
tags: vNext
---

在mvc6之前，我们如果想设置视图的位置需要继承[RazorViewEngine](http://www.hanselman.com/blog/ABetterASPNETMVCMobileDeviceCapabilitiesViewEngine.aspx)来改变查找View的默认情况, 现在mvc6中可以很容易的修改或者扩展视图路径，而不依赖于扩展`RazorViewEngine`.

#IViewLocationExpander
在MVC6中，存在一个新的接口[IViewLocationExpander](https://github.com/aspnet/Mvc/blob/master/src/Microsoft.AspNet.Mvc.Razor/IViewLocationExpander.cs),包含两个方法来定义Razor视图引擎怎样访问文件。以下就来介绍下这两个方法的含义：
{% codeblock lang:csharp  %}
    void PopulateValues(ViewLocationExpanderContext context); 
    IEnumerable<string> ExpandViewLocations(ViewLocationExpanderContext context,IEnumerable<string> viewLocations);
{% endcodeblock %}


现在我们来创建一个例子来学习以下如何使用这个接口,创建继承IViewLocationExpander的类
{% codeblock lang:csharp  %}
    public class MySharedLocationRemapper:IViewLocationExpander
    {
        #region Implementation of IViewLocationExpander 

        public void PopulateValues(ViewLocationExpanderContext context)
        {
            // do nothing
        }

        public IEnumerable<string> ExpandViewLocations(
                ViewLocationExpanderContext context, 
                IEnumerable<string> viewLocations)
        {
            //Shared替换成MyShared
            return viewLocations.Select(f => f.Replace("/Shared/", "/MyShared/"));
        }

        #endregion
    }
{% endcodeblock %}

现在我们在`Startup.ConfigureServices class`中添加ViewLocationExpander：


{% codeblock lang:csharp  %}
    services.AddMvc(); 
    services.Configure<RazorViewEngineOptions>(o =>
    {
       o.ViewLocationExpanders.Add(typeof(MySharedLocationRemapper));
    });
{% endcodeblock %}

以上就是所有的代码，通过此设置我们就可以把Shared文件夹替换成MyShared,是不是很简单?