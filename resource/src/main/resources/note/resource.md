## 共享资源服务
实例工程的共享资源模块，可以为已经授权的用户提供一些共享信息服务  
ResourceApplication 中 @EnableResourceServer 标注这个应用是一个资源服务器  
一个应用被标注为资源服务器后，在浏览器中就不能直接访问

### 查询登录用户的详细信息
在单独使用 Spring Security 安全管理的应用中，只要在控制器中使用 Principal,  
就能取得用户的完整信息，或者使用如代码清单 1 所示的代码，也能很容易地获取  
登录用户的详细信息。但是，使用 SSO 之后，这种方法就不能适用了，这时如果使  
用 getDetails() 将返回一个空值，而在控制器中使用 Principal 也只能返回登录  
用户的用户名、用户拥有的角色和登录令牌等信息而已，用户的其他信息如性别、  
邮箱等将不能取得。这是 OAuth2 基于安全的考虑而设计的，因为 SSO 涉及了第三  
方的应用请求，所以它保护了登录用户的隐私信息。

代码清单 1 
```
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
User user = (User)authentication.getDetails();
```

如果需要取得登录用户的详细信息，如性别、邮箱、所属的部门等，就只能像前面  
提到的那样，使用资源服务器提供的共享资源接口，然后通过 Zuul 路由代理服务  
获取已经登录的用户详细信息。
代码清单 2 是一个使用 Ajax 获取登录用户的详细信息的例子

代码清单 2
```
function getuserinfo(){
    $.get('./resource/user',{ts:new Date().getTime()},function(data){
        user = data;
        var $list = $('#tbodyContent').empty();
            var html = "" ;
            html += '<tr> ' +
                '<td>'+ (data.id==null?'':data.id) +'</td>' +
                '<td>'+ (data.name==null?'':data.name) +'</td>' +
                '<td>'+ (data.email==null?'':data.email) +'</td>' +
                '<td>'+ (data.department==null?'' :data.department) +'</td>' +
                '<td>'+ (data.createdate==null?'': getSmpFormatDateByLong(data.createdate,true)) +'</td>';
            html +='</tr>' ;

            $list.append($(html));
    });
}
```