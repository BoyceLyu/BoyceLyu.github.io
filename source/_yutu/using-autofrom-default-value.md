title: "AutoForm中默认值使用"
date: 2015-05-15 09:00:00
tags: AutoFrom
---
本篇介绍下框架中动态脚本的使用以及基于此构建的AutoFrom默认值功能.
#简介
在项目中使用动态脚本是很常见的功能,.Net4.0添加动态特性之后可以使用[Python](https://ironpython.codeplex.com/)以及[Ruby](http://ironruby.net/)作为脚本语言.但这两种需要学习特定语言,对于目前来说不是一个好的选择.
mono中集成了动态脚本功能[mono.csharp](http://www.mono-project.com/docs/about-mono/languages/csharp/),这样我们就可以使用C#语法来作为动态脚本嵌入项目当中.

#动态脚本的使用
脚本功能涉及到3个类

	namespace Mapuni.Frame.Tools.DlrScript	
	--IScriptService.cs
	--MonoCSharpService.cs
	--ScriptManager.cs

##IScriptService: 
>操作脚本接口类定义了*注册参数*,*注册方法*,*执行脚本*以及*执行脚本带返回值*4个方法,如果要实现其他脚本可以继承此接口

##MonoCSharpService
>Mono.Csharp中继承IScriptService接口的封装.

##ScriptManager
>提供了全局统一静态操作脚本入口,可以管理定义的参数以及方法的注册,所有操作都是在一个会话中完成.

##使用ScriptManager

###数值计算

	 var result = ScriptManager.Run("1+1");
	 --result = 2
	 
	 var result = ScriptManager.Run("5*2+1");
	 --result = 11
	 
###添加变量

	ScriptManager.RegisterParameter("True",true);
    var result = ScriptManager.Run("True");
    --result = true

	ScriptManager.RegisterParameter("myparameter","我是一个参数");
    var result = ScriptManager.Run("myparameter");
    --result = "我是一个参数"
	
	ScriptManager.RegisterParameter("count",4);
    var result = ScriptManager.Run("count+5");
    --result = 9
	
>这里需要注意:下面和上面的例子唯一区别就是注册参数是字符串类型所以最后结果是字符串类型

	ScriptManager.RegisterParameter("count","4");
    var result = ScriptManager.Run("count+5");
    --result = "45"
	
	
###添加函数
>这里需要注意下Modo.Scharp 直接执行一个函数是无法获取到返回值,可以使用以下两种方法
	ScriptManager.RegisterFunction("myfunc", (Func<string>)(()=>"test"));
	
	var result1 = ScriptManager.Run("myfunc()+/"/"");
	--result1 = test
	
	var func = ScriptManager.Run("myfunc()") as Func<string>;
	var result2 = func();
	--result2 = test
	
#AutoForm中默认值扩展

上例中提到 **Modo.Scharp 直接执行一个函数是无法获取到返回值**,所以我们在AutoForm中获取默认值是做了下判断

	defaultValue = ScriptManager.Run(cfgField.DF_DEFAULTVALUE);
    if(defaultValue is Func<string>)
    {
         efaultValue = ((Func<string>)defaultValue)();
    }

这样在直接获取函数值的时候就无需在添加`+""`,像使用变量一样 `myfunc`就可以得到想要的值.局限性么?注册方法的时候只能使用类型`Func<string>`.

目前平台提供两个默认方法,可以根据项目需求进行添加
>~/App_Start/GlobalScriptRegister.cs

###获取当前系统用户名	currentUsername
	
	ScriptManager.RegisterFunction("currentUsername", (Func<string>)CurrentLoginName);
	public static string CurrentLoginName()
    {
        var user =  HttpContext.Current.Session[UserSessionKey] as Base_UserInfo;
        if (user == null) return null;
        return user.User_Name;
    }

###获取当前系统时间		nowDatetime

	 ScriptManager.RegisterFunction("nowDatetime", (Func<string>) NowDateTime)
	 public static string NowDateTime()
     {
         return DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss fff");
     }
	 
以上就是本篇教程的全部内容,如有疑问请联系平台部.
	
	
