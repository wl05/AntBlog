这里我们使用svg-captcha插件在node.js中生成svg格式的验证码。

## 基本使用
1. 安装
```bash
npm install --save svg-captcha
```
2. 使用方式
```
const svgCaptcha = require('svg-captcha');
const code = svgCaptcha.create({
  size: 6,  //验证码长度
  width: 100,
  height: 38,
  background: '#ddd',
  noise: 4,//干扰线条数
  fontSize: 32,
  ignoreChars: '0o1i',   //验证码字符中排除'0o1i'
  color: true // 验证码的字符是否有颜色，默认没有，如果设定了背景，则默认有
});
const authCode = code.text.toLowerCase(); // 验证码字符串
const authCodeSvg = code.data; // 验证码svg
```

## 项目中实战
以注册为例子

## 参考文章
1. [NodeJs生成SVG图形验证码](https://www.cnblogs.com/kakayang/p/8794546.html)
