# 读到这里

这是一个用 Jekyll 构建的 GitHub Pages 研究博客。文章继续以 `posts/*.md` 维护，推送到 `main` 后由 GitHub Actions 自动构建并发布。

## 发布地址

- 网站：<https://limafang.github.io/blogs/>
- 仓库：<https://github.com/limafang/blogs>

推送到 `main` 后，GitHub Actions 会自动构建并发布。首次使用时，需要在仓库 Settings → Pages → Build and deployment 中选择 `GitHub Actions`。

## 本地预览

安装 Ruby 和 Jekyll 后，在仓库根目录运行：

```bash
gem install bundler jekyll
jekyll serve --livereload
```

然后打开 `http://localhost:4000/blogs/`。新增文章时复制现有 front matter，并修改标题、日期、描述和 permalink 即可。
