---

layout: post

title: "读《浏览器的工作原理》"

date: 2017-10-20

categories: [DOM]

---

* content
{:toc}

## 有用的信息
1. 预解析Speculative parsing
>  Both Webkit and Firefox do this optimization. While executing scripts, another thread parses the rest of the document and finds out what other resources need to be loaded from the network and loads them. These way resources can be loaded on parallel connections and the overall speed is better. Note - the speculative parser doesn't modify the DOM tree and leaves that to the main parser, it only parses references to external resources like external scripts, style sheets and images.
 浏览器在执行脚本代码时，会预先解析后面的script, stylesheet, 和image资源，并提前进行资源的请求加载，这样就实现了并行请求资源。
 **注意**: 这里的预解析并不会更改DOM结构，而是只有当前脚本执行完才会对DOM进行更改，也就是会**阻塞渲染**。

 2. CSS的加载
 从概念上讲，CSS不会改变DOM的结构，所以加载CSS的时候没有必要阻塞文档的解析，但是由于script 代码可能会请求节点的样式，如果不提前把样式加载并解析完成，js可能会得到错误的计算样式，所以加载并解析stylesheet时，火狐会阻塞脚本的执行，webkit只在脚本要访问某个样式属性的时候才会被阻塞。

 ## 渲染树的构建
 火狐把渲染树中中的元素称为“frames”，Webkit则称之为``renderer``(渲染器)或``render object``。一个渲染器知道如何放置并绘制自己和自己的子元素。
 Webkit中的 RenderObject 类是渲染对象的基类，其定义如下：
 ```c++
 class RenderObject {
     virtual void layout();             // 布局方法
     virtual void paint(PaintInfo);         // 绘制方法
     virtual rect repaintRect();            // 绘制rect的方法
     Node* node;            // the DOM node
     RenderStyle* style; `  // the computed style
     Renderlayer* containLayer;     // the containing z-index layer
 }
 ```
 每个渲染器会呈现一块矩形区域，这通常与CSS盒模型相对应。这个区域包含几何信息，比如长宽等。
 盒模型的类型与节点的``display``属性有关。
 下面是Webkit根据``display``属性值来决定创建何种类型的渲染器的代码。
 ```c++
 RenderObject* RenderObject::createObject(Node* node, RenderStyle* style)
 {
     Document* doc = node->document();
     RenderArena* arena = doc->renderArena();
     ...
     RenderObject* o = 0;

     switch (style->display()) {
         case NODE:
            break;
        case INLINE:
            o = new (arena) RenderInline(node);
            break;
        case BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case INLINE_BLOCK:
            o = new (arena) RenderBlock(node);
            break;
        case LIST_ITEM:
            o = new (arena) RenderListItem(node);
            break;
        ...
     }
     return o;
 }
 ```

 ### The render tree relation to the DOM tree
 渲染器和DOM 元素相关，但关系不是一对一的关系。没有视觉效果的元素将不会放倒渲染树中。比如``head``标签。样式为``display:none;``的元素也不会插入到渲染树中。

 有一些元素对应了多个视觉对象，这是因为他们往往有着复杂的结构，不能用单一的矩形来描述。比如``"select"``元素，就含有三个部分：展示区域，下拉框以及点击按钮。
 另一个例子是 broken HTML.根据 CSS 规范，一个 inline 元素必须只含有 block 元素 或者只含有 inline 元素。当含有混合的内容时，浏览器会创建一个隐藏的 block 渲染器。

 还有一些渲染器对应在不同位置的同一个DOM节点。浮动定位和绝对定位的元素脱离了文档流，被放置到树中的不同位置，并映射到真实的frame。在他们本来应该在的位置上，会有一个占位的 frame 。
 <div><img src="/notes/assets/blog_images/20171020/render-dom.png"/></div>style-context-tree.png

### The flow of constructing the tree
在Firefox 中，页面的展示会监听 DOM 的更新变动。
Webkit中，处理样式以及创建渲染器的过程称为``attachment``，每个DOM 节点会有一个``attach``方法。 Attachment 是同步的，向DOM 树中插入节点会调用节点的``attach``方法。

对 html 和 body 标签的处理会创建渲染树的根节点。根渲染对象对应于CSS规范中的 ``containing block``-包含所有 block 的最上层 block 。他的大小就是 viewport 的大小，即浏览器窗口的大小。 火狐称之为 ViewPortFrame, Webkit 称之为 RenderView. 这是document 指向的渲染对象。随着 DOM 节点的插入，渲染树的剩余节点也被创建。

## Style Computation
构建渲染树需要计算每个渲染对象的视觉属性,这是通过计算每个元素的样式属性来按成的。

样式包括多种来源的样式表，内联样式,行内样式以及HTML的样式属性，如``bgcolor``属性。样式属性会被转换为响应的CSS属性。

样式表的来源有浏览器的默认样式表，页面开发者提供的样式表以及用户样式表-这是由浏览器用户提供的，在浏览器相应的文件夹中。

样式计算有以下几个困难：
1. Style data 有着非常大的结构，保存着大量的样式属性，这会导致存储问题。

2. 如果不进行优化，寻找每个元素的匹配样式会导致性能问题。对每一个元素都遍历整个规则集合是一个沉重的任务。 CSS选择器可能有着非常复杂的结构，这有可能导致匹配一个看似正确的匹配路径实际上是无效的，这样就不得不尝试另一条路径。
  比如
    ```css
    div div div div { 
        ...
    }
    ```
    上面的规则选择了作为三层 div 的子元素的 div 元素。假如你想检查该规则是否适用于当前的 div， 就需要向上查找，在找到找到第三层时，发现不适合这个规则，就需要尝试另一条规则。

3. > Applying the rules involves quite complex cascade rules that define the hierarchy of the rules.

让我们看看浏览器是如何解决这些问题的。

### Sharing style data
Webkit nodes reference style objects(RenderStyle), 这些对象在某些情况下，可以被节点所共享，这些节点是siblings or cousins 并且满足一下条件：
 1. The elements must be in the same mouse state (e.g., one can't be in :hover while the other isn't)
 2. 元素均不能含有id属性
 3. The tag names should match.(应该是相同的意思)
 4. The class attributes should match. (类名相同)
 5. The set of mapped attributes must be identical.(待研究)
 6. The link states must match; (链接的状态需要一致)
 7. The focus states must match.
 8. 元素均不能被属性选择器所影响。比如``input[checked="true"]``,这些元素的样式会被状态所影响。
 9. 元素均不能有行内样式属性。(HTML5中行内样式属性已经被废弃了)
 10. > There must be no sibling selectors in use at all. WebCore simply throws a global switch when any sibling selector is encountered and disables style sharing for the entire document when they are present. This includes the + selector and selectors like :first-child and :last-child.
    (现在不知道还是不是这样)

### Firefox rule tree
Firefox has two extra trees for easier style computation - the rule tree and the style context tree. Webkit 也有style object 但是他们并不是存储在树中的，还有DOM节点才会指向相关的样式。
 <figure><img src="/notes/assets/blog_images/20171020/style-context-tree.png"/>
 <figcaption></figcaption>Firefox style context tree</figure>
The style context contain end values.这个值是通过按照正确顺序来应用匹配的规则并将其有逻辑值转化为具体值而计算出来的。比如百分数形式的宽度值计算为实际的像素值。

所有的匹配的规则存储在树中。底层的节点拥有更高的优先级。这个树包含找到的规则匹配的所有路径。树对规则的存储不是一次性完成的，而是每次节点样式需要计算时，计算路径被加入到树中。
 <figure><img src="/notes/assets/blog_images/20171020/tree.png"/>
 <figcaption></figcaption>tree</figure>
 假设我们需要为另一个节点匹配规则，并发现匹配的规则是 B - E - I. 在树中我们已经有了这条路径，因为我们已经计算了 A-B-E-I-L 这条路径。

 让我们看看工作时树是怎么保存的。

 ### Division into structs
 The style contexts are divided into structs.这些结构包含某一类别的样式信息，像border和颜色。结构中的所有属性都是继承或非继承的。继承属性如果没有被设置将会继承父元素的属性值，非继承属性会使用元素的默认值。

 这个树通过缓存整个结构(包含计算出的最终值)来帮助我们。如果底层的节点没有提供结构的定义，就会使用上层节点的结构(structs)。

### Computing the style contexts using the rule tree
When computing the style context for a certain element, we first compute a path in the rule tree or use an existing one。 然后我们开始使用路径中的规则来构成我们在新的 style context 中的struct。我们从路径的底层节点开始，这些节点具有最高的优先级。然后遍历整个树直到struct被完全构成。 如果在那个规则节点中没有具体的结构规则，我们就可以做很大的优化--我们向上遍历树，直到我们找到一个指定了具体的结构的节点--这样，整个struct 可以被共享，起到了很好的优化作用。

如果我们仅仅找到了部分定义，我们向上遍历树，直到struct 被填满。

如果我们没有找到struct 的定义，那我们会使用context tree 中，节点父元素的struct 。

如果具体的节点使用了一些值，我们就需要通过 计算将其转化为具体的值。我们将其缓存起来，以便供子元素使用。

如果一个元素有相邻的元素也指向同一个tree node, 那么entire style context 都会被共享使用。

来看一下具体的例子。假设我们有这样的HTML结构：
```html
<html>
	<body>
		<div class="err" id="div1">
			<p>
				this is a <span class="big"> big error </span>
				this is also a
				<span class="big"> very  big  error</span> error
			</p>
		</div>
		<div class="err" id="div2">another error</div>
  </body>
</html>
```
并且有以下规则：
```css
div {margin:5px;color:black}
.err {color:red}
.big {margin-top:3px}
div span {margin-bottom:4px}
#div1 {color:blue}
#div2 {color:green}
```
假定我们需要构建两个structs - the color struct and the margin struct.The color struct contains only one member - the color The margin struct contains the four sides.The resulting rule tree will look like this (the nodes are marked with the node name : the # of rule they point at):
 <figure><img src="/notes/assets/blog_images/20171020/rule-tree.png"/>
 <figcaption></figcaption> The rule tree</figure>
 The context tree will look like this (node name : rule node they point to):
  <figure><img src="/notes/assets/blog_images/20171020/context-tree.png"/>
 <figcaption></figcaption>The context tree</figure>

 假设我们解析HTML， 得到了第二个``<div>``标签，我们需要为这个节点创建一个 style context ，并填充他的struct。
 We will match the rules and discover that the matching rules for the <div> are 1 ,2 and 6. This means there is already an existing path in the tree that our element can use and we just need to add another node to it for rule 6 (node F in the rule tree). 
We will create a style context and put it in the context tree. The new style context will point to node F in the rule tree.

We now need to fill the style structs. We will begin by filling out the margin struct. Since the last rule node(F) doesn't add to the margin struct, we can go up the tree until we find a cached struct computed in a previous node insertion and use it. We will find it on node B, which is the uppermost node that specified margin rules.

We do have a definition for the color struct, so we can't use a cached struct. Since color has one attribute we don't need to go up the tree to fill other attributes. We will compute the end value (convert string to RGB etc) and cache the computed struct on this node.

### Manipulating the rules for an easy match
解析完样式表之后，浏览器会把所有的规则添加到某些hash map 中，比如，选择器是class,那么会把这条规则放到class map中，选择器是id，就把这条规则放到id map 中。寻找元素的匹配规则时，只需要去对应的map中寻找即可，这样就简化了匹配的过程。
> &nbsp;&nbsp;After parsing the style sheet, the rules are added one of several hash maps, according to the selector. There are maps by id, by class name, by tag name and a general map for anything that doesn't fit into those categories. If the selector is an id, the rule will be added to the id map, if it's a class it will be added to the class map etc. 
This manipulation makes it much easier to match rules. There is no need to look in every declaration - we can extract the relevant rules for an element from the maps. This optimization eliminates 95+% of the rules, so that they need not even be considered during the matching process(4.1).<br>
 &nbsp;&nbsp;Both Webkit and Firefox do this manipulation.

### Applying the rules in the correct cascade order
> The style object has properties corresponding to every visual attribute (all css attributes but more generic). If the property is not defined by any of the matched rules - then some properties can be inherited by the parent element style object. Other properties have default values.

> The problem begins when there is more than one definition - here comes the cascade order to solve the issue.

### Style sheet cascade order
A declaration for a style property can appear in several style sheets, and several times inside a style sheet. This means the order of applying the rules is very important. This is called the "cascade" order. According to CSS2 spec, the cascade order is (from low to high):
1. Browser declarations
2. User normal declarations
3. Author normal declarations
4. Author important declarations
5. User important declatations

### Specifity
The selector specifity is defined by the [CSS2 specification](http://www.w3.org/TR/CSS2/cascade.html#specificity) as follows:
1. 如果是元素的``style``属性定义的样式，记做1，否则为0，并将其赋值给 a
2. 计算选择器中，id 的个数，赋值给 b
3. 计算其他属性和伪类的个数，赋值给 c
4. 计算元素名和伪元素的个数，赋值给 d
然后将各个值连接起来,得到： a - b - c - d，然后从左到右权重依次降低。最终得出来的值最大的，优先级最高，会将其定义的样式应用给元素。

### Gradual process
> Webkit uses a flag that marks if all top level style sheets (including @imports) have been loaded. If the style is not fully loaded when attaching - place holders are used and it s marked in the document, and they will be recalculated once the style sheets were loaded.

## Layout
> When the renderer is created and added to the tree, it does not have a position and size. Calculating these values is called layout or reflow.

渲染器被创建并添加到树中时，并没有位置和大小。位置和大小的计算称为layout 或 reflow.

> HTML uses a flow based layout model, meaning that most of the time it is possible to compute the geometry in a single pass. Elements later ``in the flow`` typically do not affect the geometry of elements that are earlier ``in the flow``, so layout can proceed left-to-right, top-to-bottom through the document. There are exceptions - for example, HTML tables may require more than one pass 

> Layout is a recursive process. It begins at the root renderer, which corresponds to the element of the HTML document. Layout continues recursively through some or all of the frame hierarchy, computing geometric information for each renderer that requires it.

> All renderers have a "layout" or "reflow" method, each renderer invokes the layout method of its children that need layout.

### Dirty bit system
> In order not to do a full layout for every small change, browser use a "dirty bit" system. A renderer that is changed or added marks itself and its children as "dirty" - needing layout.

> There are two flags - "dirty" and "children are dirty". Children are dirty means that although the renderer itself may be ok, it has at least one child that needs a layout.

### Global and incremental layout
Layout can be triggered on the entire render tree - this is "global" layout. this can happen as a result of :
1. A global style change that affects all renderers, like a font size change.
2. As a result of a screen being resized

Layout can be incremental, only the dirty renderers will be layed out .

Incremental layout is triggered (asynchronously) when renderers are dirty. For example when new renderers are appended to the render tree after extra content came from the network and was added to the DOM tree.

### Asynchronous and Synchronous layout
**Incremental layout is done asynchronously**.Firefox queues "reflow commands" for incremental layouts and a scheduler triggers batch execution of these commands. Webkit also has a timer that executes an incremental layout - the tree is traversed and "dirty" renderers are layout out.

Scripts asking for style information, like "offsetHeight" can **trigger incremental layout synchronously**.

Global layout will usually be triggered **synchronously**. 

### Optimizations
When a layout is triggered by a "resize" or a change in the renderer position(and not size), the renders sizes are **taken from a cache and not recalculated**.

### The layout process
The layout usually has the following pattern:
 1. Parent renderer determines its own width.
 2. Parent goes over children and:
    1. Place the child renderer(sets its x and y)
    2. Calls child layout if needed(they are dirty or we are in a global layout or some other reason), this calculates the child's height.
3. Parent uses childrend accumulative heights and the heights of the margins and paddings to set it own height - this will be used by the parent renderer's parent.
4. Set its dirty bit to false.

### Width calculation
The renderer's width is calculated using the container block's width, the renderer's style "width" property, the margin and the borders.

## Painting
> In the painting stage, the render tree is traversed and the renderers "paint" method is called to display their content on the screen. Painting uses the UI infrastructure component. More on that in the chapter about the UI.
### Global and Incremental
Like layout, painting can also be global - the entire tree is painted - or incremental. In incremental painting, some of the renderers change in a way that does not affect the entire tree. The changed renderer invalidates it's rectangle on the screen. This causes the OS to see it as a "dirty region" and generate a "paint" event. 

### The painting order
CSS2 defines the order of the painting process - [http://www.w3.org/TR/CSS21/zindex.html](http://www.w3.org/TR/CSS21/zindex.html).This is actually the order in which the elements are stacked in the stacking contexts. This order affects painting since the stacks are painted from back to front. The stacking order of a block renderer is:
1. background color 
2. background image
3. border
4. children
5. outline

### Webkit rectangle storage
Before repainting, webkit saves the old rectangle as a bitmap. It then paints only the delta between the new and old rectangles. 

### Dynamic changes
The browsers try to do the minimal possible actions in response to a change. So changes to an elements color will cause only repaint of the element. Changes to the element position will cause layout and repaint of the element, its children and possibly siblings. Adding a DOM node will cause layout and repaint of the node. Major changes, like increasing font size of the "html" element, will cause invalidation of caches, relyout and repaint of the entire tree.

### The rendering engine's threads
The rendering engine is single threaded. Almost everything, except network operations, happens in a single thread. In Firefox and safari this is the main thread of the browser. In chrome it's the tab process main thread. 

Network operations can be performed by several parallel threads. The number of parallel connections is limited (usually 2 - 6 connections. Firefox 3, for example, uses 6).
chrome之所以快一些，就是因为每个tab页都是一个单独的主线程。这也是他占用内存多的原因。
网络请求是多个线程同时进行的，但是有个数限制。

### Event loop
The browser main thread is an event loop. It's an infinite loop that keeps the process alive. It waits for events (like layout and paint events) and processes them. 

## CSS2 visual model
### Positioning scheme
There are three schemes:
 1. Normal - the object is positioned according to its place in the document -  this means its place in the render tree is like its place in the dom tree and layed out according to its box type and dimensions
 2. Float -  the object is first layed out like normal flow, then moved as far left or right as possible.
  先按照正常的位置放置，然后尽可能的移动到左边。 
 3. Absolute - the object is put in the render tree differently than its place in the DOM tree.