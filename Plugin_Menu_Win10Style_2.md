### **【RpgmakerMV】插件编写实战系列二：仿win10菜单列表，新增窗口并显示指定文字/命令**

#### 1、在菜单命令文字旁显示图片

rpgmaker mv中命令列表默认只显示文字，如果想要把我们想要在文字旁显示图片，需要修改命令窗口中绘制文字的那一部分代码。上一节我们目的是在为菜单场景中添加图片元素进去，主要在rpg.secens.js中操作，这次我们要修改窗口的绘制方式，所以来到rpg.windows.js中，找到Window_Command类：

![1568810179082.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568810179082.png?raw=true)

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

![1568860153923.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568860153923.png?raw=true)

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

![1568891333186.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568891333186.png?raw=true)

#### 2、创建“我的任务”磁贴，用来展示文字信息；创建设置命令磁贴，用来打开设置页面；创建win开始按钮命令，点击可以返回地图界面。

在上文中，我们已经认识了Window_Command类，知道它是所有命令窗口的超类，继续查看rpg.windows.js，其中还有多个窗口类，主菜单共有三个窗口，命令窗口，角色状态窗口和金币数量展示窗口，分别对应Window_menuCommand，Window_MenuStatus和Window_Gold，金币数量展示窗口只显示金币的数量，相对较为简单，我们直接参考这个窗口来新建一个文本展示窗口，代码如下：

```JavaScript
//构造方法
function Window_Mission() {
        this.initialize.apply(this, arguments);
    }
//原型对象
    Window_Mission.prototype = Object.create(Window_Base.prototype);//Tip
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

> Tip：Object.create方法创建一个新的Window_Mission.prototype对象，并使用现有的Window_Base.prototype来提供新创建的对象的proto

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

![1568868246288.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568868246288.png?raw=true)

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
