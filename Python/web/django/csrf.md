---
title: csrf
---


# 一、简介

> [详细介绍点此](https://baike.baidu.com/item/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0/13777878)
>
> django为用户实现防止跨站请求伪造的功能，通过**中间件 django.middleware.csrf.CsrfViewMiddleware** 来完成。而对于django中设置防跨站请求伪造功能有分为全局和局部。

```python
全局：
　　中间件 django.middleware.csrf.CsrfViewMiddleware

局部：

装饰器：@csrf_protect，为当前函数强制设置防跨站请求伪造功能，即便settings中没有设置全局中间件。
装饰器：@csrf_exempt，取消当前函数防跨站请求伪造功能，即便settings中设置了全局中间件。

注：from django.views.decorators.csrf import csrf_exempt,csrf_protect
```

# 二、应用

## 2.1 普通表单

```python
veiw中设置返回值：
　　return render_to_response('Account/Login.html',data,context_instance=RequestContext(request))　　
     或者
     return render(request, 'xxx.html', data)
  
html中设置Token:
　　{% csrf_token %}
```

## 2.2 ajax

> 对于传统的form，可以通过表单的方式将token再次发送到服务端，而对于ajax的话，使用如下方式。

```html
{% load static %}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>后台登录</title>
    <link rel="shortcut icon" href="{% static 'common/img/favicon.ico' %}">
    <link rel="stylesheet" href="{% static 'user/css/reset.css' %}">
    <link rel="stylesheet" href="{% static 'user/css/style.css' %}">

</head>
<body>
<div id="particles-js">
    <div class="login">
{#        <form action="" method="post">#}
            {% csrf_token %}
            <div class="login-top">
                登录
            </div>
            <div class="login-center clearfix">
                <div class="login-center-img"><img src="{% static 'user/images/name.png' %}"></div>
                <div class="login-center-input">
                    <input type="text" name="username" id="username" placeholder="请输入您的用户名" onfocus="this.placeholder=''"
                           onblur="this.placeholder='请输入您的用户名'" value="{{ user_name }}">
                    <div class="login-center-input-text">用户名</div>
                </div>
            </div>
            <div class="login-center clearfix">
                <div class="login-center-img"><img src="{% static 'user/images/password.png' %}"></div>
                <div class="login-center-input">
                    <input type="password" name="password" id="password" value="" placeholder="请输入您的密码" onfocus="this.placeholder=''"
                           onblur="this.placeholder='请输入您的密码'">
                    <div class="login-center-input-text">密码</div>
                </div>
            </div>
            <div id="container">
            </div>
            <p style="color: red;text-align: center" id="error">{{ error }}</p>
            <div style="text-align: center">
                <button class="login-button" onclick="click_login()">登录</button>
            </div>
{#        </form>#}
    </div>
    <canvas id="c_n4" width="860" height="958"
            style="position: fixed; top: 0px; left: 0px; z-index: -1; opacity: 0.5;"></canvas>
</div>
<script src="{% static 'common/plugins/jquery/jquery.min.js' %}"></script>
<script src="{% static 'user/js/jigsaw.min.js' %}"></script>
<script src="{% static 'user/js/canvas-nest.min.js' %}"></script>
<script type="text/javascript">
    function hasClass(elem, cls) {
        cls = cls || '';
        if (cls.replace(/\s/g, '').length === 0) return false; //当cls没有参数时，返回false
        return new RegExp(' ' + cls + ' ').test(' ' + elem.className + ' ');
    }

    function addClass(ele, cls) {
        if (!hasClass(ele, cls)) {
            ele.className = ele.className === '' ? cls : ele.className + ' ' + cls;
        }
    }

    function removeClass(ele, cls) {
        if (hasClass(ele, cls)) {
            var newClass = ' ' + ele.className.replace(/[\t\r\n]/g, '') + ' ';
            while (newClass.indexOf(' ' + cls + ' ') >= 0) {
                newClass = newClass.replace(' ' + cls + ' ', ' ');
            }
            ele.className = newClass.replace(/^\s+|\s+$/g, '');
        }
    }




</script>
<script>
    var flag = null;
    jigsaw.init({
        el: document.getElementById('container'),
        width: 300, // 可选, 默认310
        height: 155, // 可选, 默认155

        onSuccess: function () {
            //滑块验证通过
            flag = true
            $("#error").text("")

        },
        onFail: function () {
            //滑块验证未通过
            flag = false
        },
        onRefresh: function () {
            //滑块刷新
            flag = false

        }
    })
    function click_login() {
        if(flag){
            //发送ajax请求
            $.ajax({
                url:"{% url 'login' %}",
                type:"POST",
                data:{
                    "username":$("#username").val(),
                    "password":$("#password").val(),
                    "csrfmiddlewaretoken": $("[name = 'csrfmiddlewaretoken']").val()
                },
                success:function (data) {

                    if(data.user_name)
                   {

                       window.location.href = "{% url 'index' %}"
                   }
                   else{
                       $("#error").text(data.error)

                   }
                },
                error:function () {
                    $("#error").text(data.error)
                }
            })
        }else{
            $("#error").text("滑块未通过")
        }
    }
</script>
</body>
</html>