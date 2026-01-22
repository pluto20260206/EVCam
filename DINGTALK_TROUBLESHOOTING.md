# 钉钉 Stream 模式配置检查清单

## 问题诊断

你遇到的 **503 错误** 通常是因为以下原因之一：

### 1. Stream 模式未正确开启

**检查步骤：**
1. 登录 [钉钉开放平台](https://open-dev.dingtalk.com/)
2. 进入你的应用 → 开发配置 → 消息推送
3. 确认选择的是 **「Stream 模式」**（不是 HTTP 推送）
4. 如果是 HTTP 推送，切换为 Stream 模式并保存

### 2. 凭证信息混淆

钉钉有两组不同的凭证：

| 凭证类型 | 用途 | 在哪里找 |
|---------|------|---------|
| AppKey + AppSecret | 获取 Access Token | 应用详情 → 基础信息 |
| ClientId + ClientSecret | Stream 连接 | 应用详情 → 开发配置 → Stream 模式 |

**常见错误：**
- ❌ 把 AppKey 填到 ClientId
- ❌ 把 AppSecret 填到 ClientSecret
- ✅ 正确：四个字段都要填，且不能重复

### 3. 应用未发布或权限不足

**检查步骤：**
1. 应用详情 → 版本管理与发布
2. 确认应用状态：
   - 开发中：只有开发者可以使用
   - 已发布：企业内所有人可以使用
3. 应用详情 → 权限管理
4. 确认已开启以下权限：
   - ✅ 企业内机器人发送消息
   - ✅ 机器人消息接收
   - ✅ 上传媒体文件

### 4. 网络问题

**测试方法：**
```bash
# 在设备上测试网络连通性
ping api.dingtalk.com

# 或在浏览器访问
https://api.dingtalk.com
```

如果无法访问，检查：
- WiFi/移动网络是否正常
- 是否有防火墙阻止
- 是否需要配置代理

## 正确的配置流程

### 步骤 1：创建应用
1. 登录钉钉开放平台
2. 创建企业内部应用
3. 记录 **AppKey** 和 **AppSecret**

### 步骤 2：开启机器人
1. 应用详情 → 应用功能 → 机器人
2. 点击「开启」
3. 配置机器人信息（名称、头像、描述）

### 步骤 3：配置 Stream 模式
1. 应用详情 → 开发配置 → 消息推送
2. 选择 **「Stream 模式」**
3. 记录 **ClientId** 和 **ClientSecret**
4. 点击保存

### 步骤 4：配置权限
1. 应用详情 → 权限管理
2. 开启以下权限：
   - 企业内机器人发送消息
   - 机器人消息接收
   - 上传媒体文件
3. 点击保存

### 步骤 5：发布应用（可选）
1. 应用详情 → 版本管理与发布
2. 创建版本并发布
3. 或保持「开发中」状态（仅开发者可用）

### 步骤 6：添加机器人到群聊
1. 打开钉钉群聊
2. 群设置 → 群机器人 → 添加机器人
3. 选择你创建的机器人
4. 点击添加

### 步骤 7：在应用中配置
1. 打开 Android 应用
2. 菜单 → 远程查看
3. 填入四个凭证：
   ```
   AppKey: ding...
   AppSecret: ...
   ClientId: ...
   ClientSecret: ...
   ```
4. 保存配置 → 启动服务

## 验证配置

### 测试 1：检查 Access Token
查看应用日志，应该看到：
```
DingTalkApiClient: 正在获取新的 Access Token...
DingTalkApiClient: Access Token 获取成功
```

如果失败，说明 AppKey 或 AppSecret 错误。

### 测试 2：检查 Stream 连接
查看应用日志，应该看到：
```
DingTalkApiClient: Stream 请求: {...}
DingTalkApiClient: Stream 响应: {...}
DingTalkStreamClient: WebSocket 连接已建立
```

如果看到 503 错误，说明：
- Stream 模式未开启，或
- ClientId/ClientSecret 错误

### 测试 3：发送测试消息
1. 在钉钉群聊中 @机器人
2. 发送「录制」
3. 应该看到机器人回复：「收到录制指令，开始录制 1 分钟视频...」

## 常见错误代码

| 错误码 | 含义 | 解决方法 |
|-------|------|---------|
| 503 | Stream 端点不可用 | 检查 Stream 模式是否开启，ClientId/ClientSecret 是否正确 |
| 401 | 认证失败 | 检查 AppKey/AppSecret 是否正确 |
| 403 | 权限不足 | 检查应用权限配置 |
| 404 | 端点不存在 | 检查 API 地址是否正确 |

## 调试技巧

### 1. 查看完整日志
在应用中：
1. 点击「日志」展开日志面板
2. 查看所有 `DingTalk*` 开头的日志
3. 特别注意错误信息

### 2. 使用 Postman 测试
测试 Access Token 获取：
```
POST https://api.dingtalk.com/v1.0/oauth2/accessToken
Content-Type: application/json

{
  "appKey": "your_app_key",
  "appSecret": "your_app_secret"
}
```

测试 Stream 端点：
```
POST https://api.dingtalk.com/v1.0/gateway/connections/open
Content-Type: application/json

{
  "clientId": "your_client_id",
  "clientSecret": "your_client_secret",
  "subscriptions": [
    {
      "type": "CALLBACK",
      "topic": "*"
    }
  ]
}
```

### 3. 联系钉钉支持
如果以上都无法解决，可以：
1. 访问 [钉钉开放平台文档](https://open.dingtalk.com/document/)
2. 加入钉钉开发者社区
3. 提交工单获取技术支持

## 参考资料

- [钉钉 Stream 模式文档](https://open.dingtalk.com/document/orgapp/stream-mode-overview)
- [钉钉机器人开发指南](https://open.dingtalk.com/document/orgapp/robot-overview)
- [钉钉 API 参考](https://open.dingtalk.com/document/orgapp/api-overview)
