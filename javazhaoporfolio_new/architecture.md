# 个人品牌平台全栈架构文档

**版本**: 1.0
**日期**: 2025-07-20
**作者**: Winston (架构师)

---

### 1. 高层级架构 (High Level Architecture)

#### **技术摘要 (Technical Summary)**

本项目将构建为一个**解耦的 (Decoupled)** 全栈应用。前端将采用 **Next.js** 框架构建，并托管于 **Vercel** 平台以获得极致的性能和全球 CDN 加速。后端将采用 **Strapi** 作为 Headless CMS，运行在独立的服务器容器中，并通过 **PostgreSQL** 数据库持久化数据。前后端通过标准的 **REST API** 进行通信。整个项目将采用 **Monorepo** 结构进行代码管理，以提升开发效率和代码一致性。

#### **高层级架构图 (High Level Architecture Diagram)**

```mermaid
graph TD
    subgraph "用户端"
        User[👤 网站访客]
    end

    subgraph "前端 (Vercel)"
        Frontend[🌐 Next.js 前端应用<br>(SSG/ISR 渲染)]
    end

    subgraph "后端 (云服务器/容器)"
        Backend[🚀 Strapi Headless CMS<br>(REST API)]
    end

    subgraph "数据库 (云服务)"
        DB[(🐘 PostgreSQL 数据库)]
    end

    User -- HTTPS --> Frontend
    Frontend -- API 请求 --> Backend
    Backend -- 读/写 --> DB
```
### 2. 技术栈 (Tech Stack)

| 类别 | 技术选型 | 建议版本 | 用途与理由 |
| :--- | :--- | :--- | :--- |
| **前端框架** | Next.js | `14.x` | **业界标杆**: 提供最佳的性能 (SSG/ISR) 和 SEO 支持，与 Vercel 部署无缝集成。 |
| **UI 组件** | Shadcn/UI | `latest` | **高度可定制**: 提供了设计精良、无障碍体验极佳的基础组件，同时给予我们完全的控制权。 |
| **CSS 方案** | Tailwind CSS | `3.x` | **高效开发**: Utility-first 的思想能极大提升样式编写速度和一致性，与 Shadcn/UI 完美配合。 |
| **前端状态管理**| React Hooks/Context | `(内置)` | **MVP 优先**: 对于 MVP 阶段的功能，React 自带的状态管理已足够，避免过度设计。 |
| **后端框架** | Strapi | `4.x` | **灵活的 Headless CMS**: 强大的内容建模能力，插件生态丰富，能满足我们所有内容管理需求。 |
| **API 风格** | REST API | `(内置)` | **成熟稳定**: Strapi 默认提供 REST API，标准、通用，易于前后端对接。 |
| **数据库** | PostgreSQL | `16.x` | **功能强大**: 性能稳定，功能丰富的开源关系型数据库，能支持未来的复杂业务。 |
| **文件存储** | Cloudflare R2 | `N/A` | **高性价比**: 用于存储 Strapi 上传的图片等媒体文件，全球分发速度快，成本极具优势。 |
| **认证授权** | Strapi Users & Permissions | `(内置)` | **开箱即用**: Strapi 自带的用户系统足以满足 MVP 阶段的需求。 |
| **测试框架** | Vitest & Playwright | `latest` | **现代高效**: Vitest 用于单元/集成测试，速度快；Playwright 用于端到端(E2E)测试，功能强大可靠。 |
| **代码仓库** | Monorepo (Turborepo) | `latest` | **统一管理**: 使用 Turborepo 管理单一代码库，优化构建流程，方便前后端代码共享。 |
| **CI/CD** | GitHub Actions | `N/A` | **无缝集成**: 与代码托管平台紧密结合，易于配置自动化测试和部署流程。 |
| **网站分析** | Vercel Analytics | `(内置)` | **简便易用**: Vercel 自带分析工具，无需额外配置，开箱即用地收集核心网站指标。 |

### 3. 数据模型 (Data Models)

#### **模型一：文章 (Article)**

* **用途**: 存储博客文章的核心内容。
* **关键属性**:
    * `title` (标题): `Text` - 文章的标题。
    * `slug` (路径): `UID` - 文章的唯一访问路径，根据标题自动生成。
    * `content` (内容): `Rich Text` - 支持 Markdown 的文章正文。
    * `cover_image` (封面图): `Media` - 文章的封面图片。
    * `published_at` (发布时间): `Datetime` - 文章的发布日期和时间。
    * `category` (分类): `Relation` - 与 “Category” 模型的一对一关系。
    * `tags` (标签): `Relation` - 与 “Tag” 模型的多对多关系。
* **TypeScript 类型定义 (`packages/shared/src/types/article.ts`)**:
    ```typescript
    import { Category } from './category';
    import { Tag } from './tag';
    import { StrapiMedia } from './strapi';

    export interface Article {
      id: number;
      attributes: {
        title: string;
        slug: string;
        content: string;
        cover_image: { data: StrapiMedia | null };
        published_at: string;
        category: { data: Category | null };
        tags: { data: Tag[] };
      };
    }
    ```

#### **模型二：分类 (Category)**

* **用途**: 用于文章的单选分类。
* **关键属性**:
    * `name` (名称): `Text` - 分类的名称，如“前端技术”。
* **TypeScript 类型定义 (`packages/shared/src/types/category.ts`)**:
    ```typescript
    export interface Category {
      id: number;
      attributes: {
        name: string;
      };
    }
    ```

#### **模型三：标签 (Tag)**

* **用途**: 用于为文章添加多个标签。
* **关键属性**:
    * `name` (名称): `Text` - 标签的名称，如“React”。
* **TypeScript 类型定义 (`packages/shared/src/types/tag.ts`)**:
    ```typescript
    export interface Tag {
      id: number;
      attributes: {
        name: string;
      };
    }
    ```

### 3. 数据模型 (Data Models) (续)

#### **模型四：作品集条目 (Portfolio Item)**

* **用途**: 存储每一个独立的个人项目或作品案例。
* **关键属性**:
    * `title` (标题): `Text` - 项目的名称。
    * `description` (描述): `Rich Text` - 项目的详细介绍。
    * `image` (图片): `Media` - 项目的展示图片或截图。
    * `project_url` (项目链接): `Text` - 指向项目线上地址或代码仓库的链接。
    * `technologies` (技术栈): `Relation` - 与 “Technology” 模型的多对多关系。
* **TypeScript 类型定义 (`packages/shared/src/types/portfolio.ts`)**:
    ```typescript
    import { Technology } from './technology';
    import { StrapiMedia } from './strapi';

    export interface PortfolioItem {
      id: number;
      attributes: {
        title: string;
        description: string;
        image: { data: StrapiMedia | null };
        project_url: string;
        technologies: { data: Technology[] };
      };
    }
    ```

#### **模型五：技术 (Technology)**

* **用途**: 作为可复用的技术标签，用于关联作品集条目。
* **关键属性**:
    * `name` (名称): `Text` - 技术的名称，如“Next.js”。
* **TypeScript 类型定义 (`packages/shared/src/types/technology.ts`)**:
    ```typescript
    export interface Technology {
      id: number;
      attributes: {
        name: string;
      };
    }
    ```

#### **模型六：关于页面 (About Page)**

* **用途**: 存储“关于我”这个独立页面的内容。这是一个**单一类型 (Single Type)**，因为网站只有一个“关于我”页面。
* **关键属性**:
    * `title` (标题): `Text` - 页面的主标题。
    * `content` (内容): `Rich Text` - 页面的详细介绍内容。
* **TypeScript 类型定义 (`packages/shared/src/types/about.ts`)**:
    ```typescript
    export interface AboutPage {
      id: number;
      attributes: {
        title: string;
        content: string;
      };
    }
    ```

#### **模型七：社交链接 (Social Link)**

* **用途**: 管理将在网站页脚等位置显示的社交媒体链接列表。
* **关键属性**:
    * `platform` (平台): `Enumeration` - 下拉选项列表，预设值如 `GitHub`, `LinkedIn`, `Twitter`, `Bilibili`, `Juejin` 等。
    * `url` (链接): `Text` - 你的个人主页链接。
    * `icon_svg` (图标): `Text` (长文本) - (可选) 允许直接粘贴平台的 SVG 图标代码，赋予前端最大的灵活性。
* **TypeScript 类型定义 (`packages/shared/src/types/social.ts`)**:
    ```typescript
    export type SocialPlatform = "GitHub" | "LinkedIn" | "Twitter" | "Bilibili" | "Juejin" | "Other";

    export interface SocialLink {
      id: number;
      attributes: {
        platform: SocialPlatform;
        url: string;
        icon_svg?: string;
      };
    }
    ```

### 4. API 规范 (API Specification)

```yaml
openapi: 3.0.0
info:
  title: "个人品牌网站 API"
  version: "1.0.0"
  description: "用于驱动个人品牌网站前端的内容 API"
servers:
  - url: "/api"
    description: "Strapi 后端 API 根路径"

paths:
  /articles:
    get:
      summary: "获取文章列表"
      description: "获取所有已发布的文章，支持分页。"
      parameters:
        - name: "populate"
          in: "query"
          schema:
            type: "string"
          description: "关联加载字段 (例如: cover_image, category, tags)"
        - name: "pagination[page]"
          in: "query"
          schema:
            type: "integer"
          description: "页码"
        - name: "pagination[pageSize]"
          in: "query"
          schema:
            type: "integer"
          description: "每页数量"
      responses:
        "200":
          description: "一个包含文章列表和分页信息的对象"
          content:
            application/json:
              schema:
                type: "array"
                items:
                  $ref: "#/components/schemas/Article"
        
  /articles/{slug}:
    get:
      summary: "获取单篇文章详情"
      description: "根据 slug 获取一篇文章的完整内容。"
      parameters:
        - name: "slug"
          in: "path"
          required: true
          schema:
            type: "string"
      responses:
        "200":
          description: "包含单篇文章详情的对象"
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Article"
        "404":
          description: "未找到文章"
          
# (此处省略了其他数据模型的端点定义，结构类似)

components:
  schemas:
    Article:
      type: "object"
      # (此处省略了 Article 模型的详细 schema 定义)
```


  ### 5. 逻辑组件 (Logical Components)

我们的系统主要由以下几个逻辑块构成：

* **前端应用 (Frontend App)**:
    * **职责**: 渲染用户界面，处理用户交互，从后端获取并展示数据。
    * **技术**: Next.js
    * **物理位置**: `apps/web`

* **后端应用 (Backend App)**:
    * **职责**: 提供内容管理的后台界面，并通过 REST API 向前端暴露数据。
    * **技术**: Strapi
    * **物理位置**: `apps/api`

* **共享类型包 (Shared Types Package)**:
    * **职责**: 定义前后端共用的 TypeScript 数据类型，确保数据一致性。
    * **技术**: TypeScript
    * **物理位置**: `packages/shared`

* **共享 UI 包 (Shared UI Package)**:
    * **职责**: (可选，作为最佳实践) 存放未来可能被多个应用共享的基础 UI 组件。
    * **技术**: React, Shadcn/UI
    * **物理位置**: `packages/ui`

### 6. 统一项目结构 (Unified Project Structure)

以下是我们采用 Turborepo 管理的 Monorepo 的具体文件目录结构：

```plaintext
your-project-name/
├── apps/                     # 独立应用
│   ├── api/                  # Strapi 后端应用
│   │   ├── config/
│   │   ├── src/
│   │   │   ├── api/          # (Strapi 自动生成的数据模型 API)
│   │   │   └── admin/
│   │   └── package.json
│   └── web/                  # Next.js 前端应用
│       ├── app/              # Next.js 14 的 App Router 目录
│       │   ├── (pages)/      # 页面路由
│       │   │   ├── about/
│       │   │   ├── blog/
│       │   │   └── portfolio/
│       │   └── layout.tsx
│       ├── components/       # UI 组件 (基于 Shadcn/UI)
│       ├── lib/              # 工具函数/API 客户端
│       └── package.json
├── packages/                 # 共享代码包
│   ├── shared/               # 共享的 TS 类型和工具函数
│   │   └── src/
│   │       ├── index.ts      # (导出所有类型)
│   │       └── types/
│   └── ui/                   # 共享的 UI 组件库
│       └── src/
├── package.json              # Monorepo 根 package.json
└── turborepo.json            # Turborepo 配置文件
```
### 7. 开发工作流 (Development Workflow)

#### **本地开发设置 (Local Development Setup)**

1.  开发者在本地克隆 Git 仓库。
2.  在项目根目录运行 `npm install`，Turborepo 会自动安装所有应用（`apps`）和包（`packages`）的依赖。
3.  在根目录创建一个 `.env` 文件，填入数据库连接信息等环境变量。
4.  运行 `npm run dev`，Turborepo 会同时启动 Next.js 前端和 Strapi 后端的开发服务器。

#### **环境配置 (Environment Configuration)**

* 我们将使用 `.env` 文件来管理环境变量。前端应用 (`apps/web`) 需要知道后端的 API 地址，后端 (`apps/api`) 需要数据库的连接信息。

    ```bash
    # apps/web/.env.local
    NEXT_PUBLIC_API_URL=http://localhost:1337

    # apps/api/.env
    DATABASE_CLIENT=postgres
    DATABASE_HOST=127.0.0.1
    DATABASE_PORT=5432
    DATABASE_NAME=your_db_name
    DATABASE_USERNAME=your_db_user
    DATABASE_PASSWORD=your_db_password
    ```

### 8. 部署架构 (Deployment Architecture)

#### **部署策略 (Deployment Strategy)**

* **前端 (`apps/web`)**: 我们将前端应用部署到 **Vercel**。Vercel 是 Next.js 的创造者，能提供一键式部署、全球 CDN 加速、自动 SSL 证书和最佳的性能优化。
* **后端 (`apps/api`)**: 我们将 Strapi 后端应用打包成一个 **Docker 镜像**。这种容器化的方式提供了极大的灵活性，你可以将它部署到任何支持 Docker 的云平台，无论是国内的**阿里云/腾讯云容器服务**，还是国外的 **Google Cloud Run**, **Render**, 或任何 VPS 服务器。

#### **CI/CD 流程 (Continuous Integration / Continuous Deployment)**

* 我们将使用 **GitHub Actions** 来自动化测试和部署流程。
    1.  当代码被推送到 `main` 分支时，工作流被触发。
    2.  自动运行所有前端和后端的测试。
    3.  如果测试通过：
        * Vercel 会自动拉取最新的前端代码，进行构建和上线。
        * GitHub Actions 会构建后端的 Docker 镜像，推送到容器镜像仓库（如 Docker Hub 或阿里云 ACR），然后触发云服务器拉取新镜像并重启服务。

#### **环境 (Environments)**

* **Production (生产环境)**: 关联 `main` 分支，每次合并到 `main` 都会触发自动部署。
* **Preview (预览环境)**: Vercel 会为每个 Pull Request 自动创建一个临时的预览环境，方便我们在合并代码前审查前端的变更。

### 9. 测试策略 (Testing Strategy)

* **测试金字塔**: 我们将遵循测试金字塔原则，以大量的单元测试为基础，辅以适量的集成测试，并用少量的端到端测试覆盖关键用户流程。
* **单元/集成测试**: 前后端应用都将使用 **Vitest** 进行测试。测试文件与源代码并列存放（例如 `component.tsx` 和 `component.test.tsx`），便于发现和维护。
* **端到端测试**: 使用 **Playwright** 编写测试脚本，模拟真实用户交互，覆盖核心的用户流程（如文章浏览、作品集查看）。
* **自动化**: 所有测试都将集成到 **GitHub Actions** CI/CD 流程中，任何测试未通过的代码都无法合并到 `main` 分支。

### 10. 安全与性能 (Security and Performance)

#### **安全 (Security)**

* **凭证管理**: 绝不将任何 API 密钥、数据库密码等敏感信息硬编码在代码中。所有凭证都必须通过环境变量 (`.env` 文件) 进行管理。
* **访问控制**: 充分利用 Strapi 内置的基于角色的访问控制（RBAC），为 API 端点设置精细的公开或私有权限。
* **输入验证**: 所有来自前端的请求，都将在 Strapi 的控制器层进行严格的输入验证，防止恶意数据注入。
* **前端安全**: 依赖 Vercel 平台提供的安全头（Security Headers）等最佳实践，防范 XSS、CSRF 等常见 Web 攻击。

#### **性能 (Performance)**

* **前端性能**: 优先使用 Next.js 的静态站点生成 (SSG) 和增量静态再生 (ISR) 功能，生成预渲染的静态页面，实现秒级加载。所有图片都将通过 Next.js 的 `<Image>` 组件进行自动化优化。
* **后端性能**: Strapi 默认提供高效的数据库查询。对于复杂查询，我们将确保在 PostgreSQL 数据库中建立适当的索引。
* **缓存策略**: Vercel 的全球 CDN 将自动缓存前端静态资源。后端 API 可在未来第二阶段根据需要引入 Redis 等缓存层。

### 11. 编码标准 (Coding Standards)

* **一致性**: 项目将配置统一的 **ESLint** 和 **Prettier** 规则，在代码提交时自动格式化，确保所有代码风格一致。
* **类型安全**: 整个项目（前后端）都将启用 TypeScript 的 `strict` 模式。所有前后端共享的数据结构，都必须从 `packages/shared` 中导入，不得在应用中重复定义。
* **代码组织**: 遵循各自框架（Next.js, Strapi）的最佳实践来组织代码，确保代码的可读性和可维护性。

### 12. 下一步 (Next Steps)

* **规划完成**: 至此，全栈架构文档已完成。项目的所有规划阶段（PRD, UI/UX Spec, Architecture）均已结束。
* **文档分片**: 下一步是在 IDE 环境中，将这份架构文档和 PRD 分片成更小的任务单元。
* **进入开发**: 分片后，Scrum Master 将开始创建第一个用户故事，正式进入开发迭代循环。



