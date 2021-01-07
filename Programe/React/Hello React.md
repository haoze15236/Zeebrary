# 引入react框架

**CDN**:通过引入react的js来使用

```html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>React hello world</title>
    <script src="https://unpkg.com/react@16/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
</head>
<body>
    <div id="app"></div>
    <script>
        var hello = React.createElement('h1',{},"hello world");
        ReactDOM.render(hello,document.getElementById('app'));
    </script>
</body>
</html>
```

**yarn**：通过yarn安装react插件

```shell
#下载react框架核心插件
yarn add react react-dom --save
```

