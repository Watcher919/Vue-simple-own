   vue源码业余时间差不多看了一年，以前在网上找帖子，发现很多帖子很零散，都是一部分一部分说，断章的很多，所以自己下定决定一行行看，经过自己坚持与努力，现在基本看完了   。这个vue源码逐行分析，我基本每一行都打上注释，加上整个框架的流程思维导图，基本上是小白也能看懂的vue源码了。

   说的非常的详细，里面的源码注释，有些是参考网上帖子的，有些是自己多年开发vue经验而猜测的，有些是自己跑上下文程序知道的，本人水平可能有限，不一定是很正确，如果有不足的地方可以联系我QQ群 ：302817612  修改，或者发邮件给我281113270@qq.com  谢谢。 



vue 如何去看vue源码呢？其实mvvm源码并没有想象中那么神秘，从12年开始到至今mvvm发展已经有了十几年历史了，从以前直接操作dom的jq发展有十几年历史，但是这十几年历史发展，并没有多大的改变，思想还是那些，模块还是分为几大块：

1. 模板转换，就是我们写的 vue 模板 或者是 react jsx 我们都可以理解是模板，然后他会经过 模板编译转换，像vue的话是进过一个方法paseHTML方法转换成ast树，里面的paseHTML用while 循环模板，然后经过正则 匹配到vue指令，还有vue的属性，事件方法等，收集到一个ast树中。
2. 数据相应，vue是一个双数据相应的框架，底层用的是Object.defineProperty 监听和挟持数据改变，然后调用回调方法更新视图更新。双数据绑定原理是：obersve()方法判断value没有没有__ob___属性并且是不是Obersve实例化的，  value是不是Vonde实例化的，如果不是则调用Obersve 去把数据添加到观察者中，为数据添加__ob__属性， Obersve 则调用defineReactive方法，该方法是连接Dep和wacther方法的一个通道，利用Object.definpropty() 中的get和set方法 监听数据。get方法中是new Dep调用depend()。为dep添加一个wacther类，watcher中有个方法是更新视图的是run调用update去更新vonde 然后更新视图。 然后set方法就是调用dep中的notify 方法调用wacther中的run 更新视图
3. 虚拟dom，vnode，在vue用vnode是通过 ast对象，在转义成vonde 需要渲染的函数，比如_c('div'  s(''))  等这类的函数，编译成vonde 虚拟dom。然后到updata更新数据 调用__patch__ 把vonde 通过diff算法变成正真正的dom元素。

  4.diif算法，vue2 的diff 算法是深度优先算法遍历，然后对比算法是通过 新旧的vnode对比先对比他们的基本属性，比如key 标签等，如果是相同则通过diff算法对比然后diff算法是新旧的vnode对比，然后有四个指针索引，两个新的vnode开始指针和新的 vnode 结束指针，两个旧的vnode开始指针和旧的 vnode 结束指针。然后先判断vnode是否为空，如果为空就往中间靠拢  开始的指针++  结束的指针 --。然后两头对比之后，在交叉对比，直到找不到相同的vnode之后如果多出的就删除，如果少的话就新增，然后对比完之后在更新到真实dom。







源码入口流程 vue源码解读流程 1.new Vue 调用的是 Vue.prototype._init  从该函数开始 经过 $options 参数合并之后 initLifecycle 初始化生命周期标志 初始化事件，初始化渲染函数。初始化状态就是数据。把数据添加到观察者中实现双数据绑定。



```
 Vue.prototype._init = function (options) { //初始化函数
  //... 省略code
  
    initLifecycle(vm); //初始化生命周期 标志
            initEvents(vm); //初始化事件
            initRender(vm); // 初始化渲染
            callHook(vm, 'beforeCreate'); //触发beforeCreate钩子函数
            initInjections(vm); // resolve injections before data/props 在数据/道具之前解决注入问题 //初始化 inject
            initState(vm);  //    //初始化状态
            initProvide(vm); // resolve provide after data/props  解决后提供数据/道具  provide 选项应该是一个对象或返回一个对象的函数。该对象包含可注入其子孙的属性，用于组件之间通信。
            callHook(vm, 'created'); //触发created钩子函数
  
  
  //... 省略code
    // 然后挂载模板，这里大概就是把模板转换成ast的入口
    vm.$mount(vm.$options.el);
  
 }
```



​    vm.$mount 进入这个挂载模板方法，判断是否有 render 函数 或者是template，如果没有则使用el.outerHTML , 实际上这里就是要拿到模板的html内容

```
 Vue.prototype.$mount = function (el, hydrating) { 
   //... 省略code
       el = el && query(el); //获取dom
         if (!options.render) {
              if (template) {
              
              }else if (template.nodeType) { 
                  template = template.innerHTML;
              } else if (el) {
                template = getOuterHTML(el);
              }
         ｝
          
         
              // render 函数 也是 ast 转换 方法
                var ref = compileToFunctions(
                    template, //模板字符串
                    {
                        shouldDecodeNewlines: shouldDecodeNewlines, //flase //IE在属性值中编码换行，而其他浏览器则不会
                        shouldDecodeNewlinesForHref: shouldDecodeNewlinesForHref, //true chrome在a[href]中编码内容
                        delimiters: options.delimiters, //改变纯文本插入分隔符。修改指令的书写风格，比如默认是{{mgs}}  delimiters: ['${', '}']之后变成这样 ${mgs}
                        comments: options.comments //当设为 true 时，将会保留且渲染模板中的 HTML 注释。默认行为是舍弃它们。
                    },
                    this
                );
         
         
       
   
     //... 省略code
      //执行$mount方法     用$mount的方法把扩展挂载到dom上
        return mount.call(
            this,
            el, //真实的dom
            hydrating //undefined
        )
 
 ｝
```





调用  Vue.prototype.$mount 方法之后 拿到模板之后 就会进入以下这几个方法，这几个方法用了很多函数式编程

```
compileToFunctions

createCompiler

createCompilerCreator

baseCompile

parse

parseHTML

```

这里比较重点的是parseHTML 他是  while (html) { //循环html 然后 然后经过正则 匹配到vue指令，还有vue的属性，事件方法等，收集到一个ast树中。

```
  function parseHTML(
        html, //字符串模板
        options //参数
    ) {
        var stack = []; // parseHTML 节点标签堆栈
        var expectHTML = options.expectHTML; //true
        var isUnaryTag$$1 = options.isUnaryTag || no; //函数匹配标签是否是 'area,base,br,col,embed,frame,hr,img,input,isindex,keygen, link,meta,param,source,track,wbr'
        var canBeLeftOpenTag$$1 = options.canBeLeftOpenTag || no; //函数 //判断标签是否是 'colgroup,dd,dt,li,options,p,td,tfoot,th,thead,tr,source'
        var index = 0;
        var last, //
            lastTag; //
        console.log(html)



        while (html) { //循环html
            last = html; //
            // Make sure we're not in a plaintext content element like script/style 确保我们不在像脚本/样式这样的纯文本内容元素中
            if (
                !lastTag || //lastTag 不存在
                !isPlainTextElement(lastTag)  // 如果标签不是script,style,textarea
            ) {

                var textEnd = html.indexOf('<'); //匹配开始标签或者结束标签的位置
                if (textEnd === 0) { //标识是开始标签
                    // Comment:
                    if (comment.test(html)) { //匹配 开始字符串为<!--任何字符串,注释标签  如果匹配上
                        var commentEnd = html.indexOf('-->'); //获取注释标签的结束位置

                        if (commentEnd >= 0) { //如果注释标签结束标签位置大于0，则有注释内容
                            console.log(html.substring(4, commentEnd))
                            if (options.shouldKeepComment) { //shouldKeepComment为真时候。获取注释标签内容

                                //截取注释标签的内容
                                options.comment(html.substring(4, commentEnd));
                            }
                            //截取字符串重新循环  while 跳出循环就是靠该函数，每次匹配到之后就截取掉字符串，知道最后一个标签被截取完没有匹配到则跳出循环
                            advance(commentEnd + 3);
                            continue
                        }
                    }

                    //这里思路是先匹配到注释节点，在匹配到这里的ie浏览器加载样式节点
                    // http://en.wikipedia.org/wiki/Conditional_comment#Downlevel-revealed_conditional_comment
                    if (conditionalComment.test(html)) {  //匹配开始为 <![ 字符串  <![endif]-->   匹配这样动态加ie浏览器的 字符串  <!--[if IE 8]><link href="ie8only.css" rel="stylesheet"><![endif]-->
                        //匹配ie浏览器动态加样式结束符号
                        var conditionalEnd = html.indexOf(']>');

                        if (conditionalEnd >= 0) {
                            //截取字符串重新循环  while 跳出循环就是靠该函数，每次匹配到之后就截取掉字符串，知道最后一个标签被截取完没有匹配到则跳出循环
                            advance(conditionalEnd + 2);
                            continue
                        }
                    }

                    // Doctype:
                    //匹配html的头文件 <!DOCTYPE html>
                    var doctypeMatch = html.match(doctype);
                    if (doctypeMatch) {
                        //截取字符串重新循环  while 跳出循环就是靠该函数，每次匹配到之后就截取掉字符串，知道最后一个标签被截取完没有匹配到则跳出循环
                        advance(doctypeMatch[0].length);
                        continue
                    }

                    // End tag:
                    //匹配开头必需是</ 后面可以忽略是任何字符串  ^<\\/((?:[a-zA-Z_][\\w\\-\\.]*\\:)?[a-zA-Z_][\\w\\-\\.]*)[^>]*>
                    var endTagMatch = html.match(endTag);
                    if (endTagMatch) {

                        var curIndex = index;
                        //标签分隔函数 while 跳出循环就是靠该函数，每次匹配到之后就截取掉字符串，知道最后一个标签被截取完没有匹配到则跳出循环
                        advance(endTagMatch[0].length);
                        console.log(endTagMatch)
                        console.log(curIndex, index)
                        //查找parseHTML的stack栈中与当前tagName标签名称相等的标签，
                        //调用options.end函数，删除当前节点的子节点中的最后一个如果是空格或者空的文本节点则删除，
                        //为stack出栈一个当前标签，为currentParent变量获取到当前节点的父节点
                        parseEndTag(
                            endTagMatch[1],
                            curIndex,
                            index
                        );
                        continue
                    }

                    // Start tag:
                    //解析开始标记 标记开始标签
                    //  获取开始标签的名称，属性集合，开始位置和结束位置，并且返回该对象
                    var startTagMatch = parseStartTag();

                    if (startTagMatch) {
                        //把数组对象属性值循环变成对象，这样可以过滤相同的属性
                        //为parseHTML 节点标签堆栈 插入一个桟数据
                        //调用options.start  为parse函数 stack标签堆栈 添加一个标签
                        handleStartTag(startTagMatch);
                        //匹配tag标签是pre,textarea，并且第二个参数的第一个字符是回车键
                        if (shouldIgnoreFirstNewline(lastTag, html)) {
                            //去除回车键空格
                            advance(1);
                        }
                        continue
                    }
                }

                var text = (void 0),
                    rest = (void 0),
                    next = (void 0);
                if (textEnd >= 0) {

                    rest = html.slice(textEnd); //截取字符串  var textEnd = html.indexOf('<'); //匹配开始标签或者结束标签的位置
                    console.log(rest)

                    while (
                        !endTag.test(rest) && //匹配开头必需是</ 后面可以忽略是任何字符串
                        !startTagOpen.test(rest) && // 匹配开头必需是< 后面可以忽略是任何字符串
                        !comment.test(rest) && // 匹配 开始字符串为<!--任何字符串
                        !conditionalComment.test(rest) //匹配开始为 <![ 字符串
                    ) {
                        console.log(rest);


                        // < in plain text, be forgiving and treat it as text
                        // <在纯文本中，要宽容，把它当作文本来对待
                        next = rest.indexOf('<', 1); //匹配是否有多个<
                        if (next < 0) {
                            break
                        }
                        textEnd += next; //截取 索引位置
                        rest = html.slice(textEnd); //获取 < 字符串 <    获取他们两符号< 之间的字符串
                    }
                    text = html.substring(0, textEnd); //截取字符串 前面字符串到 <

                    //while 跳出循环就是靠该函数，每次匹配到之后就截取掉字符串，知道最后一个标签被截取完没有匹配到则跳出循环
                    advance(textEnd);
                }

                if (textEnd < 0) { //都没有匹配到 < 符号 则表示纯文本
                    text = html; //出来text
                    html = ''; //把html至空 跳槽 while循环
                }

                if (options.chars && text) {
                    options.chars(text);
                }
            } else {
                //  处理是script,style,textarea
                var endTagLength = 0;
                var stackedTag = lastTag.toLowerCase();
                var reStackedTag = reCache[stackedTag] || (reCache[stackedTag] = new RegExp('([\\s\\S]*?)(</' + stackedTag + '[^>]*>)', 'i'));
                var rest$1 = html.replace(reStackedTag, function (all, text, endTag) {
                    endTagLength = endTag.length;
                    if (!isPlainTextElement(stackedTag) && stackedTag !== 'noscript') {
                        text = text
                            .replace(/<!\--([\s\S]*?)-->/g, '$1') // #7298
                            .replace(/<!\[CDATA\[([\s\S]*?)]]>/g, '$1');
                    }
                    //匹配tag标签是pre,textarea，并且第二个参数的第一个字符是回车键
                    if (shouldIgnoreFirstNewline(stackedTag, text)) {
                        text = text.slice(1);
                    }
                    if (options.chars) {
                        options.chars(text);
                    }
                    return ''
                });
                index += html.length - rest$1.length;
                html = rest$1;
                parseEndTag(stackedTag, index - endTagLength, index);
            }

            if (html === last) {
                options.chars && options.chars(html);
                if ("development" !== 'production' && !stack.length && options.warn) {
                    options.warn(("Mal-formatted tag at end of template: \"" + html + "\""));
                }
                break
            }
        }






        // Clean up any remaining tags
        //查找parseHTML的stack栈中与当前tagName标签名称相等的标签，
        //调用options.end函数，删除当前节点的子节点中的最后一个如果是空格或者空的文本节点则删除，
        //为stack出栈一个当前标签，为currentParent变量获取到当前节点的父节点
        parseEndTag();
        //while 跳出循环就是靠该函数，每次匹配到之后就截取掉字符串，知道最后一个标签被截取完没有匹配到则跳出循环
        function advance(n) {
            index += n; //让索引叠加
            html = html.substring(n); //截取当前索引 和 后面的字符串。
        }

        //获取开始标签的名称，收集属性集合，开始位置和结束位置，并且返回该对象
        function parseStartTag() {
            var start = html.match(startTagOpen); //匹配开始标签 匹配开头必需是< 后面可以忽略是任何字符串  ^<((?:[a-zA-Z_][\\w\\-\\.]*\\:)?[a-zA-Z_][\\w\\-\\.]*)
            console.log(start)
            console.log(start[0].length)

            if (start) {
                var match = {
                    tagName: start[1], //标签名称
                    attrs: [], //标签属性集合
                    start: index //标签的开始索引
                };
                //标记开始标签的位置，截取了开始标签
                advance(start[0].length);
                var end, attr;

                while (
                    !(end = html.match(startTagClose)) //没有到 关闭标签 > 标签
                    && (attr = html.match(attribute)) //收集属性
                ) {
                    console.log(html)
                    //截取属性标签
                    advance(attr[0].length);
                    match.attrs.push(attr); //把属性收集到一个集合
                }
                if (end) {
                    match.unarySlash = end[1]; //如果是/>标签 则unarySlash 是/。 如果是>标签 则unarySlash 是空
                    console.log(end)

                    //截取掉开始标签，并且更新索引
                    advance(end[0].length);
                    match.end = index; //开始标签的结束位置
                    return match
                }
            }
        }

        //把数组对象属性值循环变成对象，这样可以过滤相同的属性
        //为parseHTML 节点标签堆栈 插入一个桟数据
        //调用options.start  为parse函数 stack标签堆栈 添加一个标签
        function handleStartTag(match) {
            /*
            * match = {
                     tagName: start[1], //标签名称
                     attrs: [], //标签属性集合
                     start: index， //开始标签的开始索引
                     match:index ，   //开始标签的 结束位置
                    unarySlash:'' //如果是/>标签 则unarySlash 是/。 如果是>标签 则unarySlash 是空
             };
            * */

            var tagName = match.tagName; //开始标签名称
            var unarySlash = match.unarySlash; //如果是/>标签 则unarySlash 是/。 如果是>标签 则unarySlash 是空
            console.log(expectHTML)
            console.log('lastTag==')
            console.log(lastTag)
            console.log(tagName)

            if (expectHTML) {   //true

                if (
                    lastTag === 'p' //上一个标签是p
                    /*
                      判断标签是否是
                     'address,article,aside,base,blockquote,body,caption,col,colgroup,dd,' +
                     'details,dialog,div,dl,dt,fieldset,figcaption,figure,footer,form,' +
                     'h1,h2,h3,h4,h5,h6,head,header,hgroup,hr,html,legend,li,menuitem,meta,' +
                     'optgroup,option,param,rp,rt,source,style,summary,tbody,td,tfoot,th,thead,' +
                     'title,tr,track'
                     */
                    && isNonPhrasingTag(tagName)
                ) {
                    //查找parseHTML的stack栈中与当前tagName标签名称相等的标签，
                    //调用options.end函数，删除当前节点的子节点中的最后一个如果是空格或者空的文本节点则删除，
                    //为stack出栈一个当前标签，为currentParent变量获取到当前节点的父节点
                    parseEndTag(lastTag);
                }
                if (
                    canBeLeftOpenTag$$1(tagName) &&   //判断标签是否是 'colgroup,dd,dt,li,options,p,td,tfoot,th,thead,tr,source'
                    lastTag === tagName //上一个标签和现在标签相同  <li><li> 编译成 <li></li>  但是这种情况是不会出现的 因为浏览器解析的时候会自动补全如果是<li>我是li标签<li> 浏览器自动解析成  <li>我是li标签</li><li> </li>
                ) {
                    //查找parseHTML的stack栈中与当前tagName标签名称相等的标签，
                    //调用options.end函数，删除当前节点的子节点中的最后一个如果是空格或者空的文本节点则删除，
                    //为stack出栈一个当前标签，为currentParent变量获取到当前节点的父节点
                    parseEndTag(tagName);
                }
            }

            var unary = isUnaryTag$$1(tagName) || //函数匹配标签是否是 'area,base,br,col,embed,frame,hr,img,input,isindex,keygen, link,meta,param,source,track,wbr'
                !!unarySlash; //如果是/> 则为真

            var l = match.attrs.length;
            var attrs = new Array(l); //数组属性对象转换正真正的数组对象
            for (var i = 0; i < l; i++) {
                var args = match.attrs[i]; //获取属性对象
                // hackish work around FF bug https://bugzilla.mozilla.org/show_bug.cgi?id=369778
                //对FF bug进行黑客攻击:https://bugzilla.mozilla.org/show_bug.cgi?id=369778
                if (
                    IS_REGEX_CAPTURING_BROKEN &&  //这个应该是 火狐浏览器私有 标志
                    args[0].indexOf('""') === -1
                ) {
                    if (args[3] === '') {
                        delete args[3];
                    }
                    if (args[4] === '') {
                        delete args[4];
                    }
                    if (args[5] === '') {
                        delete args[5];
                    }
                }
                var value = args[3] || args[4] || args[5] || '';
                var shouldDecodeNewlines = tagName === 'a' && args[1] === 'href'
                    ? options.shouldDecodeNewlinesForHref  // true chrome在a[href]中编码内容
                    : options.shouldDecodeNewlines;  //flase //IE在属性值中编码换行，而其他浏览器则不会

                attrs[i] = {  //把数组对象属性值循环变成对象，这样可以过滤相同的属性
                    name: args[1], //属性名称
                    //属性值
                    value: decodeAttr(value, shouldDecodeNewlines) //替换html 中的特殊符号，转义成js解析的字符串,替换 把   &lt;替换 <  ， &gt; 替换 > ， &quot;替换  "， &amp;替换 & ， &#10;替换\n  ，&#9;替换\t

                };

            }

            console.log('==!unary==')
            console.log(!unary)

            if (!unary) { //如果不是单标签

                // 为parseHTML 节点标签堆栈 插入一个桟数据
                stack.push({ //标签堆栈
                    tag: tagName, //开始标签名称
                    lowerCasedTag: tagName.toLowerCase(), //变成小写记录标签
                    attrs: attrs //获取属性
                });
                //设置结束标签
                lastTag = tagName;
                console.log('== parseHTML handleStartTag stack==')
                console.log(stack)

            }


            //
            if (options.start) {

                //标签开始函数， 创建一个ast标签dom，  判断获取v-for属性是否存在如果有则转义 v-for指令 把for，alias，iterator1，iterator2属性添加到虚拟dom中
                //获取v-if属性，为el虚拟dom添加 v-if，v-eles，v-else-if 属性
                //获取v-once 指令属性，如果有有该属性 为虚拟dom标签 标记事件 只触发一次则销毁
                //校验属性的值，为el添加muted， events，nativeEvents，directives，  key， ref，slotName或者slotScope或者slot，component或者inlineTemplate 标志 属性
                // 标志当前的currentParent当前的 element
                //为parse函数 stack标签堆栈 添加一个标签
                options.start(
                    tagName,  //标签名称
                    attrs,  //标签属性
                    unary,  // 如果不是单标签则为真
                    match.start,  //开始标签的开始位置
                    match.end //开始标签的结束的位置
                );
            }


        }



        //查找parseHTML的stack栈中与当前tagName标签名称相等的标签，
        //调用options.end函数，删除当前节点的子节点中的最后一个如果是空格或者空的文本节点则删除，
        //为stack出栈一个当前标签，为currentParent变量获取到当前节点的父节点
        function parseEndTag(
            tagName,   //标签名称
            start,  //结束标签开始位置
            end    //结束标签结束位置
        ) {
            var pos,
                lowerCasedTagName;
            if (start == null) { //如果没有传开始位置
                start = index;    //就那当前索引
            }
            if (end == null) {  //如果没有传结束位置
                end = index;    //就那当前索引
            }

            if (tagName) { //结束标签名称
                lowerCasedTagName = tagName.toLowerCase(); //将字符串转化成小写
            }

            // Find the closest opened tag of the same type 查找最近打开的相同类型的标记
            if (tagName) {
                // 获取stack堆栈最近的匹配标签
                for (pos = stack.length - 1; pos >= 0; pos--) {
                    //找到最近的标签相等
                    if (stack[pos].lowerCasedTag === lowerCasedTagName) {
                        break
                    }
                }
            } else {
                // If no tag name is provided, clean shop
                //如果没有提供标签名称，请清理商店
                pos = 0;
            }


            if (pos >= 0) { //这里就获取到了stack堆栈的pos索引
                // Close all the open elements, up the stack 关闭所有打开的元素，向上堆栈
                console.log(pos)

                for (var i = stack.length - 1; i >= pos; i--) {

                    if ("development" !== 'production' && //如果stack中找不到tagName 标签的时候就输出警告日志，找不到标签
                        (i > pos || !tagName) &&
                        options.warn
                    ) {
                        options.warn(
                            ("tag <" + (stack[i].tag) + "> has no matching end tag.")
                        );
                    }
                    if (options.end) {
                        console.log(options.end)
                        //调用options.end函数，删除当前节点的子节点中的最后一个如果是空格或者空的文本节点则删除，
                        //为stack出栈一个当前标签，为currentParent变量获取到当前节点的父节点
                        options.end(
                            stack[i].tag,//结束标签名称
                            start, //结束标签开始位置
                            end //结束标签结束位置
                        );
                    }
                }
                // Remove the open elements from the stack
                //从堆栈中删除打开的元素
                // console.log(stack[pos].tag)
                // 为parseHTML 节点标签堆栈 出桟当前匹配到的标签
                stack.length = pos;
                //获取到上一个标签，就是当前节点的父节点
                lastTag = pos && stack[pos - 1].tag;
                console.log(stack)
                console.log(lastTag)




            } else if (lowerCasedTagName === 'br') {
                if (options.start) {
                    //标签开始函数， 创建一个ast标签dom，  判断获取v-for属性是否存在如果有则转义 v-for指令 把for，alias，iterator1，iterator2属性添加到虚拟dom中
                    //获取v-if属性，为el虚拟dom添加 v-if，v-eles，v-else-if 属性
                    //获取v-once 指令属性，如果有有该属性 为虚拟dom标签 标记事件 只触发一次则销毁
                    //校验属性的值，为el添加muted， events，nativeEvents，directives，  key， ref，slotName或者slotScope或者slot，component或者inlineTemplate 标志 属性
                    // 标志当前的currentParent当前的 element
                    //为parse函数 stack标签堆栈 添加一个标签
                    options.start(
                        tagName,
                        [], true,
                        start,
                        end
                    );
                }
            } else if (lowerCasedTagName === 'p') {
                if (options.start) {
                    //标签开始函数， 创建一个ast标签dom，  判断获取v-for属性是否存在如果有则转义 v-for指令 把for，alias，iterator1，iterator2属性添加到虚拟dom中
                    //获取v-if属性，为el虚拟dom添加 v-if，v-eles，v-else-if 属性
                    //获取v-once 指令属性，如果有有该属性 为虚拟dom标签 标记事件 只触发一次则销毁
                    //校验属性的值，为el添加muted， events，nativeEvents，directives，  key， ref，slotName或者slotScope或者slot，component或者inlineTemplate 标志 属性
                    // 标志当前的currentParent当前的 element
                    //为parse函数 stack标签堆栈 添加一个标签
                    options.start(
                        tagName,
                        [], false,
                        start,
                        end);
                }
                if (options.end) {
                    //删除当前节点的子节点中的最后一个如果是空格或者空的文本节点则删除，
                    //为stack出栈一个当前标签，为currentParent变量获取到当前节点的父节点
                    options.end(
                        tagName,
                        start,
                        end
                    );
                }
            }
            console.log(lastTag)

        }
    }
```



一些匹配模板正则

```
  var onRE = /^@|^v-on:/;//判断是否是 @或者v-on:属性开头的
    var dirRE = /^v-|^@|^:/; //判断是否是 v-或者@或者:  属性开头的
    var forAliasRE = /([^]*?)\s+(?:in|of)\s+([^]*)/; //匹配 含有   字符串 in  字符串   或者  字符串 of  字符串
    var forIteratorRE = /,([^,\}\]]*)(?:,([^,\}\]]*))?$/; //匹配上,  但是属于两边是 [{ , 点 , }]  所以匹配上   ,+字符串
    var stripParensRE = /^\(|\)$/g; //匹配括号 ()

    var argRE = /:(.*)$/; //匹配字符串是否含有:
    var bindRE = /^:|^v-bind:/; //开始匹配是 :或者是v-bind
    var modifierRE = /\.[^.]+/g; // 匹配以点开头的分组 不属于点 data.object.info.age  匹配到 ['.object'，'.info' , '.age']

    var decodeHTMLCached = cached(he.decode);    //获取 真是dom的textContent文本
```





 



  



具体看我源码和流程图，这里文字就不描述这么多了，流程图是下面这中的网盘，源码是vue.js,基本每一行都有注释

链接：https://pan.baidu.com/s/10IxV6mQ2TIwkRACKu2T0ng 
提取码：1fnu 


上面的vue.js 就是我基于vue源码中每行加有注释的vue.js, 其他文件就是我看vue.js源码的时候抽出来的vue.js 源码小demo

 


 作者：姚观寿
