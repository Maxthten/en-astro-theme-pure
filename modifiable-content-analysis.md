# 个人博客项目可修改内容分析报告

## 1. 配置文件分析

### 1.1 `src/site.config.ts` - 主题核心配置

**可修改内容：**

#### 基本配置
- `title` (第6行): 网站标题，显示在浏览器标签和元数据中
- `author` (第8行): 网站作者名称，用于首页和版权声明
- `description` (第10行): 网站描述，用于SEO和元数据
- `favicon` (第12行): 网站图标路径，位于public目录下
- `socialCard` (第14行): 社交媒体分享卡片图片路径
- `locale` (第16-26行): 网站语言设置，包括：
  - `lang`: 主要语言代码
  - `attrs`: 语言属性
  - `dateLocale`: 日期显示语言
  - `dateOptions`: 日期显示格式
- `logo` (第28-31行): 首页显示的logo图片路径和alt文本
- `titleDelimiter` (第33行): 标题分隔符，用于页面标题
- `prerender` (第34行): 是否启用预渲染，影响搜索功能
- `npmCDN` (第35行): NPM包CDN地址
- `head` (第38-45行): 自定义HTML头部内容，如meta标签
- `customCss` (第46行): 自定义CSS文件路径数组

#### 导航配置
- `header.menu` (第50-57行): 顶部导航菜单，包含标题和链接

#### 页脚配置
- `footer.year` (第63-64行): 版权年份格式
- `footer.links` (第65-82行): 页脚链接列表，可添加备案信息、旅行统计等
- `footer.credits` (第85行): 是否显示"Astro & Pure主题驱动"链接
- `footer.social` (第87行): 社交媒体链接，如GitHub

#### 内容配置
- `content.externalLinks` (第93-99行): 外部链接配置，包括显示文本和属性
- `content.blogPageSize` (第101行): 博客页面分页大小
- `content.share` (第103行): 分享按钮配置，支持weibo、x、bluesky

#### 集成配置
- `integ.links` (第110-129行): 友链配置，包括：
  - `logbook`: 友链日志
  - `applyTip`: 申请友链提示信息
  - `cacheAvatar`: 是否缓存头像
- `integ.pagefind` (第131行): 是否启用Pagefind搜索
- `integ.quote` (第135-148行): 随机名言配置，包括API地址和数据提取规则
- `integ.typography` (第151-157行): 排版配置，包括：
  - `class`: 排版CSS类
  - `blockquoteStyle`: 引用块样式
  - `inlineCodeBlockStyle`: 内联代码块样式
- `integ.mediumZoom` (第161-167行): 图片缩放配置
- `integ.waline` (第170-187行): Waline评论系统配置

#### 条款配置
- `terms` (第190-210行): 网站条款链接列表

### 1.2 `astro.config.ts` - Astro构建配置

**可修改内容：**
- 部署配置
- 插件配置（如MDX、React等）
- 构建选项
- 开发服务器设置

### 1.3 `uno.config.ts` - UnoCSS配置

**可修改内容：**
- 自定义主题颜色
- 字体配置
- 间距配置
- 插件配置
- 预设配置

### 1.4 `tsconfig.json` - TypeScript配置

**可修改内容：**
- TypeScript编译选项
- 类型检查严格程度
- 模块解析配置

### 1.5 `prettier.config.mjs` - Prettier代码格式化配置

**可修改内容：**
- 代码缩进
- 引号样式
- 分号配置
- 换行符配置

### 1.6 `eslint.config.mjs` - ESLint配置

**可修改内容：**
- 代码规则配置
- 插件配置
- 环境配置

## 2. 内容文件分析

### 2.1 博客文章 (`src/content/blog/`)

**可修改内容：**
- 所有`.md`和`.mdx`文件均可修改，用于发布新博客文章
- 每个文章文件包含：
  - 前置元数据（frontmatter）：
    - `title`: 文章标题
    - `publishDate`: 发布日期
    - `description`: 文章描述
    - `tags`: 文章标签
    - `heroImage`: 文章封面图
    - `language`: 文章语言
    - `draft`: 是否为草稿
    - `comment`: 是否启用评论
  - 文章正文：使用Markdown/MDX语法编写

### 2.2 文档内容 (`src/content/docs/`)

**可修改内容：**
- 所有`.md`和`.mdx`文件均可修改，用于创建和编辑文档
- 文档结构可通过创建新文件夹和文件来扩展

## 3. 页面文件分析

### 3.1 主要页面 (`src/pages/`)

**可修改内容：**
- `index.astro`: 首页内容和布局
- `about/index.astro`: 关于页面内容
- `projects/index.astro`: 项目页面内容
- `links/index.astro`: 友链页面内容
- `terms/`: 网站条款页面，包括：
  - `copyright.md`: 版权声明
  - `disclaimer.md`: 免责声明
  - `privacy-policy.md`: 隐私政策
  - `terms-and-conditions.md`: 服务条款

### 3.2 页面路由

**可修改内容：**
- 可通过在`src/pages/`目录下创建新的`.astro`、`.md`或`.mdx`文件来添加新页面
- 支持动态路由，如`blog/[...id].astro`用于博客文章详情页

## 4. 组件文件分析

### 4.1 用户自定义组件 (`src/components/`)

**可修改内容：**
- `about/`: 关于页面组件，包括：
  - `Substats.astro`: 统计数据组件
  - `ToolSection.astro`: 工具展示组件
- `home/`: 首页组件，包括：
  - `ProjectCard.astro`: 项目卡片组件
  - `Section.astro`: 首页分区组件
  - `SkillLayout.astro`: 技能展示布局组件
- `links/`: 友链组件
- `projects/`: 项目页面组件
- `waline/`: Waline评论相关组件
- `BaseHead.astro`: 页面头部组件

### 4.2 主题组件定制

**定制方法：**
- 通过Swizzling方法自定义主题组件：
  1. 复制主题组件到`src/components/`目录
  2. 修改组件代码
  3. 更新引用路径

**可定制的核心组件：**
- `Header.astro`: 头部导航组件
- `Footer.astro`: 页脚组件
- `ThemeProvider.astro`: 主题提供者组件
- 其他页面组件如`Hero.astro`、`PostPreview.astro`等

## 5. 布局文件分析

### 5.1 布局组件 (`src/layouts/`)

**可修改内容：**
- `BaseLayout.astro`: 基础布局组件，包含header、footer和主题提供者
- `BlogPost.astro`: 博客文章布局
- `CommonPage.astro`: 通用页面布局
- `ContentLayout.astro`: 内容页面布局
- `IndividualPage.astro`: 独立页面布局

## 6. 资源文件分析

### 6.1 静态资源 (`public/`)

**可修改内容：**
- `favicon/`: 网站图标文件，可替换为自定义图标
- `icons/`: 自定义图标文件
- `images/`: 图片资源，如社交媒体卡片图片
- `scripts/`: 自定义脚本
- `links.json`: 友链数据

### 6.2 源资源 (`src/assets/`)

**可修改内容：**
- `icons/`: 应用图标
- `projects/`: 项目展示图片
- `styles/`: 自定义样式文件
- `tools/`: 工具图标
- 其他图片资源，如头像、二维码等

## 7. 样式文件分析

### 7.1 CSS文件 (`src/assets/styles/`)

**可修改内容：**
- `app.css`: 应用主样式文件
- `global.css`: 全局样式文件

### 7.2 UnoCSS样式

**可修改内容：**
- 通过`uno.config.ts`自定义主题样式
- 在组件和页面中使用UnoCSS工具类
- 创建自定义CSS类和组件样式

## 8. 自定义组件使用

**可使用的用户组件：**
- 容器组件：`Card`、`Collapse`、`Aside`、`Tabs`、`MdxRepl`
- 列表组件：`CardList`、`Timeline`、`Steps`
- 文本组件：`Button`、`Spoiler`、`FormattedDate`、`Label`、`Svg`
- 资源组件：`Icon`

**使用方法：**
- 在MDX文件中直接导入使用
- 在Astro组件中导入使用

## 9. 内容创建指南

### 9.1 添加新博客文章

**步骤：**
1. 在`src/content/blog/`目录下创建新文件夹或文件
2. 使用Markdown/MDX语法编写文章
3. 添加必要的前置元数据
4. 可使用`bun new <post-slug>`命令快速创建文章模板

### 9.2 添加新页面

**步骤：**
1. 在`src/pages/`目录下创建新的`.astro`、`.md`或`.mdx`文件
2. 根据需要选择合适的布局组件
3. 编写页面内容

## 10. 主题定制最佳实践

### 10.1 配置优先原则
- 优先修改`src/site.config.ts`中的配置项，避免直接修改组件
- 只有在配置项无法满足需求时，才考虑自定义组件

### 10.2 组件定制方法
- 使用Swizzling方法自定义主题组件，保持升级兼容性
- 避免直接修改`packages/pure/`目录下的核心代码

### 10.3 样式定制方法
- 优先使用UnoCSS工具类和主题配置
- 如需自定义样式，可创建独立的CSS文件并在`customCss`中引用

## 11. 升级考虑

- 保持`src/site.config.ts`配置项的兼容性
- 使用Swizzling方法自定义的组件需要手动更新
- 定期查看主题更新日志，了解配置项变化

## 总结

本博客项目提供了丰富的可修改内容，从配置项到组件、从内容到样式，都可以根据需求进行定制。建议按照"配置优先、组件其次、样式最后"的原则进行定制，以保持良好的升级兼容性。同时，参考项目文档和示例，确保定制内容符合主题设计规范和最佳实践。