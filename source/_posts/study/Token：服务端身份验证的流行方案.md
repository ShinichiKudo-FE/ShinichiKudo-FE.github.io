---
title: Token：服务端身份验证的流行方案
date: 2018-05-23 18:42:30
tags: "Token"
---
# 身份认证

服务端提供资源给客户端，但是某些资源是有条件的。所以服务端要能够识别请求者的身份，然后再判断所请求的资源是否可以给请求者。

token是一种身份验证的机制，初始时用户提交账号数据给服务端，服务端采用一定的策略生成一个字符串（token），token字符串中包含了少量的用户信息，并且有一定的期限。服务端会把token字符串传给客户端，客户端保存token字符串，并在接下来的请求中带上这个字符串。

它的工作流程大概是这样

## Token机制

> 在这样的流程下，我们需要考虑下面几个问题：
* 如何根据token获取用户的信息？
* 确保识别伪造的token？
* 这里是指token不是经过服务端来生成的。
* 如何应付冒充的情况？
* 非法客户端拦截了合法客户端的token，然后使用这个token向服务端发送请求，冒充合法客户端。

## 用户匹配

服务端在生成token时，加入少量的用户信息，比如用户的id。服务端接收到token之后，可以解析出这些数据，从而将token和用户关联了起来。

## 防伪造

一般情况下，建议放入token的数据是不敏感的数据，这样只要服务端使用私钥对数据生成签名，然后和数据拼接起来，作为token的一部分即可。比如JWT，参考JSON Web Token - 在Web应用间安全地传递信息。

在我知道JWT之前，我先了解的是另一种模式。基于加密的算法，对数据进行加密，把加密的结果作为token（`http://blog.leapoahead.com/2015/09/06/understanding-jwt/`）。

本文主要讨论后一种模式。

## 防冒充

### 加干扰码

服务端在生成token时，使用了客户端的UA作为干扰码对数据加密，客户端进行请求时，会同时传入token、UA，服务端使用UA对token解密，从而验证用户的身份。
<!-- more -->
如果只是把token拷贝到另一个客户端使用，不同的UA会导致在服务端解析token失败，从而实现了一定程度的防冒充。但是攻击者如果猜到服务端使用UA作为加密钥，他可以修改自己的UA。

### 有效期

给token加上有效期，即使被冒充也只是在一定的时间段内有效。这不是完美的防御措施，只是尽量减少损失。

服务端在生成token时，加入有效期。每次服务端接收到请求，解析token之后，判断是否已过期，如果过期就拒绝服务。

### token刷新

如果token过期了，客户端应该对token续期或者重新生成token。

> 这取决于token的过期机制。
服务器缓存token及对应的过期时间
这个时候就可以采用续期的方式，服务器更新过期时间，token再次有效。
token中含有过期时间
这个时候需要重新生成token。

在token续期或者重新生成token的时候，需要额外加入数据来验证身份。因为token已经过期了，即token已经不能用来验证用户的身份了。这个时候可以请求用户重新输入账号和密码，但是用户体验稍差。

另一种方式是使用摘要。服务端生成token，同时生成token的摘要，然后一起返回给客户端。客户端保存摘要，一般请求只需要用到token，在刷新token时，才需要用到摘要。服务端验证摘要，来验证用户的身份。因为摘要不会频繁的在客户端和服务端之间传输，所以被截取的概率较小。

## Token工作流程

### 生成token

一般在登录的时候生成token。Token管理者负责根据用户的数据生成token和摘要，摘要用来给APP端刷新token用，类似于微信登录中的refresh_token。

生成token的过程中，ua的充作干扰码。没有相同的ua，就无法解析生成的token字符串。如果客户端是混合开发的模式，生成token和使用token的代理可能不同（比如一个是h5，一个是原生），ua就会不同，token就无法成功的使用。可以选择其他的客户端数据作为干扰码，
> 注意考虑下面的问题：
* 不同的客户端，干扰码应该不同
* 干扰码的很大一个作用是防冒充，如果选择的充当干扰码的客户端数据没有区分性，就达不到效果。选择充当干扰码的数据，在哪些情况下会变化？这些情况是否合理？
* 比如干扰码数据中含有app的版本号，那么app版本升级就会导致干扰码变化。服务端根据新的干扰码，无法解析旧的token，此时用户必须重新登录。这种情况是合理的吗？如果不合理，干扰码中就不应该含有app的版本号。

### 拦截验证

客户端的每一次请求，都必须携带token、ua，拦截器会对敏感资源的访问进行拦截，然后根据ua解析token，解析不成功，表示token和ua不匹配。解析成功之后，判断token是否已过期，如果是，拒绝服务。所有都ok的情况下，拦截器放行，请求传达到业务服务者。

### Token刷新

当token过期，用户需要刷新token。刷新token本质上是这样的：

客户端需要把token、摘要、ua都传给服务端，服务端对token重新生成摘要，通过判断两个摘要是否相同来验证这次请求刷新token的客户端，是不是上次请求生成token的客户端。验证通过，服务端需要使用用户数据重新生成token，用户数据则来自用ua解析token的结果。

## HTML&JS等前端知识系列之Ajax post请求带有token向Django请求

我们 在母板上写入这段代码：

```js
<script type="text/javascript">
        // 个人定义大函数，不是重点，可以忽略
        $(document).ready(function(){
            get_sys_load();
            var csrftoken = getCookie('csrftoken');
            var active_node = $("#mainnav-menu a[href='{{ request.path }}']");
            active_node.parent().addClass("active-link");
            if (active_node.parent().parent().hasClass("collapse")){
                active_node.parent().parent().addClass("in");
            }

        });//end doc ready
        // 个人定义大函数，不是重点，可以忽略
        function  get_sys_load(){
            $.ajax({
                url: "{% url 'get_server_host_status' %}",
                type: "GET",
                dataType: "json",
                success: function(callback){
                    for( i in callback){
                            console.log(i,callback[i]);
                            $('#'+ i +'_display').text(callback[i]);
                            $('#'+ i +'_width').text(callback[i]);
                            $('#'+ i +'_attr').css('width',callback[i]+'%')

                    }// end for
                },// end sucess func
                error: function(callback){
                    alert(callback)
                }// end error func
            })
        }
        // 这个才是重点的代码，必须写
        function getCookie(name) {
            var cookieValue = null;
            if (document.cookie && document.cookie !== '') {
                var cookies = document.cookie.split(';');
                for (var i = 0; i < cookies.length; i++) {
                    var cookie = jQuery.trim(cookies[i]);
                    // Does this cookie string begin with the name we want?
                    if (cookie.substring(0, name.length + 1) === (name + '=')) {
                        cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                        break;
                    }
                }
            }
            return cookieValue;
        }

        // 这个才是重点的代码，必须写
        function csrfSafeMethod(method) {
            // these HTTP methods do not require CSRF protection
            return (/^(GET|HEAD|OPTIONS|TRACE)$/.test(method));
        }
        $.ajaxSetup({
            beforeSend: function(xhr, settings) {
                if (!csrfSafeMethod(settings.type) && !this.crossDomain) {
                    xhr.setRequestHeader("X-CSRFToken", csrftoken);
                }
            }
        });
    </script>
    {% block  bottom-js %}
    {% endblock %}
```

我们在子板上调用这端js代码，调用的前提是子板的html页面必须嵌套了这个 csrf_token, 代码如下

```js
html页面的代码：
==================
<span>{% csrf_token %}</span>
==================



======JQuery代码 =======

{% block  bottom-js %}
<script>

    function run_cmd(){
        var csrftoken = getCookie('csrftoken');     // 调用获取token的方法来获取token，getCookie是前面定义好的函数。
        var input_cmd = $('textarea').val();
        $.ajax({
            url:"{% url 'put_cmd' %}",
            type:'POST',
            dataType:'json',
            token: csrftoken,                  // 把token塞入头部
            data:{'host_id':$('#host_id').text(),'minion_name':$('#minion_id').text(),'cmd':input_cmd},
            success: function(callback){
                console.log(callback)
            }, // end success
            error: function (callback) {
                console.log(callback);
                $('code').append(callback)
            }
        })
    }

    function show_result(content){

    }

</script>

{% endblock %}
```

此时，我们在提交post请求，就能够正常提交了，可以参考官网的信息：
[ajax token](https://docs.djangoproject.com/en/dev/ref/csrf/#ajax)。

## 登录后保存token到cookie中

```js
// 1.引入相应JS

　　<script src="web/js/jquery-1.9.1.min.js"></script>

　　<script src="web/js/jquery.cookie.js"></script>

// 2.保存cookie

　　$.cookie("名称","值") //$.cookie('cookieName', 'cookieValue');

　　$.cookie("名称","值",{expires:失效时间})

// 3.获取cookie

　　$.cookie("名称") //$.cookie('cookieName');

// 4.删除cookie

　　$.cookie("名称",null)


```

## Token机制相比Cookie机制的优点

* **支持跨域访问:**cookie是不允许跨域访问的，这一点对token机制是不存在的，前提是传输的用户认证信息通过HTTP头传输.

* **无状态(也称：服务端可扩展行):**Token机制在服务端不需要存储session信息，因为Token 自身包含了所有登录用户的信息，只需要在客户端的cookie或本地介质存储状态信息.

* **更适用于cdn:** 可以通过内容分发网络请求你服务端的所有资料（如：javascript，HTML,图片等），而你的服务端只要提供API即可

* **去耦**: 不需要绑定到一个特定的身份验证方案。Token可以在任何地方生成，只要在你的API被调用的时候，你可以进行Token生成调用即可.

* **更适用于移动应用:** 当你的客户端是一个原生平台（iOS, Android，Windows 8等）时，Cookie是不被支持的（你需要通过Cookie容器进行处理），这时采用Token认证机制就会简单得多。

* **CSRF**:因为不再依赖于Cookie，所以你就不需要考虑对CSRF（跨站请求伪造）的防范。

* **性能:** 一次网络往返时间（通过数据库查询session信息）总比做一次HMACSHA256计算 的Token验证和解析要费时得多.不需要为登录页面做特殊处理: 如果你使用Protractor 做功能测试的时候，不再需要为登录页面做特殊处理.

* **基于标准化:**你的API可以采用标准化的 JSON Web Token (JWT). 这个标准已经存在多个后端库（.NET, Ruby, Java,Python, PHP）和多家公司的支持（如：Firebase,Google, Microsoft）.
