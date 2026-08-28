# 终极考核 AI 密钥安全说明（小米 MiMo）

本微课第 7 章「终极考核」的 AI 调用已切换至小米 MiMo（OpenAI 兼容端点），
第 2 章「AI 对话模拟练习」仍使用 DeepSeek。本文说明 MiMo 密钥的安全存储与调用方式。

## 一、密钥如何加载（三不原则）

- **不硬编码**：源码 `index.html` 中只含占位符 `__MIMO_API_KEY__`，绝无明文密钥。
- **不入仓库**：真实密钥只存放在本地 `.env` 文件中，已被 `.gitignore` 排除，禁止提交到 Git。
- **不打印输出**：构建脚本 `_build.js` 与页面代码的任何分支都不会把密钥输出到终端、日志或控制台。

加载链路：
```
.env / 环境变量 MIMO_API_KEY
   → node _build.js（hex 混淆注入）
   → index.html 中 _HEX2 = '<hex>'
   → 页面 _rK2() 解码 → createOpenAIClient({ apiKey, baseURL }) → Authorization: Bearer
```

## 二、安全存储建议

1. 密钥只写在本地 `.env`（与本目录同级），设置文件权限仅本人可读：
   - macOS/Linux：`chmod 600 .env`
   - Windows：文件属性 → 安全 → 仅当前用户
2. 服务器部署时，使用平台的环境变量功能注入 `MIMO_API_KEY`，不要把 `.env` 上传。
3. 更换/轮换密钥：在 MiMo 控制台重置后，更新 `.env` 并重新执行 `node _build.js`。

## 三、构建与部署

```bash
# 首次：准备密钥
cp .env.example .env        # 编辑 .env 填入真实密钥（sk- 开头）

# 每次修改 index.html 源码后重新构建
node _build.js              # 注入密钥，生成可部署版本
node _build.js --check      # 校验：无占位符残留、hex 可解码、无明文密钥

# 部署 index.html（静态托管即可，无需服务器）
```

## 四、安全边界（务必知悉）

- 前端 hex 混淆**仅增加提取难度，并非加密**：任何有浏览器的访问者理论上都能
  从页面源码中还原密钥。纯静态部署下密钥最终必然出现在客户端。
- 适用场景：内部培训、访问受限的环境；如对安全要求高，请改用**服务端代理**
  （由后端持有密钥转发请求，前端只存代理地址）。
- MiMo 端点如未放行浏览器跨域（CORS），直连会被浏览器拦截，需自建同源代理转发。

## 五、第 7 章调用要点

- 端点：`https://api.xiaomimimo.com/v1/chat/completions`
- 模型：`mimo-v2.5-pro`（请求自动关闭 thinking 以稳定 temperature）
- 认证：`Authorization: Bearer <key>`
- 评分使用 `response_format: { type: 'json_object' }` 结构化输出
