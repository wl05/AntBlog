这里我们使用svg-captcha插件在node.js中生成svg格式的验证码。

## 基本使用
1. 安装
```bash
npm install --save svg-captcha
```
2. 使用方式
```js
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
![image](https://github.com/antbaobao/AntBlog/blob/master/node/imgs/1.png?raw=true)
在这个注册表单中我们需要验证码，获取验证码的controller代码如下：
```js
 // controller/user.js
 generateAuthCode() {
    const { ctx, app } = this;
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

    const authCode = code.text.toLowerCase();
    const ip = ctx.helper.getIp(ctx);
    app.redis.set(`${ip}_authCode`, authCode, 'EX', 10 * 60); // 设置authCode 并指定过期时间十分钟
    return ctx.helper.success(ctx, { authCode: code.data, ip });
  }
```

意思就是产生验证码，然后把验证码的字符串存到的redis里面，这里key值用的是ip+'_authCode',同时给验证码设置过期时间。然后返回svg的验证码给前端。然后注册提交的时候再从redis里面比对提交的验证码是否存在。
OK，我们来看看注册controller的具体实现。
```js
async signup() {
    const { ctx, app, service } = this;
    const validator = struct({
      name: 'string',
      gender: 'string',
      email: 'string',
      password: 'string',
      authCode: 'string'
    });
    try {
      validator(ctx.request.body);
    } catch (err) {
      return ctx.helper.error(ctx, error_002[ 0 ], error_002[ 1 ]);
    }
    try {
      const user_f = await service.user.findByName({ name: ctx.request.body.username });
      if (user_f) { // 账号已存在
        return ctx.helper.error(ctx, user_002[ 0 ], user_002[ 1 ]);
      } else {
        // 判断验证码是否有效
        const ip = ctx.helper.getIp(ctx);
        const authCode = await app.redis.get(`${ip}_authCode`);
        console.log(authCode, ctx.request.body.authCode.toLowerCase());
        if (authCode !== ctx.request.body.authCode.toLowerCase()) {
          return ctx.helper.error(ctx, user_003[ 0 ], user_003[ 1 ]);
        }
        const salt = rand(160, 36);
        ctx.request.body.password = sha1(ctx.request.body.password + salt);
        ctx.request.body.salt = salt;
        const user = await service.user.create(ctx.request.body);
        console.log('user=======', user);
        let email = user.email;
        ctx.helper.sendUserEmail(ctx, email);
        return ctx.helper.success(ctx);
      }
    } catch (e) {
      console.log(e);
      return ctx.helper.error(ctx, error_001[ 0 ], error_001[ 1 ]);
    }

  }
```

前端实现逻辑就不展示了。具体的代码可以参考[我的网站](https://github.com/antbaobao/AntVueBlogFront)

## 参考文章
1. [NodeJs生成SVG图形验证码](https://www.cnblogs.com/kakayang/p/8794546.html)
