---
title: 'diss Symbol😏'
date: '2019-06-27'
spoiler: Symbol到底有何用处
---

> 说实话，作为js的第7种原始类型，感觉Symbol在我们工作中的价值真的不大，要么是注册全局常量，除去了赋值的烦恼，要么是作为对象property唯一key，要么是利用内置Symbol重新改造原生对象的内部行为。

从第一种作用开始diss吧，从使用场景来说，第一种真的很鸡肋，我寻遍了所有我的代码中的常量，我发现他们的赋值都是有真正意义上的用处的，比如要和服务端对齐，要和产品对齐，就拿[es6中文文档](http://caibaojian.com/es6/symbol.html)这里的例子来说吧，我认为是没有说全面的，请看代码：

```
function getArea(shape, options) {
  var area = 0;

  switch (shape) {
    case 'Triangle': // 魔术字符串
      area = .5 * options.width * options.height;
      break;
    /* ... more code ... */
  }

  return area;
}

getArea('Triangle', { width: 100, height: 100 }); // 魔术字符串
```
乍一看，真的是消除了魔术字符串。那么另外一个问题来了，什么时候会用到getArea，一定是产品的需求告诉我们，这个时候需要绘制一个三角形，那么这段代码是要给谁看的，给我们的开发同事一起看的，如果仅仅是用Symbol来代替，是不是有点不清不楚，用Triangle却让人一目了然，但是Triangle确实是个魔术字符串，无意义的字符串出现在代码中一看就不专业。请看下一段优化代码：

```
var shapeType = {
  triangle: 'Triangle'
};

function getArea(shape, options) {
  var area = 0;
  switch (shape) {
    case shapeType.triangle:
      area = .5 * options.width * options.height;
      break;
  }
  return area;
}

getArea(shapeType.triangle, { width: 100, height: 100 });
```
哦？我理解的是用`const TRIANGLE = 'Triangle'`，好歹提出一个变量，没想到可以`shapeType.triangle`，这样既解决了triangle让人一目了然，又把字符串提出来了。再看最后一段代码：

```
const shapeType = {
  triangle: Symbol()
};
````
优秀啊！糟糕，我被我自己说服了，收回第一种作用很鸡肋的说法，对不起打扰了（姨母笑

再看第二种，说一个我在日常工作中真的有遇到这样的场景，上家公司写react的时候，和别人讨论connect取state的时候是否要展开，哈，我坚持要展开，因为在组件中取的时候方便啊，哈，别人坚持不要展开，因为有可能组件还需要另一个state，展开有可能key重复啊。当时还没用过symbol，所以我妥协了，可是就算现在遇到了symbol，so what?我写的时候怎么能知道别人的哪个key会和我重复啊，别人也不知道我的啊，难道key全都要用symbol来写吗？

```
var mySymbol = Symbol();
var hisSymbol = Symbol();
var herSymbol = Symbol();
var whoseSymbol = Symbol();

var a = {
  [mySymbol]: 'Hello!'
  [hisSymbol]: 'Symbol!',
  [herSymbol]: 'hhhh',
  [whoseSymbol]: 'hhhh~'，
};

a[mySymbol] // "Hello!"
```
疯都疯了吧？所以第二种是真吐槽，不接受反驳（暗中观察，你难道还有其他法门？

第三种我也得吐槽，为什么？因为大多数人的工作场景都用不了！听有人说这是一种元编程，一听我就觉得不得了高大上超纲了赶紧在线膜拜，最后理解的结果就是：用程序写程序，简单粗暴见得最多的就是代码生成器，那在js中用symbol去改变js这门语言的实现也就是一种元编程能力，这种机制也必须是js提供给我们的，称之为反射。所以我们会说js具有了元编程方面的能力。这里有一段散发陈旧味道的比较原始的[元编程代码](https://www.cnblogs.com/liuyanlong/archive/2013/05/27/3102161.html)，可以瞧一瞧。科普结束。来看看别人家的代码，改造js从现在开始：

```
class Even {
  static [Symbol.hasInstance](obj) {
    return Number(obj) % 2 === 0;
  }
}

1 instanceof Even // false
2 instanceof Even // true
12345 instanceof Even // false
```

诶？仿佛很有趣很牛逼哦！那我可不可以这样写？

```
class Even {
  static isInstance(obj) {
    return Number(obj) % 2 === 0;
  }
}

Even.isInstance(1) // false
Even.isInstance(2) // true
Even.isInstance(12345) // false
```
为什么我非要用Symbol.hasInstance啊？为了装逼。欧，对不起打扰了（姨母笑）。话说回来，如果装逼没人捧的话也很容易乏味的哦！仔细想想，装逼的前提是共享啊，共享的话别人是不是得和你达成共识，你要告诉别人你们在做的是一门新的语言，叫js+，js+里面有内置对象Even，继承于Object，真的很厉害哦，如果有这样的岗位，请叫上我。可惜没有，可惜我还是在写业务写bug搬砖，普适性不强啊！

还有一种说法，前瞻性，这是一个非常有意思的词，前瞻性🤔要考虑到多远才算是有用的，考虑到未来别人会用到这个命名，所以我要让这个命名唯一，我认为不如换种说法，换成“这组信息是元信息的时候我们就用symbol去定义”，一来元信息内部使用，对象固有属性；二来symbol唯一性，虽不是完全私有但大多数情况下外部可以忽略不易被发现，在于`Object.getOwnPropertyNames()`和`Object.keys()`都取不到它。

事情就是这样的，科普性的文章贴在参考资料里，这里只记录思考过程，欢迎diss与battle哦!

> ps: 写完我又认真看了看下面两篇文章，找到了一个[nodejs](https://nodejs.org/api/util.html#util_util_inspect_object_options)使用`Symbol.toStringTag`的地方，惊喜，一定要点开看看哦！

Symbol科普：

[http://caibaojian.com/es6/symbol.html](http://caibaojian.com/es6/symbol.html)

[https://juejin.im/post/5a0e65c1f265da430702d6b9#heading-8](https://juejin.im/post/5a0e65c1f265da430702d6b9#heading-8)
