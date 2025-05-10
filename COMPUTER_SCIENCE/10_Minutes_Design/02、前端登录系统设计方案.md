> 这是自己一个小项目登录的真实设计，但是是让 AI 帮忙总结的 HLD。
## 🧱 一、系统背景

### 1. 第三方 SSO 登录流程

- 用户点击“登录”按钮，将跳转至第三方 SSO 登录页面
- 登录成功后，第三方将用户 **重定向**回前端页面，并在 URL 中 **携带 `session-token`**

例如：
```
https://frontend.example.com/login?session-token=abc123
```

---

## 🔐 二、认证流程设计

### 1. 登录流程（处理 session-token）

前端在登录页接收到 `session-token` 后：

```
[SSO 登录成功]
     ↓
[前端登录页接收 session-token]
     ↓
[调用后端 /api/auth/validate-session]
     ↓
[后端验证 token → 设置 HttpOnly Cookie]
     ↓
[前端拉取用户信息，存入内存]
```

---

## 🔄 三、前后端交互流程

### ✅ 前端请求设置（React）

```ts
fetch('https://api.example.com/api/auth/validate-session', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ sessionToken }),
  credentials: 'include', // 关键：允许带 cookie
})
```

- 所有接口请求都需设置：`credentials: 'include'`
- Axios 示例：`axios.post(..., { withCredentials: true })`

---

### ✅ 后端响应设置（Next.js）

在 Next.js 的 API Route 中设置响应头（动态设置 CORS）：

```ts
const allowedOrigins = ['https://frontend-a.com', 'https://frontend-b.com']
const origin = req.headers.origin

if (allowedOrigins.includes(origin)) {
  res.setHeader('Access-Control-Allow-Origin', origin)
  res.setHeader('Access-Control-Allow-Credentials', 'true') // 允许前端带 cookie
  res.setHeader('Access-Control-Allow-Methods', 'GET,POST,OPTIONS')
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type')
}

// 设置 Cookie（HttpOnly + Secure）
res.setHeader('Set-Cookie', `auth_token=xxx; HttpOnly; Secure; SameSite=None; Path=/`)
```

⚠️ 跨域 + Cookie 必须使用：
- `Access-Control-Allow-Origin: <具体域名>`
- `Access-Control-Allow-Credentials: true`
- Cookie 设置为 `HttpOnly; Secure; SameSite=None`

---

## 🧠 四、前端用户状态管理方案

### ✅ 使用 React Context 存储用户信息（避免 localStorage）

```tsx
// auth-context.tsx
const AuthContext = createContext({ user: null, setUser: () => {} })

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null)

  useEffect(() => {
    fetch('https://api.example.com/api/auth/me', {
      credentials: 'include',
    })
      .then((res) => res.json())
      .then((data) => {
        if (data.success) setUser(data.user)
      })
  }, [])

  return (
    <AuthContext.Provider value={{ user, setUser }}>
      {children}
    </AuthContext.Provider>
  )
}
```

- 优点：
  - 数据保存在内存中，安全（避免 XSS 读 localStorage）
  - 全局可访问
  - 页面刷新自动重拉用户信息

---

## 🔐 五、Cookie 鉴权机制

### ✅ Cookie 设置建议

| 属性         | 说明                                       |
|--------------|--------------------------------------------|
| HttpOnly     | JS 无法访问，防 XSS                        |
| Secure       | 仅 HTTPS 下传输                            |
| SameSite=None| 跨站点请求时允许携带 Cookie（必须为 None）|
| Path=/       | 全站点有效                                 |

例如：
```http
Set-Cookie: auth_token=xxx; HttpOnly; Secure; SameSite=None; Path=/
```

---

## 🧪 六、完整登录流程总结

```plaintext
1. 用户点击登录 → 跳转至 SSO 登录页面
2. 登录成功 → SSO 重定向回前端页面，带上 session-token
3. 前端读取 session-token → 请求后端 /validate-session
4. 后端验证 token → 设置认证 Cookie（HttpOnly）
5. 前端拉取用户信息 /api/auth/me → 存入 Context（内存）
6. 后续所有请求自动携带 Cookie，完成鉴权
7. 页面刷新 → 继续读取 Cookie + 拉取用户信息
```

---

## 🧰 七、安全最佳实践总结

| 项目                       | 建议做法 |
|----------------------------|----------|
| Token 存储                 | 不存 localStorage，全部放在 Cookie（HttpOnly） |
| Cookie 设置                | `HttpOnly; Secure; SameSite=None` |
| 前端状态存储               | 使用 React Context，存储在内存 |
| 前端请求设置               | `credentials: 'include'` |
| 后端 CORS 设置             | 动态设置 `Access-Control-Allow-Origin`，不能是 `*` |
| CSRF 防护（可选）         | 对敏感接口增加 CSRF Token 校验 |
| 页面路由保护               | 使用 `AuthProvider + PrivateRoute` 控制访问 |
| HTTPS                     | 前后端都使用 HTTPS |

---

## ✅ 附加建议（可选扩展）

- 使用 React Router + ProtectedRoute 实现路由级权限控制
- 使用 Redux Toolkit 或 Zustand 管理状态（替代 Context）
- 实现 Token 刷新机制（如果 Cookie 过期）
- 添加 WebSocket 鉴权支持（如有实时通信需求）
