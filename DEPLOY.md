# Artisan Billiards 网站部署指南

> 网站大小：64KB（纯静态单页应用）

---

## 方案一：Vercel（推荐，最快最稳）

### 方法 A：命令行部署（推荐）

```bash
# 1. 安装 Vercel CLI
npm i -g vercel

# 2. 进入项目目录
cd /Users/fengshuai/Desktop/billiard-website

# 3. 一键部署
vercel --prod

# 首次运行会提示登录，按指引用 GitHub/Google 账号授权即可
# 部署完成后会返回一个 https://xxx.vercel.app 的域名
```

### 方法 B：拖拽部署（零代码）

1. 打开 https://vercel.com/new
2. 选择 "Import Git Repository" 或直接将项目文件夹拖拽到页面
3. 点击 Deploy，30 秒后自动上线
4. 自动生成 HTTPS 域名，全球 CDN 加速

**优点**：免费、自动 HTTPS、全球 CDN、每次推送自动重新部署

---

## 方案二：Netlify Drop（最简单，30 秒上线）

1. 打开 https://app.netlify.com/drop
2. 将 `billiard-website` 文件夹直接拖拽到网页中
3. 立刻获得一个随机域名（如 `https://abundant-cactus-123456.netlify.app`）
4. 点击 "Site settings" → "Domain management" 可绑定自定义域名

**优点**：无需注册、无需命令行、秒级部署

---

## 方案三：GitHub Pages（免费，适合开发者）

```bash
# 1. 初始化 Git 仓库
cd /Users/fengshuai/Desktop/billiard-website
git init
git add .
git commit -m "Initial commit"

# 2. 在 GitHub 创建新仓库（如 billiard-website）
# 访问 https://github.com/new

# 3. 关联并推送
git remote add origin https://github.com/你的用户名/billiard-website.git
git branch -M main
git push -u origin main

# 4. 开启 GitHub Pages
# 进入仓库 → Settings → Pages → Source 选择 "Deploy from a branch"
# Branch 选择 "main"，文件夹选 "/ (root)"，点击 Save

# 5. 等待 1 分钟，访问 https://你的用户名.github.io/billiard-website
```

**优点**：完全免费、与 Git 版本控制结合、支持自定义域名

---

## 方案四：国内云存储（国内访问最快）

### 腾讯云 COS + CDN

1. 登录 [腾讯云控制台](https://console.cloud.tencent.com/cos)
2. 创建存储桶，访问权限选 **公有读私有写**
3. 进入存储桶 → 文件列表 → 上传 `index.html`
4. 基础配置 → 静态网站 → 开启，索引文档填 `index.html`
5. （可选）绑定自定义域名 + 开启 CDN 加速

### 阿里云 OSS

1. 登录 [阿里云控制台](https://oss.console.aliyun.com/)
2. 创建 Bucket，读写权限选 **公共读**
3. 上传 `index.html`
4. 基础设置 → 静态页面 → 默认首页设为 `index.html`
5. 访问 Bucket 域名即可

**优点**：国内访问速度极快、支持自定义域名、费用极低（每月约 ¥1-5）

---

## 方案五：自有服务器 / VPS

```bash
# 连接服务器后执行
ssh root@你的服务器IP

# 进入网站目录
cd /var/www/html

# 上传文件（本地终端执行）
scp /Users/fengshuai/Desktop/billiard-website/index.html root@你的服务器IP:/var/www/html/

# 或使用 Nginx 配置
sudo tee /etc/nginx/sites-available/billiard << 'EOF'
server {
    listen 80;
    server_name 你的域名.com;
    root /var/www/billiard;
    index index.html;
    
    location / {
        try_files $uri $uri/ =404;
    }
    
    # 开启 gzip
    gzip on;
    gzip_types text/html text/css application/javascript;
}
EOF

sudo ln -s /etc/nginx/sites-available/billiard /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## 方案对比

| 方案 | 难度 | 费用 | 国内速度 | HTTPS | 自定义域名 |
|------|------|------|----------|-------|-----------|
| Vercel | ⭐ 低 | 免费 | ⭐⭐ 中等 | ✅ 自动 | ✅ |
| Netlify Drop | ⭐ 极低 | 免费 | ⭐⭐ 中等 | ✅ 自动 | ✅ |
| GitHub Pages | ⭐⭐ 中 | 免费 | ⭐⭐ 中等 | ✅ 自动 | ✅ |
| 腾讯云 COS | ⭐⭐ 中 | ¥1-5/月 | ⭐⭐⭐⭐⭐ 极快 | ✅ | ✅ |
| 自有服务器 | ⭐⭐⭐ 高 | ¥50+/月 | ⭐⭐⭐⭐ 快 | 需配置 | ✅ |

---

## 快速决策

- **想立刻看到效果** → Netlify Drop（拖拽即部署）
- **长期维护 + 自动化** → Vercel（推送即部署）
- **国内客户为主** → 腾讯云 COS / 阿里云 OSS
- **已有服务器** → Nginx 托管

---

## 绑定自定义域名（通用步骤）

无论你选哪个平台，绑定域名流程类似：

1. 在平台设置中找到 "Domains" 或 "自定义域名"
2. 添加你的域名（如 `www.artisanbilliards.com`）
3. 去域名注册商（阿里云/GoDaddy/Cloudflare）添加 DNS 记录：

```
类型    主机记录    记录值
CNAME   www         cname.vercel-dns.com.   # Vercel
CNAME   www         xxx.netlify.app.        # Netlify
A       @           185.199.108.153         # GitHub Pages
```

4. 等待 DNS 生效（通常 5-30 分钟）
5. 平台自动签发 SSL 证书，HTTPS 可用

---

> 需要我帮你执行其中某个方案吗？告诉我你的偏好即可。