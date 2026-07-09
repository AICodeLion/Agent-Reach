# 常见问题排查

## Claude / Cowork 提示无法安装

**症状：** 把安装文档发给 Claude、Cowork 或其他托管 Agent 后，Agent 回复说不能安装，常见理由包括：

- shell 在临时 Linux 沙盒里，不是你的 Windows/macOS 电脑
- `~/.agent-reach/` 会话结束后会消失
- 网络或平台策略禁止 `curl`、`yt-dlp`、浏览器自动化或 MCP 服务
- 无法访问你本机 Chrome/Edge 里的登录态

**原因：** Agent Reach 要安装在真正使用它的持久环境里。临时托管沙盒即使能执行 `pip install`，也只是装进沙盒，不会装到你的电脑；需要浏览器登录态的平台也无法读取你本机浏览器。

**解决方案：**

1. 在你的本机终端、持久化开发容器、远程开发 VM 或 VPS 里安装。
2. 如果使用 Agent，确认它的 shell 连接的是目标机器，而不是一次性的托管沙盒。
3. 如果平台策略禁止命令行联网或抓取网页，换用本机终端安装，或使用支持本机 shell 的 Agent。

可以把这句话发给 Agent：

```text
请先确认你的 shell 是否运行在我的本机或持久化服务器上。如果你在临时托管沙盒里，不要安装 Agent Reach；请告诉我需要在本机终端执行哪些命令。
```

---

## 雪球 / Xueqiu: API 返回 400

**症状：** `agent-reach doctor` 显示雪球 ⚠️，报 `HTTP Error 400`

**原因：** 雪球 API 需要登录 Cookie，无法通过匿名访问获取。

**解决方案：** 在 Chrome 里登录 xueqiu.com，然后运行：

```bash
agent-reach configure --from-browser chrome
```

再次运行 `agent-reach doctor` 确认恢复 ✅。Cookie 过期后重新运行即可。

---

## Twitter/X: twitter-cli 连接失败

**症状：** `twitter search` 或其他命令返回错误

**原因：** twitter-cli 需要 AUTH_TOKEN 和 CT0 环境变量才能访问 Twitter API。如果你的网络环境需要代理才能访问 x.com，需要配置代理。

**解决方案：**

### 方案 1：设置环境变量代理

```bash
export HTTP_PROXY="http://user:pass@host:port"
export HTTPS_PROXY="http://user:pass@host:port"
twitter search "test" -n 1
```

### 方案 2：使用全局代理工具

让代理工具接管所有网络流量，这样 twitter-cli 的请求也会走代理：

```bash
# macOS — ClashX / Surge 开启"增强模式"
# Linux — proxychains 或 tun2socks
proxychains twitter search "test" -n 1
```

### 方案 3：不用 twitter-cli，用 Exa 搜索替代

twitter-cli 不可用时，可以直接用 Exa 搜索 Twitter 内容：

```bash
mcporter call 'exa.web_search_exa(query: "site:x.com 搜索词", numResults: 5)'
```

### 方案 4：检查认证

```bash
twitter check
```

> 如果返回 "Missing credentials"，需要设置 AUTH_TOKEN 和 CT0 环境变量。
>
> **Fallback：** 如果你已经安装了 bird CLI（`npm install -g @steipete/bird`），它也能正常工作。Agent Reach 会自动检测已安装的工具。
