#            Selenium with C# #

# 1.环境搭建 #

Step-1：准备所需的开发环境、浏览器驱动、Selenium-Webdriver、单元测试框架，Visual Studio 2017

Step-2：使用Visual Studio创建一个单元测试项目，如下图所示：



![img](https:////upload-images.jianshu.io/upload_images/3349421-a29bfeea41966b84.png?imageMogr2/auto-orient/strip|imageView2/2/w/800/format/webp)



Step-3：添加引用Webdriver，Selenium Webdriver目前支持的.NET平台有3.5和4.0，4.5等以上，一般选择4.5以上。



![img](https:////upload-images.jianshu.io/upload_images/3349421-1d020eb3dfda6dec.png?imageMogr2/auto-orient/strip|imageView2/2/w/616/format/webp)



-Step-4：创建第一个单元测试用例，并编译通过



# 2 .Web元素定位 #



使用Selenium来做自动化测试，一般的流程是:

**查找定位元素——>操作元素——>断言**



那么第一步我们需要能够完成查找并定位元素，Selenium目前提供了8种基本定位方法，可根据实际情况进行选择，如下示：

| 定位方法        | 示例                                                         |
| --------------- | ------------------------------------------------------------ |
| ID              | FindElement(By.Id("user"))                                   |
| Name            | FindElement(By.Name("username"))                             |
| LinkText        | FindElement(By.LinkText("Login"))                            |
| PartialLinkText | FindElement(By.PartialLinkText("Next"))                      |
| XPath           | FindElement(By.Xpath("//div[@id="login"]/input"))            |
| TagName         | FindElement(By.TagName("body"))                              |
| ClassName       | FindElement(By.ClassName("table"))                           |
| CSS             | FindElement(By.CssSelector("#login   > input[type="text"]")) |

## 启动浏览器



Selenium支持的浏览器较多，我拿目前三大使用较多的浏览器（IE、Chrome、Firefox）做一个示例，来演示如何启动三大浏览器，代码如下：

```cs
using System;
using System.Collections.ObjectModel;
//MS自带的单元测试框架
using Microsoft.VisualStudio.TestTools.UnitTesting;
//Webdriver引用
using OpenQA.Selenium;
//IE引用
using OpenQA.Selenium.IE;
//Chrome引用
using OpenQA.Selenium.Chrome;
//Firefox引用
using OpenQA.Selenium.Firefox; 

namespace SeleniumDemo
{
    [TestClass]
    //定义浏览器类型枚举
    public enum DriverType { IE, Chrome, Firefox };
    public class Lesson02
    {
        //获取浏览器驱动程序路径
        string driverPath = AppDomain.CurrentDomain.BaseDirectory;
        //定义使用浏览器类型
        DriverType dt = DriverType.Chrome;
        IWebDriver driver;
        //获取测试需要使用哪一种浏览器驱动
        public IWebDriver GetDirverType(DriverType driverType)
        {
            switch (driverType)
            {
                case DriverType.IE:
                    driver = new InternetExplorerDriver(driverPath);
                    break;
                case DriverType.Chrome:
                    driver = new ChromeDriver();
                    break;
                case DriverType.Firefox:
                    //3.0之后的Selenium Webdriver才需要这样，2.0版本的可以直接以这种形式：driver = new FirefoxDriver();
                    FirefoxOptions fo = new FirefoxOptions();
                    //Firefox安装路径
                    fo.BrowserExecutableLocation = @"C:\Program Files\Mozilla Firefox\firefox.exe";
                    FirefoxDriverService fds = FirefoxDriverService.CreateDefaultService(driverPath);
                    driver = new FirefoxDriver(fds, fo, TimeSpan.FromSeconds(100));
                    break;
                default:
                    break;
            }
            return driver;
        }
        [TestMethod]
        public void Demo02()
        {
            driver = GetDirverType(dt);
            //访问百度
            driver.Navigate().GoToUrl("https://www.baidu.com");
            //查找搜索输入框，输入Selenium
            IWebElement searchText = driver.FindElement(By.Id("kw")); 
            //在输入前清空内容
            searchText.Clear();
            searchText.SendKeys("Selenium");
            //查找元素
            IWebElement searchBtn = driver.FindElement(By.Id("su"));
            //点击搜索按钮
            searchBtn.Click();
            //退出浏览器
            driver.Quit();
            //driver.Close();
        }
    }
}
```

**Close（）与Quit（）区别：**

> driver.Close()：在完成操作后，仅关闭浏览器窗口,不会关闭Webdriver会话（我们在测试每个用例时，都会启动一个浏览器驱动进程，如果使用Close()方法，则浏览器驱动进程不会关闭，则使用Quit()则会自动关闭）

> driver.Quit():在完成操作后，同时关闭浏览器窗口和Weddriver会话



## 使用8种基本定位方法

在介绍基本的8种定位方法前，我们可以看看使用各浏览器自带的Web开发工具或插件（Firebug/Firepath等）来查看如何进行元素查看和定位：

> 使用IE自带的Web开发工具（可使用F12快捷键进行操作）如下所示：
>
> ![img](https:////upload-images.jianshu.io/upload_images/3349421-43bd856da402ecce.png?imageMogr2/auto-orient/strip|imageView2/2/w/919/format/webp)
>
> 在Firefox中使用Firebug+Firepath进行定位元素，如下所示：
>
> ![img](https:////upload-images.jianshu.io/upload_images/3349421-f59f187476567b37.png?imageMogr2/auto-orient/strip|imageView2/2/w/1102/format/webp)
>
> 使用Chrome自带Web开发工具进行定位元素，如下所示：
>
> ![img](https:////upload-images.jianshu.io/upload_images/3349421-d74aaebd010f5fb9.png?imageMogr2/auto-orient/strip|imageView2/2/w/1028/format/webp)



在使用Selenium Webdriver进行元素定位时，通常使用

FindElement或FindElements

方法结合By类返回的元素句柄来定位元素。其中By类的常用定位方式共八种，现分别介绍如下：

#### By.Id() ####

通过元素的id属性来定位元素，具有很强的唯一性，速度也较快。

> Button： driver.FindElement(By.Id("submit_btn")).Click();
>  Link:    driver.FindElement(By.Id("cancel_link")).Click();
>  Text:    driver.FindElement(By.Id("username")).SendKeys("admin ") ;
>  HTML Div元素: driver.FindElement(By.Id("alert_div")).getText();

#### By.Name() ####

通过元素的name属性来查找元素，在当前页面中可以不唯一，可能会存在同名的情况。

> driver.FindElement(By.Name("comment")).SendKeys("comment ") ;

#### By. XPath() ####

XPath 是一门在 XML 文档中查找信息的语言,可在 XML 文档中对元素和属性进行遍历。

> driver.FindElement(By.XPath("//input[@id='kw']"));

#### By.TagName() ####

该方法可通过元素的标签名称来定位查找元素，通常该方法能定位到的元素会存在一个或多个，建议结合FindElements()方法使用。

> driver.FindElements(By.TagName("button"));

#### By. ClassName() #### 

ClassName方法是通过元素的CSS样式表所引用的伪类名称来定位查找元素。通过情况下，一个元素会存在多个CSS样式值，如下所示：

(Web code:)

```xml
<a href="https://www.baidu.com" class="btn btn-default">百度一下</a>
<input type="submit" value="提交" class="btn btn-default btn-primary"/>
```

我们可以以下任何一种方法进行定位元素：

> driver.FindElement(By.ClassName("btn-primary ")).Click(); //提交按钮
>  driver.FindElement(By.ClassName("btn ")).Click(); //百度一下链接

#### By.CssSelector() ####

在CSS中，选择器是一种模式，用于选择需要添加样式的元素，更多资料可参考：http://www.w3school.com.cn/cssref/css_selectors.asp

> driver.FindElement(By.CssSelector("#kw"));

#### By.LinkText() ####

通过文本链接来查找元素，该链接必须能在页面中为可见状态。在一些页面会出现在出现某个操作后，链接才能显示出来，如果没有上一步操作，是无法直接使用该方法进行定位元素的。

> driver.FindElement(By.LinkText("新闻")).Click();

#### By. PartialLinkText() ####

PartialLinkText是对Link定位的一种补充，在文本链接比较长或文本链接动态生成的时候，可以使用该方法截取一部分文本进行定准。

> driver.FindElement(By.PartialLinkText("新")).Click();

#### 多元素定位 ####

通过**FindElements**可以一次返回多个匹配元素，使用方法与FindElement很像。示例如下：

> ReadOnlyCollection<IWebElement> elements = driver.FindElements(By.ClassName("mnav"));





# 3 .按钮 #

Button通常有两种形式：标准按钮和提交按钮，如下图所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-032b13e0b11e11e3.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/495/format/webp)



> 其中标准的按钮通常是由**button**标签创建，而提交按钮通常是由**input**创建，且通常在**form**里面。



HTML 源码如下：

```xml
<button id="button" class="nav" data-id="124" style="font-size:20px">标准按钮</button> <br />
<form name="input" action="index.html" method="post" >
   用户名：<input type="text" name="username" /><br /><br />
   密  码：<input type="password" name="pwd" /><br/>
   <input type="submit" name="submit" value="注册-提交按钮" />
</form>
```



在Web页面中，有一些元素看起来非常像是按钮，但有一部分是通过CSS来实现的。



## 通过文本点击标准按钮

```cs
driver.FindElement(By.XPath("//button[contains(text(),'标准按钮')]")).Click();
```



## 通过文本点击提交按钮

在HTML里面提交按钮通常位于**form**内，按钮的名字是通过属性**value**来显示的。而在显示的文字中可能会包含一些额外的空格或不可见的字符。源码如下：

```xml
<input type="submit" name="submit" value="测试 按钮  " />
```



通过以下的脚本进行会失败，因为存在空格。报错信息如下图所示：

```cs
 driver.FindElement(By.XPath("//input[@value='测试 按钮']")).Click();
```



![img](https:////upload-images.jianshu.io/upload_images/3349421-a90ccb9529288bb1.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/812/format/webp)

4-2 空格导致出错

这时，我们只需要修改一下脚本里面的文本值即可：

```cs
driver.FindElement(By.XPath("//input[@value='测试 按钮  ']")).Click();
```

（网页显示方式）

通常换一种元素定位，或者只填写空格前或空格后的值（填写的值为唯一的）。

driver.FindElement(By.XPath("//input[@value='测试']")).Click();

或

driver.FindElement(By.XPath("//input[@value='按钮']")).Click();

## 提交表单

在Selenium官方文档里面，提交表单通常是由**Submit**方法实现，下面所示脚本是演示用户登录操作：

```cs
            IWebElement username = driver.FindElement(By.Name("username"));
            username.SendKeys("UserName");
            IWebElement pwd = driver.FindElement(By.Name("pwd"));
            pwd.SendKeys("pwd");
            username.Submit();
```



上面是一种方法，而在实际页面中，我们都是通过点击提交按钮来进行操作的，以下是实现方法：

```cs
        driver.FindElement(By.Name("username")).SendKeys("UserName");
        driver.FindElement(By.Name("pwd")).SendKeys("pwd");
        driver.FindElement(By.XPath("//input[@value='注册-提交按钮']")).Click();


```

尽管在一个form里面不太可能会存在多个提交按钮，但如果存在这种情况，使用**Submit** 方法仅会点击第一个提交按钮，这样会造成混乱。



## 通过ID点击按钮

如果元素有ID，那么通过ID来定位元素是最好的方式。对于测试人员而言，通常会碰到元素中没有ID的情况，与其花很长时间来研究如何定位元素，还不如在这个时候多与开发沟通，增加元素的ID，这样也能减小测试的难度。

```cs
driver.FindElement(By.Id("button")).Click();
```



## 通过Name点击按钮

```cs
driver.FindElement(By.Name("submit")).Click();
```



## 通过图片点击按钮

在测试的过程中会遇到另一种类型的按钮。在一个form内，图片看起来非常像按钮，如下图所示：

html:

```xml
<input type="image" src="images/button_login.jpg" />
```


![img](https://upload-images.jianshu.io/upload_images/3349421-51d74e28d44d711a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/48/format/webp)

4-3 图片登录按钮.jpg



```cs
driver.FindElement(By.XPath("//input[contains(@src, 'button_login.jpg')]")).Click();
```



## 通过JavsScript点击按钮

在用尽一切方法都不能点击按钮时，可以考虑用JavaScript来实现点击按钮，如下所示：

```cs
 IWebElement btn = driver.FindElement(By.Name("submit")); 
 ((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].click()", btn);
```





# 4 .文本框 #

文本框在Web页面中，通常可以允许用户输入一些文本并发送到服务器。通常有两种表现形式：**密码文本**和**普通文本**，在密码文本中输入文本通常是经常特殊处理的，常见的是将用户输入的文本用**●**或** * **显示，在普通文本中也允许输入单选或多行的文字。如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-c831686a54e7b5e7.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/617/format/webp)

5-1 文本框示例_c2i.jpg

> HTML源码如下：



```xml
  用户名: <input type="text" name="username" id="userID"> <br /><br />
  密 码: <input type="password" name="password" id="passID"> <br /><br />
  注释: <br />
  <textarea id="commentsID" rows="3" cols="50" name="comments"></textarea>
```



## 通过Name定位输入文本

通过使用**name**属性来识别定位文本框，是开发经常使用的一种方法。示例如下:

```cs
driver.FindElement(By.Name("username")).SendKeys("username");
```



## 通过Id定位输入文本

通过使用Web元素的Id来识别定位文本框，应该是测试过程最简单和最快的方法了，示例如下：

```cs
driver.FindElement(By.Id("userID")).SendKeys("username");
```



## 向密码文本输入文本

在Selenium中，密码文本框的处理其实与普通文本框是一样的，只是密码文本框中输入的字符是经过处理以特殊字符展现而已。示例代码如下：

```cs
driver.FindElement(By.Id("passID")).SendKeys("password");
```



## 清空输入的文本

对同一个文本框而言，在使用**SendKeys()**方法时，有时候会将旧的文本和新输入的文本合并在一起，从而导致测试失败。因此在需要输入文本的地方，我们可以先调用**Clear()**方法清空文本，再输入新的文本值。



```cs
 driver.FindElement(By.Id("userID")).SendKeys("username");
 driver.FindElement(By.Id("userID")).SendKeys(" test");
```

以上的示例最终输入的文本值变会变成**username test**

而正确的方法如下所示：

```cs
 river.FindElement(By.Id("userID")).Clear();
 driver.FindElement(By.Id("userID")).SendKeys("username");
```



对于密码框也建议采取同样的方法，避免输入文本被合并而导致测试失败。

## 在文本框中输入多行文本

在Selenium中，处理多行文本框中普通文本框一样，示例如下：

```cs
driver.FindElement(By.Id("commentsID")).SendKeys("comments \r\n Test Multiline Text");
```



> 其中 **\r\n** 代表换行，开始新的一行。最终效果图如下：

![img](https://upload-images.jianshu.io/upload_images/3349421-2760858474021292.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/498/format/webp)



## 断言文本的值

在测试过程中，我们有时候需要获取到用户输入的文本信息，来验证是否与预期保持一致，这时候我们使用断言**Assert**来实现，示例代码如下：

```cs
IWebElement textValue = driver.FindElement(By.Id("userID"));
textValue.SendKeys("testTextValue");
Console.WriteLine(textValue.GetAttribute("value"));
Assert.AreEqual<string>("testTextValue", textValue.GetAttribute("value"));
```



> 最终的结果如下图所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-d37c27afc43e38fd.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1036/format/webp)



## 给元素设置焦点

在做自动化的过程中，有时候需要给元素设置焦点。而在Selenium中没有专门设置焦点的方法，但变通一下，我们可能通过发送一个空的按钮响应来解决该问题。

```cs
driver.FindElement(By.Id("commentsID")).SendKeys("");
```



当然，也可以采用JavaScript来解决该问题，示例代码如下：

```cs
IWebElement elements = driver.FindElement(By.Id("commentsID"));
((IJavaScriptExecutor)driver).ExecuteScript("arguments[0].focus();", elements);
```



## 改变只读或禁用状态文本框的值

只读属性的文本框在浏览器中是不可编辑的，而禁用状态的文本框通常是以灰色显示的。如下图所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-ae25141898b91e44.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/629/format/webp)

5-4 只读和禁用状态文本框

HTML源码如下所示

```xml
<!--只读状态的文本框-->
只读状态文本框：<input type="text" name="readOnlyText" id="readOnlyId" readonly="true" value="只读状态"/>
<br /><br />
<!--禁用状态的文本框-->
禁用状态文本框：<input type="text" name="disableText" id="disableTextId" disabled="true" value="禁用状态" />
```



如果文本框被设置只读属性，如果继续按照以下方法进行输入，则会报错。

```cs
driver.FindElement(By.Id("readOnlyId")).SendKeys("Read Only Test");
```



报错信息如下：

![img](https://upload-images.jianshu.io/upload_images/3349421-7943fe29e59416ec.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/586/format/webp)

5-5 只读状态报错

**注意：这段代码在IE中会报错，而在Chrome中却不会报错**

在一些地方用常规方法不能解决时，我们可以尝试采用JavaScript来解决此类问题，示例代码如下所示：

```cs
((IJavaScriptExecutor)driver).ExecuteScript("document.getElementById('readOnlyId').value='改变只读文本的值'"); 
((IJavaScriptExecutor)driver).ExecuteScript("document.getElementById('disableTextId').value='改变禁用文本的值'");
```



最终实现的效果如下：

> 

![img](https://upload-images.jianshu.io/upload_images/3349421-607d3324ce6286dd.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/618/format/webp)

5-6 改变值





# 5.单选按钮 #

单选按钮通常用在需要与用户进行交互且只能选一个选项的情况。下面即是一个典型的单选按钮示例：

![img](https://upload-images.jianshu.io/upload_images/3349421-80b44714cf1c4316.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/570/format/webp)





```xml
 请选择您的性别：
 <input type="radio" name="gender" value="male" id="male" checked="checked">男
 <input type="radio" name="gender" value="female" id="female" />女<br />
```



## 选中单选按钮

通常情况下，在一个单选按钮组里面，它们的名字是一样的。所以我们使用定位的时候，不建议使用**Name**属性进行定位，建议使用Id、XPath和CssSelector进行定位，如下所示：

```scss
driver.FindElement(By.XPath("//input[@name='gender' and @value='male']")).Click();
Thread.Sleep(1000);
driver.FindElement(By.Id("female")).Click();
```



## 清除单选按钮的选中状态

对于一个已经选中的单选按钮进行多次点击，是不会带来任何影响的，下面的示例代码在测试过程中依然可以正常通过。

```cs
driver.FindElement(By.Id("female")).Click();
driver.FindElement(By.Id("female")).Click(); //已经选中，多次点击没有任何影响
```



如果一个单选按钮被选中，在Selenium中清除选中状态的常用方法是点击另一个单选按钮。下面的示例使用**Clear()**方法清除单选按钮的选中状态将会抛出异常**Invalid Element State:Element Must Be User-Editable in Order to Clear It.**



```cs
            driver.FindElement(By.Id("female")).Click();
            try
            {
                driver.FindElement(By.Id("female")).Clear();
            }
            catch (Exception ex)
            {
                throw ex;
            }
```



报错截图如下所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-509b1e6a84c8807e.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1143/format/webp)

6-2 清除单选按钮选中状态报错_c2i.jpg

下面的示例演示了既抛出了异常也按预期的目标实现了点击，示例如下：



```cs
      driver.FindElement(By.Id("female")).Click();
      try
      {
          driver.FindElement(By.Id("female")).Clear();
      }
      catch (Exception ex)
      {
          Console.WriteLine("不能清除单选状态，报错信息为：\n"+ex.ToString());

      }
      finally
      {
          driver.FindElement(By.Id("male")).Click();
      }
```



## 判断单选按钮状态

```cs
    IWebElement femaleEle = driver.FindElement(By.Id("female"));
    IWebElement maleEle = driver.FindElement(By.Id("male"));
    bool flag = femaleEle.Selected;
    if (flag)
    {
        maleEle.Click();
        Assert.IsTrue(maleEle.Selected);
    }
    else
    {
        femaleEle.Click();
        Assert.IsTrue(femaleEle.Selected);
        Assert.IsFalse(maleEle.Selected);
    }
```



## 通过循环点击单选按钮

在这之前我基本上仅使用方法**FindElement()**来查找定位元素，而在Selenium中还有另外一种方法**FindElements()**，示例如下：

```csharp
    ReadOnlyCollection<IWebElement> elements = driver.FindElements(By.Name("gender"));
    Assert.AreEqual<int>(2,elements.Count);
    foreach (IWebElement item in elements)
    {
        if (item.GetAttribute("value")=="female")
        {
            item.Click();
        }
    }
```



 与方法**FindElement()**不一样的是，方法**FindElements()**会返回能匹配到条件的集合，这对一些比较难定位的元素是非常有用的一个方法。 

## 选中第N个单选按钮

在Name属性值相同且单选按钮也比较多的情况下，除了上面的方法循环判断来点击外，也可以通过XPath和以下方法来点击单选按钮，示例如下：



```cs
    driver.FindElements(By.Name("gender"))[1].Click();
    Assert.IsTrue(driver.FindElements(By.Name("gender"))[1].Selected);

    driver.FindElement(By.XPath("//input[1]")).Click();
    Assert.IsTrue(driver.FindElement(By.XPath("//input[1]")).Selected);
```



 **请注意里面的下标，C#下标默认是从0开始，而XPath里面下标默认是从1开始。** 



# 6 .复选框



复选框也是Web页面常见的一种用户交互控件，可以允许用户一次选择多个选项。常见的多选按钮示例如下图所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-11e05a4b481bffaf.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/540/format/webp)

 HTML源码如下： 

```xml
请选择你常用的交通出行方式:<br /><br />
    <input type="checkbox" name="metro" checked="checked" id="metro" />地铁
    <input type="checkbox" name="bus" id="bus" />公交
    <input type="checkbox" name="bike" id="bike" />自行车
    <input type="checkbox" name="walk" id="walk" />步行
    <input type="checkbox" name="driving" id="driving" />自驾
```





## 通过Name选中复选框

```cs
  driver.FindElement(By.Name("metro")).Click();
```



## 通过ID选中复选框

对于复选框的点击而言实际第一次点击将选中，再次点击将清除选中状态（针对第一次状态是未选中状态而言），下面将演示在测试脚本中如何确认复选框是否为选中状态。

```cs
   IWebElement metroCheckbox = driver.FindElement(By.Id("metro"));
   if (!metroCheckbox.Selected)
   {
       metroCheckbox.Click();
   }
```

## 通过连续调用FindElement方法选中复选框

针对要定位的复选框具有多个相同的属性时，我们可以使用XPath定位，也可以使用连续调用FindElement方法。HTML源码如下：

```xml
<div id="div1">
  <input type="checkbox" name="test" checked="checked" value="on" />复选框位于第一个div里面
</div>
<div id="div2">
  <input type="checkbox" name="test" checked="checked" value="on" />复选框位于第二个div里面
  </div>
```





 定位的脚本代码如下所示： 

```cs
 driver.FindElement(By.Id("div2")).FindElement(By.Name("test")).Click();
```





## 清除复选框选中状态

对于清除复选框的状态，我们只需要再次点击复选框即可，与单选按钮一样，不能使用**Clear()**方法。因为复选框的状态而言总共就两种状态，选中和未选中状态。

```cs
  IWebElement drivingCheckbox = driver.FindElement(By.Id("driving"));
  if (drivingCheckbox.Selected)
  {
      drivingCheckbox.Click();
  }
  else
  {
      drivingCheckbox.Click();
  }
```



## 断言复选框的状态

判断复选框的方法比较简单，只需要使用**Assert**类中的方法即可。

```cs
   IWebElement bikeCheckbox = driver.FindElement(By.Id("bike"));
   Assert.IsFalse(bikeCheckbox.Selected);
   bikeCheckbox.Click();
   Assert.IsTrue(bikeCheckbox.Selected);
```



# 7.下拉列表



常见的下拉列表框如下图所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-43b2b911011c2023.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/431/format/webp)

 HTML源码如下： 

```xml
<select name="China" id="ChinaProvince">
    <option value="">--请选择区域--</option>
    <option value="pdditrict">浦东新区</option>
    <option value="hpditrict">黄浦区</option>
    <option value="xhditrict">徐汇区</option>
    <option value="cnditrict">长宁区</option>
    <option value="sjditrict">松江区</option>
</select>
```

## Selenium中SelectElement类结构

![img](https://upload-images.jianshu.io/upload_images/3349421-0ea5f5ea2529ccd7.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/611/format/webp)

8-1 SelectElement Class.jpg



使用SelectElement类需引用命名空间OpenQA.Selenium.Support.UI 在标准的Selenium中，选择下拉列表的类是Select(如Java)，但在C#中Select是关键字。

## 通过文本选择列表

```css
IWebElement selectElem = driver.FindElement(By.Name("China"));
SelectElement selectObj = new SelectElement(selectElem);
selectObj.SelectByText("浦东新区");
```





在使用**SelectElement**需要引用命名**OpenQA.Selenium.Support.UI** 。在标准的Selenium中，选择下拉列表的类是**Select**(比如Java)，但在C#中Select是关键字，所以才换了一个名字

## 通过Value值选择列表

```cs
IWebElement selectElem = driver.FindElement(By.Name("China"));
SelectElement selectObj = new SelectElement(selectElem);
selectObj.SelectByValue("pdditrict");
```



## 通过Index选择列表

有些时候，我们并不关心所选择的列表是否为我们所要选择的列表，只需要确定有列表被选中即可，这时我们可以使用通过Index来选择，特别是一些动态生成下拉列表的情况下，使用Index来选择是最好的方式。代码如下所示：

```cs
IWebElement selectElem = driver.FindElement(By.Name("China"));
SelectElement selectObj = new SelectElement(selectElem);
selectObj.SelectByIndex(1);
```



## 通过循环选择列表

虽然在Selenium中，使用上面几种方法已经可以实现选择下拉列表了。在实际情况如果需要循环选择某些选项，该怎么写脚本了？详细代码如下:

```cs
IWebElement selectElem = driver.FindElement(By.Name("China"));
ReadOnlyCollection<IWebElement> options = driver.FindElements(By.TagName("option"));
//循环每个列表选中一次
for (int i = 0; i < options.Count; i++)
{
    options[i].Click();
}
//循环选中一个列表
for (int i = 0; i < options.Count; i++)
{
    if (options[i].Text.Contains("徐汇"))
    {
        options[i].Click();
    }
}
```



## 选择多个列表选项

下拉列表中不仅支持单选也支持多选，多选下拉列表示例如下所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-67c213b55705d3a7.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/481/format/webp)

8-3 下列列表多选_c2i.jpg

HTML源码如下：



```xml
请选择你向往的城市：<br /><br />
<select name="city" id="city"  multiple="multiple" size="10">
    <option value="">--请选择城市--</option>
    <option value="shanghai">上海</option>
    <option value="nanjing">南京</option>
    <option value="wuhan">武汉</option>
    <option value="chongqing">重庆</option>
    <option value="chengdu">成都</option>
    <option value="kunming">昆明</option>
    <option value="guilin">桂林</option>
</select>
```

 示例代码如下： 

```cs
IWebElement cityEle = driver.FindElement(By.Id("city"));
SelectElement citySelect = new SelectElement(cityEle);
citySelect.SelectByText("上海");
citySelect.SelectByValue("wuhan");
citySelect.SelectByIndex(6);
Assert.AreEqual(3,citySelect.AllSelectedOptions.Count);
```

最终的运行效果如下所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-8f48e127d9735208.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/453/format/webp)

8-4 多选运行效果_c2i.jpg

## 清除单个选中列表

清除所有选择列表只需要调用**SelectElement**类的方法**DeselectByText**、**DeselectByValue**、**DeselectByIndex**方法即可



```css
IWebElement cityEle = driver.FindElement(By.Id("city"));
SelectElement citySelect = new SelectElement(cityEle);
citySelect.SelectByText("上海");
citySelect.SelectByValue("wuhan");
citySelect.SelectByIndex(6);
Assert.AreEqual(3,citySelect.AllSelectedOptions.Count);
citySelect.DeselectByText("上海");
citySelect.DeselectByValue("wuhan");
citySelect.DeselectByIndex(6);
Assert.AreEqual(0, citySelect.AllSelectedOptions.Count);
```



## 清除所有选择列表

清除所有选择列表只需要调用SelectElement类的方法**DeselectAll**方法即可。

```cs
IWebElement cityEle = driver.FindElement(By.Id("city"));
SelectElement citySelect = new SelectElement(cityEle);
citySelect.SelectByText("上海");
citySelect.SelectByValue("wuhan");
citySelect.SelectByIndex(6);
Assert.AreEqual(3,citySelect.AllSelectedOptions.Count);
citySelect.DeselectAll();
Assert.AreEqual(0, citySelect.AllSelectedOptions.Count);
```



## 通过选择项断言

下拉列表中断言可以通过SelectElement类中**SelectedOption**方法来断言选择的列表是否正确，代码如下：

```cs
IWebElement selectElem = driver.FindElement(By.Name("China"));
SelectElement selectObj = new SelectElement(selectElem);
selectObj.SelectByText("浦东新区");
Assert.AreEqual("浦东新区", selectObj.SelectedOption.Text);
```



## 通过下拉列表Value属性断言

```cs
IWebElement selectElem = driver.FindElement(By.Name("China"));
SelectElement selectObj = new SelectElement(selectElem);
selectObj.SelectByText("浦东新区");
Assert.AreEqual("pdditrict", selectObj.SelectedOption.GetAttribute("value"));
```



## 多选下拉列表断言

因多选下拉列表可允许用户选中多个选项，那么我们可以将用户选中的项做为一个集合来对待，然后进行断言，代码如下：

```cs
IWebElement cityEle = driver.FindElement(By.Id("city"));
SelectElement citySelect = new SelectElement(cityEle);
citySelect.SelectByText("上海");
citySelect.SelectByValue("wuhan");
citySelect.SelectByIndex(6);

IList<IWebElement> selected = citySelect.AllSelectedOptions;
Assert.AreEqual(3, selected.Count);
Assert.AreEqual("上海",selected[0].Text);
```



# 8.Frames

虽然在Web中可以使用Frame进行页面布局，但这并不是一种推荐的设计方案。但现实中仍有部分网站仍在使用，甚至部分网站还使用了iFrames。在Selenium API 中也提供关于 Frame 定位操作的一些方法，将在本文中进行介绍。

## Selenium中Frame方法

在Web页面中如果需要定位到Frame中的元素时，需要切换到该Frame中，常使用的方法主要为：**Frame(int frameIndex)**、**Frame(string frameName)**和**DefaultContent()**。另外，还有一个方法是**ParentFrame()**，可以切换到当前Frame的上一层Frame中，主要用于存在多Frame嵌套的情况。下面是Selenium API提供的方法，如下所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-5065679cfe5ad084.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/539/format/webp)

9-2 Selenium Frame方法.jpg

## 测试Frames

常见的Frame页面布局一般分为三块，**顶部的导航，左边的菜单和右边的信息展示栏**，如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-09191621584fa07b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

9-1 Frame示例_c2i.jpg

HTML源码如下所示：

```xml
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
  <frameset rows="40%,60%">
    <frame name="nav" id="navId" src="https://www.baidu.com">
      <frameset cols="30%,60%">
        <frame name="menu" id="menuId" src="https://www.baidu.com">
        <frame name="content" id="contentId" src="https://www.baidu.com">
      </frameset>
  </frameset>
</html>
```





- 使用Selenium API进行Frame操作通常的步骤如下：

1. 通过index或Name定位到Frame
2. 切换到该Frame内部
3. 在该Frame下面进行操作
4. 切换到默认页面（即包含所有Frame的页面）
5. 如果还需要到其他Frame进行定位的话，则重复以上步骤

下面示例代码将演示依次从上到下，从左到右切换到不同的Frame，详细如下

```cs
driver.SwitchTo().Frame("nav");
driver.FindElement(By.Id("kw")).SendKeys("切换到nav Frame中");
driver.SwitchTo().DefaultContent();
//如果通过FrameIndex来定位切换Frame，下标从0开始
driver.SwitchTo().Frame(1);
driver.FindElement(By.Id("kw")).SendKeys("切换到menu Frame中");
driver.SwitchTo().DefaultContent();
driver.SwitchTo().Frame("content");
driver.FindElement(By.Id("kw")).SendKeys("切换到content Frame中");
```



 **如果通过FrameIndex来定位切换Frame，下标从0开始** 

最终的测试结果如下所示：

![img](https://upload-images.jianshu.io/upload_images/3349421-cba418417829c3e6.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1034/format/webp)

9-3 Frame代码切换_c2i.jpg

## 测试iFrames

一种常见的iFrame示例如下：

![img](https://upload-images.jianshu.io/upload_images/3349421-e9195b0714dd47da.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/644/format/webp)

9-4 iFrame示例_c2i.jpg

iFram.html源码:

```xml
<!--iFram.html-->
<body>
    <div id="divID">
        <h3>测试iFrame</h3>
        测试用户名：<input type="text" id="userName" />*在iFrame外面<br/><br />
        <iframe frameborder="1" id="iFrame1" name="iFrame1" src="iFrameLogin.html"></iframe>    *在iFrame里面
     </div>
    <br/><br/>
    <p><input type="checkbox" id="checkbox1" />测试主页面选中1</p>
    <p><input type="checkbox" id="checkbox2" />测试主页面选中2</p>
</body>
```

 FrameLogin.html源码 :

```xml
<!--iFrameLogin.html-->
<body>
    <table border="0" cellspacing="2" cellpadding="2">
        <tbody>
            <tr>
                <td>
                    用户名：<input type="text" id="userNameId" />
                </td>
            </tr>
            <tr>
                <td>
                    密    码：<input type="password" id="pwdId">
                </td>
            </tr>
        </tbody> 
    </table>
    <input type="submit" id="submitId" value="登录" />
</body>
```



 下面我们来演示在分别主页面中和iFrame页面中输入用户名，示例代码如下： 

```cs
driver.FindElement(By.Id("userName")).SendKeys("我在主页面中");
driver.SwitchTo().Frame("iFrame1");
driver.FindElement(By.Id("userNameId")).SendKeys("我在iFrame页面中");
driver.FindElement(By.Id("pwdId")).SendKeys("pwd");
driver.SwitchTo().DefaultContent();//跳出iframe
driver.FindElement(By.Id("checkbox1")).Click();
driver.FindElement(By.Id("checkbox2")).Click();
```







最终运行的效果如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-d4053cb2227ee163.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/621/format/webp)

9-5 iFrame运行效果图_c2i.jpg

## 测试多iFrames

一个页面可包含多个iFrame，一个典型的示例如下：

![img](https:////upload-images.jianshu.io/upload_images/3349421-23f41dd4c8b40d15.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/737/format/webp)

9-6 多个iFrame示例_c2i.jpg

该示例中存在3个iFrame，见图中红色框标识。各个HTML源码如下所示：
 iFram.html源码:

```xml
<!--iFram.html-->
<body>
    <div id="divID">
        <h3>测试iFrame</h3>
        测试用户名：<input type="text" id="userName" />*在iFrame外面<br /><br />
        <iframe frameborder="1" name="mainFrame" id="mainFrameId" height="400" width="450" src="iFrameQuery.html"></iframe>父iFrame
    </div>
    <br /><br />
    <p><input type="checkbox" id="checkbox1" />测试主页面选中1</p>
    <p><input type="checkbox" id="checkbox2" />测试主页面选中2</p>
</body>
```

 iFrameQuery.html源码 :

```xml
<!--iFrameQuery.html-->
<body>
    <iframe frameborder="1" name="subFrame1" id="subFrame1" src="iFrameLogin.html"></iframe>子iFrame-1
    <br /><br />
    <iframe frameborder="1" name="subFrame2" id="subFrame2" src="iFrameLogin.html"></iframe>子iFrame-2
    <br /><br />
    在子iFrame中:<input type="text" id="text1" width="50">
</body>
```



 iFrameLogin.html源码 :

```xml
<!--iFrameLogin.html-->
<body>
    <table border="0" cellspacing="2" cellpadding="2">
        <tbody>
            <tr>
                <td>
                    用户名：<input type="text" id="userNameId" />
                </td>
            </tr>
            <tr>
                <td>
                    密    码：<input type="password" id="pwdId">
                </td>
            </tr>
        </tbody> 
    </table>
    <input type="submit" id="submitId" value="登录" />
</body>
```



 以下示例代码演示了依次从最外层的iFrame切换到内层iFrame，并在同级iFrame中进行定位操作，再切换到外层iFrame进行操作，详细代码如下： :



```cs
driver.FindElement(By.Id("userName")).SendKeys("我在主页面中");
//第一次切换iFrame
driver.SwitchTo().Frame("mainFrame");
Thread.Sleep(500);
//第二次切换iFrame
driver.SwitchTo().Frame("subFrame1");
driver.FindElement(By.Id("userNameId")).SendKeys("我在子iFrame-1");
driver.FindElement(By.Id("pwdId")).SendKeys("pwd");
//第三次切换iFrame，切换同级父iFrame，这时其实是位于mainFrame中
driver.SwitchTo().ParentFrame();
//第四次切换iFrame
driver.SwitchTo().Frame("subFrame2");
driver.FindElement(By.Id("userNameId")).SendKeys("我在子iFrame-2");
driver.FindElement(By.Id("pwdId")).SendKeys("pwd");
//第六次切换iFrame-方法一,推荐方法一
driver.SwitchTo().ParentFrame();
//第六次切换iFrame-方法二
//driver.SwitchTo().DefaultContent();
//driver.SwitchTo().Frame("mainFrame");
driver.FindElement(By.Id("text1")).SendKeys("我已经切换到父iFrame啦");
//第六次切换iFrame
driver.SwitchTo().DefaultContent();
driver.FindElement(By.Id("checkbox1")).Click();
driver.FindElement(By.Id("checkbox2")).Click();
```



 注意方法**SwitchTo().ParentFrame()**和**SwitchTo().DefaultContent()**的区别 



最终的运行效果如下：

![img](https://upload-images.jianshu.io/upload_images/3349421-8405e76da9a553f5.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/654/format/webp)

9-7 多iFrame演示效果_c2i.jpg

# 9.浏览器操作



如何来定位Web页面中基本的元素，在之前的文章已经介绍过了。在本文中将介绍如何来控制和操作浏览器。我们先看看Selenium API 中提供的方法。

## Selenium API 方法

Selenium API对Webdriver定义如下：

![img](https:////upload-images.jianshu.io/upload_images/3349421-02b57dfe9624f008.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/947/format/webp)

10-1 SeleniumAPI方法.jpg

> 从中我们可以看到主要的方法有**Manage()**、**Navigate()**、**ITargetLocator()**、**Quit()**等

## 访问指定网址

示例代码如下：

```cs
 string url = "https://www.baidu.com";
 driver.Navigate().GoToUrl(url);
```



 从API里面可以看到IWebdriver里面有一个属性**Url**，也可以采用以下方式实现与上面一样的目的，如下所示： 

```cs
driver.Url = "https://www.baidu.com";
```





## 访问同一网站的不同页面

**driver.Navigate().GoToUrl()**中所输入的网址是一个完整的**URL**，但对于测试同一网站中的不同页面，其实不需要每次都输入完整的URL。我们可以创建一个方法来进行拼接，即可达到同样的目的，也能提高代码的复用度。我们以访问CSDN博客为例，详细代码如下：

```cs
        IWebDriver driver = new ChromeDriver();
        string mainUrl = "http://blog.csdn.net";
        //创建访问方法
        public void VisitWebSite(string subUrl)
        {
            driver.Navigate().GoToUrl(mainUrl+subUrl);
        }

        [TestMethod]
        public void TestVisitWebSite()
        {
            //访问huilan_same的博客
            VisitWebSite("/huilan_same");
            Thread.Sleep(1000);
            //访问niubitianping的博客
            VisitWebSite("/niubitianping");
            //访问博客主页
            VisitWebSite("/");
            Thread.Sleep(1000);
            //访问Hadoop专栏
            VisitWebSite("/column/details/hadoop-force.html");
            driver.Quit();
        }
```



 这种方式还有另外一个好处就是如果你需要在不同的服务器上面运行和测试相同的Web程序，而仅仅是更改了网址的部分名称，那么只需要更改一行代码，如下所示： 

```cs
string mainUrl = "http://www.csdn.net";
```



## 浏览器导航操作

在进行Web页面浏览时，通过会有**前进**、**后退**、**刷新**等操作，通过Selenium API可以得知，实现该操作主要方法为**Navigate()**

```cs
            //后退
            driver.Navigate().Back();
            //前进
            driver.Navigate().Forward();
            //刷新
            driver.Navigate().Refresh();
```



## 移动浏览器

在测试过程中，我们可以将浏览器窗口移动到指定的位置，从而避免对其他操作或本身生成影响。而把浏览器移动到窗口其他位置并不影响最终的测试结果。示例代码如下：

```cs
driver.Manage().Window.Position = new System.Drawing.Point(200,500);
Thread.Sleep(2000);
driver.Manage().Window.Position = new System.Drawing.Point(0, 0);
```



 位置(0,0)则代表所处位置是屏幕的左上方。 

## 设置浏览器窗口大小

浏览器窗口大小设定主要包含三种，**用户自定义大小**、**最大化**和**最小化**

- 用户自定义浏览器大小

```cs
driver.Manage().Window.Size = new System.Drawing.Size(800,600);
```

 用户自定义浏览器大小，需要使用到.NET里面的**System.Drawing**。 

- 浏览器最小化

在Selenium API 中并没有提供最小化浏览器的方法，但我们可以通过移动窗口到指定位置实现同样的目的，示例代码如下：

```cs
driver.Manage().Window.Position = new System.Drawing.Point(-1500, 0);
Thread.Sleep(2000);
driver.FindElement(By.PartialLinkText("博客专家")).Click();
Thread.Sleep(2000);
driver.Manage().Window.Position = new System.Drawing.Point(300, 400);
driver.Manage().Window.Maximize();
```

## 拖动滚动条

有些Web控件或元素在默认窗口大小，无法直接看见，需要调整滚动条位置才能看见。这个时候我们在测试的过程中，可以通过**JavaScript**来模拟拖动滚动条。代码如下：

```cs
   driver.Url = "http://blog.csdn.net/";
   driver.Manage().Window.Maximize();
   Thread.Sleep(1000);
   IWebElement eles = driver.FindElement(By.PartialLinkText("公司简介"));
   int elesPostionX = eles.Location.X;
   int elesPostionY = eles.Location.Y;
   string js = "window.scroll(" + elesPostionX + "," + elesPostionY + ")";
   ((IJavaScriptExecutor)driver).ExecuteScript(js);
   eles.Click();
```



## 在窗口和Tabs页面中切换

在Web页面中，如果一个链接含有**target="_blank"**标签时，则代表在点击链接后，将在浏览器新窗口或新Tab中打开链接**（取决于浏览器的设置）**。针对这种情况，我们可以使用方法**SwitchTo()**来进行切换到不同的窗口或Tab中。示例代码如下：

```cs
driver.Url = "http://blog.csdn.net/";
driver.Manage().Window.Maximize();
Thread.Sleep(1000);
IWebElement eles = driver.FindElement(By.PartialLinkText("公司简介"));
int elesPostionX = eles.Location.X;
int elesPostionY = eles.Location.Y;
string js = "window.scroll(" + elesPostionX + "," + elesPostionY + ")";
((IJavaScriptExecutor)driver).ExecuteScript(js);
//进行点击后将在新的Tab选项卡中打开
eles.Click();
ReadOnlyCollection<string> windowsHandles = driver.WindowHandles;
//切换到新窗口
driver.SwitchTo().Window(windowsHandles[1]);
Assert.IsTrue(driver.PageSource.Contains("创立于1999年，是中国最大的开发者服务平台"));
//返回最初的窗口
driver.SwitchTo().Window(windowsHandles[0]);
ReadOnlyCollection<IWebElement> elements = driver.FindElements(By.XPath("//div[@id='pub_footerall']/dl/dd[1]/a"));
Assert.AreEqual<int>(9,elements.Count);
driver.Quit();
```



# 10.用户交互

## Selenium API

Selenium API中的**Actions**类(==注意不要跟.NET自带的委托**Action**混淆==)提供了一些方法用于用户与浏览器进行较为复杂的交互操作，可以让用户通过键盘和鼠标等进行一系列的操作。详细的方法，我们可以通过查看Actions类，如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-a76114c01945e147.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/726/format/webp)

11-1 Selenium API.jpg

> 通过Actions类我们大致可以提取出以下几种交互方式
>
> -  **鼠标事件**
>    Click()
>    ClickAndHold()
>    ContextClick()
>    DoubleClick()
>    DragAndDrop()
>    DragAndDropToOffset()
>    MoveByOffset()
>    MoveToElement()
>    Release()
> -  **键盘事件**
>    KeyDown()
>    KeyUp()
>    SendKeys()

## 使用方法

- 第一步：需要引用**OpenQA.Selenium.Interactions**

```cs
using OpenQA.Selenium.Interactions;
```



- 第二步：调用方法

```cs
new Actions(driver).+"需要调用的方法"+.Perform();
```

## 鼠标单击控件

这个在之前的文章已经多次使用过了，示例代码如下：

```cs
IWebDriver driver=new ChromeDriver();
driver.Url = "https://www.baidu.com";
IWebElement eles = driver.FindElement(By.LinkText("关于百度"));
Actions actionsObj = new Actions(driver);
actionsObj.Click(eles).Perform();
```



 上面是通过**Actions**类来实现鼠标单击，其实也可以直接使用**IWebElement**中自带的**Click**方法 

## 鼠标双击控件

```cs
IWebDriver driver=new ChromeDriver();
IWebElement eles=driver.FindElement(By.Id("id"));
Actions actionsObj = new Actions(driver);
actionsObj.DoubleClick(eles).Perform();
```



## 鼠标单击拖动

下面的代码演示的是从第1个控件单击并拖动至第5个控件，并选中这5个控件，如下所示：

```cs
driver.Url = "http://www.jqueryui.org.cn/demo/5640.html";
Thread.Sleep(1000);
driver.Manage().Window.Maximize();
ReadOnlyCollection<IWebElement> items = driver.FindElements(By.XPath("//ol[@id='selectable']/li"));
Assert.AreEqual<int>(7, items.Count);
Actions actionsObj = new Actions(driver);
actionsObj.ClickAndHold(items[0]).ClickAndHold(items[4]).Release().Perform();
```



> 最后一行代码连续调用了两次**ClickAndHold()**方法，==第1次调用代表是起始控件，第2次调用代表是结束控件，注意里面的下标是从0开始==

- 鼠标单击拖动前后的状态

- ![img](https:////upload-images.jianshu.io/upload_images/3349421-cc0a4446575fa096.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

  11-2 鼠标单击拖动前后的状态_c2i.jpg

## 移动鼠标到指定控件

```cs
IWebElement eles=driver.FindElement(By.Id("id"));
Actions actionsObj = new Actions(driver);
actionsObj.MoveToElement(eles).Perform();
```



## 鼠标拖放

拖放操作因操作简单易懂，在Web页面中应用也越来越广，特别是一些银行页面中。以下代码演示如何通过Selenium API来完成操作。

- 方法一：通过方法**DragAndDrop()**实现

```cs
driver.Url = "http://www.jqueryui.org.cn/demo/5622.html";
Thread.Sleep(1000);
driver.Manage().Window.Maximize();
IWebElement soureEle = driver.FindElement(By.Id("draggable"));
IWebElement targetEle = driver.FindElement(By.Id("droppable"));
Actions actionObj = new Actions(driver);
actionObj.DragAndDrop(soureEle, targetEle).Perform();
IWebElement textEle = driver.FindElement(By.XPath("//div[@id='droppable']/p"));
Assert.IsTrue(textEle.Text.Contains("Dropped!"));
```

- 方法二：通过方法**MoveToElement**实现

```cs
driver.Url = "http://www.jqueryui.org.cn/demo/5622.html";
Thread.Sleep(1000);
driver.Manage().Window.Maximize();
IWebElement soureEle = driver.FindElement(By.Id("draggable"));
IWebElement targetEle = driver.FindElement(By.Id("droppable"));
Actions actionObj = new Actions(driver);
actionObj.ClickAndHold(soureEle).MoveToElement(targetEle).Release(targetEle).Perform();
IWebElement textEle = driver.FindElement(By.XPath("//div[@id='droppable']/p"));
Assert.IsTrue(textEle.Text.Contains("Dropped!"));
```



- 鼠标拖放前后的状态

- ![img](https:////upload-images.jianshu.io/upload_images/3349421-69e2f26b7d11842d.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/618/format/webp)

  11-3 鼠标拖放前后状态_c2i.jpg

## 拖动进度条或滑块

进度条或滑块一般都是通过JavaScript或JQuery实现，方便用户直观的调整值。一种使用JavaScript实现的滑块如下所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-dfbae521ed3d4c50.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/413/format/webp)





 示例代码如下： 

```cs
driver.Url = "http://demo.lanrenzhijia.com/2015/drag1218/";
Thread.Sleep(1000);
driver.Manage().Window.Maximize();
IWebElement origianlValueEle = driver.FindElement(By.Id("title"));
Assert.AreEqual("0", origianlValueEle.Text);
IWebElement sliderEle = driver.FindElement(By.Id("btn"));
Actions actionsObj = new Actions(driver);
actionsObj.DragAndDropToOffset(sliderEle, 100, 0).Perform();
Assert.AreEqual("51%", origianlValueEle.Text);
```



滑块移动前后的状态

![img](https:////upload-images.jianshu.io/upload_images/3349421-1692119e2e772efb.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/474/format/webp)

11-5 移动滑块代码实现_c2i.jpg

## 单击鼠标右键菜单

在部分网站中会有一些操作需要通过右键菜单来实现，对于这种情况Selenium API也提供一种方法**ContextClick()**方法，详细演示如下所示：

```cs
driver.Url = "http://www.helloweba.com/demo/2017/basicContext/";
Thread.Sleep(1000);
driver.Manage().Window.Maximize();
IWebElement btnEle = driver.FindElement(By.XPath("//button[contains(@class,'btn-success context')]"));
Actions actionsObj = new Actions(driver);
//点击右键菜单
actionsObj.ContextClick(btnEle).Perform();
IWebElement rightMenu = driver.FindElement(By.XPath("//div[@class='basicContext']/table/tbody/tr[2]"));
rightMenu.Click();

IAlert alert = driver.SwitchTo().Alert();
string alertText = alert.Text;
alert.Accept();
Assert.AreEqual<string>("Item clicked!",alertText);
```

## 键盘操作

在之前的文章中我们经常使用**SendKeys()**方法向页面中输入文本，下面我们将演示，在百度搜索中输入一段文字，然后通过键盘全选，再删除的案例，代码如下：

```cs
driver.Url = "https://www.baidu.com/";
Thread.Sleep(1000);
driver.Manage().Window.Maximize();
IWebElement searchBoxEle = driver.FindElement(By.Id("kw"));
searchBoxEle.SendKeys("在百度搜索框中输入一段文字");
Thread.Sleep(1000);
Actions actionObj = new Actions(driver);
//使用Ctrl+A全选输入的文字
actionObj.Click(searchBoxEle).KeyDown(Keys.Control).SendKeys("a").KeyUp(Keys.Control).Perform();
//使用键盘Backspace删除刚才输入的文字
actionObj.SendKeys(Keys.Backspace).Perform();
```



> 注意上面代码最后一行使用了**Actions**类中的**SendKeys()**方法，该方法与**IWebElement**接口中的**SendKeys()**方法不一样，注意区别，如下所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-fe4f4ecf90356233.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

11-6 SendKeys方法的区别.jpg

# 11.设置等待时间

当浏览器加载解析页面时，页面中各个元素可能并不能被同时加载完成的。在网络环境比较好的情况，用户或许觉得不到，但在网络环境较差的时候，我们就能看到浏览器在加载显示页面时，是有先后顺序的。如果在使用Selenium进行定位时，经常会出现**ElementNotVisibleException**等错误。针对这种问题，Selenium API提供了两种类型的等待：**显式等待**和**隐藏等待**。Selenium API提供等待的方法如下：

![img](https:////upload-images.jianshu.io/upload_images/3349421-14a69d640846e7a7.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/802/format/webp)

12-5 WebDriverWait类.jpg

如下图所示，在输入用户名和密码后，点击“SUBMIT”将会进行验证，同时页面上也会显示验证进度。

![img](https:////upload-images.jianshu.io/upload_images/3349421-4d2fad3a299e88ad.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/912/format/webp)

12-1 等待示例-1_c2i.jpg

在得到服务器的响应后，下面会显示验证的结果，如下图红色框中提示。

![img](https:////upload-images.jianshu.io/upload_images/3349421-a50b3d57ea7b5fad.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/911/format/webp)

12-2 等待示例验证结果_c2i.jpg

从测试的角度而言，希望验证的结果能立刻得到服务器的响应并被正确返回和显示在页面中，在某些情况下，网络会存在一些延迟。如果我们测试这种情况下的场景，应该怎么做了？在通常的测试中，会存在两种方法：**等待足够长的时间**或者**在可接受的最长时间内进行验证**。

## Sleep方法

Sleep()方法并不是Selenium提供的API，而是.NET平台自带的。这种方法对我们调试非常有用，可以让我们以慢镜头观看整个测试过程，在使用时需要添加引用**using System.Threading;** 在点击“SUBMIT”后，我们设定一个足够长的时间里等待直到服务器返回验证结果，代码如下所示：

```cs
    IWebDriver driver = new ChromeDriver();
    driver.Url = "http://www.freejs.net/demo/98/index.php";
    driver.Manage().Window.Maximize();
    driver.FindElement(By.Id("username")).SendKeys("Test");
    driver.FindElement(By.Id("password")).SendKeys("1");
    driver.FindElement(By.Name("submit")).Click();
    Thread.Sleep(10000);
    string text=driver.FindElement(By.Id("status")).Text;
    Assert.IsTrue(text.Contains("invalid username or password"));
```



> **Thread.Sleep(10000)**意味着在点击SUBMIB按钮后，需要等待10秒钟，然后再验证从服务器返回的结果。在这段时间返回的结果是正确的，则测试通过，否则测试失败。换句话说，如果服务器在第11秒返回了正确的结果，测试也是失败的。

## 显示等待

很显然，在上面的代码中直接指定一个时间也并不是一个好的解决方案，为什么了？如果指定的操作提前完成，而仍然需要等待直到达到指定时间，这样显然有些浪费时间了。那么我们有没有更好的方式了？在Selenium提供了一种方式：显式等待，它指的是让Webdriver等待某个条件成立时继续执行，否则则是在达到设定的时间后，抛出超时异常。

```cs
IWebDriver driver = new ChromeDriver();
driver.Url = "http://www.freejs.net/demo/98/index.php";
driver.Manage().Window.Maximize();
driver.FindElement(By.Id("username")).SendKeys("Test");
driver.FindElement(By.Id("password")).SendKeys("1");
driver.FindElement(By.Name("submit")).Click();
//Thread.Sleep(10000);
WebDriverWait explicitWait = new WebDriverWait(driver, TimeSpan.FromSeconds(10));
string text = explicitWait.Until(d => d.FindElement(By.Id("status"))).Text;
Assert.IsTrue(text.Contains("invalid username or password"));
```



 在上面的代码里面涉及到了C#中的**委托**和**lambda表达式**，大家可以了解一下 

## 隐式等待

隐式等待允许Web页面中的元素在特定的时间内加载完成。如果超出了设定的时间，指定的元素仍没有被加载完，则抛出异常。**默认的时间为 0 **



```cs
IWebDriver driver = new ChromeDriver();
driver.Url = "http://www.freejs.net/demo/98/index.php";
driver.Manage().Window.Maximize();
driver.FindElement(By.Id("username")).SendKeys("Test");
driver.FindElement(By.Id("password")).SendKeys("1");
driver.FindElement(By.Name("submit")).Click();
//driver.Manage().Timeouts().ImplicitWait=TimeSpan.FromSeconds(10);
driver.Manage().Timeouts().ImplicitlyWait(TimeSpan.FromSeconds(10));
string text = driver.FindElement(By.Id("status")).Text;
Assert.IsTrue(text.Contains("invalid username or password"));
```



**driver.Manage().Timeouts().ImplicitlyWait(TimeSpan.FromSeconds(10));**这种可能在后续版本将被移除，官方推荐我们使用**driver.Manage().Timeouts().ImplicitWait=TimeSpan.FromSeconds(10);**，Selenium API中的截图如下：

![img](https://upload-images.jianshu.io/upload_images/3349421-a09e307f0d34dd59.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/980/format/webp)



# 12.弹出窗口

在本节，将介绍Web页面常见和一些弹出窗口，如**上传文件**和**弹出窗口**。而大部分的弹出窗口，如选择和上传文件，大都使用的是本地的Windows窗口，而Selenium仅能操作浏览器，这样就对测试形成了挑战。Selenium API处理弹出窗口的主要是IAlert接口，详细如下所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-06be89f73eb6e8b5.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/685/format/webp)

13-3 IAlert类.jpg

## 文件上传

下图是一种常见的文件上传弹出窗口：

![img](https:////upload-images.jianshu.io/upload_images/3349421-b258c1c3645c1949.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/683/format/webp)

13-1 文件上传示例_c2i.jpg

HTML源码如下：

```xml
<body>
  请选择上传文件:<input type="file" name="fileUpload" id="fileUpload" size="50" />
</body>
```



 相应的定位代码如下： 

```cs
    string filePath = @"C:\类和对象.jpg";
    driver.FindElement(By.Id("fileUpload")).SendKeys(filePath);
```

 在以上的代码里面，直接将上传的文件路径写成绝对路径了，这样做灵活性不够，扩展性较差，并不推荐。建议在做测试的时候，将测试文件放在测试项目里面，这样我们就可以使用相对路径来选择文件。示例代码如下： 

```cs
string filPath = MyClass.GetFilePath() + @"testData\类和对象.jpg";
driver.FindElement(By.Id("fileUpload")).SendKeys(filePath);
```

## JavaScript弹出窗口

JavsScript弹出窗口是通过使用Javascript创建，主要用来确认操作和提示消息。一种常见的弹出框如下所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-cb39ed48d049b942.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/251/format/webp)

13-2 弹出窗口示例.jpg

HTML源码如下：

```xml
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="utf-8" />
    <title></title>
</head>
<script type="text/javascript">
    function show_confirm() 
    {
        var r = confirm("请点击");
    }
</script>
</head>
<body>
    请选择上传文件： <input type="file" name="fileUpload" id="fileUpload" size="50" />
    <br /><br />
    <input type="button" onclick="show_confirm()" value="显示确认对话框" />
</body>
</html>
```



定位示例代码：

- 方法一：使用Selenium API中的IAlert接口

```cs
    driver.FindElement(By.XPath("//input[@value='显示确认对话框']")).Click();
    IAlert alert = driver.SwitchTo().Alert();
    if (alert.Text.Contains("请点击"))
    {
        alert.Accept();
    }
    else
    {
        alert.Dismiss();
    }
```



- 方法二：使用JavaScript

```cs
((IJavaScriptExecutor)driver).ExecuteScript("window.confirm=function(){return true;}");
((IJavaScriptExecutor)driver).ExecuteScript("window.alert=function(){return true;}");
((IJavaScriptExecutor)driver).ExecuteScript("window.prompt=function(){return true;}");
driver.FindElement(By.XPath("//input[@value='显示确认对话框']")).Click();
```



## 模态对话框

随着JavaScript的发展，出现一些非常灵活易用的JavaScript库，如Bootstrap，这些JavaScript库逐渐替换了Javascript默认的提示框。一种典型的对话框如下所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-d3499ee44d4c0c44.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/603/format/webp)

13-4 模态对话框示例_c2i.jpg

相比较好于原生态的JavaScript提示框，对于模态对话框，写测试脚本要容易很多，示例代码如下：





```cs
driver.FindElement(By.Id("modalpopup")).Click();
Thread.Sleep(500);
driver.FindElement(By.XPath("//button[contains(@class,'btn btn-default')]")).Click();
```



## 设定超时时间

在测试过程中，弹出窗口未及时处理，将导致测试失败。对于这种情况，我们通过给测试方法添加属性**TimeoutAttribute**来指定最长的时间。



```cs
    [TestMethod]
    [Timeout(10*1000)] //超时时间为10秒
    public void TestModalPopUp()
    {
     //实现代码
    }
```

 在执行用例时，如果超出设定的时间，会出现以下报错
**Test 'TestModalPopUp' exceeded execution timeout period.** 



# 13.Assert

断言（Assert）就是检查点，用来判定测试是否通过的标志。在一个测试脚本中，如果没有断言，则该测试用例也失去了测试的意义。我们先来看看微软提供的单元测试框架中的Assert类，如下图所示：

![img](https:////upload-images.jianshu.io/upload_images/3349421-35470b0e5665bfb4.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

14-1 Assert类.jpg

微软框架中提供的断言有很多个，我们常用的有**Assert.IsTrue()、Assert.IsFalse()、Assert.AreEqual()、Assert.AreNotEqual()等包含常规类型和泛型**

在Selenium中常被用于断言的条件如下所示：

>  

- 页面标题（等于或包含）
- 页面内容（包含或不包含某些内容）
- 页面源码（包含或不包含某些内容）
- 元素中输入的内容（等于或包含）
- 元素显示的内容（等于或包含）
- 元素的状态（选中、禁用、显示状态）

## 断言页面标题

```cs
    driver.Url = "https://www.baidu.com";
    driver.Manage().Window.Maximize();
    Assert.AreEqual("百度一下，你就知道",driver.Title);
```



## 断言页面内容

```cs
    driver.Url = "https://www.baidu.com";
    driver.Manage().Window.Maximize();
    string eleText=driver.FindElement(By.Id("setf")).Text;
    Assert.IsTrue(eleText.Contains("把百度设为主页"));
```



## 断言页面源码

```cs
    driver.Url = "https://www.baidu.com";
    driver.Manage().Window.Maximize();
    string expectText = "京公网安备11000002000001号";
    Assert.IsTrue(driver.PageSource.Contains(expectText));
```



## 断言复选框状态

```cs
    IWebElement checkEle=driver.FindElement(By.Id("isChecked"));
    Assert.IsTrue(checkEle.Selected);
```



## 断言按钮状态

 HTML源码如下： 

```xml
<label id="label">测试Label内容</label>
```

 断言代码： 

```cs
    string labelText=driver.FindElement(By.Id("label")).Text;
    string expectText = "测试Label内容";
    Assert.AreEqual<string>(expectText, labelText);
```

## 断言span文本内容

HTML源码如下：

```xml
<span id="span">测试Span内容</span>
```



 从上面的HTML源码可以看到span其实跟Label是一样的，断言代码也会很相似，如下所示 

```cs
    string spanText = driver.FindElement(By.Id("span")).Text;
    string expectText = "测试Span内容";
    Assert.AreEqual<string>(expectText, spanText);
```



## 断言div文本内容

HTML源码如下：

```xml
<div id="div1">
    Test div
    <div id="divChild1">divChild1</div>
    <div id="divChild2">divChild2</div>
 </div>
```



 断言代码： 

```cs
Assert.AreEqual<string>("Test div\r\ndivChild1\r\ndivChild2", driver.FindElement(By.Id("div1")).Text);
Assert.AreEqual<string>("divChild1", driver.FindElement(By.Id("divChild1")).Text);
Assert.IsTrue(driver.FindElement(By.Id("divChild2")).Text.Contains("divChil"));
```



## 断言表格文本内容

HTML源码如下：

```xml
<table id="testTable" cellpadding="1" border="1" width="50%">
    <tr id="row1">
        <td id="cell1">Cell-1-1</td>
        <td id="cell2">Cell-1-2</td>
    </tr>
    <tr id="row2">
        <td id="cell3">Cell-2-1</td>
        <td id="cell4">Cell-2-2</td>
    </tr>
</table>
```



 断言代码： 

```cs
   IWebElement table= driver.FindElement(By.Id("testTable"));
   string allTableText =table.Text;
   string expectText="Cell-1-1 Cell-1-2\r\nCell-2-1 Cell-2-2";
   Assert.AreEqual(expectText, allTableText);
   string js = "return arguments[0].outerHTML;";
   var htmlSource = ((IJavaScriptExecutor)driver).ExecuteScript(js, table).ToString();
   string resultText="<td id=\"cell3\">Cell-2-1</td>";
   Assert.IsTrue(htmlSource.Contains(resultText));
```



## 断言单元格内容

```cs
    IWebElement cell = driver.FindElement(By.Id("cell1"));
    string expectText = "Cell-1-1";
    Assert.AreEqual<string>(expectText, cell.Text);
```



## 断言表格行内容

```cs
    IWebElement row = driver.FindElement(By.Id("row2"));
    string expectText = "Cell-2-1 Cell-2-2";
    Assert.AreEqual<string>(expectText, row.Text);
```



## 断言图片当前状态

在一些网页页面中，当某些事件发生后，才会触发另外一些事件，例如下图所示的，点击按钮进行显示和隐藏图片：

![img](https://upload-images.jianshu.io/upload_images/3349421-42cc5b5293e68bcd.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/521/format/webp)

14-2 图片显示和隐藏_c2i.jpg

HTML源码如下：

```xml
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta charset="GB2312" />
    <title>测试图片当前状态</title>
    <script>
        function ShowOrHideImg()
        {
            var target = document.getElementById('imgDiv');
            if (target.style.display=='block') {
                target.style.display = 'none';
            }
            else {
                target.style.display = 'block';
            }
        }
    </script>
</head>
<body>
    <div id="btn" class="button">
      <input type="button" id="button" value="显示/隐藏图片" onclick="ShowOrHideImg()" />
    </div>
    <div id="imgDiv"class="divName" style="display:none;">
      ![](img\Selenium.jpg)
    </div>
</body>
</html>
```



 断言代码如下： 

```cs
    driver.FindElement(By.Id("button")).Click();
    IWebElement imgEle = driver.FindElement(By.Id("img"));
    Assert.IsTrue(imgEle.Displayed);
    driver.FindElement(By.Id("button")).Click();
    Assert.IsFalse(imgEle.Displayed);
```



 断言在日常测试过程是非常重要的一个环节，在测试过程中我们需要找到合适的断言点，以此来判断测试能否通过，所以需要我们结合自身项目和Assert类来写好更好的自动化测试用例。 