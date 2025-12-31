# BestBlogs.dev OAuth 登录与密码重置功能

## 功能概述

本次更新为 BestBlogs.dev 添加了以下功能：
1. **Google OAuth2 登录** - 支持用户使用 Google 账号登录
2. **GitHub OAuth2 登录** - 支持用户使用 GitHub 账号登录
3. **忘记密码** - 用户可通过邮箱重置密码
4. **设置密码** - OAuth 用户可为账户设置本地密码
5. **Header 用户信息** - 登录状态下显示用户头像和下拉菜单
6. **用户设置页面** - 提供个人资料、密码设置、订阅管理功能

---

## 一、本次修改的文件清单

### 后端新增文件

| 文件路径 | 说明 |
|----------|------|
| `bestblogs-common/src/main/java/dev/bestblogs/foundation/enums/OAuthProviderEnum.java` | OAuth 提供商枚举 |
| `bestblogs-common/src/main/java/dev/bestblogs/adapter/OAuthAdapter.java` | OAuth 适配器接口 |
| `bestblogs-common/src/main/java/dev/bestblogs/adapter/impl/GoogleOAuthAdapter.java` | Google OAuth 适配器 |
| `bestblogs-common/src/main/java/dev/bestblogs/adapter/impl/GitHubOAuthAdapter.java` | GitHub OAuth 适配器 |
| `bestblogs-common/src/main/java/dev/bestblogs/adapter/ao/OAuthUserInfo.java` | OAuth 用户信息 DTO |
| `bestblogs-api/src/main/java/dev/bestblogs/api/controller/OAuthController.java` | OAuth 控制器 |
| `bestblogs-api/src/main/java/dev/bestblogs/api/service/OAuthService.java` | OAuth 服务 |
| `bestblogs-api/src/main/java/dev/bestblogs/api/controller/request/ForgotPasswordRequest.java` | 忘记密码请求 DTO |
| `bestblogs-api/src/main/java/dev/bestblogs/api/controller/request/ResetPasswordRequest.java` | 重置密码请求 DTO |
| `bestblogs-api/src/main/java/dev/bestblogs/api/controller/request/SetPasswordRequest.java` | 设置密码请求 DTO |
| `bestblogs-api/src/main/java/dev/bestblogs/api/controller/response/PasswordResetValidateResponse.java` | 密码重置验证响应 DTO |
| `bestblogs-admin-api/src/main/java/dev/bestblogs/api/admin/controller/dto/config/OAuthConfigDto.java` | OAuth 配置管理 DTO |

### 后端修改文件

| 文件路径 | 修改内容 |
|----------|----------|
| `bestblogs-common/src/main/java/dev/bestblogs/entity/UserEntity.java` | 添加 OAuth 和密码重置字段 |
| `bestblogs-common/src/main/java/dev/bestblogs/domain/model/UserModel.java` | 添加 OAuth 关联和密码重置方法 |
| `bestblogs-common/src/main/java/dev/bestblogs/domain/mapper/UserMapper.java` | 更新字段映射 |
| `bestblogs-common/src/main/java/dev/bestblogs/common/parameter/ConfigKey.java` | 添加 OAuth 配置键 |
| `bestblogs-common/src/main/java/dev/bestblogs/common/parameter/ParameterReadService.java` | 添加 OAuth 配置读取方法 |
| `bestblogs-api/src/main/java/dev/bestblogs/api/controller/UserController.java` | 添加密码重置端点，更新 convertToAuthInfo 返回更多用户信息 |
| `bestblogs-api/src/main/java/dev/bestblogs/api/controller/dto/UserAuthInfo.java` | 添加 userName, displayName, avatarUrl, oauthProvider, hasPassword 字段 |
| `bestblogs-api/src/main/java/dev/bestblogs/api/service/UserService.java` | 添加密码重置逻辑 |
| `bestblogs-api/src/main/java/dev/bestblogs/api/service/UserOpLogService.java` | 添加操作日志方法 |
| `bestblogs-admin-api/src/main/java/dev/bestblogs/api/admin/controller/ParameterConfigAdminController.java` | 添加 OAuth 管理接口 |

### 前端新增文件

| 文件路径 | 说明 |
|----------|------|
| `bestblogs-app/src/app/[lang]/auth/callback/page.tsx` | OAuth 回调处理页 |
| `bestblogs-app/src/app/[lang]/forgot-password/page.tsx` | 忘记密码页 |
| `bestblogs-app/src/app/[lang]/reset-password/page.tsx` | 重置密码页 |
| `bestblogs-app/src/app/[lang]/settings/page.tsx` | 用户设置页面（个人资料、密码设置、订阅管理） |

### 前端修改文件

| 文件路径 | 修改内容 |
|----------|----------|
| `bestblogs-app/src/app/[lang]/signin/page.tsx` | 添加社交登录按钮和忘记密码链接 |
| `bestblogs-app/src/app/[lang]/signup/page.tsx` | 添加社交登录按钮 |
| `bestblogs-app/src/lib/api.ts` | 添加 OAuth、密码重置、setPassword API 方法 |
| `bestblogs-app/src/types/User.ts` | 添加 userName, displayName, avatarUrl, oauthProvider, hasPassword 字段 |
| `bestblogs-app/src/components/common/UserInfo.tsx` | 重写组件：未登录显示登录按钮，已登录显示头像下拉菜单 |
| `bestblogs-app/messages/zh.json` | 添加中文翻译（Login、Settings、Header.user） |
| `bestblogs-app/messages/en.json` | 添加英文翻译（Login、Settings、Header.user） |

---

## 二、配置获取与设置

### 2.1 Google OAuth 配置

#### 获取配置步骤：

1. 访问 [Google Cloud Console](https://console.cloud.google.com/)
2. 创建新项目或选择现有项目
3. 导航到 **APIs & Services > Credentials**
4. 点击 **Create Credentials > OAuth client ID**
5. 选择 **Web application**
6. 配置：
   - **Name**: BestBlogs OAuth
   - **Authorized JavaScript origins**:
     - `https://www.bestblogs.dev`
     - `http://localhost:3000` (开发环境)
   - **Authorized redirect URIs**:
     - `https://api.bestblogs.dev/api/auth/oauth/google/callback`
     - `http://localhost:8080/api/auth/oauth/google/callback` (开发环境)
7. 点击 **Create**，获取 **Client ID** 和 **Client Secret**

#### 设置配置：

**方式一：管理后台配置（推荐）**

访问管理后台 `/api/admin/parameter/oauth` 接口设置：

```json
{
  "googleClientId": "your-google-client-id.apps.googleusercontent.com",
  "googleClientSecret": "your-google-client-secret",
  "googleRedirectUri": "https://api.bestblogs.dev/api/auth/oauth/google/callback"
}
```

**方式二：环境变量配置（后备）**

```properties
bestblogs.oauth.google.client-id=your-google-client-id.apps.googleusercontent.com
bestblogs.oauth.google.client-secret=your-google-client-secret
bestblogs.oauth.google.redirect-uri=https://api.bestblogs.dev/api/auth/oauth/google/callback
```

### 2.2 GitHub OAuth 配置

#### 获取配置步骤：

1. 访问 [GitHub Developer Settings](https://github.com/settings/developers)
2. 点击 **New OAuth App**
3. 配置：
   - **Application name**: BestBlogs
   - **Homepage URL**: `https://www.bestblogs.dev`
   - **Authorization callback URL**: `https://api.bestblogs.dev/api/auth/oauth/github/callback`
4. 点击 **Register application**
5. 获取 **Client ID**
6. 点击 **Generate a new client secret** 获取 **Client Secret**

#### 设置配置：

**方式一：管理后台配置（推荐）**

访问管理后台 `/api/admin/parameter/oauth` 接口设置：

```json
{
  "githubClientId": "your-github-client-id",
  "githubClientSecret": "your-github-client-secret",
  "githubRedirectUri": "https://api.bestblogs.dev/api/auth/oauth/github/callback"
}
```

**方式二：环境变量配置（后备）**

```properties
bestblogs.oauth.github.client-id=your-github-client-id
bestblogs.oauth.github.client-secret=your-github-client-secret
bestblogs.oauth.github.redirect-uri=https://api.bestblogs.dev/api/auth/oauth/github/callback
```

### 2.3 通用 OAuth 配置

```json
{
  "frontendCallbackUrl": "https://www.bestblogs.dev/auth/callback",
  "passwordResetTokenValidityMinutes": 60
}
```

### 2.4 管理后台 API

| 接口 | 方法 | 说明 |
|------|------|------|
| `/api/admin/parameter/oauth` | GET | 获取 OAuth 配置 |
| `/api/admin/parameter/oauth` | POST | 保存 OAuth 配置 |

---

## 三、API 端点说明

### OAuth 相关

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/auth/oauth/{provider}/authorize` | GET | 获取 OAuth 授权 URL (provider: google/github) |
| `/api/auth/oauth/{provider}/callback` | GET | OAuth 回调处理 |

### 密码重置相关

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/user/password/forgot` | POST | 发送密码重置邮件 |
| `/api/user/password/reset/validate` | GET | 验证重置 token |
| `/api/user/password/reset` | POST | 重置密码 |
| `/api/user/password/set` | POST | 设置密码（需登录） |

---

## 四、Header 用户信息与设置页面

### 4.1 Header 用户信息组件

**位置**: Header 右上角

**功能说明**:
- **未登录状态**: 显示"登录"按钮，点击跳转到登录页
- **已登录状态**: 显示用户头像
  - 头像优先使用 OAuth 提供商头像（`avatarUrl`）
  - 无头像时显示用户名首字母缩写

**下拉菜单内容**:
- 用户显示名称（`displayName` 或 `userName`）
- 用户邮箱
- 设置链接（跳转 `/settings`）
- 退出登录

### 4.2 用户设置页面

**路径**: `/settings`

**页面结构**:

```
用户设置页面
├── 页面标题（账户设置）
├── 个人资料卡片
│   ├── 用户头像
│   ├── 显示名称
│   ├── 邮箱
│   └── OAuth 关联信息（如：已关联 Google 账号）
├── 密码设置卡片
│   ├── 新密码输入框
│   ├── 确认密码输入框
│   └── 设置/修改密码按钮
├── 订阅管理卡片
│   ├── 每周精选推送说明
│   └── 管理订阅按钮（跳转 /manage-subscription）
└── 账户操作卡片
    └── 退出登录按钮
```

**设计规范符合度**:
- ✅ 国际化：使用 `useTranslations('Settings')`
- ✅ 暗黑模式：所有组件支持 `dark:` 变体
- ✅ 响应式设计：移动端和桌面端适配
- ✅ 颜色系统：使用 ink、paper、slate 设计令牌
- ✅ 字体排版：页面标题使用 `font-heading` 衬线字体

---

## 五、测试场景清单

### 5.1 Google OAuth 登录测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| G1 | 新用户首次 Google 登录 | 创建新账户，自动登录，跳转首页 | [ ] |
| G2 | 已有账户（邮箱相同）Google 登录 | 关联 OAuth 信息，自动登录 | [ ] |
| G3 | 已登录用户再次点击 Google 登录 | 正常登录，保持会话 | [ ] |
| G4 | Google 授权后取消 | 返回登录页，显示错误提示 | [ ] |
| G5 | Google 配置未设置时点击登录 | 显示"提供商未配置"错误 | [ ] |
| G6 | 中文界面 Google 登录 | 所有文案显示中文 | [ ] |
| G7 | 英文界面 Google 登录 | 所有文案显示英文 | [ ] |

### 5.2 GitHub OAuth 登录测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| H1 | 新用户首次 GitHub 登录 | 创建新账户，自动登录，跳转首页 | [ ] |
| H2 | 已有账户（邮箱相同）GitHub 登录 | 关联 OAuth 信息，自动登录 | [ ] |
| H3 | GitHub 邮箱私有时登录 | 获取主邮箱，正常登录 | [ ] |
| H4 | GitHub 授权后取消 | 返回登录页，显示错误提示 | [ ] |
| H5 | GitHub 配置未设置时点击登录 | 显示"提供商未配置"错误 | [ ] |
| H6 | 中文界面 GitHub 登录 | 所有文案显示中文 | [ ] |
| H7 | 英文界面 GitHub 登录 | 所有文案显示英文 | [ ] |

### 5.3 忘记密码测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| F1 | 已注册邮箱请求重置 | 发送重置邮件，显示成功提示 | [ ] |
| F2 | 未注册邮箱请求重置 | 显示成功提示（安全考虑不泄露信息） | [ ] |
| F3 | 邮箱格式错误 | 前端验证提示格式错误 | [ ] |
| F4 | 中文界面发送重置邮件 | 邮件内容为中文 | [ ] |
| F5 | 英文界面发送重置邮件 | 邮件内容为英文 | [ ] |
| F6 | 重复请求重置（1分钟内） | 正常发送（或可添加频率限制） | [ ] |

### 5.4 重置密码测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| R1 | 有效 token 重置密码 | 密码重置成功，自动登录 | [ ] |
| R2 | 过期 token 重置密码 | 显示"链接已过期"错误 | [ ] |
| R3 | 无效 token 重置密码 | 显示"无效链接"错误 | [ ] |
| R4 | 已使用 token 再次重置 | 显示"链接已失效"错误 | [ ] |
| R5 | 密码少于6位 | 前端验证提示密码太短 | [ ] |
| R6 | 两次密码不一致 | 前端验证提示密码不匹配 | [ ] |
| R7 | 未验证用户重置密码 | 重置成功，用户状态变为已验证 | [ ] |

### 5.5 设置密码测试（OAuth 用户）

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| S1 | OAuth 用户首次设置密码 | 密码设置成功 | [ ] |
| S2 | 已有密码用户设置密码 | 密码更新成功 | [ ] |
| S3 | 未登录访问设置密码 | 跳转登录页 | [ ] |
| S4 | 密码少于6位 | 前端验证提示密码太短 | [ ] |

### 5.6 账户关联测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| L1 | 本地注册后 Google 登录（相同邮箱） | 关联成功，可两种方式登录 | [ ] |
| L2 | Google 注册后本地登录 | 设置密码后可本地登录 | [ ] |
| L3 | Google 登录后 GitHub 登录（相同邮箱） | 同一账户，关联两个 OAuth | [ ] |
| L4 | 未验证用户 OAuth 登录 | 自动标记为已验证 | [ ] |

### 5.7 操作日志测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| O1 | Google 登录 | 记录 OAUTH_LOGIN 日志 | [ ] |
| O2 | GitHub 登录 | 记录 OAUTH_LOGIN 日志 | [ ] |
| O3 | 重置密码 | 记录 PASSWORD_RESET 日志 | [ ] |
| O4 | 设置密码 | 记录 PASSWORD_SET 日志 | [ ] |

### 5.8 管理后台配置测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| A1 | 获取 OAuth 配置 | 返回当前配置值 | [ ] |
| A2 | 保存 Google 配置 | 配置生效，可用 Google 登录 | [ ] |
| A3 | 保存 GitHub 配置 | 配置生效，可用 GitHub 登录 | [ ] |
| A4 | 清空配置后登录 | 显示"提供商未配置"错误 | [ ] |

### 5.9 边界与异常测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| E1 | OAuth state 过期（10分钟后回调） | 显示登录失败错误 | [ ] |
| E2 | OAuth state 重复使用 | 显示登录失败错误 | [ ] |
| E3 | 网络中断时 OAuth 回调 | 显示网络错误提示 | [ ] |
| E4 | 邮箱为空的 OAuth 用户 | 提示需要邮箱权限 | [ ] |
| E5 | 刷新重置密码页面 | token 仍然有效，可继续操作 | [ ] |

### 5.10 Header 用户信息测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| U1 | 未登录访问首页 | Header 显示"登录"按钮 | [ ] |
| U2 | 已登录访问首页 | Header 显示用户头像 | [ ] |
| U3 | 点击头像显示下拉菜单 | 显示用户名、邮箱、设置、退出 | [ ] |
| U4 | 点击"设置"链接 | 跳转到 /settings 页面 | [ ] |
| U5 | 点击"退出登录" | 清除会话，跳转首页 | [ ] |
| U6 | OAuth 用户显示头像 | 显示 OAuth 提供商头像 | [ ] |
| U7 | 无头像用户显示 | 显示名称首字母缩写 | [ ] |

### 5.11 设置页面测试

| 序号 | 测试场景 | 预期结果 | 通过 |
|------|----------|----------|------|
| P1 | 未登录访问设置页 | 跳转登录页 | [ ] |
| P2 | 已登录访问设置页 | 显示个人资料信息 | [ ] |
| P3 | OAuth 用户查看资料 | 显示 OAuth 关联信息 | [ ] |
| P4 | 设置密码成功 | 显示成功提示，hasPassword 更新 | [ ] |
| P5 | 密码太短（<6位） | 显示错误提示 | [ ] |
| P6 | 两次密码不一致 | 显示错误提示 | [ ] |
| P7 | 点击管理订阅 | 跳转到订阅管理页 | [ ] |
| P8 | 点击退出登录 | 清除会话，跳转首页 | [ ] |
| P9 | 中文界面设置页 | 所有文案显示中文 | [ ] |
| P10 | 英文界面设置页 | 所有文案显示英文 | [ ] |
| P11 | 暗黑模式设置页 | 所有组件正确适配暗黑模式 | [ ] |

---

## 六、注意事项

1. **HTTPS 要求**: Google OAuth 生产环境必须使用 HTTPS
2. **邮箱验证**: OAuth 登录的用户自动标记为邮箱已验证
3. **同一邮箱**: 不同 OAuth 提供商使用相同邮箱会关联到同一账户
4. **Token 有效期**:
   - 密码重置 Token 默认 60 分钟过期
   - OAuth State 10 分钟过期
5. **配置优先级**: 数据库配置 > 环境变量配置

---

## 七、技术实现要点

### 7.1 OAuth2 授权码流程

```
1. 用户点击 Google/GitHub 登录
2. 前端调用 /api/auth/oauth/{provider}/authorize
3. 后端生成 state 并返回授权 URL
4. 用户跳转到 Google/GitHub 授权页面
5. 用户授权后回调 /api/auth/oauth/{provider}/callback
6. 后端验证 state，用授权码换 token
7. 获取用户信息，创建/关联用户
8. 生成 JWT token，重定向到前端回调页
9. 前端保存 token，跳转首页
```

### 7.2 密码安全

- 密码使用 BCrypt 哈希存储
- HTTPS 传输保护
- 密码重置 Token 一次性使用
- 不泄露邮箱是否注册的信息

### 7.3 配置动态化

OAuth 配置支持：
1. 管理后台动态修改（数据库存储）
2. 环境变量作为后备
3. 修改后自动刷新缓存

---

*文档创建日期: 2024-12-30*
*最后更新: 2025-12-31*
