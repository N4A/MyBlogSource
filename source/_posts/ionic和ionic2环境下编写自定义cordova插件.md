---
title: ionic和ionic2环境下编写自定义cordova插件
date: 2017-04-17 21:40:11
tags: [ionic, ionic2, cordova, android]
---

## 1 增加android的平台

对于一个ionic项目，在主目录下通过以下命令行增加android平台。

```shell
cordova platform add android
```

<!-- more -->

然后在platforms目录下就会出现一个android文件夹：

![android directory](/img/ionic/android_cat.png)

之后可以使用配置了android开发环境的eclipse选中上图android目录打开该项目；也可以使用Android Studio打开上图android目录，这时候需要要配置gradle,直接选择确定下载即可（此时建议开启全局代理），注意项目的路径中不能含有非ascii码字符（例如中文），否则gradle会build失败。

本例使用Android Studio来演示，eclipse环境下只要找到对应的文件即可。

## 2 使用Android Studio开发插件源码

### 2.1 介绍项目目录

打开项目后，项目目录如下：

![android project](/img/ionic/android_studio.png)

该项目有两个模块，第一个是CordovaLib，里面是cordova库的代码，我们就不用管了（如果是用eclipse打开这会显示为一个单独的项目）

第二个模块便是我们自己的项目了，就如普通的android项目目录一样。我们在项目主目录WWW目录下的代码都被原封不动的拷贝到assets/www目录下，又额外添加了一些cordova js库，这些文件我们也不用管。

### 2.2 开发插件源码

如下图，新建一个org.apache.cordova.hello包，改包名可随意取。然后在改包下新建一个HelloPlugin.java文件：

![Hello Plugin](/img/ionic/hello.png)

HelloPlugin的代码如下：

```java
package org.apache.cordova.hello;

import android.widget.Toast;
import org.apache.cordova.CallbackContext;
import org.apache.cordova.CordovaPlugin;
import org.json.JSONArray;
import org.json.JSONException;

/**
 * Created by duocai on 2017/4/10.
 */
//所有的自定义plugin都需要继承CordovaPlugin这个类
public class HelloPlugin extends CordovaPlugin {

  //然后需要复写这个execute方法
  @Override
  public boolean execute(String action, JSONArray args, CallbackContext callbackContext)
    throws JSONException {

    if (action.equals("hello")) {
      String text = "";
      for (int i = 0; i < args.length(); i++) {
        text += args.getString(i) + "\n"; //这取决于js代码传进来的参数类型。
      }
      Toast.makeText(this.cordova.getActivity(), text, Toast.LENGTH_LONG).show();
      callbackContext.success("success: " + text);//使用回掉的方法，返回信息
      //callbackContext.error("dd"); //对应有返回错误的方法
      return true;
    }

    return false;
  }
}

```

execute有三个参数，这个对应js的调用代码会更好理解（详见2.4部分）：

```javascript
cordova.exec(callbackContext.success, callbackContext.error, PluginName, action, args);
```

其中action就对应action；args在js端就是普通的js数组；callbackContext.success, callbackContext.error分别是成功和失败时的两个回掉函数，前面java代码中：

```java
callbackContext.success("success: " + text);//使用回掉的方法，返回信息
```

这一行就调用了第一个回掉函数。

PluginName的意义就是指定了当前的java文件（详见2.3本分）

### 2.3 在配置文件中配置HelloPlugin：

2.1的目录结构中，我们可以看到res/xml目录下有一个config.xml配置文件，打开文件增加配置如下：

```xml
<?xml version='1.0' encoding='utf-8'?>
<widget id="com.ionicframework.starter" version="0.0.1" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0"> <!-- xmlns若是报找不到文件错误也不用管-->
    <name>HelloCordova</name>
    <description>
        An Ionic Framework and Cordova project.
    </description>
    <author email="you@example.com" href="http://example.com.com/">
      Your Name Here
    </author>
    <content src="index.html" />
    <access origin="*" />
    <preference name="loglevel" value="DEBUG" />
    <preference name="webviewbounce" value="false" />
    <preference name="UIWebViewBounce" value="false" />
    <preference name="DisallowOverscroll" value="true" />
    <preference name="SplashScreenDelay" value="2000" />
    <preference name="FadeSplashScreenDuration" value="2000" />
    <preference name="android-minSdkVersion" value="16" />
    <preference name="BackupWebStorage" value="none" />
    <!-- 前面为自动生成，含义由名字可以看出个大概>

    <!--新增配置文件 name="HelloPlugin"指定了js端调用时需要传递的PluginName参数-->
    <feature name="HelloPlugin">
      <!--value的值为刚刚开发的HelloPlugin文件的路径-->
      <param name="android-package" value="org.apache.cordova.hello.HelloPlugin" />
    </feature>

</widget>

```

### 2.4 通过js代码调用插件

```javascript
/**
 * Created by duocai on 2016/9/6.
 */
angular.module('ctrl.camera', [])

  .controller('CameraCtrl', function($scope) {
    var success = function(data) {
      alert(data);
    }
    
    //参数与前面的java代码对应如下
    //js: cordova.exec(callbackContext.success, callbackContext.error, PluginName, action, args);
    //java: public boolean execute(String action, JSONArray args, CallbackContext callbackContext)
    // 此外 js 多余的 PluginName 在2.3的配置文件中配置
    //<feature name="HelloPlugin"> // PluginName
      //<param name="android-package" value="org.apache.cordova.hello.HelloPlugin" />
    //</feature>
    cordova.exec(success, null, "HelloPlugin", "hello", ["hello", "world", "!!!"]);
  });

```

其余没有注释的部分是angular controller的语法，就不再赘述。

至此就利用cordova的库完成了通过js代码调用原生java代码。

### 2.5  效果演示

![demo](/img/ionic/demo1.jpg)

其中下面小黑框是java代码的输出，上面大黑框是js代码的输出。

## 3 配置为符合ionic插件格式的插件

我们都知道使用在开发ionic项目时，是不需要上面那样在android studio中使用插件的。这是cordova 的插件进行了进一步的封装，知道了上面的过程后，我们就知道ionic的插件格式要求到底干了什么事。

在开发之前需要做一点准备，每次执行cordova build android 的时候，都会复写platforms/android目录下文件。所以我们将刚刚的android项目目录改名为android plugin development。然后在执行：

```shell
cordova platform add android
```

来增加 android 平台

### 3.1 插件目录

![my plugin](/img/ionic/plugin.png)

按上图所示建一个插件目录。src和www无所谓怎么建，但这样建条理更清晰，plugin.xml一定要在其所在的位置。

### 3.2 各个文件内容

1. HelloPlugin.java就是上面的文件直接拷贝过来即可

2. plugin.xml:

    ```xml
    <?xml version="1.0" encoding="utf-8"?>
    <plugin id="org.cordova.HelloPlugin" version="0.0.1"
            xmlns="http://apache.org/cordova/ns/plugins/1.0"
            xmlns:android="http://schemas.android.com/apk/res/android">
      <name>HelloPlugin</name>
      <description>Description</description>
    
      <!-- js 部分 -->
      <!-- plugin 'org.cordova.HelloPlugin'(看上面id) 下 定义module HelloPlugin -->
      <!-- 配置后会将HelloPlugin.js配置到android项目assets/www/plugins 目录下，做了语法补充完整，
    	有兴趣可以自己打开看看 -->
      <js-module name="HelloPlugin" src="www/HelloPlugin.js">
        <clobbers target="HelloPlugin"/>
      </js-module>
    
      <!--android 部分就是
      1. 在第2部分提到的res/xml/config.xml添加plugin配置
      2. 将对应的文件拷贝到对应的目录下-->
      <platform name="android">
        <config-file parent="/*" target="res/xml/config.xml">
          <!--新增配置文件 name="HelloPlugin"指定了js端调用时需要传递的名称参数-->
          <feature name="HelloPlugin">
            <!--value的值为刚刚开发的HelloPlugin文件的路径-->
            <param name="android-package" value="org.apache.cordova.hello.HelloPlugin" />
          </feature>
        </config-file>
        <source-file src="src/android/HelloPlugin.java" target-dir="src/org/apache/cordova/hello"/>
      </platform>
    </plugin>
    ```


3. HelloPlugin.js

    ```java
    /**
    * Created by admin on 2017/4/10.
    */
    var exec = require('cordova/exec');
    
     // 符合格式要求可在其它js文件中调用该模块
     module.exports = {
    
       show: function (message) {
      //基本上等同于前面第2部分所说的cordova.exec()
         exec(null,null, "HelloPlugin", "hello", [message]);
       }
    
     };
    ```

4. 调用插件，依然以前面的CameraCtrl为例

    ```javascript
      /**
    * Created by duocai on 2016/4/10.
    */
    angular.module('ctrl.camera', [])
    
     .controller('CameraCtrl', function($scope) {
       HelloPlugin.show("hello world !!!") // HelloPlugin在前面的plugin.xml里配置
     });
    ```

### 3.3 运行结果

   ![demo](/img/ionic/demo2.png)

## 4 进一步分析

1. 比较第二部分和第三部分，如果是要自己写插件的化，不如待到前端写的差不多了，然后直接转移到android studio项目来写，如第二部分那样扩展原生代码，并通过js调用。修改少量的js代码和html。这样做使得扩展功能变得简单，但是这样做的后果是没有多平台。
2. 比较第二部分和第三部分，可以得出js-module不是必须配置的，配置之后则可以使用cordova进一步封装的语法来调用。针对于自己的插件，我们可以直接调用cordova.exec(), 但是这样做的后果是模块不分离。

## 5 对于ionic2的说明【2017-6-22更新】
angular升级到angular2，ionic也随之升级。那么针对于ionic2该如何使用自定义插件呢？这里只说使用是因为cordova并没有升级，所以它的工作原理和插件的编写并不会有所改变，只是使用方式要发生变化。那么到底如何使用呢。

1. ionic2自带的插件，我看了一下使用npm管理到@ionic-native空间下，又进行了一层封装，这对于自定义插件来说，使用成本较高，不推荐。
2. 在前面所说的在android studio中直接编程是否依然有效呢。答案当然是依然有效。Typescript在编译时依然会编译成js es5的语法，所以在android studio中，依然只能看到普通的js代码，在这里面使用cordova.exec()方法的效果当然是与ionic1中一样了。（但是编译后的js文件难读，对编程不友好，这里也不推荐使用）
3. 除了ionic内置插件的使用方式（这是一种好的方式）外，还有没有其它使用方式呢，答案是有的，而且方法很多，下面会说一种推荐的方法。

下面会以ionic2 tabs样式的初始项目做一个讲解。

### 5.1 继续在android stuio中编写
当我们使用android studio如上所示打开项目时，目录结构依然不变，不过我们自己写的代码都被编译到assets/www/build/main.js 这个文件中，这个文件很长也很难阅读，我们可以通过搜索直接定位要修改的位置（搜索方法名，类名之类的），例如我搜索ionic2 tabs样式初始项目中ContactPage这个类定位到如下位置：
```javascript
var ContactPage = (function () {
    function ContactPage(navCtrl) {
        this.navCtrl = navCtrl;
    }
    return ContactPage;
}());
```
接下来在构造函数(构造函数会在页面初始加载时被调用)中添加前面2.4所说的直接调用（前提是要像第2部分所说的配置好）
```javascript
var ContactPage = (function () {
    function ContactPage(navCtrl) {
        this.navCtrl = navCtrl;
        var success = function(data) {
	      alert(data);
	    }
	    cordova.exec(success, null, "HelloPlugin", "hello", ["hello", "world", "!!!"]);
    }
    return ContactPage;
}());
```
之后，编译运行，效果如下图所示，与前面所展示无异。但是现在这个js文件不像是ionic1中的直接拷贝而是编译形成的，难以阅读，也不可能被再复用，所以不推荐使用这种方式。仅供大家了解一下插件进一步是怎么工作的。
![这里写图片描述](/img/ionic/demo3.png)

### 5.3 封装cordova插件并在ionic2中调用（推荐）
封装插件，前面第三部分说的非常清楚，封装方式与前面无异。额外补充一点就是前面好像忘了说明这个插件是怎么用的。使用方式很简单，例如将前面HelloPlugin目录放在当前项目的目录下，然后使用命令行：
 ionic plugin add HelloPlugin
命令格式是**ionic plugin add 插件目录路径**，刚刚这个例子使用相对路径。
添加好插件后，接下来说明如何在ionic2中使用，前面已经说了使用方式有很多种，接下来说明一种推荐的，以ionic2 tabs样式初始项目src/pages/contact/contact.ts这个组件为例，它原始如下所示：
```typescript
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';

@Component({
  selector: 'page-contact',
  templateUrl: 'contact.html'
})
export class ContactPage {

  constructor(public navCtrl: NavController) {
  }

}

```
 然后为了使用HelloPlugin插件，应当做如下修改：
```typescript
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
// 利用declare语法声明HelloPlugin对象，不需要初始华。这个语法有点像C++里的声明语法。
declare let HelloPlugin;
@Component({
  selector: 'page-contact',
  templateUrl: 'contact.html'
})
export class ContactPage {

  constructor(public navCtrl: NavController) {
    // 这里使用就像前面第三部分所说的一样，但是为了在Typescript中使用HelloPlugin这个全局变量，需要前面的声明，不然typescript编译时会报错，因为找不到HelloPlugin
    HelloPlugin.show("hello world !!!")
  }
}
```
然后说面一下HelloPlugin这个变量名字是从哪里来的。看前面3.2.2 plugin.xml配置文件中的如下部分:
```xml
<--就是在这里配置的-->
<js-module name="HelloPlugin" src="www/HelloPlugin.js">
        <clobbers target="HelloPlugin"/>
      </js-module>
```
这种使用插件的方式非常简单，只需在文件的开始部分使用
**declare let 对象名**
来声明这个对象就可以在typescript中直接使用，你完全可以把他当成是一种import，来引用你在插件中定义的全局对象。所以这是一种推荐的使用方式。
最后效果如下：
![这里写图片描述](http://img.blog.csdn.net/20170622221238308?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvSkFWQV9ONEE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 5.4 刚刚的declare是怎么工作的。
我们都知道一个html文件如果引用了多个js文件，那么全局变量在这些文件中是可以直接相互引用的。刚刚的declare所做的事情就是：尽管我没有初始化这个对象，但是我会在其它某个文件中，声明并初始化这个全局对象，所以接下来文件中typescript要绕过对这个对象的检查。
我们看一下编译后main.js文件（前面提到过）：
``` javascript
var ContactPage = (function () {
    function ContactPage(navCtrl) {
        this.navCtrl = navCtrl;
        // Test Hello plugin
        HelloPlugin.show("hello world !!!");
    }
    return ContactPage;
}());
```
然后在整个js文件中，再也找不到第二个HelloPlugin。所以前面的declare let HelloPlugin在typescript中实现就是告知Typescript忽略接下来对Helloplugin的语法检查，将其原封保留即可。
等到编译成js文件，就完全和ionic1是一样的环境了。