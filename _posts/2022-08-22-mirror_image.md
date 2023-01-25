---
layout: post
title: 使用Cloudflare搭建网站镜像
author: Yang
---

# Cloudflare Workers

## 搭建维基百科镜像[^1]

可以使用 Cloudflare Workers 搭建维基百科镜像网站。Cloudflare Workers 有免费版本，但是有配额限制，免费版本每天的请求数量不能超过100000次。请求数次日不累计，会清零重新计算。

点击"Create a Worker"

输入以下代码

```
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  try{
    u = (new URL(request.url))
    if(u.protocol == 'http:'){
      u.protocol = 'https:'
      return (new Response('', {status: 301, headers: {'Location': u.href}}))
    }

    h = (new Headers(request.headers))
    ua = h.get('User-Agent')

    v = ''
    ual = ua.toLowerCase()
    if(ual.indexOf('mobile') !== -1 || ual.indexOf('android') !== -1 || ual.indexOf('like mac os x') !== -1){v = '.m'}

    host = u.hostname
    path = u.pathname

    //u.searchParams.append('variant','zh-cn')
    argv = u.search

    if(path == '/'){return (new Response('', {status: 302, headers: {'Location': '/wiki/'}}))}

    uri = path+argv

    d = await fetch('https://zh'+v+'.wikipedia.org'+uri)
    return (new Response(d.body, {status: d.status, headers: d.headers}))
  }catch(e){
    return (new Response(e, {status: 500}))
  }
}
```

点击保存即可使用

注：
以上代码不支持POST方法，而编辑页面需要使用POST方法，所以只能查看不能编辑页面。
默认的字词转换模式为“不转换”，如果想使用其他字词转换模式，请删除上方内容中u.searchParams.append('variant','zh-cn')之前的//字样，然后将zh-cn替换为您想要的字词转换模式。

[^1]:参考文献：[Help:如何访问维基媒体旗下项目](https://meta.wikimedia.org/wiki/Help:%E5%A6%82%E4%BD%95%E8%AE%BF%E9%97%AE%E7%BB%B4%E5%9F%BA%E5%AA%92%E4%BD%93%E6%97%97%E4%B8%8B%E9%A1%B9%E7%9B%AE)

---

## 搭建谷歌镜像

点击"Create a Worker"

输入以下代码

```
// Website you intended to retrieve for users.
const upstream = 'www.google.com'
 
// Custom pathname for the upstream website.
const upstream_path = '/'
 
// Website you intended to retrieve for users using mobile devices.
const upstream_mobile = 'www.google.com'
 
// Countries and regions where you wish to suspend your service.
const blocked_region = ['KP', 'SY', 'PK', 'CU']
 
// IP addresses which you wish to block from using your service.
const blocked_ip_address = ['0.0.0.0', '127.0.0.1']
 
// Whether to use HTTPS protocol for upstream address.
const https = true
 
// Whether to disable cache.
const disable_cache = true
 
// Replace texts.
const replace_dict = {
    '$upstream': '$custom_domain',
    '//google.com': ''
}
 
addEventListener('fetch', event => {
    event.respondWith(fetchAndApply(event.request));
})
 
async function fetchAndApply(request) {
 
    const region = request.headers.get('cf-ipcountry').toUpperCase();
    const ip_address = request.headers.get('cf-connecting-ip');
    const user_agent = request.headers.get('user-agent');
 
    let response = null;
    let url = new URL(request.url);
    let url_hostname = url.hostname;
 
    if (https == true) {
        url.protocol = 'https:';
    } else {
        url.protocol = 'http:';
    }
 
    if (await device_status(user_agent)) {
        var upstream_domain = upstream;
    } else {
        var upstream_domain = upstream_mobile;
    }
 
    url.host = upstream_domain;
    if (url.pathname == '/') {
        url.pathname = upstream_path;
    } else {
        url.pathname = upstream_path + url.pathname;
    }
 
    if (blocked_region.includes(region)) {
        response = new Response('Access denied: WorkersProxy is not available in your region yet.', {
            status: 403
        });
    } else if (blocked_ip_address.includes(ip_address)) {
        response = new Response('Access denied: Your IP address is blocked by WorkersProxy.', {
            status: 403
        });
    } else {
        let method = request.method;
        let request_headers = request.headers;
        let new_request_headers = new Headers(request_headers);
 
        new_request_headers.set('Host', url.hostname);
        new_request_headers.set('Referer', url.hostname);
 
        let original_response = await fetch(url.href, {
            method: method,
            headers: new_request_headers
        })
 
        let original_response_clone = original_response.clone();
        let original_text = null;
        let response_headers = original_response.headers;
        let new_response_headers = new Headers(response_headers);
        let status = original_response.status;
        
        if (disable_cache) {
            new_response_headers.set('Cache-Control', 'no-store');
        }
 
        new_response_headers.set('access-control-allow-origin', '*');
        new_response_headers.set('access-control-allow-credentials', true);
        new_response_headers.delete('content-security-policy');
        new_response_headers.delete('content-security-policy-report-only');
        new_response_headers.delete('clear-site-data');
        
        if(new_response_headers.get("x-pjax-url")) {
            new_response_headers.set("x-pjax-url", response_headers.get("x-pjax-url").replace("//" + upstream_domain, "//" + url_hostname));
        }
        
        const content_type = new_response_headers.get('content-type');
        if (content_type.includes('text/html') && content_type.includes('UTF-8')) {
            original_text = await replace_response_text(original_response_clone, upstream_domain, url_hostname);
        } else {
            original_text = original_response_clone.body
        }
        
        response = new Response(original_text, {
            status,
            headers: new_response_headers
        })
    }
    return response;
}
 
async function replace_response_text(response, upstream_domain, host_name) {
    let text = await response.text()
 
    var i, j;
    for (i in replace_dict) {
        j = replace_dict[i]
        if (i == '$upstream') {
            i = upstream_domain
        } else if (i == '$custom_domain') {
            i = host_name
        }
 
        if (j == '$upstream') {
            j = upstream_domain
        } else if (j == '$custom_domain') {
            j = host_name
        }
 
        let re = new RegExp(i, 'g')
        text = text.replace(re, j);
    }
    return text;
}
 
 
async function device_status(user_agent_info) {
    var agents = ["Android", "iPhone", "SymbianOS", "Windows Phone", "iPad", "iPod"];
    var flag = true;
    for (var v = 0; v < agents.length; v++) {
        if (user_agent_info.indexOf(agents[v]) > 0) {
            flag = false;
            break;
        }
    }
    return flag;
}
```

点击保存即可使用
