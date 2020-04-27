---
title: 只有20行Javascript代码！手把手教你写一个页面模板引擎
date: 2018-02-24 10:04:24
tags: "Js"
categories: "Js"
---
[原文地址](http://blog.jobbole.com/56689/)

```javascript
var TemplateEngine = function(html, options) {
    var re = /<%([^%>]+)?%>/g, reExp = /(^( )?(if|for|else|switch|case|break|{|}))(.*)?/g, code = 'var r=[];\n', cursor = 0; //正则捕获所有以<%开头，以%>结尾的片段
    var add = function(line, js) {
        js? (code += line.match(reExp) ? line + '\n' : 'r.push(' + line + ');\n') :
            (code += line != '' ? 'r.push("' + line.replace(/"/g, '\\"') + '");\n' : '');
        return add;
    }//使用条件判断和循环,复杂逻辑
    while(match = re.exec(html)) {
        add(html.slice(cursor, match.index))(match[1], true);
        cursor = match.index + match[0].length;
    }
    add(html.substr(cursor, html.length - cursor));
    code += 'return r.join("");';//把所有的字符串放在一个数组里，在程序的最后把它们拼接起来。
    return new Function(code.replace(/[\r\t\n]/g, '')).apply(options);//把原本返回模板字符串的语句替换成如下的内容
}
```