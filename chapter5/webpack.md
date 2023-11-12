#### webpack 打包同步引用和异步引用的区别

-   先说一下 import 和 require 的异同：import 和 require 本质上都是同步导入，也就是 buildtime 的时候就把引用的东西都导入了，区别就是 import 导入的是一个引用，而 require 引入的相当于 const（如果导入的是值，就是不变的； 如果导入的是对象，则值是可变的）。如果想要异步，runtime 的时候才知道值的内容，需要使用 import()或者 require.ensure()

-   webpack 打包的区别在同步和异步处理的不同区别是，同步打包会打到一个包里，比如最简版就是打到 bundles.js。而异步处理会打出 0.js, 1.js, 2.js 这样几个不同的引入包文件，等运行的时候，用哪个去加载哪个。异步的实现大概就是这样的，遇到异步模块时，使用**webpack_require**.e 函数去把异步代码加载进来。该函数会在 html 的 head 中动态增加 script 标签，src 指向指定的异步模块存放的文件

以下是异步引用打包后的实际代码：

```
/* webpack/runtime/load script */
 	(() => {
 		var inProgress = {};
 		var dataWebpackPrefix = "test-webpack:";
 		// loadScript function to load a script via script tag
 		__webpack_require__.l = (url, done, key, chunkId) => {
 			if(inProgress[url]) { inProgress[url].push(done); return; }
 			var script, needAttach;
 			if(key !== undefined) {
 				var scripts = document.getElementsByTagName("script");
 				for(var i = 0; i < scripts.length; i++) {
 					var s = scripts[i];
 					if(s.getAttribute("src") == url || s.getAttribute("data-webpack") == dataWebpackPrefix + key) { script = s; break; }
 				}
 			}
 			if(!script) {
 				needAttach = true;
 				script = document.createElement('script');

 				script.charset = 'utf-8';
 				script.timeout = 120;
 				if (__webpack_require__.nc) {
 					script.setAttribute("nonce", __webpack_require__.nc);
 				}
 				script.setAttribute("data-webpack", dataWebpackPrefix + key);

 				script.src = url;
 			}
 			inProgress[url] = [done];
 			var onScriptComplete = (prev, event) => {
 				// avoid mem leaks in IE.
 				script.onerror = script.onload = null;
 				clearTimeout(timeout);
 				var doneFns = inProgress[url];
 				delete inProgress[url];
 				script.parentNode && script.parentNode.removeChild(script);
 				doneFns && doneFns.forEach((fn) => (fn(event)));
 				if(prev) return prev(event);
 			}
 			var timeout = setTimeout(onScriptComplete.bind(null, undefined, { type: 'timeout', target: script }), 120000);
 			script.onerror = onScriptComplete.bind(null, script.onerror);
 			script.onload = onScriptComplete.bind(null, script.onload);
 			needAttach && document.head.appendChild(script);
 		};
 	})();
```
