1. 什么是跨域？
    想要知道什么是跨域，就要先认识下同源策略(CORP)，指的是浏览器对不同源的脚本或者文本的访问方式进行的限制。
    其中同源指的是 **协议+域名+端口** 三者一致。它是浏览器最基本也最核心的安全功能，能够有效地预防 XSS CSRF 等攻击。
    跨域指的就是没有遵守同源策略的规定，访问其他网站的资源失败。
    
2. 如何解决跨域？
+ Jsonp
    原理：在 Html 页面中通过相应的标签从不同的域名下加载静态资源文件是被浏览器允许的。
    所以我们可以通过动态创建 script 标签，再去请求一个带参网址来实现跨域通信。
    ```javascript
    let script = doument.createElement('script');

    script.src = 'http://www.abc.com/login?username=xiaoming&cb=callback';

    document.body.appendChild(script);

    function callback(res) {
        console.log(res);
    }
    ```
    使用 Jsonp 实现跨域，其**兼容性好**，但**不支持 POST 请求**，且不安全会造成** XSS **攻击。
+ CORS
    只需服务端设置头 Access-Control-Allow-Origin 即可。
+ Http-proxy
+ postMessage
+ 正反代理
    总结一句话到底是正向代理还是反向代理，只需要判断客户端知不知道真正返回数据的服务器在谁，知道就是正向代理，不知道就是反向代理
+ Nginx
+ document.domain + iframe
+ window.name
+ location.hash
+ Websocket