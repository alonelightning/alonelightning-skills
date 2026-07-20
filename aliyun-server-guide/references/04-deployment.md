# 环节四：部署（连接 → 环境 → 上线应用）

目标：把用户的网站/应用在服务器上跑起来，能用 `http://<公网IP>` 访问。以 Ubuntu 为例。

**本环节几乎全部可自动化**，agent 有两条执行通道，按可用性选择：

| 通道 | 说明 | 适用 |
|------|------|------|
| ① 云助手 RunCommand（推荐） | 走阿里云 API 在实例内执行脚本，**免 SSH、免密码、免开 22 端口**；公共镜像默认预装云助手 Agent | agent 有 AK 即可用，最稳 |
| ② 本机 SSH | `ssh -i key.pem root@IP` 直连执行 | agent 所在机器可出网到服务器 22 端口 |
| ③ 用户手动（兜底） | 控制台 Workbench 网页终端，用户复制粘贴命令 | 前两者均不可用时 |

**云助手用法（通道 ①）**：
```
# 执行命令（同步等待可加 --Timeout；脚本内容需 base64）
aliyun ecs RunCommand --RegionId <region> --InstanceId.1 <id> --Type RunShellScript --CommandContent <base64脚本> --Timeout 600

# 查执行结果
aliyun ecs DescribeInvocationResults --RegionId <region> --InvokeId <invokeId>
```
（PowerShell 生成 base64：`[Convert]::ToBase64String([Text.Encoding]::UTF8.GetBytes($script))`）

**手动兜底（通道 ③）操作指引**：控制台 https://ecs.console.aliyun.com → 实例列表 → 目标实例 → 「远程连接」→ Workbench → 输入 root + 密码 → 网页终端。agent 把每条命令发给用户粘贴，并让用户回传输出。

> 部署会修改服务器内容。**首次部署到全新服务器**属常规操作，播报即可；**覆盖已有内容/重新部署**必须先征得同意。

## 步骤 1：基础加固 【自动】

```bash
apt update && apt upgrade -y
```
纪律：安全组只放行必要端口；数据库端口绝不对公网开放；不用弱密码。

## 步骤 2：环境路线决策

- **用户想自己以后可视化管理** → 宝塔面板路线（安装可自动，后续使用是用户点鼠标）
- **用户全托管给 agent** → 命令行路线（agent 全自动，推荐）

### 路线 A：宝塔面板 【安装自动 + 使用人工】

1. 【自动】按宝塔官网 https://www.bt.cn/new/download.html 当前给出的 Ubuntu 安装脚本执行（脚本会输出面板地址+初始账号密码）。
2. 【自动】安全组放行面板端口（默认 8888，装完建议改）：AuthorizeSecurityGroup 或指引控制台。
3. 【人工】用户浏览器打开面板 → 一键装 LNMP → 建站点/装 WordPress（agent 口头指引每一步点哪）。
4. ⚠️ 提醒：面板改默认端口、设强密码、开面板 SSL。

### 路线 B：命令行 【全自动】

**静态网站：**
```bash
apt install -y nginx
# 网站文件放 /var/www/html/
```

**Node.js 应用：**
```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | bash - && apt install -y nodejs nginx
npm install -g pm2
pm2 start app.js --name myapp && pm2 save && pm2 startup
```

**Python 应用（Flask/FastAPI/Django）：**
```bash
apt install -y python3-pip python3-venv nginx
# venv 装依赖，gunicorn/uvicorn 启动，systemd 守护
```

**Nginx 反向代理**（/etc/nginx/sites-available/default）：
```nginx
server {
    listen 80;
    server_name _;
    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
改完 `nginx -t && systemctl reload nginx`。

**数据库：**
```bash
apt install -y mysql-server
mysql_secure_installation
```
只监听 127.0.0.1；重要数据 mysqldump + 快照双保险（开快照策略有少量费用 → 需同意）。

## 步骤 3：上传代码 【自动】

| 方式 | 命令/说明 |
|------|----------|
| git clone（有仓库，最推荐） | 云助手/SSH 里执行 `git clone <repo>` |
| scp/SFTP | `scp -r ./dist root@IP:/var/www/html/`（agent 本机有代码时） |
| 云助手拉取 OSS/URL | 代码打包传可下载 URL 后 `curl -o` 拉取 |
| 手动兜底 | 宝塔面板文件管理拖拽上传，或 FinalShell 等图形工具 |

代码在用户电脑上而 agent 拿不到 → 指引用户用 Workbench 的文件上传或图形 SSH 工具。

## 步骤 4：验证 【自动】

```
curl -I http://<公网IP>
```
返回 200/301 即部署成功。失败排查顺序：① 应用进程在不在（pm2 list / systemctl status）② Nginx 配置（nginx -t）③ 安全组 80 是否放行 ④ 服务器内 `curl 127.0.0.1` 是否通（区分应用问题和网络问题）。

## 上线前检查清单

- [ ] IP 访问正常
- [ ] 进程守护（pm2/systemd）已配置，重启自动拉起（可 `aliyun ecs RebootInstance` 后验证——重启会短暂中断服务 → 需同意）
- [ ] 数据库不对公网开放、强密码
- [ ] 备份策略（快照/mysqldump 定时）
- [ ] 内地服务器：页脚预留备案号位置（备案下来后填，链接到 beian.miit.gov.cn）

## 常见故障速查

| 现象 | 大概率原因 |
|------|-----------|
| SSH 连不上 | 安全组没放 22 / 用了内网 IP / 凭证错（云助手通道不受影响，可继续操作） |
| IP 通域名不通 | 解析未生效，或域名未备案被阻断 |
| 网站间歇性挂 | 内存不足（free -h）、无进程守护 |
| 数据库拒连 | 服务没起、密码错、监听地址不对 |
| 访问慢 | 带宽跑满（控制台监控）、图片未压缩 |
