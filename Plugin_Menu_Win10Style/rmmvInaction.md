# 【RpgmakerMV】插件编写实战

**工具&软件：**Visual studio code，PS，Rpgmaker MV

**Tip：**涉及到的JS知识

**参考：**对应《JavaScript高级程序设计》中涉及到这部分知识点的章节，或者其他参考资料

**目标：**仿win10开始菜单

**目录：**

[TOC]

RMMV插件目录结构：

<img src="F:\New server\MV_Online\phoneApp\js\img\1568810179082.png" alt="1568561611621" style="zoom: 67%;" />

### **【RpgmakerMV】插件编写实战一：跳过标题，为窗口添加win瓷砖风格背景&自定义菜单窗口位置、大小、透明度**

#### 1、使用插件跳过标题界面

首先我们在你的游戏\js\plugins中新建一个js文件，这里我命名为win10_menu.js，使用vscode打开编辑，并建立一个**私有作用域**，代码如下：

`(function(){})();`

> **Tip：**
>
> 与私有作用域相对的是全局作用域，一般的变量和函数会放在全局作用域中，但当程序较大，有多人参与时，过多的全局变量和函数很容易导致命名冲突。而私有作用域实际是一个模仿块级作用域的匿名函数，在匿名函数中定义的任何变量，都会在执行结束时被销毁。通过建立私有作用域，每个开发人员既可以使用自己的变量，又不必担心搞乱全局作用域。
>
> **参考：**7.3 模仿块级作用域

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

<img src="F:\New server\MV_Online\phoneApp\js\img\layout.png" alt="1568782193464" style="zoom:50%;" />

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

![1568797351921](F:\New server\phoneApp\js\img\1568797351921.png)

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



### **【RpgmakerMV】插件编写实战系列二：仿win10菜单列表，新增窗口并显示指定文字/命令**

#### 1、在菜单命令文字旁显示图片

rpgmaker mv中命令列表默认只显示文字，如果想要把我们想要在文字旁显示图片，需要修改命令窗口中绘制文字的那一部分代码。上一节我们目的是在为菜单场景中添加图片元素进去，主要在rpg.secens.js中操作，这次我们要修改窗口的绘制方式，所以来到rpg.windows.js中，找到Window_Command类：

<img src="F:\New server\MV_Online\phoneApp\js\img\1568810179082.png" alt="1568810179082" style="zoom: 67%;" />

它是选择命令窗口的超类，其中可以找到Window_Command.prototype.drawItem函数，其中有一个drawText方法：this.drawText(this.commandName(index), rect.x, rect.y, rect.width, align);，可以看到是这条代码绘制了命令的文本。另外，在Window_Base（所有窗口的超类）中还有一个drawIcon（绘制图标）的方法，

```javascript
Window_Base.prototype.drawIcon = function(iconIndex, x, y) {
    var bitmap = ImageManager.loadSystem('IconSet');
    var pw = Window_Base._iconWidth;
    var ph = Window_Base._iconHeight;
    var sx = iconIndex % 16 * pw;
    var sy = Math.floor(iconIndex / 16) * ph;
    this.contents.blt(bitmap, sx, sy, pw, ph, x, y);
};
```

它接受三个参数，iconIdex是图标的序号，它来自game\img\system\IconSet.png，从0开始数起，第一个图标是0，第二个是1，依此类推，其中，12、13、14、15、28、29、30、31的位置是我们自己添加的图标，x，y是图标显示的坐标信息。

![1568860153923](F:\New server\phoneApp\js\img\1568860153923.png)

所以我们通过修改Window_Command.prototype.drawItem，并利用drawIcon这个方法便可以在文本旁边显示图片，代码如下：

```JavaScript
Window_Command.prototype.drawItem = function(index) {
        var rect = this.itemRectForText(index);
        var align = this.itemTextAlign();
        this.resetTextColor();
        this.changePaintOpacity(this.isCommandEnabled(index));
        var scenesToDraw = [Scene_Menu]//这里只在主菜单也就是Scene_Menu中绘制图标
        var commandIcon = {};
        if(scenesToDraw.indexOf(SceneManager._scene.constructor) >= 0){ 
        var prep = {};
        var commandName = this.commandName(index);
        prep[0] = "物品";
        prep[1] = "12";
        commandIcon[prep[0]] = Number(prep[1]);
        prep[2] = "技能";
        prep[3] = "13";
        commandIcon[prep[2]] = Number(prep[3]);
        /*这里将物品的图标设置为第七个icon，
        commandIcon[commandName]会遍历所有的命令找到对应物品的之后将其图标设置为第七
        个icon，其他的因为还没绘制故不显示*/
        this.drawIcon(commandIcon[commandName], rect.x-4, rect.y+2);
        rect.x += 30;
        rect.width -= 30;}
        this.drawText(this.commandName(index), rect.x, rect.y, rect.width, align);
    };
```

对比这两句代码：

```
this.drawIcon(commandIcon[commandName], rect.x-4, rect.y+2)；
this.drawText(this.commandName(index), rect.x, rect.y, rect.width, align);
```

drawText方法同样在Windows_Base中，text是显示的文字，然后是坐标信息，最大宽度和排列。

```JavaScript
Window_Base.prototype.drawText = function(text, x, y, maxWidth, align) {
    this.contents.drawText(text, x, y, maxWidth, this.lineHeight(), align);
};
```

this.commandName(index)，是根据传入的index来决定显示哪个命令文本，每一个命令都对应一个index，假如“物品”对应index = 0，“技能”对应index = 1，那么this.commandName(0) = “物品”，this.commandName(1) = “技能”。

rect是这么定义的， var rect = this.itemRectForText(index)，这里不需要理解 this.itemRectForText(）改变了什么，只要知道，经过它处理后，rect.x，rext.y便对应了当前传入的index代表的命令文本的位置，梳理整个过程，假设index = 0，“物品”命令的x坐标是（1，1）：

```javascript
drawItem(0) ===> 
rect = this.itemRectForText(0) ，rect.x = 1, rect.y = 1 ===> 
this.commandName(0) = “物品” ===>
this.drawText(物品, 1, 1);//后两项参数可为空
```

理解drawText，有助于理解drawIcon，commandIcon和prep都是数组，我们模仿重复以下上面的过程，当index = 0传入时，drawIcon最终会经历怎样的处理过程：

```javascript
drawItem(0) ===> 
rect = this.itemRectForText(0) ，rect.x = 1, rect.y = 1 ===> 
var commandName = this.commandName(0) = “物品” ===>
prep[0] = "物品";
prep[1] = "12";
commandIcon["物品"] = Number("12")
commandIcon[commandName] = commandIcon["物品"] = Number("12") ===>
（rect.x-4, rect.y+2） =（1-4，1+2）=（-3，3） ===>
this.drawIcon(12, -3, 3)
```

这样我们就把iconSet.png图片中的第12个icon画在了物品命令的旁边，最终效果图如下。

![1568891333186](F:\New server\MV_Online\phoneApp\js\img\1568891333186.png)

#### 2、创建“我的任务”磁贴，用来展示文字信息；创建设置命令磁贴，用来打开设置页面；创建win开始按钮命令，点击可以返回地图界面。

在上文中，我们已经认识了Window_Command类，知道它是所有命令窗口的超类，继续查看rpg.windows.js，其中还有多个窗口类，主菜单共有三个窗口，命令窗口，角色状态窗口和金币数量展示窗口，分别对应Window_menuCommand，Window_MenuStatus和Window_Gold，金币数量展示窗口只显示金币的数量，相对较为简单，我们直接参考这个窗口来新建一个文本展示窗口，代码如下：

```JavaScript
//构造方法
function Window_Mission() {
        this.initialize.apply(this, arguments);
    }
//原型对象
    Window_Mission.prototype = Object.create(Window_Base.prototype);//Object.create方法创建一个新的Window_Mission.prototype对象，并使用现有的Window_Base.prototype来提供新创建的对象的proto
    Window_Mission.prototype.constructor = Window_Mission;//增强对象
//初始化
    Window_Mission.prototype.initialize = function(x, y) {
        var width = this.windowWidth();
        var height = this.windowHeight();
        Window_Base.prototype.initialize.call(this, x, y, width, height);//继承父类Window_Base的proto
        this.refresh();
    };
    Window_Mission.prototype.windowWidth = function() {
        return 180;
    };
    Window_Mission.prototype.windowHeight = function() {
        //return this.fittingHeight(1);
        return 180;
    };
    Window_Mission.prototype.refresh = function() {
        var x = this.textPadding();
        var width = this.contents.width - this.textPadding() * 2;
        this.contents.clear();
        this.drawTextEx("新的任务", 20, 100);//drawTextEx(text, x, y),这个方法也是在rpg.windows.js中定义的，接受三个参数，要显示的文本，以及文本在窗口中显示位置的坐标
    };
    Window_Mission.prototype.open = function() {
        this.refresh();
        Window_Base.prototype.open.call(this);
    };
```

 Window_Mission()类创建好之后，并不会直接显示，还需要将之实例化，也就是将之在菜单场景中显示出来，通过以下代码来完成：

```JavaScript
//实例化对象
Scene_Menu.prototype.createMissionWindow = function() {
        this._missionWindow = new Window_Mission(0, 0);
        this._missionWindow.y = 100;
        this._missionWindow.x = 664;
        this._missionWindowOrg = [this._missionWindow.x,this._missionWindow.y]
        this.addWindow(this._missionWindow);
    };
```

还记得我们之前添加图片到菜单场景时所提到的，新增加的元素要到Scene_Menu.prototype.create中去“登记”一下，将this.createMissionWindow()放入Scene_Menu.prototype.create中，测试游戏，我们便已经添加好了一个新的窗口。

效果如图：

<img src="C:\Users\xipudata\AppData\Roaming\Typora\typora-user-images\1568868246288.png" alt="1568868246288" style="zoom: 67%;" />

创建win开始按钮和新增上面这个窗口原理是一样的，只是之前绘制的是文字，这里我们需要添加一个命令。直接来看这部分代码：

```javascript
 //创建关闭菜单的菜单命令
    function Window_gotoMapCommand() {
        this.initialize.apply(this, arguments);
    }
    
    Window_gotoMapCommand.prototype = Object.create(Window_Command.prototype);
    Window_gotoMapCommand.prototype.constructor = Window_gotoMapCommand;
    
    Window_gotoMapCommand.prototype.initialize = function(x, y) {
        Window_Command.prototype.initialize.call(this, x, y);

    };
    
    Window_gotoMapCommand.prototype.windowWidth = function() {
        return 80;
    };
    Window_gotoMapCommand.prototype.windowHeight = function() {
        return 80;
    };
    
    Window_gotoMapCommand.prototype.numVisibleRows = function() {
        //return this.maxItems();
    };
    
    Window_gotoMapCommand.prototype.makeCommandList = function() {
        this.addOptionsCommand();
    };
    //命令的名字，以及是否开启命令，enabled
    Window_gotoMapCommand.prototype.addOptionsCommand = function() {
            var enabled = this.isOptionsEnabled();
            this.addCommand("", 'gotoMap', enabled);
    };
 
    Window_gotoMapCommand.prototype.isOptionsEnabled = function() {
        return true;
    };
    Window_gotoMapCommand.prototype.drawItem = function(index) {
        var rect = this.itemRectForText(index);
        var align = this.itemTextAlign();
        this.resetTextColor();
        this.changePaintOpacity(this.isCommandEnabled(index));
        this.drawText(this.commandName(index), rect.x, rect.y, rect.width, align);
    };
    //通过这个方法返回地图界面
    Scene_Menu.prototype.commandWin = function() {
        SceneManager.push(SceneManager.goto(Scene_Map)); //回到地图界面
    };

    Scene_Menu.prototype.createGotoMapWindow = function() {
        this._gotoMapWindow = new Window_gotoMapCommand(0, 0);
        this._gotoMapWindow.y = 640;
        this._gotoMapWindow.x = 0;
        this._gotoMapWindow.setHandler('gotoMap',   this.commandWin.bind(this));//将commandWin方法通过’gotoMap‘这个标识和之前的addOptionsCommand添加的命令联系起来
        this.addWindow(this._gotoMapWindow);
    };
//将下面这句代码添加到var _wpMenu_create = Scene_Menu.prototype.create中去
this.createGotoMapWindow();

```



### **【RpgmakerMV】插件编写实战系列三：实现磁贴滑动效果&将选择高亮框扩展覆盖至整块磁贴，添加插件描述、作者、帮助及命令参数**

通过前两节的内容，我们已经可以：1、修改菜单窗口的大小、位置；2、为菜单添加图片背景，为每个窗口添加不同的背景；3、增加新的窗口；4、在菜单命令文字旁绘制icon。

此时我们已经大概模仿出了win10菜单的布局，但是，当我们打开win10菜单的时候，菜单中的磁贴是有一个向上滑动效果的，更有动感也更活泼。同时，每一行的磁贴也并不是以同样的速率向上划动的，中间存在一个延迟。

除了有这样一个滑动效果外，我们无论点击磁贴的哪个位置，都可以进入相关界面，但rmmv默认的选择框是窄窄的长方形，如下图：

![1568973517240](F:\New server\MV_Online\phoneApp\js\img\1568973517240.png)

这里我们需要将这个框的大小覆盖至整块磁贴，也就是增加框的高度，如图：

![1568973613856](F:\New server\MV_Online\phoneApp\js\img\1568973613856.png)

#### 1、磁贴滑动效果的实现

动态效果实现的思路是，在菜单没打开的时候通过resetPosition方法给菜单的各个窗口一个位移，新增的resetPosition方法修改的是默认窗口的位置，需要把该方法放到Scene_Menu.prototype.create中去。这里是给所有窗口的y轴坐标加了100，比如原先A窗口的坐标是（100，100），那么在菜单打开之前，其坐标就被移动到了（100, 200)，同时， this._goldWindow.contentsOpacity = 0这句代码，将窗口里文字的透明度也改为0，使得文字的显示也有一个淡入的效果。

当我们打开菜单时，执行updateSlide，这个方法和updateLayout一样都放在Scene_Menu.prototype.updateSprites中去，各个菜单窗口再移动到默认的位置，并且原先窗口的透明度被改成了0，不再显示，此时只显示我们的磁贴，形成一个动态的效果。

我们以角色状态窗口为例：

```javascript
//将this.updateSlide()添加到之前创建的Scene_Menu.prototype.updateSprites中
        this.updateSlide();	
//将this.resetPosition()添加到之前创建的Scene_Menu.prototype.create中
        this.resetPosition()
    
    Scene_Menu.prototype.resetPosition = function() {
        var slide = 100 //元素滑动的距离 
        this._statusWindow.y = this._statusWindowOrg[1] + slide;
        this._statusWindow.contentsOpacity = 0;
    };
    Scene_Menu.prototype.updateSlide = function() {
        var slideSpeed = 7; //移动的速度
        var opcSpeed = 12;	//窗口文本不透明度渐变速度
        this._statusWindow.contentsOpacity += opcSpeed;
        
        if (this._statusWindow.y > this._statusWindowOrg[1]) {
            this._statusWindow.y -= slideSpeed;
            if (this._statusWindow.y < this._statusWindowOrg[1]) {this._statusWindow.y = this._statusWindowOrg[1]};
        };
    };
```

<img src="F:\New server\MV_Online\phoneApp\js\img\1568976987666.png" alt="1568976987666" style="zoom:50%;" />

<img src="F:\New server\MV_Online\phoneApp\js\img\1568977013699.png" alt="1568977013699" style="zoom:50%;" />

#### 2、将选择高亮框扩展覆盖至整块磁贴

所有的命令窗口，在我们选中这个命令的时候，都会有一个一闪一闪的蓝色高亮框，但这个框一般都是长方形的，我们前面修改了一些窗口的尺寸，这次我们修改蓝色高亮框的尺寸。

修改前：

![1569164098890](F:\New server\MV_Online\phoneApp\js\img\1569164098890.png)

修改后：

![1569164148115](F:\New server\MV_Online\phoneApp\js\img\1569164148115.png)

我们可以在rpg.windows.js中找到Window_Selectable.prototype.updateCursor，修改它即可，但不要直接操作这个函数，我们新建一个窗口——Window_optionCommand，代码如下：

```javascript
//将this.createOptionWindow()放到前面创建的Scene_Menu.prototype.create中
this.createOptionWindow();//新增一个设置命令窗口

//命令窗口
function Window_optionCommand() {
        this.initialize.apply(this, arguments);
    }
    
    Window_optionCommand.prototype = Object.create(Window_Command.prototype);
    Window_optionCommand.prototype.constructor = Window_optionCommand;
    
    Window_optionCommand.prototype.initialize = function(x, y) {
        Window_Command.prototype.initialize.call(this, x, y);

    };
    
    Window_optionCommand.prototype.windowWidth = function() {
        return 100;
    };
    Window_optionCommand.prototype.windowHeight = function() {
        return 100;
    };
    
    Window_optionCommand.prototype.numVisibleRows = function() {
        //return this.maxItems();
    };
    
    Window_optionCommand.prototype.makeCommandList = function() {
        this.addOptionsCommand();
    };
    
    Window_optionCommand.prototype.addOptionsCommand = function() {
            var enabled = this.isOptionsEnabled();
            this.addCommand("", 'options', enabled);
    };
 
    Window_optionCommand.prototype.isOptionsEnabled = function() {
        return true;
    };
    Window_optionCommand.prototype.drawItem = function(index) {
        var rect = this.itemRectForText(index);
        var align = this.itemTextAlign();
        this.resetTextColor();
        this.changePaintOpacity(this.isCommandEnabled(index));
        this.drawText(this.commandName(index), rect.x, rect.y, rect.width, align);
    };
    Scene_Menu.prototype.commandGameEnd = function() {
        SceneManager.push(Scene_Options);
        //SceneManager.push(SceneManager.goto(Scene_Map)); 回到地图界面
        //window.open("http://www.baidu.com"); 打开一个网页
    };
    Scene_Menu.prototype.createOptionWindow = function() {
        this._optionWindow = new Window_optionCommand(0, 0);
        this._optionWindow.y = 430;
        this._optionWindow.x = 654;
        this._optionWindowOrg = [this._optionWindow.x,this._optionWindow.y]
        this._optionWindow.setHandler('options',   this.commandGameEnd.bind(this));
        this.addWindow(this._optionWindow);
    };
```

Window_CommandGameEnd继承了Window_Command的属性和方法，command继承了Window_Selectable的属性和方法，所以这里我们通过下面这段代码来单独修改新建的设置窗口的蓝色高亮框的大小。

```JavaScript
//修改高亮选择的大小
    Window_optionCommand.prototype.updateCursor = function() {
        if (this._cursorAll) {
            var allRowsHeight = this.maxRows() * this.itemHeight();
            this.setCursorRect(0, 0, this.contents.width, allRowsHeight);
            this.setTopRow(0);
        } else if (this.isCursorVisible()) {
            var rect = this.itemRect(this.index());
            rect.x += 0; 
            rect.y += 0; 
            rect.width += 0;
            rect.height += 180;//设置高度为180
            this.setCursorRect(rect.x, rect.y, rect.width, rect.height);
        } else {
            this.setCursorRect(0, 0, 0, 0);
        }
    };
```

注意这里的itemRect方法，它在Window_Selectable.prototype.itemRect中有定义，如果你想要修改整体高亮选择框的尺寸，可以在这个函数里修改。

```javascript
Window_Selectable.prototype.itemRect = function(index) {
    var rect = new Rectangle();
    var maxCols = this.maxCols();
    rect.width = this.itemWidth();
    rect.height = this.itemHeight();//高，如果修改为180，则将之改为rect.height = 180；
    rect.x = index % maxCols * (rect.width + this.spacing()) - this._scrollX;
    rect.y = Math.floor(index / maxCols) * rect.height - this._scrollY;
    return rect;
};
```

#### 3、添加插件描述、作者、帮助及命令参数

rpgmaker mv自带了Community_Basic.JS, itemBook.js等插件，我们模仿这几个插件即可。

上文我们知道了如何将选择高亮框扩展覆盖至整块磁贴，并且将其高度设置为了180，即rect.height += 180，这次我们添加一个参数，使我们可以在插件管理器中修改这个数值。

所有的插件描述、作者、帮助及命令参数都将被放到 /*: */ 里面，并且以 @ 开头，如：

插件作者：@author text

插件描述：@plugindesc text

插件帮助：@help text

命令参数：@param height number

命令参数的描述：@desc text

命令参数的默认值：@default 180

完整代码如下：

```javascript
 /*:
@plugindesc 菜单美化插件编写实例
@author B站：Qcampus  微信公众号：河上一周
@help
仿win10菜单。
@param height
@desc 选择高亮框的高度
@default 180
插件命令：
ToggleloveWall —— 打开或关闭表白墙
 */
var parameters = PluginManager.parameters('win10_menu_part1');
　  var selectHeight = Number(parameters['height'] || 180);
    //加载图片
    ImageManager.loadWPMenus = function(filename) {
        return this.loadBitmap('img/wpmenu/', filename, 0, true);//从img/wpmenu/文件夹中加载指定图片文件，本例中所有的图片都放在这个文件夹
    };
//Window_optionCommand.prototype.updateCursor中的rect.height += 180改为：
rect.height += selectHeight；
```

效果如图，全部完成后效果如图，教程结束

![1569235424235](F:\New server\MV_Online\phoneApp\js\img\1569235424235.png)

<img src="F:\New server\phoneApp\js\img\1569235449898.png" alt="1569235449898" style="zoom:50%;" />

<img src="F:\New server\MV_Online\phoneApp\js\img\1569235911234.png" alt="1569235911234" style="zoom:50%;" />