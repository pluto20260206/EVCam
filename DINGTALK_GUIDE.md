# 钉钉远程控制功能使用指南

## 功能概述

本应用新增了钉钉远程控制功能，允许你通过钉钉群聊远程控制设备录制视频。主要特性：

- 通过钉钉 @机器人 发送「录制」指令
- 自动录制 1 分钟视频
- 自动上传视频到钉钉群聊
- 无需公网 IP，使用钉钉 Stream 模式

## 配置步骤

### 1. 创建钉钉机器人应用

1. 登录 [钉钉开放平台](https://open-dev.dingtalk.com/)
2. 创建企业内部应用
3. 开启机器人能力
4. 配置 Stream 模式（重要！）
5. 记录以下信息：
   - AppKey
   - AppSecret
   - ClientId
   - ClientSecret

### 2. 在应用中配置

1. 打开应用，点击左上角菜单按钮 ☰
2. 选择「远程查看」
3. 填入钉钉应用的凭证信息：
   - AppKey
   - AppSecret
   - ClientId
   - ClientSecret
4. 点击「保存配置」
5. 点击「启动服务」

### 3. 使用远程录制

1. 确保服务已启动（连接状态显示「已连接」）
2. 在钉钉群聊中 @机器人
3. 发送「录制」指令
4. 设备将自动：
   - 开始录制 1 分钟视频
   - 录制完成后自动停止
   - 上传视频到钉钉群聊

## 注意事项

### 权限要求

- 摄像头权限
- 录音权限
- 存储权限
- 网络权限

### 网络要求

- 设备需要联网
- 建议使用 WiFi 或 4G/5G 网络
- 上传视频需要稳定的网络连接

### 钉钉配置要点

1. **必须使用 Stream 模式**：在钉钉开放平台配置机器人时，选择 Stream 模式而非 Webhook 模式
2. **机器人权限**：确保机器人有发送消息和上传文件的权限
3. **群聊设置**：将机器人添加到需要使用的钉钉群聊中

### 录制说明

- 录制时长固定为 1 分钟
- 录制所有已配置的摄像头（最多 4 路）
- 视频保存在 `/DCIM/MultiCam/` 目录
- 上传完成后视频仍保留在本地

## 故障排查

### 连接状态显示「未连接」或 503 错误

**503 错误通常表示钉钉 Stream 端点不可用，可能的原因：**

1. **检查钉钉应用配置**
   - 确认应用已开启 Stream 模式（不是 Webhook 模式）
   - 在钉钉开放平台 → 应用详情 → 开发配置 → 消息推送 → 选择「Stream 模式」
   - 确认 ClientId 和 ClientSecret 正确（不是 AppKey 和 AppSecret）

2. **检查凭证信息**
   - AppKey 和 AppSecret：用于获取 Access Token
   - ClientId 和 ClientSecret：用于建立 Stream 连接
   - 这是两组不同的凭证，不要混淆

3. **检查应用权限**
   - 确认应用已发布（或在开发环境中）
   - 确认应用有「机器人消息接收」权限
   - 确认应用有「企业内机器人发送消息」权限

4. **检查网络连接**
   - 确认设备可以访问 `api.dingtalk.com`
   - 尝试在浏览器访问 https://api.dingtalk.com 测试连通性
   - 检查是否有防火墙或代理阻止连接

5. **查看详细日志**
   - 打开应用日志面板（点击「日志」展开）
   - 查看 `DingTalkApiClient` 和 `DingTalkStreamClient` 的日志
   - 特别注意 Access Token 获取是否成功

### 连接状态一直显示「未连接」（非 503 错误）

1. 检查网络连接
2. 确认钉钉凭证信息正确
3. 确认钉钉应用已开启 Stream 模式
4. 查看应用日志面板的错误信息

### 收不到录制指令

1. 确保在钉钉群聊中 @机器人
2. 确认发送的指令是「录制」（不区分大小写）
3. 检查服务是否已启动
4. 查看连接状态是否为「已连接」

### 视频上传失败

1. 检查网络连接
2. 确认视频文件已生成（查看 `/DCIM/MultiCam/` 目录）
3. 检查钉钉应用是否有文件上传权限
4. 查看应用日志面板的错误信息

### 录制失败

1. 确认摄像头权限已授予
2. 确认摄像头未被其他应用占用
3. 返回「录制界面」检查摄像头预览是否正常
4. 查看应用日志面板的错误信息

## 技术架构

### 核心组件

- **DingTalkConfig**: 配置存储
- **DingTalkApiClient**: 钉钉 API 调用
- **DingTalkStreamClient**: WebSocket 长连接
- **DingTalkCommandReceiver**: 指令解析和处理
- **VideoUploadService**: 视频上传服务
- **RemoteViewFragment**: 配置界面

### 工作流程

```
钉钉群聊 (@机器人 + 「录制」)
    ↓
钉钉服务器 (Stream 推送)
    ↓
DingTalkStreamClient (WebSocket 接收)
    ↓
DingTalkCommandReceiver (解析指令)
    ↓
MainActivity.startRemoteRecording() (开始录制)
    ↓
MultiCameraManager (录制 1 分钟)
    ↓
VideoUploadService (上传视频)
    ↓
钉钉群聊 (显示视频文件)
```

## 开发者信息

### API 端点

- Access Token: `https://api.dingtalk.com/v1.0/oauth2/accessToken`
- Stream 连接: `https://api.dingtalk.com/v1.0/gateway/connections/open`
- 发送消息: `https://api.dingtalk.com/v1.0/robot/oToMessages/batchSend`
- 上传文件: `https://oapi.dingtalk.com/media/upload`

### 依赖库

- OkHttp 4.12.0 (HTTP 客户端和 WebSocket)
- Gson 2.10.1 (JSON 解析)

## 更新日志

### v0.3 (当前版本)

- 新增钉钉远程控制功能
- 支持远程录制 1 分钟视频
- 自动上传视频到钉钉群聊
- 新增「远程查看」配置界面

### v0.2

- 视频录制每 1 分钟分段存储
- 增加录制回看功能
- 修复多个 bug

### v0.1

- 实现基础的摄像头调用
- 视频预览、保存功能
