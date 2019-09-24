### **【RpgmakerMV】插件编写实战系列三：实现磁贴滑动效果&将选择高亮框扩展覆盖至整块磁贴，添加插件描述、作者、帮助及命令参数**

通过前两节的内容，我们已经可以：1、修改菜单窗口的大小、位置；2、为菜单添加图片背景，为每个窗口添加不同的背景；3、增加新的窗口；4、在菜单命令文字旁绘制icon。

此时我们已经大概模仿出了win10菜单的布局，但是，当我们打开win10菜单的时候，菜单中的磁贴是有一个向上滑动效果的，更有动感也更活泼。同时，每一行的磁贴也并不是以同样的速率向上划动的，中间存在一个延迟。

除了有这样一个滑动效果外，我们无论点击磁贴的哪个位置，都可以进入相关界面，但rmmv默认的选择框是窄窄的长方形，如下图：

![1568973517240.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568973517240.png?raw=true)

这里我们需要将这个框的大小覆盖至整块磁贴，也就是增加框的高度，如图：

![1568973613856.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568973613856.png?raw=true)

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

![1568976987666.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568976987666.png?raw=true)

![1568977013699.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1568977013699.png?raw=true)

#### 2、将选择高亮框扩展覆盖至整块磁贴

所有的命令窗口，在我们选中这个命令的时候，都会有一个一闪一闪的蓝色高亮框，但这个框一般都是长方形的，我们前面修改了一些窗口的尺寸，这次我们修改蓝色高亮框的尺寸。

修改前：

![1569164098890.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1569164098890.png?raw=true)

修改后：

![1569164148115.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1569164148115.png?raw=true)

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

效果如图，全部完成后菜单插件管理器界面如图。

教程涉及的rmmv工程文件已在教程第一部分提供下载，win10_menu_part1是截至教程结束所完成的插件，win10_menu是完整的插件，谢谢观看！

![1569235424235.png](https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1569235424235.png?raw=true)

<img src="https://github.com/QSociety/RMMVTutorial/blob/master/Plugin_Menu_Win10Style/img/1569235449898.png?raw=true" alt="1569235449898.png" style="zoom: 50%;" />

