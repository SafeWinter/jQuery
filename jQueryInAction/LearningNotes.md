//2019-03-29
1. create a custom jQuery filter
demo1: 
```js
$.expr[':'].requiredLevel = $.expr.createPseudo( function( filterParam ) {
	return function (element, context, isXml) {
		return element.getAttribute('data-level') > 2 && element.getAttribute() > 100;
	};
});
var $requiredLevels = $('.levels li:requiredLevel');
```

demo2:
```js
$.expr[':'].pointsHigherThan = $.expr.createPseudo ( function ( filterParam ) {
	var points = parseInt(filterParam, 10);
	return function ( element, context, isXml ) {
		return element.getAttribute('data-points') > points;
	};
});
var $higherPointsLevels = $('.levels li:pointsHigherThan(50)');
```

demo3:
```js
$.expr[':'].onlyText = $.expr.createPseudo(function(param){
	return function(element, context, isXml){
		return element.innerHTML.match(/^\w+$/);
	};
});
var $myResult = $(':onlyText');
```

Note:
1) function createPseudo() is a jQuery utility, belonging to the underlying Sizzle selectors engine

2) 最内层的匿名函数的返回结果是一个 boolean 值，表征 element 是否被保留：
```js
function(element, context, isXml){
	return element.innerHTML.match(/^\w+$/);
};
```

3) 引用了 jQuery 对象的标识符，最好加前缀【$】，以避免重复套用 $();

4) both will bo - li:nth-child(3n-2)  ===  li:nth-child(3n+1);

//2019-03-30
第三章勘误：
英文版第54页：
```js
$('<img>',
  {
    src: 'images/little.bear.png',
    alt: 'Little Bear',
    title:'I woof in your general direction',
    click: function() {
      alert($(this).attr('title'));
    }
  })
  .appendTo('body');
```
实测时 src 的路径有误，应改为'../images/...':
```js
$('<img>',
  {
    src: '../images/little.bear.png',
    alt: 'Little Bear',
    title:'I woof in your general direction',
    click: function() {
      alert($(this).attr('title'));
    }
  })
  .appendTo('body');
```

### 关于运行第三章的示例代码：
由于使用了 iframe 标签，直接用浏览器打开本地 html 文件将报错：
```js
lab.operations.html:185 Uncaught DOMException: Blocked a frame with origin "null" from accessing a cross-origin frame.
    at HTMLFormElement.<anonymous> (file:///D:/apache-tomcat-8.0.36/webapps/jqia3/chapter-3/lab.operations.html:185:29)
    at HTMLFormElement.dispatch (file:///D:/apache-tomcat-8.0.36/webapps/jqia3/js/jquery-1.11.3.min.js:4:8549)
    at HTMLFormElement.r.handle (file:///D:/apache-tomcat-8.0.36/webapps/jqia3/js/jquery-1.11.3.min.js:4:5252)
```
错误语句位于 `chapter-3/dom.sample.page.html L185`：
```js
var $set = frame.perform(operation);
```
来自 dom.sample.page.html L121:
```js
         function perform(operation) {
            $('*').removeClass('found-element');
            eval('var $set=' + operation + ';');
            $set.addClass('found-element');
            return $set;
         }
```
### 解决办法：
1. 运行tomcat服务器（实测 tomcat V8.0.36）；
2. 将随书源代码文件夹重命名为 jqia3 后放入 webapps 文件夹；
3. 从 bin 文件夹启动服务器，输入如下 url 即可: 
http://localhost:8080/jqia3/chapter-3/lab.operations.html


### 关于 alert($('img').length) 无效的问题：
```js
Ignored call to 'alert()'. The document is sandboxed, and the 'allow-modals' keyword is not set.
Uncaught TypeError: Cannot read property 'addClass' of undefined
    at perform (dom.sample.page.html:124)
    at HTMLFormElement.<anonymous> (lab.operations.html:185)
    at HTMLFormElement.dispatch (jquery-1.11.3.min.js:4)
    at HTMLFormElement.r.handle (jquery-1.11.3.min.js:4)
```
原因：
由于使用了 iframe 的 sandbox 属性，默认禁用了 alert 警告框，需添加 allow-modals 显式声明：（lab.operations.html L69-70）
```xml
<iframe id="frame" name="frame" src="dom.sample.page.html" seamless="seamless"
        sandbox="allow-same-origin allow-scripts allow-modals"></iframe>
```
添加关键词 allow-modals 后正常。

//2019-04-06
用原生JS格式化日期（hh:mm:ss.SSS）
```js
function formateDate(date) {
  return ( 
    ( (date.getHours()   < 10 ? '0' : '') + date.getHours()   ) + ':' +
    ( (date.getMinutes() < 10 ? '0' : '') + date.getMinutes() ) + ':' +
    ( (date.getSeconds() < 10 ? '0' : '') + date.getSeconds() ) + '.' +
    ( (date.getMilliseconds() < 10 ? '00' : (date.getMilliseconds() < 100 ? '0' : '')) + date.getMilliseconds() ) 
    );
}
```

可用于$().on()方法的事件类型：（Events）
`blur`  `change`  `click`  `dblclick`  `error`  `focus`  `focusin`  `focusout`  
`keydown`  `keypress`  `keyup`  `load`  `mousedown`  `mouseenter`  `mouseleave`  
`mousemove`  `mouseout`  `mouseover`  `mouseup`  `ready`  `resize`  `scroll`  
`select`  `submit`  `unload`

//第六章 jQuery 事件要点梳理
1. 准备工作：（附录学习）
1.1. JavaScript对象基础
  JavaScript 的 Object 对象，是其属性（property）的集合。每个属性包含一个名称（name）和值（value）

  名称是string类型的，而值可以是任意类型：Number, String, Boolean, Object 等等

  这意味着，Object 实例的主要目的，是为这些带名称的数据类型提供一个容器。（This means the primary purpose of an Object instance is to serve as a container for a
named collection of other types）
  
  JavaScript 对象是无序的属性集合。
  属性由名字和值构成。
  对象可以用 new 或 对象字面量 进行声明。
  数组也可以用 new 或 数组字面量 进行声明。
  顶级变量是 window 对象的属性（Top-level variables are properties of window）

1.2. 作为一等公民的“函数”
  和 JavaScript 其他数据类型一样，函数也被视为对象，由 JavaScript 的构造函数创建，因此可以：
  1）赋给一个变量；
  2）赋给一个对象的属性；
  3）作为参数进行传递；
  4）作为某个函数的返回值；
  5）使用字面量进行创建。

  正因为 函数 function 与 Number, String, Boolean, Date ... 一样，它也被称为 一等对象（first-class object）

  ES6 中可以通过 myFunc.name 获取函数名称（String型）

  如果一个函数没有显式返回一个值，则默认返回一个 undefined

  以下语句是等效的，最外层、全局位置的函数，也是 window 的一个属性：
```js
  function hello() { 
    alert('Hi there!'); 
  }
  hello = function hello() { 
    alert('Hi there!'); 
  }
  window.hello = function hello() { 
    alert('Hi there!'); 
  }
```

  标准模式定义的函数是可以自动提升作用域的，而赋给一个变量的函数则不行。
```js
  funcDecl();     //The message is alerted correctly
  funcExpr();     //raises an error

  function funcDecl() {
    alert('function declaration');
  }
  var funcExpr = function() {
    alert('function expression');
  };
```
  用作回调的回调函数：
    异步编程最常见的一个概念————回调函数
  
  回调函数名字的由来（callback function）
  以计时器方法 setTimeout 为例：
```js
    function hello() { alert('Hi there!'); }
    setTimeout(hello, 5000);
```
  及时满 5 秒后，该方法将回到自身（而非外部），向自身内部的一个函数发起调用命令，这个函数就被称为回调函数。
  Because the `setTimeout()` method makes a call back to a function in your own code, that function is termed a callback function.

  this 究竟是指什么？
  在以“类”为基础的面向对象语言中，this 通常指代的是声明或定义这个方法的某个类的实例。而在 JavaScript 中，function 是一等对象，不会被其他对象所定义，这里的 this 所指代的对象，术语叫 函数上下文（function context），是具体引用了（而不是定义了）这个函数的某个对象。
  In class-based OO languages, this generally references the instance of the class for which the method has been declared.
  In JavaScript, where functions are first-class objects that aren’t declared as part of anything, the object referenced by this — termed the function context — is determined not by how the function is declared but by how it’s invoked.

  这意味着同一个函数会由于调用者的不同而具有不同的上下文。
  例如：
```js
  var ride = {
    make: 'Yamaha',
    model: 'XT660R',
    year: 2014,
    purchased: new Date(2015, 7, 21),
    owner: {
    name: 'Spike Spiegel',
    occupation: 'bounty hunter'
    },
    whatAmI: function() {
      return this.year + ' ' + this.make + ' ' + this.model;
    }
  };
```
  当中的 this 指的是属性 whatAmI 对应的 ride 对象，因为 ride 的 whatAmI 属性引用了这个函数。对 ride 而言，函数不归 ride 独有，只是通过一个属性进行了引用。（详见 appendix.fig2.function.context.png）


  闭包（closure）
  指的是 function 实例，与某个局部变量耦合在一起。该变量确保了 function 实例的正常执行。
  回调函数在声明的时候引用局部变量的能力，是编写高效 JavaScript 代码的有力工具。有了闭包，声明时引用的局部变量在后期调用时仍然在函数作用域内（其实质是单独创建了一个专属于闭包的作用域链）

```js
  IIFE（Immediately-Invoked Function Expression立即调用的函数表达式）
  (function() {
    // The code of the function goes here...
  })();
```
  IIFI可用于创建私有变量，不破坏外层作用域链。也可用于某事件的回调函数声明，此时可能用到闭包引用某个局部变量。


6.1. 浏览器的事件模型

6.1.1. 第0级DOM模型：
```js
  document.getElementById('example').onmouseover = function(event) {
    console.log('At ' + formatDate(new Date()) + ' Crackle!');
  };
```
缺点：
1）一个事件只能绑定一个事件函数
2）需要浏览器的特性检验：
```js
  var target = event.target || event.srcElement;
```

事件冒泡阶段与捕获阶段
```js
var elements = document.getElementsByTagName('*');
for(var i = 0; i < elements.length; i++) {
  (function(current) {
      current.onclick = function(event) {
        event = event || window.event;
        var target = event.target || event.srcElement;
        console.log(
          'At ' + formatDate(new Date()) +
          ' For ' + current.tagName + '#'+ current.id +
          ' target is ' + target.tagName + '#' + target.id
      );
    };
  })(elements[i]);
}
```
禁用冒泡：
```js
event.stopPropagation();
```
（老版IE 用 `event.cancelBubble = true`）

禁用默认动作：(href跳转、submit提交等)
`event.preventDefault(); 或 return false;`


6.1.2. 第2级DOM模型：
```js
var element = document.getElementById('example');
element.addEventListener('click', function(event) {
    console.log('At ' + formatDate(new Date()) + ' BOOM once!');
  }, false);
element.addEventListener('click', function(event) {
    console.log('At ' + formatDate(new Date()) + ' BOOM twice!');
  }, false);
element.addEventListener('click', function(event) {
    console.log('At ' + formatDate(new Date()) + ' BOOM thrice!');
  }, false);
```
优点：
1）同一事件可添加多个事件函数，依次执行；
2）新增 capture 阶段；
不足：
同样面临浏览器的版本兼容性问题。

冒泡与捕获阶段的示意图，详见 chapter6.fig6.4.bubble.capture.png

IE8及更早版本的特有事件模型
attachEvent(eventName, handler)
eventName: onclick, onmouseover, onkeydown ...
handler: function(event){ ... }
event: window.event

jQuery 事件模型
(1)添加事件：
  $().on(eventType[, selector][, data], handler);
  $().on(eventsHash[, selector][, data]);
  
  其中 eventHash 是一个 JavaScript 的对象，类似 HashMap<String, Object>
  
  值得注意的是，selector 并非过滤器的意思，这里是用作事件代理的对象

  同时，handler 指代一个具体的 function，它接收一个 event 事件对象

  jQuery 中的 return false; 等同于同时执行了 event.preventDefault(); 与 event.stopPropagation(); 而 JavaScript 中的 return false; 仅仅等效于 event.preventDefault();

  方法中的 data 项可以通过 event.data 进行访问：
```js
  $('#my-button').on('click', {
      name: 'John Resig'
    }, function (event) {
      console.log('The name is: ' + event.data.name);
    }
  );
```
  多个事件名可以用空格隔开：
```js
  $('button')
    .on('click', function(event) {
      console.log('Button clicked!');
    })
    .on('mouseenter mouseleave', myFunctionHandler);

  .on( eventHash ) 示例：
  $('button').on({
    click: function(event) {
      console.log('Button clicked!');
    },
    mouseenter: myFunctionHandler,
    mouseleave: myFunctionHandler
  });
```

(2)事件代理
  用于事先不确定是否有该标记的事件绑定，handler 直接绑定到父标记上，同时通过 selector 指定可能出现的标记：
```js
  $('#my-list').on('mouseover', 'li', function(event) {
    console.log('List item: ' + ($(this).index() + 1));
  });
```
  相应的 JavaScript 版本：
```js
  document.getElementById('my-list').addEventListener('mouseover',
    function(event) {
      if (event.target.nodeName === 'LI') {
        console.log('List item: ' +
          (Array.prototype.indexOf.call(
            document.getElementById('my-list').children,
            event.target
            ) + 1)
        );
      }
    },
    false
  );
```

  常见写法：
```js
  $(document).on('mouseover', '#my-list li', function(event) {
    console.log('List item: ' + ($(this).index() + 1));
  });
```
  但绑定很多代理事件到 document 上或 DOM 根节点附近会拖慢性能，因为冒泡机制会从 target 对象开始逐一匹配与 selector 相符的元素来触发事件。因此最佳实践尽量靠近触发的元素绑定代理事件。

  只触发一次事件：
  one(eventType[, selector][, data], handler);

  one(eventsHash[, selector][, data]);



  事件的删除:
  off(eventType[, selector][, handler]);
  off(eventsHash, [, selector]);
  off();
  要删除一个函数 handler, 必须事先声明且不能为匿名函数, 或者使用带命名空间的 eventType

  $('*').off('.fred');
  注意: 事件类型的命名空间是加后缀字符串, 如"click.myApp.myName"


6.2.3. 事件实例
  传到 jQuery 中的的 event 是经过处理的 event 实例, 是 jQuery 定义的一个属性 jQuery.Event, 且对常见的属性名和方法进行了标准化, 封装了浏览器的特性检验
  jQuery 中的 event 方法:
  preventDefault();
  stopPropagation();          //禁用绑在 target 上的当前事件的冒泡机制，其他事件不受影响
  stopImmediatePropagation(); //禁用绑在 target 上的所有事件的冒泡机制
  isDefaultPrevented();
  isPropagationStopped();
  isImmediatePropagationStopped();


6.2.4. 触发事件处理器
  trigger(eventType[, data]);
  data: (Any) Data to be passed to the handlers. If an array is provided, the elements are passed to the handler as different parameters.
```js
  $('#foo').on('click', function(event, par1, par2, par3){
    console.log(par1, par2, par3);
  });
  ==>
  $('#foo').trigger('click', [1, 2, 3]);
```
  如果之希望触发 handler 而不希望冒泡或触发默认动作，则需要调用 triggerHandler：
  `triggerHandler(eventType[, data]);`

  trigger vs triggerHandler：
  1）trigger 对所有 jQuery 集合生效，triggerHandler 仅对 jQuery 集首个元素生效；
  2）trigger 允许链式操作，triggerHandler 不允许（未指定返回值的情况下，默认返回 undefined）；
  3）trigger 会冒泡，triggerHandler 不会；

  示例：
```xml
  <div id="wrapper">
      <button id="btn">Click me!</button>
      <input type="text" id="address"/>
  </div>
```
```js
  <script>
  $('#wrapper')
      .on('focus', function() {
          console.log('Div focused');
      })
      .on('click', function() {
          console.log('Div clicked');
      });
  
  $('#address')
      .on('focus', function() {
          console.log('Input focused');
      })
      .triggerHandler('focus');
  
  $('#btn')
      .on('click', function() {
          console.log('Button clicked');
      });
  </script>
```

  快捷方法：简化 on 方法与 trigger 方法：
  通式：
    eventName([data,] handler)
  适用的方法名：
  `blur`   `change`   `click`   `dblclick`   `focus`   `focusin`   `focusout`   
  `keydown`   `keypress`   `keyup`   `mousedown`   `mouseenter`   `mouseleave`   
  `mousemove`   `mouseout`   `mouseover`   `mouseup`   `ready`   `resize`   
  `scroll`   `select`   `submit`
  其中，data 的值通过 handler 中的 event.data 访问

  jQuery 3 弃用了 `load()`, `unload()`, 和 `error()`


  划过元素的事件：hover()

  `mouseout` 与 `mouseover` 存在的弊病，进入子元素后同样判定为划出父元素。解决方法：换用 `mouseenter` 与 `mouseleave`：
```js
  $(element).mouseenter(function1).mouseleave(function2);
  .hover(enterHandler, leaveHandler);
  .hover(handler);
```
  自定义事件：
```js
  $('#btn').on('customEvent', function(){
      alert('customEvent');
  });
  $('#anotherBtn').click(function() {
      $('#btn').trigger('customEvent');
  });
```
  注意：自定义事件不能通过快键方式调用：
```js
  $('#btn').customEvent();  //error ocurred
```

  使用事件的命名空间
```js
  $('.my-class').on('click.editMode', myFunction);
  $('*').off('.editMode');
```

  事件命名空间是区分大小写的
```js
  $('.elements').on('click.myApp.myName', myFunction);
  $('.elements').trigger('click.myapp');
```

  同时触发同一命名空间下的多个事件：
```js
  $('.elements').on('click.myApp.myName', myFunction);
  $('.other-elements').on('click.myApp', myOtherFunction);
```
  调用含 myApp 的命名空间的事件函数：
```js
  $('.elements, .other-elements').trigger('click.myApp');
```