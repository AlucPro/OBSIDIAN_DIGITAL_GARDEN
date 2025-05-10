> 最近和同事讨论登录安全时，我们聊到了一个真实遇到，藏得很深的安全问题。
> 它发生在用户登录后，在“绑定三方账号”功能时，被伪造攻击。
> 惯例，下边是我们按照自己的真实经验，让 AI 代笔。


## 一、背景介绍

在现代 Web 应用中，提供微信、QQ、GitHub 等第三方账号绑定功能已经非常常见。用户在登录网站后，可以选择绑定自己的第三方账号，实现快捷登录或账号互通。

然而，如果绑定接口设计不安全，攻击者可能通过脚本注入或诱导点击的方式，将你的网站账号绑定到他们自己的第三方账号上，从而实现账号劫持。

---

## 二、常规绑定流程

以“绑定微信账号”为例，一个典型流程如下：

1. 用户登录网站；
2. 用户点击【绑定微信】按钮；
3. 跳转或弹出微信授权窗口；
4. 用户授权后，微信返回一个授权 key（如 code）；
5. 前端拿到 key，请求后端绑定接口；
6. 后端完成绑定，将微信账号与当前用户账号关联。

---

## 三、安全风险分析

如果攻击者在你登录网站后，通过 XSS 或其他方式在页面中注入恶意脚本，就可能诱导你点击一个伪造的“绑定按钮”，而绑定的是**攻击者自己的微信账号 key**。

### 攻击流程如下：

1. 用户正常登录；
2. 恶意脚本弹出伪装绑定窗口；
3. 用户点击确认；
4. 前端发起绑定请求，带的是**黑客的微信 key**；
5. 后端识别用户已登录，直接完成绑定；
6. 攻击者的微信账号与用户网站账号绑定成功，以后可通过微信登录用户账号。

---

## 四、安全设计方案

为了防止上述攻击，我们需要确保：

- 发起绑定请求的是当前登录用户本人；
- 请求中的 key 是用户自己的操作产生的；
- 绑定请求不能被脚本伪造。

### 设计核心

在绑定请求中引入：

- 一个用户身份标识 `identityToken`，仅保存在前端内存中；
- 一个 `sign` 签名字段，由前端使用加密算法对 `identityToken` 加密生成；
- 后端对 `sign` 和 `identityToken` 进行验证。

---

## 五、安全绑定流程

1. 用户登录后，后端返回一个 `identityToken`；
2. 前端将该 `identityToken` 保存在内存中（**不能存入 cookie**）；
3. 用户点击绑定时，前端使用加密算法对 `identityToken` 生成签名 `sign`；
4. 前端向绑定接口发送：`key`, `identityToken`, `sign`；
5. 后端：
   - 验证 `sign` 是否是 `identityToken` 的合法加密结果；
   - 验证 `identityToken` 是否属于当前登录用户；
   - 验证 key 是否有效；
   - 验证通过后，完成绑定。

---

## 六、前端示例代码（JavaScript）

```javascript
// utils/crypto.js
const SECRET_KEY = 'your-shared-secret'; // 与后端约定密钥

function generateSign(token) {
  // 简化加密示例，实际应使用 AES/HMAC 等更安全算法
  return btoa(`${token}:${SECRET_KEY}`);
}
```

```javascript
// bindWechat.js
import { generateSign } from './utils/crypto';

async function bindWechatAccount(wechatKey) {
  const token = sessionStorage.getItem('identity_token');
  if (!token) {
    alert('未登录或身份过期');
    return;
  }

  const sign = generateSign(token);

  const payload = {
    key: wechatKey,
    token,
    sign
  };

  const response = await fetch('/api/bind/wechat', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(payload)
  });

  const result = await response.json();
  if (!response.ok) {
    alert(result.error || '绑定失败');
  } else {
    alert('绑定成功');
  }
}
```

---

## 七、后端示例代码（JavaScript / Node.js + Express）

```javascript
// utils/crypto.js
const SECRET_KEY = 'your-shared-secret';

function verifySign(token, sign) {
  const expected = Buffer.from(`${token}:${SECRET_KEY}`).toString('base64');
  return expected === sign;
}

module.exports = { verifySign };
```

```javascript
// server.js
const express = require('express');
const bodyParser = require('body-parser');
const { verifySign } = require('./utils/crypto');

const app = express();
app.use(bodyParser.json());

const sessionMap = new Map(); // token -> userId

// 模拟登录接口
app.post('/api/login', (req, res) => {
  const userId = 'user_123';
  const token = `token_${Date.now()}`;
  sessionMap.set(token, userId);
  res.json({ token });
});

// 模拟微信绑定接口
app.post('/api/bind/wechat', (req, res) => {
  const { key, token, sign } = req.body;

  if (!verifySign(token, sign)) {
    return res.status(400).json({ error: '签名验证失败' });
  }

  const userId = sessionMap.get(token);
  if (!userId) {
    return res.status(401).json({ error: '身份无效' });
  }

  if (!key || !key.startsWith('wx_')) {
    return res.status(400).json({ error: '微信授权 key 无效' });
  }

  // 执行绑定（伪代码）
  console.log(`用户 ${userId} 成功绑定微信账号 ${key}`);
  return res.json({ success: true });
});

app.listen(3000, () => {
  console.log('Server running at http://localhost:3000');
});
```

---

## 八、安全加固建议

- 签名中加入时间戳 + 过期时间，防止重放攻击；
- 后端限制绑定接口频率（防刷）；
- token 建议为 JWT 或随机字符串，短期有效；
- 敏感数据全部走 HTTPS；
- 前端签名逻辑不要暴露密钥，复杂算法建议服务端生成签名；
- 登录状态建议结合 access_token + refresh_token 实现。
