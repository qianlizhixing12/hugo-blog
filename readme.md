# 安装hugo

```bash
sudo apt install hugo
```

# 创建站点并关联github仓

```bash
hugo new site hugo-blog
cd hugo-blog
git init
git remote add origin https://github.com/username/username.github.io.git
```

# 配置GitHub Actions自动部署

```bash
mkdir -p .github/workflows
touch .github/workflows/hugo.yaml
# 配置.github/workflows/hugo.yaml内容后
git add .
git commit -m "Initial commit"
git push -u origin main
# 配置github网站，GitHub仓库 → Settings → Pages → Source 选择 GitHub Actions
```

# 下载配置主题

```bash
git submodule add https://github.com/alex-shpak/hugo-book themes/hugo-book
echo "theme = 'hugo-book'" >> hugo.toml
# 把 draft: true 改为 draft: false 或直接删除这行（草稿不会被发布）
```

# 创建博客

```bash
hugo new content content/docs/asm-learn.md
```

# 本地预览

```bash
hugo server -D
```

# 本地调试

```bash
# 识别文章列表
hugo list all
```
