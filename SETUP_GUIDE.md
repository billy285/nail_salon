# 邮件和电话提醒设置指南

## 概述

本网站现在已包含邮件和电话提醒功能的框架。以下是完成设置所需的步骤。

## 一、邮件功能设置 (Formspree)

### 1. 创建 Formspree 账户
- 访问 https://formspree.io/
- 注册一个免费账户
- 验证您的邮箱地址

### 2. 创建表单
- 登录后，点击 "New Form"
- 输入表单名称（如 "Luxury Nails Booking"）
- 输入您想接收邮件的邮箱地址（如 info@luxurynails.com）
- 创建后，您会获得一个 Form ID（格式类似：`xaykbodj`）

### 3. 更新代码中的 Formspree ID

在 `index.html` 文件中，找到以下三处并替换 `your-form-id`：

1. **预约邮件** (第 2410 行左右):
```javascript
const formspreeEndpoint = 'https://formspree.io/f/your-form-id';
```

2. **联系邮件** (第 2526 行左右):
```javascript
const formspreeEndpoint = 'https://formspree.io/f/your-form-id';
```

3. **提醒邮件** (第 2160 行左右):
```javascript
const formspreeEndpoint = 'https://formspree.io/f/your-form-id';
```

### 4. 启用邮件发送

在 `sendEmail` 函数中（第 2444-2468 行），取消注释实际的发送代码：

```javascript
// 取消注释下面的代码
fetch(endpoint, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(data)
})
.then(response => {
    if (response.ok) {
        console.log('Email sent successfully');
    } else {
        console.error('Error sending email');
    }
})
.catch(error => {
    console.error('Error:', error);
});
```

## 二、电话/SMS 提醒设置

### 选项 A: 使用 Twilio (推荐)

1. **创建 Twilio 账户**
   - 访问 https://www.twilio.com/
   - 注册并获得免费试用额度
   - 获取您的 Account SID, Auth Token, 和 Twilio 电话号码

2. **设置后端服务器**
   
   由于浏览器安全限制，您需要一个后端服务器来发送 SMS。创建一个简单的 Node.js 服务器：

   ```javascript
   // server.js
   const express = require('express');
   const twilio = require('twilio');
   const cors = require('cors');
   
   const app = express();
   app.use(cors());
   app.use(express.json());
   
   const accountSid = 'YOUR_TWILIO_ACCOUNT_SID';
   const authToken = 'YOUR_TWILIO_AUTH_TOKEN';
   const twilioPhone = '+1234567890'; // 您的 Twilio 号码
   
   const client = twilio(accountSid, authToken);
   
   app.post('/send-sms', (req, res) => {
       const { to, message } = req.body;
       
       client.messages.create({
           body: message,
           from: twilioPhone,
           to: to
       })
       .then(message => res.json({ success: true, sid: message.sid }))
       .catch(error => res.json({ success: false, error: error.message }));
   });
   
   app.listen(3001, () => console.log('Server running on port 3001'));
   ```

3. **更新前端代码**
   
   在 `sendBookingReminder` 函数中添加：
   ```javascript
   fetch('http://localhost:3001/send-sms', {
       method: 'POST',
       headers: { 'Content-Type': 'application/json' },
       body: JSON.stringify({
           to: booking.phone,
           message: `Reminder: Your Luxury Nails appointment on ${booking.date} at ${booking.time}. Book ID: ${booking.bookingId}`
       })
   });
   ```

### 选项 B: 使用其他 SMS 服务

- Plivo: https://www.plivo.com/
- Nexmo/Vonage: https://www.vonage.com/
- 国内服务商：阿里云短信、腾讯云短信等

## 三、提醒功能说明

### 当前功能

1. **预约确认邮件**: 客户提交预约后立即发送
2. **联系表单邮件**: 客户发送联系消息后发送
3. **预约提醒**: 预约前 24 小时自动发送邮件和 SMS
4. **本地存储**: 所有预约保存在浏览器 localStorage 中

### 提醒检查频率

- 页面加载时立即检查
- 之后每小时检查一次
- 只对未发送过提醒的预约发送提醒

## 四、测试建议

1. **测试邮件功能**:
   - 使用自己的邮箱地址进行预约测试
   - 检查 Formspree 仪表板确认邮件发送

2. **测试提醒功能**:
   - 创建一个 24 小时内的预约进行测试
   - 使用浏览器开发者工具的 Console 查看日志

3. **查看存储的预约**:
   - 在浏览器 Console 中输入: `showStoredBookings()`

## 五、生产环境注意事项

1. **后端服务器**: 对于生产环境，您需要：
   - 部署后端服务器（使用 Heroku, Vercel, AWS 等）
   - 使用 HTTPS
   - 添加身份验证保护 API 端点

2. **数据持久化**: 
   - 当前使用 localStorage（仅限单个浏览器）
   - 生产环境建议使用数据库（MongoDB, PostgreSQL 等）

3. **安全考虑**:
   - 不要在前端暴露 API 密钥
   - 验证所有用户输入
   - 使用 HTTPS

## 六、需要帮助？

如有问题，请查看：
- Formspree 文档: https://formspree.io/help
- Twilio 文档: https://www.twilio.com/docs

祝您使用愉快！
