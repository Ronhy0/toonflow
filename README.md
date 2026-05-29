 |   | 

**中文** | [English](./docs/README.en.md)



# 知剧AI

**AI 短剧创作平台 **  
**小说秒转剧本，剧本延展分镜 **  
**前后端与桌面端一体化交付 🔥**

 🚀 **一站式短剧工程**：AI 剧本生成 × 小说转剧本 × 分镜辅助 × 管理后台 × 桌面端封装，全流程覆盖！

  


---

# 🌟 主要功能

知剧AI 是一个面向短剧创作与管理的源码开放项目，提供完整的前端站点、后端 API、管理后台和 Electron 桌面端封装，适合个人学习、研究、非商业本地部署和功能体验。

- ✅ **AI 剧本生成**  
 支持从创意到结构化剧本的生成与编辑，帮助快速搭建短剧内容骨架。
- ✅ **小说转剧本**  
 支持小说项目、章节、剧情线到剧本的转换流程，适合影视改编类创作场景。
- ✅ **分镜辅助**  
 围绕剧本场景生成分镜、封面和画面素材，提升后续视频制作效率。
- ✅ **桌面端封装**  
 支持 Electron 联调与安装包构建，可输出 Windows / macOS 桌面客户端。

---

# 📦 应用场景

- AI 短剧创作平台
- 小说影视化改编工具
- 私有化内容生产系统
- 桌面端短剧创作工作台
- 面向运营团队的后台管理系统
- 学习和研究型项目参考

---

# 🔰 使用指南

## 📺 视频教程









**知剧AI 快速上手教程**
👉 [点击观看（待补充）](#)





---

# 🚀 安装

## 前置条件

在安装和使用本软件之前，请准备以下内容：

- ✅ Node.js：README 最低为 **18**；当前前端（Nuxt 3.21+）的 `engines` 要求 **`^20.19.0` 或 `>=22.12.0`**，建议使用 **Node 22 LTS** 以免 `npm ci` / `nuxt prepare` 失败。
- ✅ npm `>= 9`（随 Node 发行版附带即可）

## 本机安装

### 1. 下载与安装


| 操作系统    | GitHub 下载                                                 | 网盘下载           | 说明      |
| ------- | --------------------------------------------------------- | -------------- | ------- |
| Windows | [Release](https://github.com/openfrees/toonflow/releases) | [网盘下载（待补充）](#) | 官方发布安装包 |
| macOS   | [Release](https://github.com/openfrees/toonflow/releases) | [网盘下载（待补充）](#) | 官方发布安装包 |
| Linux   | [Release](https://github.com/openfrees/toonflow/releases) | [网盘下载（待补充）](#) | 待提供     |


> 当前仓库已支持 Windows 和 macOS 打包。Release 页地址、网盘地址请后续补上。

### 2. 启动服务

安装完成后，启动程序即可开始使用本服务。

> ⚠️ **首次登录**  
> 账号：`手机号`　｜　密码：`随便设置`  
> 没有任何验证本地登录，所以手机密码随便设置

> 📌 **使用 AI 剧本 / 小说转剧本 / 分镜与配图等功能前**，必须在个人中心完成「模型设置」并绑定场景，详见下文 [大模型与 API Key 配置](#大模型与-api-key-配置)。

## 大模型与 API Key 配置

本项目的 AI 能力（对话、剧本生成、小说 Agent、分镜与封面等）**依赖真实的大模型 HTTP API**。仓库**不包含**任何厂商自带的 API Key，也**不在** `.env` 中预置可开箱即用的 LLM 密钥；你需要自行准备 Key，并在应用内完成配置。

### 工作原理概览

1. **按用户存储**：每个登录用户在「模型设置」里添加一条或多条模型配置（文字模型 `text`、图片模型 `image`），其中 **API Key、Base URL、模型 ID** 等为必填项。
2. **场景绑定**：仅有模型记录不够，还需要把模型绑定到具体业务场景（例如「剧本生成 / 文字」对应 `script_gen`，「生图」对应 `image_gen`）。未绑定或解密失败时，相关接口会返回提示，要求先到模型设置中配置。
3. **调用时组装配置**：后端从数据库读取你的模型与绑定关系，解密 API Key 后，组装成与内部 `ai_chat` 模块一致的结构，再通过 [Vercel AI SDK](https://ai-sdk.dev/)（`@ai-sdk/deepseek` / `@ai-sdk/openai`）发起请求。
4. **与「加密主密钥」的区别**：`storyweaver-api/config/config.default.js` 中的 `config.encryption.secret` 用于 **AES 加密存储在数据库里的用户模型 API Key**，不是大模型厂商的 Key。私有化部署时请改为强随机串，并勿提交到公开仓库。

### 在前端如何操作

1. 使用本地或 Docker 方式启动服务并完成登录（见上文「首次登录」）。
2. 进入 **个人中心 → 模型设置**（对应前端页面 `storyweaver-web/pages/user/model-config.vue`）。
3. **新增模型**：选择类型（文字 / 图片），填写名称、**API Key**、**Base URL**、**模型 ID**（Model），可按需调整 `max_tokens`、温度等。
4. **场景绑定**：在界面中将「剧本生成 / 文字」绑定到你的文字模型，将「配图 / 生图」等绑定到图片模型（具体文案以界面为准）。保存后，各 AI 功能才会使用该 Key 调用对应服务商。

### 填写说明（文字模型）

| 字段 | 说明 |
| --- | --- |
| **API Key** | 服务商控制台发放的密钥，服务端会加密后写入数据库。 |
| **Base URL** | 兼容 OpenAI API 的网关地址（须带协议，如 `https://api.deepseek.com` 或自建代理根地址）。 |
| **模型 ID** | 服务商文档中的模型名（如 `deepseek-chat`、`gpt-4o` 等）。 |

后端对「提供商标识」的处理逻辑简述：**名称为 `deepseek` 的提供商**会走 DeepSeek 官方 SDK 路径；**其它标识**会走 **OpenAI 兼容** 客户端（`createOpenAI` + 你填写的 `baseURL`），便于对接 One API、Azure OpenAI 兼容网关、本地 vLLM 等。

### 填写说明（图片模型）

除 Key、Base URL、模型名外，还可通过扩展字段配置尺寸等（具体以模型设置页为准）。生图相关接口会从 `image_gen` 场景绑定中读取解密后的 Key 与参数。

### 后端与接口（二次开发参考）

- **控制器**：`storyweaver-api/app/controller/api/modelConfig.js`（列表、创建、更新、删除、测试连接、场景绑定等）。
- **业务服务**：`storyweaver-api/app/service/api/modelConfig.js`（含 `getEffectiveAiConfig`：按用户与场景返回有效配置或 `null`）。
- **数据表**：`user_model`（模型与加密后的 `api_key`）、`user_model_scene`（用户与场景、模型 ID 的绑定）。模型定义见 `storyweaver-api/app/model/user_model.js`。
- **实际请求**：`storyweaver-api/app/lib/ai_chat.js`（根据配置选择 DeepSeek 或 OpenAI 兼容通道）。

业务控制器（如剧本生成 `scriptGenerate.js`）在调用 AI 前会取 `getEffectiveAiConfig`；若返回空，会提示 **「请先在模型设置中配置并绑定文字模型」**（错误码等业务细节以接口为准）。

### 安全建议

- 生产环境务必修改默认的 `config.encryption.secret` 与 JWT、数据库等敏感配置（通过 `config.local.js` / 环境变量等你已在用的 Egg 配置方式覆盖，勿把真实密钥写入 Git）。
- API Key 仅保存在服务端数据库中且为密文；前端编辑时按掩码展示，请勿在浏览器控制台或日志中明文打印 Key。

### 关于「全站默认 Key」

当前开源版本的设计是 **「谁使用谁配置自己的 Key」**，代码路径中**没有**「未绑定时自动回退到环境变量里的平台统一 Key」的逻辑。若你希望企业内网统一部署一套默认模型，需要在后端增加一层配置（例如从环境变量组装默认 `ai` 配置），并在 `getEffectiveAiConfig` 返回 `null` 时合并该默认项；这属于二次开发扩展，本 README 不展开实现细节。

## Docker 部署

### 前置条件

- 已安装 [Docker](https://docs.docker.com/get-docker/)（版本 20.10+）
- 已安装 [Docker Compose](https://docs.docker.com/compose/install/)（版本 2.0+）

### 本地 Docker 一键运行

本仓库已经内置根目录 `Dockerfile` 和 `docker-compose.yml`，可直接在本机通过 Docker 跑起前后端。

当前 Docker 方案特点：

- 采用 `localhost` 模式启动后端，使用 SQLite 持久化数据
- 前端在镜像构建阶段静态生成，并拷贝到 `storyweaver-api/web`
- 前台页面、后端 API、管理后台统一走同一个地址

### 基础镜像（Docker Hub 拉取失败时）

构建默认使用 **AWS Public ECR** 上的官方 Node 镜像（`public.ecr.aws/docker/library/node:20-bookworm-slim`），与 Docker Hub `library/node` 同源，可避免访问 `auth.docker.io` 时出现 `connection reset` / token 失败。

若你希望仍从 Docker Hub 拉取，在构建前执行：

```shell
export TOONFLOW_NODE_IMAGE=node:20-bookworm-slim
docker compose up -d --build
```

### 运行前说明

`docker-compose.yml` 位于项目根目录，所以命令必须在**仓库根目录**执行：

```shell
cd /你的项目根目录
```

### 启动命令

```shell
# 1. 构建镜像并后台启动容器
# 首次启动或代码更新后，优先用这个命令
docker compose up -d --build

# 2. 查看容器运行状态
docker compose ps

# 3. 持续查看应用日志
# 适合排查启动失败、接口报错、数据库初始化等问题
docker compose logs -f storyweaver-app

# 4. 停止并移除容器
docker compose down
```

### 访问地址

启动成功后可直接访问：

- 前台首页：`http://127.0.0.1:7006/`
- 后端健康检查：`http://127.0.0.1:7006/api/health`
- 管理后台：`http://127.0.0.1:7006/admin`

### 数据持久化

Docker 运行产生的数据默认保存在宿主机以下目录：

```shell
./docker-data/storyweaver-api/data
./docker-data/storyweaver-api/logs
./docker-data/storyweaver-api/run
```

其中：

- `data`：SQLite 数据库文件
- `logs`：后端运行日志
- `run`：运行时文件

### 服务端口说明


| 端口     | 用途        | 说明            |
| ------ | --------- | ------------- |
| `7006` | 后端 API 服务 | Egg.js 默认开发端口 |
| `7005` | 前端开发服务    | Nuxt 3 默认开发端口 |


### 数据持久化

当前 Compose 已默认挂载以下目录：

```yaml
volumes:
  - ./docker-data/storyweaver-api/data:/app/storyweaver-api/data
  - ./docker-data/storyweaver-api/logs:/app/storyweaver-api/logs
  - ./docker-data/storyweaver-api/run:/app/storyweaver-api/run
```

### 常用操作命令

```shell
# 重新构建并启动（更新版本时使用）
docker compose up -d --build

# 仅启动已有镜像（代码未变时使用）
docker compose up -d

# 查看容器状态
docker compose ps

# 查看实时日志
docker compose logs -f storyweaver-app

# 停止服务
docker compose down

# 进入容器调试
docker exec -it storyweaver-app sh
```

> ⚠️ 首次登录信息见 [本机安装 - 启动服务](#2-启动服务)

---

# 🔧 开发流程指南

> [!CAUTION]
> 🚧 **PR 提交规范** 🚧
>
>
>
> 请将 PR 提交到指定开发分支（待补充具体分支名）

## 开发环境准备

- **Node.js**：18 为历史最低说明；**实际开发与安装依赖请使用 Node 20.19+ 或 22 LTS**（与 Nuxt / Vite 工具链一致）。
- **npm**：推荐随 Node 自带的当前主版本。
- **Linux 仅跑 Electron 桌面端时**：需系统图形栈相关库（如 `libatk`、`gtk3` 等）；无图形环境可只跑 `storyweaver-api` + `storyweaver-web` 或 Docker。

## 快速启动项目

1. **克隆项目**
  **从 GitHub 克隆：**
   **从 Gitee 克隆（国内推荐）：**
2. **安装依赖**
  请先在项目根目录下执行以下命令以安装依赖项：
3. **启动开发环境**
  本项目包含 **后端 API 服务** 和 **前端页面** 两部分，请根据需要选择启动方式：
  - **方式一：仅启动后端服务（开发调试用）**
    ```bash
    cd storyweaver-api
    npm run dev
    ```
    > ⚠️ 此命令仅启动后端 API 服务，**不包含前端页面**。如需同时使用前端页面，请配合前端项目单独启动，或使用 Electron 桌面模式。
  - **方式二：启动前端页面**
    ```bash
    cd storyweaver-web
    npm run dev
    ```
  - **方式三：启动 Electron 桌面客户端（推荐完整体验）**
    ```bash
    cd electron
    npm run dev
    ```
    > 此命令会启动 Electron 桌面窗口，配合后端服务使用，开箱即用。适合想要完整体验所有功能的开发者。
   **三种模式对比：**

  | 命令                      | 启动内容         | 前端页面 | 适用场景             |
  | ----------------------- | ------------ | ---- | ---------------- |
  | `npm run dev`（api）      | 仅后端 API      | ❌ 无  | 后端开发调试、配合前端项目联调  |
  | `npm run dev`（web）      | 仅前端页面        | ✅ 有  | 前端开发调试           |
  | `npm run dev`（electron） | Electron 桌面端 | ✅ 内置 | 完整功能体验、桌面客户端开发调试 |

4. **项目打包**
  - 前端本地静态生成：
  - 前端生产环境构建：
    ```bash
    npm run build:prod
    ```
  - 打包为 Windows 平台可执行程序：
    ```bash
    cd electron
    npm run build:win
    ```
  - 打包为 Mac 平台可执行程序：
    ```bash
    cd electron
    npm run build:mac
    ```
  - 全部平台打包：
    ```bash
    cd electron
    npm run build:all
    ```
5. **代码质量检查**
  ```bash
   cd storyweaver-api
   npm run test
  ```

## 前端开发

前端代码位于 `storyweaver-web`，基于 Nuxt 3 + Vue 3 + Pinia + TypeScript 构建。

常用命令：


| 命令                       | 说明                        |
| ------------------------ | ------------------------- |
| `npm run dev`            | 本地开发（加载 `.env.localhost`） |
| `npm run generate:local` | 本地模式静态生成                  |
| `npm run generate:prod`  | 生产环境静态生成                  |
| `npm run build:prod`     | 生产环境 SSR 构建               |


## 项目结构

```
📂 storyweaver-web/              # Nuxt 3 前端
📂 storyweaver-api/              # Egg.js 后端与管理后台
│  ├─ 📂 app/
│  │  ├─ 📂 controller/          # 控制器
│  │  ├─ 📂 service/             # 业务逻辑
│  │  ├─ 📂 model/               # 数据模型
│  │  ├─ 📂 middleware/          # 中间件
│  │  ├─ 📂 view/                # Nunjucks 模板（管理后台）
│  │  └─ 📂 public/              # 静态资源
│  ├─ 📂 config/                 # 配置文件
│  └─ 📂 database/               # 数据库迁移
📂 electron/                     # Electron 桌面端封装
│  ├─ 📄 main.js                 # Electron 入口
│  ├─ 📂 scripts/                # 构建脚本
│  └─ 📂 icons/                  # 应用图标
📄 brand-mark-main.svg           # 品牌图标源文件
📄 LICENSE                       # 许可协议
📄 README.md                     # 项目说明
```

---

# 🔗 相关仓库


| 仓库                  | 说明              | GitHub                                              | Gitee                                             |
| ------------------- | --------------- | --------------------------------------------------- | ------------------------------------------------- |
| **知剧AI**            | 完整项目（本仓库，推荐）    | [GitHub](https://github.com/openfrees/toonflow.git) | [Gitee](https://gitee.com/open_free/toonflow.git) |
| **storyweaver-web** | 前端源代码（当前仓库子目录）  | 同上                                                  | 同上                                                |
| **storyweaver-api** | 后端源代码（当前仓库子目录）  | 同上                                                  | 同上                                                |
| **electron**        | 桌面端源代码（当前仓库子目录） | 同上                                                  | 同上                                                |


> 💡 **提示**：如果您只是想使用知剧AI，直接下载本仓库的客户端即可。如需二次开发或定制，请克隆完整仓库。

---

# 📝 开发计划

我们正持续优化产品，以下为近期开发重点：

1. 核心功能升级

- `🧩 AI 剧本生成增强` 支持更多 AI 模型接入，优化剧本结构与对白质量
- `📄 小说转剧本优化` 支持多格式文本导入，强化章节解析与剧情线梳理

1. 生产流程优化

- `👗 分镜管理增强` 强化分镜与剧本场景的关联，支持批量生成与调整
- `📦 批量处理/任务队列` 支持多章节同时处理，后台任务管理，进度实时监控

1. 部署与体验增强

- `🐳 Docker 部署模板` 提供开箱即用的 docker-compose.yml
- `📱 多端适配` 优化移动端体验，完善桌面端功能

---

# 👨‍👩‍👧‍👦 微信交流群







拉群小助手:



> 微信群二维码待补充，有了之后把上面注释取消即可。

---

# 💌 联系我们



📧 邮箱：[toms6688@foxmail.com](mailto:toms6688@foxmail.com?subject=知剧AI咨询)

---

# 📜 许可证

知剧AI 采用**个人学习与非商业研究许可协议**发布，许可证详情见 `[LICENSE](./LICENSE)`。

你可以在遵守许可协议全部条款的前提下：

- 获取、阅读、下载和保存本项目源码
- 为个人学习、技术研究、功能验证和非商业本地部署目的使用
- 在非商业前提下修改源码，并保留原始版权和许可声明

明确禁止：

- 将本项目或其衍生版本用于任何商业目的
- 将本项目用于收费服务、SaaS、云托管、代部署、代运营或技术外包交付
- 销售、转售、出租、授权、分发源码、安装包、镜像或二次开发版本

如需获得商业授权许可，请通过邮箱与我们联系。

---

# 🙏 致谢

感谢以下开源项目为知剧AI提供强大支持：

- [Nuxt](https://nuxt.com/) - 基于 Vue 的全栈框架
- [Vue.js](https://vuejs.org/) - 渐进式 JavaScript 框架
- [Egg.js](https://www.eggjs.org/) - 企业级 Node.js 框架
- [Sequelize](https://sequelize.org/) - Node.js ORM 框架
- [Electron](https://www.electronjs.org/) - 跨平台桌面应用开发框架
- [Pinia](https://pinia.vuejs.org/) - Vue 状态管理库
- [SQLite](https://www.sqlite.org/) - 轻量级嵌入式数据库
- [Sharp](https://sharp.pixelplumbing.com/) - 高性能 Node.js 图像处理库
- [AI SDK](https://ai-sdk.dev/) - 面向 TypeScript 的 AI 工具包
- [Zod](https://zod.dev/) - TypeScript 优先的模式验证库





##### copyright © 知剧AI

