---
name: long-screenshot
description: 将长截屏图片转化为适配手机尺寸的网页，并部署到 GitHub Pages 生成公网访问链接
---

# 长截屏转网站 + 部署（多站点版）

每次调用创建独立子目录，互不覆盖。旧页面不会消失。

## 输入

- 图片路径（必填）
- 页面标题（可选，默认取文件名）
- 自定义 slug（可选，默认用时间戳生成）

---

## 第一阶段：生成 slug

slug 是 URL 中的目录名，要求字母数字短横线。

优先级：
1. 用户手动指定 slug
2. 标题拼音/英文部分（如果标题含 Latin 字符则提取）
3. 时间戳 fallback：`shot-YYYYMMDD-HHmmss`

```bash
SLUG="shot-$(date +%Y%m%d-%H%M%S)"
```

---

## 第二阶段：生成 HTML

### 1. 转换为 base64

```bash
base64 -w 0 "<图片路径>" > /tmp/base64_tmp.txt
```

### 2. 拼接 HTML

分三段拼接（head + base64 + tail），用临时文件避免 here-string 限制：

**head.txt**：
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=3.0, viewport-fit=cover">
<title>页面标题</title>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: #000; overflow-y: auto; -webkit-overflow-scrolling: touch; }
#loading { position: fixed; top: 50%; left: 50%; transform: translate(-50%,-50%); color: #aaa; font-size: 14px; z-index: 10; }
img { display: block; width: 100%; height: auto; user-select: none; -webkit-user-select: none; pointer-events: none; }
</style>
</head>
<body>
<div id="loading">加载中...</div>
<img src="data:image/jpeg;base64,
```

**tail.txt**：
```html
" onload="document.getElementById('loading').style.display='none'">
</body>
</html>
```

### 3. 写文件到子目录

```bash
mkdir -p <slug>
cat head.txt /tmp/base64_tmp.txt tail.txt > <slug>/index.html
```

---

## 第三阶段：部署到 GitHub

### 4. 提交并推送

此项目已有 GitHub 仓库（PEPSIcyk/school-run），直接复用，**无需重新开启 Pages**。

```bash
git add <slug>/index.html
git commit -m "添加长截屏页面：<标题>"
git push
```

### 5. 输出

```
网站已部署

公网地址：https://pepsicyk.github.io/school-run/<slug>/
```

无需再次开启 GitHub Pages（已开启过即可）。
