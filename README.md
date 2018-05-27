# vue-router-h5-history
vue-router的HTML5 History 模式，这种模式充分利用 history.pushState API 来完成 URL 跳转而无须重新加载页面。

```js
const router = new VueRouter({
  mode: 'history',
  routes: [...]
})
```
当你使用 history 模式时，URL 就像正常的 url，例如 `http://yoursite.com/user/id`， 就是长这样的！

不过这种模式要玩好，还需要后台配置支持。因为我们的应用是个单页客户端应用，如果后台没有正确的配置，当用户在浏览器直接访问 `http://oursite.com/user/id` 就会返回 404，这就尴尬了。

所以，你要在服务端增加一个覆盖所有情况的候选资源：如果 URL 匹配不到任何静态资源，则应该返回同一个 index.html 页面，这个页面就是你 app 依赖的页面。

目前后端服务器有**Apache、nginx、原生 Node.js、基于 Node.js 的 Express、Internet Information Services (IIS)、Caddy、Firebase 主机**等。

先给一个官方标准版配置的传送门，请戳这里→ [https://router.vuejs.org/zh/guide/essentials/history-mode.html](https://router.vuejs.org/zh/guide/essentials/history-mode.html)
既然是官方给出的配置，那肯定就会说的很官方咯~

按照官方给出的示例，将道理是成功的，但是官方给出的只局限于这个项目在服务器的**根目录**下！
如果你的项目没有放在根目录下，那么就是这么的不讲道理。

下面是在实际开发中碰到的问题。

我们的后端服务器是nginx，所以按照官方的给出的配置，找到nginx.conf，照抄代码，修改完后重启服务器，讲道理是成功的。
```
server
{
    listen 80;
    server_name oursite.com;
    location /test/ {            
        alias  /usr/local/test/;
        index  index.htm index.html;
        #上面的就是一些常规配置，下面这个才是重点
        try_files $uri $uri/ /index.html;

        #这里没有采用官方给出处理404错误页面的方案
        #方案一（把所有没有后缀名的请求如果404都跳转到index.html，我们没有采用）
        #error_page 404 /test/index.html;

        #方案二（404的方式，不是特别完美。会有浏览器留下404的状态（容易被第三方劫持），以下方式可以避免被第三方劫持！）
        if (!-e $request_filename) {
            rewrite ^/(.*) /test/index.html last;
            break;
        }
    }
}
```

但是，事实就是这么的不讲道理，我们并没有成功！这就令人很是郁闷了，心中一万只神兽羊驼，奔腾而过~ 为什么没有成功呢，都是按部就班做的呀，完全不讲道理！

而真相只有一个，那就是~~~ 

**路由文件中的路径有问题**
```js
const router = new VueRouter({
  mode: 'history',
  routes: [
    { path: '/test/', component: YourComponent },
    { path: '/test/a', component: YourComponent },
    { path: '/test/b:x', component: YourComponent }
  ]
})
```
在路由文件中所有的路径前面加上服务器下项目所在的文件名即可，当然也包括`<router-link>`和`this.$router.push()`中的路径，不然又是不讲道理的。

这只是在nginx服务器下的一种解决方案，至于别的服务器应该也是同理的。

如果您有更好更优的方案，欢迎交流，谢谢。
