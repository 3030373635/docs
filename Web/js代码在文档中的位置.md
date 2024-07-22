## 位置一

**简介**

```
在 body 节点之前编写 js 代码, 但需要利用 window.onload 事件,　
该事件在当前文档完全加载之后被触发, 所以其中的代码可以获取到当前文档的任何节点.
```

**示例**

```javascript
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript">
    //onload的使用方式
    window.onload=function(){
        var bt=document.getElementsByTagName("button")[0];
        bt.onclick=function(){
            alert("have click on");
        }
    }
</script>
</head>
<body>
    <button>click on me</button>
</body>
</html>
```

## 位置二

**简介**

```
直接在html代码程序里写js。

　　缺点: 
　　　　①. js 和 html 强耦合, 不利用代码的维护
　　　　②. 若 click 相应函数是比较复杂的, 则需要先定义一个函数, 然后再在 onclick 属性中完成对函数的引用, 比较麻烦
```

**示例**

```javascript
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
    <button onclick="alert('hello JS 2')"></button>
</body>
</html>
```

## 位置三

**简介**

```
将js直接放在html文档的后部分，这样在html文档加载完成后再加载js。

　　但是，不符合习惯。
```

**示例**

```javascript
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
    <button>click on me</button>
    <script type="text/javascript">
        var bts=document.getElementsByTagName("button");
        bts[0].onclick=function(){
            alert("hello js 3");
        }
    </script>
</body>
</html>