# 想象盒子 · 博客 DIY 报告(初代自定义版)

> 版本:v1.0(初代自定义)· tag `v1-imagination-box`
> 日期:2026-09 · 构建:Hugo + hugo-theme-venin
> 用途:人和 AI 后续改进前,先读这份,知道做过什么、从哪继续、坑在哪。

---

## 1. 站点定位与设计内核

- 站名:**想象盒子**(海绵宝宝 & 派大星纸盒梗,"All you need is imagination")
- 人格:QA1 腔调——每篇文字发生在一个场景里,像单元剧;博客=幻想空间/舞台
- 主文案/设定源头:Obsidian vault `个人博客/QA1.md`、`博客构建蓝图.md`
- 版权署名:zzq;协议暂定 CC BY-NC-SA 4.0(在页脚与 copyright,未最终确认)

## 2. 技术栈

| 层 | 选型 | 备注 |
|---|---|---|
| 生成器 | Hugo 0.165 extended | 需要 dart-sass 入 PATH 才能 `css.Sass`(venin 主题用) |
| 主题 | eltr.ac eltrac/hugo-theme-venin | **git submodule**,src.eltr.ac 自托管,直连可 clone |
| 原子 CSS | UnoCSS(uno.css) | pnpm 生成,产物已拷入 site `assets/css/uno.css` 并提交 → **CI 不用 node** |
| 构建节点 | node 24 + pnpm 12 | corepack 装到 `~/.corepack`(node 目录只读) |
| 部署 | GitHub Actions(现 workflow 未适配,见 §7 待办) | 远端 FumonLemon/mygarden → blog.lemonfumo.top |

## 3. 关键文件地图(site 层,全部在 E:\mygarden)

主题本体不直接改(保 submodule 干净),定制全在 site 覆写:

```
hugo.toml                          # 全部站点参数(见下)
content/
  _index.md                        # 首页底部正文(开场白块;真正开场白主人待写)
  about.md / links.md              # 单页(占位,待写)
  article/                         # ★ 文章 section 必须叫 article(主题约定)
    6 篇示例文(带 description+categories+tags,标"示例"待替换)
static/
  logo.png                         # 开盖盒(抠图,裁边 754x512)
  logo-closed.png                  # 合盖盒(抠掉黑底,747x512;与开盖同高)
  favicon.png                      # 亮色网页图标=开盒(512²,主体 512x388)
  favicon-closed.png               # 暗色网页图标=合盒(512²,主体 446x391)
  apple-touch-icon.png             # 移动端主屏图标(180²,开盒)
  css/kraft.css                    # ★ 全部定制样式的家:配色变量/背景纹/过渡/logo 尺寸
layouts/
  _partials/head/css.html          # 覆写:去 pagefind 404 css;kraft.css 挪到主题样式之后(必须最后!)
  _partials/footer.html            # 覆写:去 Eltrac 硬编码版权/IndieWeb ring,换 zzq 版权+许可
  _partials/header/lang-switch.html# 覆写:单语站,清空语言切换按钮
  _partials/posts/webmention.html  # 覆写:禁远程 GetRemote(空 url 会直接构建报错)
  _partials/scripts.html           # 覆写:去 GoatCounter;加 ★ logo/favicon 主题联动 JS
themes/hugo-theme-venin/           # submodule,零本地改动
```

## 4. 设计体系(全部改动记录)

### 4.1 kraft 纸盒配色(核心)
- 亮色 `:root` / 暗色 `.dark` 两套 CSS 变量,覆盖 venin `_variables.scss` 的默认(黑白灰 oklch)
- 亮:`--background-color:#F3E7CE`(米黄护眼)、块底 `#ECDCBA`、正文深棕 `#46352A`、弱字 `#8A7250`、边 `#D9C49E`、链接琥珀 `#B45309`
- 暗:底 `#2B231B`、文 `#E8D8BC`、链接 `#D98E3F` 等
- 字体沿用 venin(Noto Serif SC 系),只换色

### 4.2 背景织纹:graph-paper 方格纸
- 主题自带 body 背景是个骷髅感大 SVG(作者选的,180x180 硬编码 data-uri),kraft.css 里 `body{background-image}` 覆盖
- **最终采用 v2 透明度**:亮 `fill-opacity='0.02'` / 暗 `0.01`,8x8 格、`background-size:16px`
  - 迭代史:v1(0.07/0.03, tag `bg-v1-op07`)→ 用户嫌亮色下与归档页虚线重合 → v2(tag `bg-v2-op02`,**在用**)
  - 换版本方法:`git checkout bg-v1-op07 -- static/css/kraft.css`(注意 tag 版没有后续新增的过渡块,要补)
- 颜色写死在 data-uri 里(不跟变量),亮/暗两套各一行

### 4.3 主题切换
- 机制:venin 自带 `darkmode.js` 给 `<html>` 切 `.dark` + 右上角 auto/light/dark 下拉
- 0.45s 颜色淡变:kraft.css `html,body{transition:background-color .45s,color .45s}`
- ★ logo/favicon 双态联动(`layouts/_partials/scripts.html` 内):MutationObserver 盯 `html.class`,变暗 → 顶栏 logo 换合盖图 + favicon 换合盖;变亮反向。淡切 0.14s(先淡出、换 src、淡入),预载两张图防空白
- 语义:切暗 = 合盖("呆在盒子里面");文章页大 logo 保持开盖 = "正在盒里读"

### 4.4 图标素材处理(坑多,详见 §6)
- 顶栏 logo 高锁定(3rem/4rem)防开合两图比例不同导致跳版
- favicon 两图用**统一缩放因子**,保证盒身高度一致(388 vs 391)

## 5. 内容写作约定(写文章时注意)

- 文章放 `content/article/*.md`,front matter:
  ```yaml
  title: "标题"
  date: 2026-09-06
  description: "首页大卡/列表摘要用,优先于正文自动摘要"
  categories: ["故事"]   # 首页分类胶囊/归档;现有:故事/数学/技术/随笔
  tags: ["示例"]
  ```
- description 没有就用正文首段自动摘要
- 数学公式渲染未配置(别写 `$...$`,会露源码;要支持走 passthrough 配置,见 hugo-blog-development skill)
- 首页大卡 = 最新一篇(带 description 最出效果);列表 12 篇封顶
- 分类页/标签页/归档(/article/ 按年月)主题自带

## 6. 坑清单(血泪)

1. **kraft.css 加载顺序**:venin 的 `head/css.html` 把 `extraCSSFiles` 放在主题样式**之前** → 覆盖无效。已覆写该 partial 把 kraft.css 挪到最后。以后凡是"改了 kraft.css 没效果",先查这个
2. **uno.css 是 pnpm 产物**:仓库没成品,主题 layouts 一旦改动(或新增原子类),必须 `pnpm run build:uno:prod` 再拷回 `assets/css/uno.css`,否则新 class 无样式
3. **pnpm 12 构建脚本门禁**:`pnpm-workspace.yaml` 只认顶层键,写 `dangerouslyAllowAllBuilds: true`(`pnpm:` 嵌套与 `onlyBuiltDependencies` 均被忽略,实测)
4. **corepack EPERM**:node 装在 D:\software\nodejs 只读 → `corepack enable --install-directory ~/.corepack` 绕开
5. **dart-sass**:Hugo 0.145+ 的 `css.Sass` 需要 PATH 里有 dart-sass(E:\hugo-bin\dart-sass)。本地 build/server 都要先 `$env:PATH = "E:\hugo-bin\dart-sass;" + $env:PATH`。**CI 里没有 → 部署 workflow 必挂(待办)**
6. **venin 内容约定**:文章 section 必须叫 `article`(模板按类型分目录);posts 等其它 section 无模板会出问题
7. **webmention partial**:不配 `params.webmention.src` 会执行 `GetRemote ""` → 构建报错,必须覆写禁掉
8. **footer/lang-switch/GoatCounter**:作者站私货(硬编码 Eltrac 版权、EN 切换、stats.geedea.pro 统计),全要覆写清掉
9. **素材"抠图"假象**:AI 抠图常有半透明残影,`getbbox` 会把鬼影算进内容 → 图标巨大。必须 alpha 阈值清零 + 行列投影取主体连续区
10. **favicon 缓存极顽固**:本地看新图标要开新标签页或 DevTools Disable cache,光刷新 tab 无效
11. **中文断言脚本**:temp .ps1 带中文必须存 UTF-8 BOM,否则 powershell.exe -File 按 ANSI 解析,中文全乱码假失败
12. **venin 有 3 个 deprecation WARN**(主题旧 API),不影响构建,别修

## 7. 当前状态与待办(从这里继续)

已做 ✓:venin 换装、kraft 配色(亮/暗)、graph-paper 背景(v2)、主题下拉+0.45s 淡变、logo 开合联动(淡切)、favicon 双态联动、footer/隐私本地化、示例文 x6、纸盒 logo/favicon 素材落地

待办(优先级排序):
1. **部署**:Actions workflow 需加 dart-sass 步骤(或 CI 里生成 uno.css 的替代方案);pagefind 搜索组件 css 已从 head 摘除,搜索按钮点了无反应,上线前决定删留
2. 顶栏 logo"真开合动画"(flap 旋转)未做——现方案是两图淡切;真动画需 SVG 分层重构,素材与配色素材已备,等用户拍板是否接受矢量版
3. 示例文全部替换成真内容;`_index.md` 开场白、about、links 待主人执笔
4. 文章分类体系命名(用户提过灵感:西部牛仔武器/罗马军团等)未定
5. hello.md 是早期占位,内容与盒子风格不合,可删可改
6. 数学渲染(passthrough/KaTeX)按需加
7. 站内搜索/评论(茶水间/webmention/fediverse)远期选项,未动

## 8. 常用命令速查

```powershell
# 本地预览(先给 dart-sass 入 PATH)
$env:PATH = "E:\hugo-bin\dart-sass;" + $env:PATH
E:\hugo-bin\hugo.exe server --bind 127.0.0.1 --port 1313

# 纯构建验证
E:\hugo-bin\hugo.exe --gc

# 改了主题 layouts / 新增原子类后,重生成 uno.css 并拷回
cd themes\hugo-theme-venin
$env:PATH = "$env:USERPROFILE\.corepack;" + $env:PATH   # pnpm 所在
pnpm run build:uno:prod
copy assets\css\uno.css ..\..\assets\css\uno.css

# 存档点(版本回滚)
git tag / git checkout <tag> -- <file>
```

素材源:C:\Users\Lenovo\Desktop\博客素材(原图与各代抠图都在,含 old/ 归档)
