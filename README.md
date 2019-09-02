# 前端硬核知识
这里列出了一些前端的知识，可以从底层去看看前端的那些东西
## 从系统角度去看页面渲染

### 1.页面渲染流程（HTML/CSS/JS）
HTML解析过程的输入由一串代码点(一个Unicode代码点并表示为一个四到六个十六进制数)组成，这些代码点通过标记化阶段，然后是树构造阶段，最终输出一个Document对象。

![parsing-model-overview](https://raw.githubusercontent.com/luobinbinchina/FE-Technology/master/image/parsing-model-overview.jpg)

#### 浏览器渲染一个页面是按照
1. 自上而下
2. 遇CSS/JS阻塞
3. 边解析边构建

#### DOMTree解析
样式（link、style）与脚本文件（script）加载并解析完成后，DOMTree基本完成并触发DOMContentLoaded事件，此时可通过JS操作DOM节点。

#### 构建RenderTree(呈现树)
DOMTree解析后开始构建RenderTree(呈现树)，目的是将CSS样式注入到DOM节点上（webkit内核将该过程称作附着，其他浏览器概念不尽相同），RenderTree的每一个节点都有为之相对应的DOM节点的CSS框，框的的类型取决于DOM节点的display（block/inline等），但DOM节点不太一定有与之相对应的RenderTree节点且位置也不一定相同。

#### 计算布局
DOMTree、RenderTree构造完后，开始进行布局并计算RenderTree节点的大小和位置，简单来说布局的过程是浏览器根据窗口（包括显示器）实际大小来计算RenderTree节点实际的显示大小和位置。

#### 页面绘制
布局完成后就开始进行RenderTree节点绘制，最终完成页面的渲染流程
