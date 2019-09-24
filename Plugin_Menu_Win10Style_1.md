# 【RpgmakerMV】插件编写实战

**工具&软件：**Visual studio code，PS，Rpgmaker MV

**Tip：**涉及到的JS知识

**参考：**其他参考资料

**目标：**仿win10开始菜单

**任务：**

- [x] 添加图片背景
- [x] 调整菜单窗口的布局：位置，大小，透明度
- [x] 在命令文字旁绘制图片Icon
- [x] 添加新的窗口，并在窗口内显示文字或者设置一个命令
- [x] 动态效果的实现（磁贴向上滑动动画）
- [x] 修改选择高亮框的大小

**目录：**

[【RpgmakerMV】插件编写实战一：跳过标题，为窗口添加win瓷砖风格背景&自定义菜单窗口位置、大小、透明度](https://qsociety.github.io/RMMVTutorial/Plugin_Menu_Win10Style_1)

[【RpgmakerMV】插件编写实战系列三：实现磁贴滑动效果&将选择高亮框扩展覆盖至整块磁贴，添加插件描述、作者、帮助及命令参数](https://qsociety.github.io/RMMVTutorial/Plugin_Menu_Win10Style_2)

[【RpgmakerMV】插件编写实战系列二：仿win10菜单列表，新增窗口并显示指定文字/命令](https://qsociety.github.io/RMMVTutorial/Plugin_Menu_Win10Style_2)



<img src="https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1569235911234.png?raw=true" alt="1569235911234.png" style="zoom:50%;" />

**RMMV插件目录结构：**

![pluginDesc.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/pluginDesc.png?raw=true)

### **【RpgmakerMV】插件编写实战一：跳过标题，为窗口添加win瓷砖风格背景&自定义菜单窗口位置、大小、透明度**

#### 1、使用插件跳过标题界面

首先我们在你的游戏\js\plugins中新建一个js文件，这里我命名为win10_menu.js，使用vscode打开编辑，并建立一个**私有作用域**，代码如下：

`(function(){})();`

> **Tip：**
>
> 与私有作用域相对的是全局作用域，一般的变量和函数会放在全局作用域中，但当程序较大，有多人参与时，过多的全局变量和函数很容易导致命名冲突。而私有作用域实际是一个模仿块级作用域的匿名函数，在匿名函数中定义的任何变量，都会在执行结束时被销毁。通过建立私有作用域，每个开发人员既可以使用自己的变量，又不必担心搞乱全局作用域。
>

我们想要先跳过标题动画，这样方便测试，标题也是一个场景，也就是scene，如RMMV插件目录结构所示，我们可以在rpg_scenes.js中找到这样一段代码：

```javascript
Scene_Boot.prototype.start
```

看到boot，start，goto(Scene_Map)等词的字面意思，我们大概可以猜到可以通过改写这段代码跳过标题，修改后如下，把它放入之前建立的私有作用域中：

```javascript
(function(){

    Scene_Boot.prototype.start = function() {
        Scene_Base.prototype.start.call(this);//让Scene_Boot继承Scene_Base的属性和方法
        SoundManager.preloadImportantSounds();
        this.checkPlayerLocation();//检查角色位置
        DataManager.setupNewGame();//开始新游戏
        SceneManager.goto(Scene_Map);//进入地图场景
        this.updateDocumentTitle();
    };
    
})();
```

在rpgmaker mv插件管理器中载入我们编辑好的插件，运行游戏测试，这时会直接进入地图界面。

> **Tip：**在一个子构造函数中，可以通过调用父构造函数的 `call` 方法来实现继承。写一个方法，然后让另外一个新的对象来继承它，而不是在新对象中再写一次这个方法，这里 Scene_Boot便继承了Scene_Base的属性和方法。
>
> **参考：**https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call

#### 2、修改菜单窗口位置、大小、透明度，并为窗口添加瓷砖背景

我们先尝试载入一张图片作为整个菜单的背景，本例我们选择phoneApp\img\wpmenu\layout.png.

rpgmaker mv中载入图片文件有这样一种方法，先把它放在前边，供需要时调用：

```JavaScript
ImageManager.loadWPMenus = function(filename) {
        return this.loadBitmap('img/wpmenu/', filename, 0, true);//从img/wpmenu/文件夹中加载指定图片文件，本例中所有的图片都放在这个文件夹
    };
```

> Tip：loadBitmap是从模块的可执行文件中加载指定的位图资源的函数。

先在RMMV自带的Community_Basic.js插件中修改游戏分辨率为1200X720，然后为了方便我们布局背景图片的位置，可以在PS中将layout.png的尺寸同样调整为1200X720，RMMV中坐标原点的位置（0，0）在左上角，背景图片大小设置为游戏分辨率大小，就不用我们再去调整图片坐标点的位置，可以直接在这张图片里决定好背景图要在的位置，我们得到的图片如下，是win10开始菜单底图的样式：

<img src="https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/layout.png?raw=true" alt="layout.png" style="zoom:50%;" />

在rpg_scenes.js中，我们可以在Scene_MenuBase类中找到Scene_MenuBase.prototype.createBackground，通过它我们可以便在菜单场景中载入图片元素。

不要直接修改这个函数，会影响整体，Scene_Menu是Scene_MenuBase的子类，继承了它的属性和方法，所以Scene_Menu.prototype.createBackground也是可以的，我们在插件中定义一个新的私有变量并把对应的函数赋值给它，代码如下：

```javascript
var _wpMenu_createBackground = Scene_Menu.prototype.createBackground;
    Scene_Menu.prototype.createBackground = function(){
        _wpMenu_createBackground.call(this);
        this._field = new Sprite();//可以把这个_field理解成一块黑板，接下来我们所有的图片元素都得贴到这块黑板上
        this.addChild(this._field);
    };
```

> **Tip：**下划线是一种常用的记号，通常变量前加下划线表示“私有变量”，函数名前加下划线表示“私有函数”，没有特别的含义，只是为了维护方便。

光上面的代码是不够的，我们可以在Scene_Menu代码部分找到这样一个函数：Scene_Menu.prototype.create，可以发现主菜单显示的命令窗口，角色状态窗口，金币数量展示窗口都是在这个函数中定义的，所以接下来我们加载图片的方法Scene_Menu.prototype.createSprites也需要先在这里“登记”一下，将this,createSprites()放入Scene_Menu.prototype.create中，同上文的处理方式一样，将这个函数赋值给私有变量_wpMenu_create，代码如下：

```JavaScript
var _wpMenu_create = Scene_Menu.prototype.create;
    Scene_Menu.prototype.create = function() {
        _wpMenu_create.call(this);
        this.createSprites();//增加图片元素
    };
    //各种图片都放在这里
    Scene_Menu.prototype.createSprites = function(){
        this.createLayout();//主背景图
    }
    //加载主背景图的方法的执行
    Scene_Menu.prototype.createLayout = function() {
        this._layout = new Sprite(ImageManager.loadWPMenus("layout"));//会自动从img/wpmenu/文件夹中找到对应名字的图片加载进来
        this._field.addChild(this._layout);	
    };
```

到这里我们测试游戏，layout.png图片便已显示在菜单背景中。

> 这里结合以上内容介绍一些JavaScript面向对象程序中继承和原型链的相关知识，帮助大家理解。
>
> 前面的内容我们提到了Scene_Menu和Scene_MenuBase两种类型，他们分别有自己的属性和方法，比如Scene_Menu.prototype.create就是Scene_Menu的一个方法，且Scene_Menu继承了Scene_MenuBase的属性和方法。
>
> 原来存在于Scene_MenuBase中的所有属性和方法，现在也存在于Scene_Menu.prototype 中了。在确立了继承关系之后，我们可以给Scene_Menu.prototype添加新的方法，比如creat方法，这样就在继承了Scene_MenuBase 的属性和方法的基础上又添加了一个新方法。
>
> 当以读取模式访问一个实例属性时，首先会在实例中搜索该属性。如果没有找到该属性，则会继续搜索实例的原型。在通过原型链实现继承的情况下，搜索过程就得以沿着原型链继续向上。就拿上面的例子来说，调用 this.createSprites()会经历三个搜索步骤：
>
> 1）搜索实例；2）搜索Scene_Menu.prototype；3）搜索Scene_MenuBase.prototype，最后一步才会找到该方法。在找不到属性或方法的情况下，搜索过程总是要一环一环地前行到原型链末端才会停下来。
>
> 关系图如下：
>
> ![2019-8-16-scene_menu.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/2019-8-16-scene_menu.png?raw=true)

接下来我们调整主菜单各个窗口到合适的位置，以命令窗口，_commandWindow为例，将以下代码添加到Scene_Menu.prototype.createSprites = function(){ }中，跟在this.createLayout()后面即可：

```javascript
Scene_Menu.prototype.createSprites = function(){
        //主背景图
        this.createLayout();
        //定义窗口坐标、宽、高信息
        this._commandWindow.x = 50;
        this._commandWindow.y = 90;
        this._commandWindowOrg = [this._commandWindow.x,this._commandWindow.y];
        this._commandWindow.width = 130;
        this._commandWindow.height = 550;
    }
```

重新测试游戏，命令窗口是不是已经改变了位置和大小？

同样的方法，可以为不同的窗口添加不同的图片背景，只是这一次需要将载入的图片坐标和窗口的坐标绑定在一起，并且要将窗口的透明度调整为0，为了实现这些，我们还需要创建Scene_Menu.prototype.update，Scene_Menu.prototype.updateSprites，Scene_Menu.prototype.updateLayout，当检测到用户加载了背景图后，会自动刷新，并执行updateLayout中设置的内容，完整代码如下，复制到你的插件中即可，本节结束：

<img src="https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568797351921.png?raw=true" alt="1568797351921.png" style="zoom:50%;" />

```javascript
(function(){
    //加载图片
    ImageManager.loadWPMenus = function(filename) {
        return this.loadBitmap('img/wpmenu/', filename, 0, true);//从img/wpmenu/文件夹中加载指定图片文件，本例中所有的图片都放在这个文件夹
    };
    //跳过标题
    Scene_Boot.prototype.start = function() {
        Scene_Base.prototype.start.call(this);//让Scene_Boot继承Scene_Base的属性和方法
        SoundManager.preloadImportantSounds();
        this.checkPlayerLocation();//检查角色位置
        DataManager.setupNewGame();//开始新游戏
        SceneManager.goto(Scene_Map);//进入地图场景
        this.updateDocumentTitle();
    };
    //载入图片元素
    var _wpMenu_createBackground = Scene_Menu.prototype.createBackground;
    Scene_Menu.prototype.createBackground = function(){
        _wpMenu_createBackground.call(this);
        this._field = new Sprite();//可以把这个_field理解成一块黑板，接下来我们所有的图片元素都得贴到这块黑板上
        this.addChild(this._field);
    };
   
    var _wpMenu_create = Scene_Menu.prototype.create;
    Scene_Menu.prototype.create = function() {
        _wpMenu_create.call(this);
        this.createSprites();//增加图片元素
    };
    //各种图片都放在这里
    Scene_Menu.prototype.createSprites = function(){
        //主背景图
        this.createLayout();
        //角色状态背景图
        this.createLayoutStatus();
        //定义命令窗口坐标、宽、高信息
        this._commandWindow.x = 50;
        this._commandWindow.y = 90;
        this._commandWindowOrg = [this._commandWindow.x,this._commandWindow.y];
        this._commandWindow.width = 130;
        this._commandWindow.height = 550;
        //定义角色状态窗口坐标、宽、高信息
        this._statusWindow.x = 190;
        this._statusWindow.y = 100;
        this._statusWindowOrg = [this._statusWindow.x,this._statusWindow.y]
        this._statusWindow.width = 474;
        this._statusWindow.height = 450; 
    }
    //加载主背景图的方法的执行
    Scene_Menu.prototype.createLayout = function() {
        this._layout = new Sprite(ImageManager.loadWPMenus("layout"));//会自动从img/wpmenu/文件夹中找到对应名字的图片加载进来
        this._field.addChild(this._layout);	
    };
    Scene_Menu.prototype.createLayoutStatus = function() {
        this._layoutStatus = new Sprite(ImageManager.loadWPMenus("LayoutStatus"));
        this._field.addChild(this._layoutStatus);	
    };

    var _wpMenu_update = Scene_Menu.prototype.update;
    Scene_Menu.prototype.update = function() {
    _wpMenu_update.call(this);
    //检测到有_layout存在，则调用updateSprites()方法
    if (this._layout) {this.updateSprites()};
};
    Scene_Menu.prototype.updateSprites = function() {
        this.updateLayout();	
   };
    Scene_Menu.prototype.updateLayout = function() {
        this._layoutStatus.x = this._statusWindow.x;
        this._layoutStatus.y = this._statusWindow.y;
        this._layoutStatus.opacity = this._statusWindow.contentsOpacity;
        this._statusWindow.opacity = 0;	
    };

})();
```


