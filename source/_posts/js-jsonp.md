---
title: JSONP的原理与实现
category: Javascript
tags:
  - Javascript
date: 2016-06-10 14:34:16
---
之前在做统计分析系统的时候需要跨域请求数据.Jquery已经封装了jsonp的实现,但是考虑到统计代码的大小,决定弃用jquery.自己实现一下.
jsonp的原理其实很简单,通过script标签发送一个callback参数到后端,后端返回数据时,使用这个callback函数包裹一下.这样,在前端接收到后端的返回后,就可以执行这个callback函数.
在使用jsonp时需要注意,jsonp只能发送get请求.
下面是jsonp的实现代码.
```javascript
var _ajax = {
    ajax_jsonp: {
        //获取当前时间戳
        now: function () {
            return (new Date()).getTime();
        },
        //获取16位随机数
        rand: function () {
            return Math.random().toString().substr(2);
        },
        //url组装
        parse_data: function (data) {
            var ret = "";
            if (typeof data === "string") {
                ret = data;
            }
            else if (typeof data === "object") {
                for (var key in data) {
                    ret += "&" + key + "=" + encodeURIComponent(data[key]);
                }
            }
            //加上时间戳
            ret += "&_time=" + this.now();
            return ret;
        },
        /**
        * jsonp调用的方法
        * @param {object} url 请求的url
        * @param {object} data 请求的data
        */
        jsonp: function (url, data) {
            var name;
            url = url + (url.indexOf("?") === -1 ? "?" : "&") + this.parse_data(data);

            //检测callback的函数名是否已经定义
            var match = /callback=([a-zA-Z\._]+)/.exec(url);
            if (match && match[1]) {
                name = match[1].replace(/\./g, "_");
            }
            else {
                //如果url中没有callback,直接返回
                throw new Error("invalid callback param");
            }

            //创建script标签
            var script = document.createElement("script");
            script.type = "text/javascript";
            script.src = url;

            //在head里面插入script元素
            var head = document.getElementsByTagName("head");
            if (head && head[0]) {
                head[0].appendChild(script);
            }
        }
    }
};
```
调用
```javascript
_ajax.ajax_jsonp.jsonp(_url, data);
#比如_url中的callback是这样的参数: ?callback=window.getdata
#则在js中注册这个window.getdata函数即可
window.getdata = function(data)
{
    console.log(data);
}
```

