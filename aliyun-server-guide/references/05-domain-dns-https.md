# 环节五：域名、解析与 HTTPS

目标：用户拥有域名 → 解析到服务器 → HTTPS 正常访问。

## 步骤 1：查询域名是否可注册 【自动】

- **自动**（需域名 API 权限；域名 API 固定走杭州 endpoint）：
  ```
  aliyun domain CheckDomain --DomainName example.com --endpoint domain.aliyuncs.com
  ```
  `Avail=1` 可注册。可批量帮用户查几个候选。
- **手动兜底**：https://wanwang.aliyun.com/domain/ 搜索框直接查。

## 步骤 2：购买域名 【推荐人工，需同意（花钱）】

- **自动化现状（已核实）**：阿里云有域名注册 OpenAPI（`SaveSingleTaskForCreatingOrderActivate`），但前提是账户已创建**已实名的域名信息模板**且余额充足，流程链条长；对一次性购买的新手，控制台更直观。**默认走手动指引**。
- **手动指引**：
  1. https://wanwang.aliyun.com/domain/ 搜索心仪域名（.com 约 70~80 元/年，首年常有优惠，续费标准价，以页面为准）
  2. 加入清单 → 结算。首次买需创建「信息模板」（持有人信息）并完成**模板实名认证**（身份证，通常几小时~1 天过审）。
  3. ⚠️ 域名持有人实名必须与备案主体一致（个人备案→个人模板；企业备案→企业模板）。
  4. 支付【用户本人】。
- **验证**【自动】：`aliyun domain QueryDomainList --endpoint domain.aliyuncs.com` 或让用户回报。

## 步骤 3：域名解析 【自动】【切换线上解析需同意】

⚠️ 内地服务器：域名未备案时 HTTP 访问会被阻断。**备案通过前不要解析主域名对外**，先用 IP 调试；备案通过后再做本步骤。（海外/香港服务器无此限制，随时解析。）

- **自动**（云解析 DNS API，需 AliyunDNSFullAccess）：
  ```
  # 添加 A 记录：主域名 和 www 都指向服务器公网 IP
  aliyun alidns AddDomainRecord --DomainName example.com --RR @ --Type A --Value <公网IP>
  aliyun alidns AddDomainRecord --DomainName example.com --RR www --Type A --Value <公网IP>

  # 查看现有记录
  aliyun alidns DescribeDomainRecords --DomainName example.com
  ```
  - 若域名不在阿里云买的：需先把域名 DNS 服务器改到阿里云解析或在原注册商处加 A 记录（给对应指引）。
  - **修改/删除已有解析记录 = 影响线上访问 → 必须先征求同意。**
- **手动兜底**：https://dns.console.aliyun.com → 域名 → 解析设置 → 添加记录：类型 A、主机记录 `@` 和 `www`、记录值填公网 IP。
- **验证**【自动】：`nslookup example.com` / `Resolve-DnsName example.com` 返回服务器 IP 即生效（通常几分钟）。

## 步骤 4：HTTPS 证书 【自动】

备案通过、域名可正常访问后做。

- **自动（推荐，Let's Encrypt，免费自动续期）**——通过云助手/SSH 在服务器执行：
  ```bash
  apt install -y certbot python3-certbot-nginx
  certbot --nginx -d example.com -d www.example.com --non-interactive --agree-tos -m <用户邮箱> --redirect
  ```
  前置：80/443 已放行、域名解析已生效、Nginx 配置里 server_name 填了域名。
- **备选**：阿里云「数字证书管理服务」可申领免费个人测试证书（有效期短、需定期换、额度政策以官网为准）：https://yundun.console.aliyun.com/?p=cas ——需要用户控制台点选，属手动路径。
- **宝塔用户**：面板「网站 → SSL」一键申请【指引用户点】。
- **验证**【自动】：`curl -I https://example.com` 返回 200 且无证书错误；提醒用户浏览器访问确认小锁图标。

## 完成检查

- [ ] IP 访问 ✅ → 域名访问 ✅ → HTTPS ✅
- [ ] http 自动跳转 https（certbot --redirect 已处理）
- [ ] 证书自动续期已就位（certbot 装机自带 systemd timer，可 `systemctl list-timers | grep certbot` 验证）
- [ ] 内地站点：页脚已放备案号并链接 beian.miit.gov.cn；30 天内完成公安备案（见环节六）
