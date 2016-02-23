#ES7 异步函数
[原文传送门](https://jakearchibald.com/2014/es7-async-functions/)
```
They're brilliant. They're brilliant and I want laws changed so I can marry them.
```
原文如斯，不想翻译。
##结合Async的Promise
在 [HTML5Rocks article on promises](http://www.html5rocks.com/en/tutorials/es6/promises/#toc-parallelism-sequencing)中，最后一个例子展示了如何为一个故事加载JSON数据，然后用同样的方式去获取章节的JSON数据，最后在这些数据都返回之后渲染他们。

代码如下：
```javascript```
function loadStory() {
  return getJSON('story.json').then(function(story) {
    addHtmlToPage(story.heading);

    return story.chapterURLs.map(getJSON)
      .reduce(function(chain, chapterPromise) {
        return chain.then(function() {
          return chapterPromise;
        }).then(function(chapter) {
          addHtmlToPage(chapter.html);
        });
      }, Promise.resolve());
  }).then(function() {
    addTextToPage("All done");
  }).catch(function(err) {
    addTextToPage("Argh, broken: " + err.message);
  }).then(function() {
    document.querySelector('.spinner').style.display = 'none';
  });
}
```
看起来不算太坏，但是···
##是时候上ES7异步函数了···
```javascript````
async function loadStory() {
  try {
    let story = await getJSON('story.json');
    addHtmlToPage(story.heading);
    for (let chapter of story.chapterURLs.map(getJSON)) {
      addHtmlToPage((await chapter).html);
    }
    addTextToPage("All done");
  } catch (err) {
    addTextToPage("Argh, broken: " + err.message);
  }
  document.querySelector('.spinner').style.display = 'none';
}
```
通过使用异步函数（[草案](https://github.com/lukehoban/ecmascript-asyncawait)）,你可以等待`await`一个`Promise`的完成。这将以非阻塞的方式暂停这个函数的进行，并且等待`Promise`的`resolve`函数的执行并返回结果。如果`Promise`抛出异常，你可以通过`catch`去处理这个异常。

作者语：我原本在箭头函数中使用`await`，[显然这是不对的](https://twitter.com/mraleph/status/449192750735704065)，所以我用`for`循环代替，关于“[为什么`await`不能在箭头函数中使用](https://github.com/lukehoban/ecmascript-asyncawait/issues/7)”这个疑惑，Domenic 的解答使我茅塞顿开。

`loadStory`返回一个`Promise`对象，所以可以在另一个异步函数中继续使用他。
```javascript```
(async function() {
  await loadStory();
  console.log("Yey, story successfully loaded!");
}());
```
##直到ES7时代的来临···
你可以通过[`Traceur transpiler`](http://goo.gl/Dc6V1B)来使用包括异步函数在内的 ES6/7 新特性。此外，你还可以通过 ES6 的`generator`特性来实现异步函数。

你仅需要额外的的很小一段代码，一个[`spawn`](https://gist.github.com/jakearchibald/31b89cba627924972ad6)函数。然后你就可以使用`generator`来实现异步函数：
```javascript```
function loadStory() {
  return spawn(function *() {
    try {
      let story = yield getJSON('story.json');
      addHtmlToPage(story.heading);
      for (let chapter of story.chapterURLs.map(getJSON)) {
        addHtmlToPage((yield chapter).html));
      }
      addTextToPage("All done");
    } catch (err) {
      addTextToPage("Argh, broken: " + err.message);
    }
    document.querySelector('.spinner').style.display = 'none';
  });
}
```
在上面的代码中，我将一个`generator`函数作为参数传入`spawn`函数中，因为在`function *()`中存在星号，因此你可以称之为`generator`函数。`spawn`函数在`generator`函数中调用`.next()`，当执行到`yield`语句时接收到一个`Promise`对象，然后等待这个异步操作成功再调用`.next()`（或者失败后再调用`.throw()`）。

ES7 将`spawn`函数引入至规范中，并且可以更方便地使用。通过标准化规范来简化异步编程简直太棒了！
##额外阅读
- [JavaScript promises, there and back again](http://www.html5rocks.com/en/tutorials/es6/promises/) - Promise入门

- [`Promise.resolve()` is not the opposite of `Promise.reject()`](https://jakearchibald.com/2014/resolve-not-opposite-of-reject/) - 关于Promise的误解
