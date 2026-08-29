# Gmail MCP Server - DSH 配置指南

## 📋 前置条件

1. **Node.js 14+** 已安装
2. **Google Cloud Console 账号**
3. **DSH (DeepSeek Harness)** 已安装

## 🚀 快速设置

### 1. 获取 Google OAuth 凭证

1. 访问 [Google Cloud Console](https://console.cloud.google.com/)
2. 创建新项目或选择现有项目
3. 启用 Gmail API：
   - 访问 [Gmail API](https://console.cloud.google.com/apis/library/gmail.googleapis.com)
   - 点击 "Enable"
4. 配置 OAuth 同意屏幕：
   - 访问 [OAuth 同意屏幕](https://console.cloud.google.com/apis/credentials/consent)
   - 选择 "External" 用户类型
   - 添加你的邮箱作为测试用户
   - 添加 Gmail 相关的 scopes
5. 创建 OAuth 2.0 凭证：
   - 访问 [Credentials](https://console.cloud.google.com/apis/credentials)
   - 点击 "Create Credentials" -> "OAuth client ID"
   - 选择 "Desktop app" 作为应用类型
   - 下载 JSON 凭证文件

### 2. 配置环境变量

创建 `.env` 文件（参考 `.env.example`）：

```bash
cd D:\Projects\Gmail-MCP-Server
cp .env.example .env
```

编辑 `.env` 文件，填入你的凭证：

```
GOOGLE_CLIENT_ID=你的Client ID
GOOGLE_CLIENT_SECRET=你的Client Secret
GOOGLE_REDIRECT_URI=http://localhost:3000/oauth2callback
```

### 3. 初次认证

运行以下命令进行 Google OAuth 认证：

```bash
cd D:\Projects\Gmail-MCP-Server
npm run auth
```

这将：
1. 启动本地服务器
2. 打开浏览器进行 Google 登录
3. 授权 Gmail 访问权限
4. 保存 tokens.json 到项目目录

### 4. 配置 DSH

#### 方式1：使用现有配置文件

已创建 `D:\Projects\TimeSpaceExchange\gmail-mcp.cordis.yml`，DSH 会自动加载。

#### 方式2：手动配置

在 DSH 的 agent preset 或 host composition 中添加：

```yaml
- insert:
    - id: gmail-mcp
      name: '@deepseek-ai/dsh-mcp-client'
      config:
        serverName: gmail
        transport: stdio
        command: node
        args:
          - D:\Projects\Gmail-MCP-Server\dist\index.js
        env:
          GOOGLE_CLIENT_ID: 你的Client ID
          GOOGLE_CLIENT_SECRET: 你的Client Secret
          GOOGLE_REDIRECT_URI: http://localhost:3000/oauth2callback
          TOKEN_PATH: D:\Projects\Gmail-MCP-Server\tokens.json
```

### 5. 重启 DSH

重启 DSH 以加载新的 MCP 服务器。

## 🔧 可用工具

配置完成后，你将可以使用以下 Gmail 工具：

### 邮件操作
- `search_emails` - 搜索邮件
- `get_email` - 获取邮件详情
- `send_email` - 发送邮件
- `draft_email` - 创建草稿
- `reply_to_email` - 回复邮件
- `forward_email` - 转发邮件

### 标签管理
- `list_labels` - 列出所有标签
- `create_label` - 创建新标签
- `update_label` - 更新标签
- `delete_label` - 删除标签
- `manage_label` - 应用/移除标签

### 过滤器
- `create_filter` - 创建过滤器
- `list_filters` - 列出过滤器
- `delete_filter` - 删除过滤器

### 附件
- `download_attachment` - 下载附件
- `list_attachments` - 列出附件

### 线程
- `get_thread` - 获取邮件线程
- `list_threads` - 列出线程

## 📧 邮件整理示例

### 自动分类订阅邮件

```json
{
  "template": "fromSender",
  "parameters": {
    "senderEmail": "newsletter@example.com",
    "labelIds": ["Label_订阅"],
    "archive": true
  }
}
```

### 账单自动标记

```json
{
  "template": "containingText",
  "parameters": {
    "searchText": "invoice",
    "labelIds": ["Label_账单"],
    "markImportant": true
  }
}
```

### 大附件邮件归类

```json
{
  "template": "largeEmails",
  "parameters": {
    "sizeInBytes": 10485760,
    "labelIds": ["Label_大文件"]
  }
}
```

## 🔍 高级搜索语法

Gmail MCP 支持 Gmail 的搜索语法：

| 操作符 | 示例 | 说明 |
|--------|------|------|
| `from:` | `from:john@example.com` | 特定发件人 |
| `to:` | `to:mary@example.com` | 特定收件人 |
| `subject:` | `subject:"meeting notes"` | 主题包含 |
| `has:attachment` | `has:attachment` | 有附件 |
| `after:` | `after:2024/01/01` | 指定日期后 |
| `before:` | `before:2024/02/01` | 指定日期前 |
| `is:` | `is:unread` | 未读邮件 |
| `label:` | `label:work` | 特定标签 |

组合使用：`from:john@example.com after:2024/01/01 has:attachment`

## ❓ 故障排除

### 认证失败
- 检查 Google Cloud Console 中的 OAuth 配置
- 确保已启用 Gmail API
- 确保已添加测试用户

### MCP 服务器无法启动
- 检查 Node.js 版本（需要 14+）
- 确保已运行 `npm install`
- 检查 dist/index.js 是否存在

### 工具未显示
- 重启 DSH
- 检查 DSH 日志中的错误信息
- 验证 cordis.yml 配置

## 📚 更多资源

- [Gmail API 文档](https://developers.google.com/gmail/api)
- [Gmail 搜索语法](https://support.google.com/mail/answer/7126229)
- [MCP 协议](https://modelcontextprotocol.io/)
- [DSH 文档](https://github.com/DeepSeek-ai/deepseek-harness)
