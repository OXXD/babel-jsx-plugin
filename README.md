# Babel 插件如何实现 JSX 到 JS 的编译

## 1. 解析 `JSX` 文件的插件

`jsx-praser.js`

```javascript
module.exports = function () {
  return {
    manipulateOptions: function manipulateOptions(opts, parserOpts) {
      parserOpts.plugins.push("jsx");
    }
  };
};
```

## 2. 转换 `JSX` => `React.createElement`

它的实现原理是这样的。Babel 读取代码并解析，生成 AST，再将 AST 传入插件层进行转换，在转换时就可以将 JSX 的结构转换为 React.createElement 的函数。如下代码所示：

`jsx-plugin.js`

```javascript
module.exports = function (babel) {
  var t = babel.types;
  return {
    name: "custom-jsx-plugin",
    visitor: {
      JSXElement(path) {
        var openingElement = path.node.openingElement;
        var tagName = openingElement.name.name;
        var args = []; 
        args.push(t.stringLiteral(tagName)); 
        var attribs = t.nullLiteral(); 
        args.push(attribs); 
        var reactIdentifier = t.identifier("React"); //object
        var createElementIdentifier = t.identifier("createElement"); 
        var callee = t.memberExpression(reactIdentifier, createElementIdentifier)
        var callExpression = t.callExpression(callee, args);
        callExpression.arguments = callExpression.arguments.concat(path.node.children);
        path.replaceWith(callExpression, path.node); 
      },
    },
  };
};
```

## 3. `.babelrc`

```javascript
{
  "plugins": ["jsx-plugin", "./jsx-parser"]
}
```

## 4. 编译

```jsx
function Test() {
  return <div><a>Hello~~~</a></div>
}
```

```shell
babel hello.jsx
```

## References

- [Babel 的插件开发文档](https://www.babeljs.cn/docs/plugins)
- [Babel Plugins](https://babeljs.io/docs/en/plugins)
- [前端面试宝典之 React 篇 - Babel 插件如何实现 JSX 到 JS 的编译](https://kaiwu.lagou.com/course/courseInfo.htm?courseId=566&sid=20-h5Url-0&buyFrom=2&pageId=1pz4#/detail/pc?id=5793)