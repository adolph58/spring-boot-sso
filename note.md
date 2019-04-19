### 6.4.2 登录登录设计
登录登出的设计，虽然在 Spring Security 中已经实现，但是对于使用 SSO 的客户端  
来说，必须进行一些合理的调整，否则如果设计不当，就有可能出现无法正常退出或者  
登录失败的情况。

代码清单 6-7 是客户端的一个登出设计，使用一个重定向链接 "redirect:/#/"  来刷  
新当前访问页面，从而触发系统检查用户的授权状态，如果用户未被授权，则引导用户  
到登录认证服务器中登录。

代码清单 6-7 客户端登录控制器
```
    @RequestMapping("/login")
    public String login() {
        return "redirect:/#/";
    }
```

代码清单 6-8 是客户端的一个登出设计，退出时首先通过用户确认，然后使用 POST  
方式执行表单 logoutform 的退出提交请求，这个请求已由 Spring Security 实现，  
执行一些清除会话和登录状态等操作，并将当前操作界面重定向到登出成功页面上。

这里需要注意的是，用户这时只是退出当前客户端而已，并没有在 SSO 服务端中执  
行过退出请求，也就是说，在 SSO 认证服务端中，还保存着用户的登录状态。如果  
这时返回原来的客户端，或者访问其他由授权的客户端，都不会要求用户登录并能  
正常访问。为了能真正的退出，还必须在 SSO 服务端中再执行一次退出请求。这个  
退出请求必须由程序来处理，而不能要求用户转到 SSO 服务端中再执行一次退出。

代码清单 6-8 客户端登出设计
```$xslt
<a href="javascript:void(0)" id="logout">[退出]</a>
……
<form th:action="@{/logout}" method="post" id="logoutform">
</form>
<div class="footBox" th:replace="fragments/footer :: footer"></div>
<script type="text/javascript">
    $(function () {
        $("#logout").click(function () {
            if (confirm('您确定退出吗？')) {
                $("#logoutform").submit();
            }
        });
    });

```

在客户端配置中有一个成功退出的链接地址，当用户在客户端中成功退出时，将被  
重定向到这个链接地址。代码清单 6-9 是这个链接的页面设计，它只做一件简单的  
事情，即在当前页面中做一个跳转链接，转到 SSO 服务端中执行退出请求。

代码清单 6-9 跳转到 SSO 服务端中执行登出
```
<script>
    function to_sso(){
        location.href = "http://localhost/signout";
    }
</script>
<body onload="to_sso()">
</body>
```

代码清单 6-10 是 SSO 服务端的登出控制器设计，只有请求这个链接，才能让用户  
完全退出当前的登录状态。程序中使用 request.logout() 来请求 Spring Security  
执行退出请求，然后返回 SSO 的登录界面，以刷新当前的页面状态。

代码清单 6-10 SSO 服务端登出控制器
```
@RequestMapping("/signout")
    public String signout(HttpServletRequest request) throws Exception{
        request.logout();
        return "tologin";
    }
```

代码清单 6-11 是接收上面的请求后，返回的一个页面设计，它同样也只做一件简单  
的事情，即将当前页面跳转到用户登录界面上。

代码清单 6-11 跳转到登录界面
```
<script>
    <!--
    function new_window(){
        location.href = "/login";
    }
    //-->
</script>
<body onload="new_window()">
</body>
```

这样，用户不管在哪个客户端中执行退出，通过上面的跳转，最终都将被引导到 SSO   
服务端的登录界面上。通过这些流程的处理，用户的一个退出请求才能彻底退出登录  
状态。当然，上面这些跳转对于用户来说，是完全透明的。

用户打开任何一个客户端的页面进行登录，登录成功后将被引导到最初打开的页面上。  
如果用户直接在 SSO 认证服务端上登录，必须有一个成功登录后的默认主页，这个  
页面可以配置一些其他客户端的导航链接。因为把登录成功的默认主页设计放置在  
客户端 1 的主页上，所以如果用户从 SSO 服务器中直接登录，就可以在 SSO 服务  
器的主页上做一个跳转，如代码清单 6-12 所示，将跳转到客户端 1 的主页上。

代码清单 6-12 登录成功的默认链接设计
```
<script>
    <!--
    function new_window(){
        location.href = "http://localhost:8081/";
    }
    //-->
</script>

<!-- onload="new_window()"-->
<body onload="new_window()">
......请稍候

</body>
```