其实非常的简单
1. ```npm install nodemailer --save```
2. 参考如下代码
```js
const nodemailer = require('nodemailer');
/**
 * @param ctx
 * @param cnd
 * describe: sendUserEmail
 */
/**
 * @param ctx
 * @param cnd
 * describe: sendUserEmail
 */
exports.sendUserEmail = (ctx, toEmail) => {
  const expiraton = 1800;
  const code = require('crypto')
    .randomBytes(16)
    .toString('hex');
  ctx.app.redis.set(`${code}`, toEmail, 'EX', expiraton);
  const config_email = {
    host: 'smtp.qq.com',
    post: 25, // SMTP 端口
    //secureConnection: true, // 使用 SSL
    auth: {
      user: '2929712050@qq.com',
      //这里密码不是qq密码，是你设置的smtp授权码
      pass: 'rdbbqlgolefhdecc'
    }
  };
  const transporter = nodemailer.createTransport(config_email);
  const html = '感谢您的注册，请点击下面的链接激活您的账号，如果链接在邮箱中打不开，您可以试试将其复制到浏览器地址栏中<div>' + ctx.app.config.baseUrl + '?code=' + code + '&account=' + toEmail + '</div>';
  const data = {
    from: '2929712050@qq.com', // 发件地址
    to: toEmail, // 收件人
    subject: '注册激活-汪乐的个人网站', // 标题
    //text: html // 标题 //text和html两者只支持一种
    html: html // html 内容
  };
  transporter.sendMail(data, function(err, info) {
    if (err) {
      return (err);
    }
    return (info.response);
  });
};
```
简单说明一下，这段代码是我用来发送[我的网站](https://github.com/antbaobao/AntVueBlogFront)注册接口激活码的一段代码，重点看```config_email```以下的部分就可以了。

这里我使用的是qq邮箱，需要获取相应的授权码，也很简单
1. 登录你的qq邮箱网页，找到设置。
2. 切换到账户，找到```POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务```开启```POP3/SMTP服务```
3. 开启成功后你就可以得到你的授权码了。

## 参考资料
1. [如何使用nodejs发邮件](https://www.cnblogs.com/chenjg/p/nodemailer.html)
