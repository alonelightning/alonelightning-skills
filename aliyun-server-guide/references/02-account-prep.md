# 环节二：账号与凭证准备

目标：让 agent 具备替用户操作阿里云的能力。四个步骤：注册账号 → 实名认证 → 创建 AccessKey → 配置阿里云 CLI。前两步**只能用户本人做**（人工指引），后两步半自动。

> 播报模板（进入本环节时）：
> 「在我能帮你自动买服务器、自动部署之前，需要先准备好你的阿里云账号和一把"钥匙"（叫 AccessKey）。注册和实名认证必须你本人做，我一步步教你；之后配置钥匙的部分我来。」

## 步骤 1：注册阿里云账号 【人工】

- **自动化**：❌ 注册涉及手机短信验证，必须本人。
- **指引**：
  1. 打开 https://account.aliyun.com/register/qr_register.htm （或 aliyun.com 右上角"免费注册"）
  2. 手机号注册，或用支付宝/淘宝账号快捷登录。
  3. 设置登录密码，牢记。
- **验证**：让用户确认能登录 https://home.console.aliyun.com （控制台首页）。

## 步骤 2：实名认证 【人工，关键前置】

- **自动化**：❌ 涉及身份证+人脸核验，必须本人。
- **指引**：
  1. 打开 https://account.console.aliyun.com/v2/#/authc/home （账号中心 → 实名认证）
  2. 个人：身份证 + 支付宝/App 人脸认证，几分钟完成。
  3. 企业：营业执照 + 法人信息（企业建站/企业备案必须用企业实名）。
- ⚠️ 提醒用户：**备案主体要和账号实名一致**。给公司建站就一开始用企业实名，避免备案时主体对不上。
- **验证**：让用户回报"已认证"状态；或后续 CLI 配好后调任意 API 不报实名错误即间接验证。

## 步骤 3：创建 AccessKey 【人工创建，agent 指引】【需同意】

- **自动化**：❌ AK 创建在控制台完成且 Secret 只显示一次，必须用户操作。
- **先征求同意，话术**：
  > 「AccessKey 是操作你阿里云账户的钥匙，我拿到后可以帮你查价、下单（下单前仍会先问你）、配置服务器。它权限很大，所以：① 建议创建一个权限受限的子账号钥匙而不是主账号钥匙；② 你随时可以在控制台禁用它。你同意创建并交给我使用吗？」
- **推荐路径（RAM 子用户 AK，安全）**：
  1. 打开 RAM 控制台 https://ram.console.aliyun.com/users → 「创建用户」
  2. 登录名随意（如 `agent-ops`），勾选「使用永久 AccessKey 访问」→ 创建
  3. **立即保存** AccessKey ID 和 Secret（Secret 只显示一次）
  4. 给该用户「添加权限」，按需授予（最小集）：
     - 买服务器/部署：`AliyunECSFullAccess`
     - 域名解析：`AliyunDNSFullAccess`
     - 查账单余额（可选）：`AliyunBSSReadOnlyAccess`
     - 域名注册（如需 API 买域名）：`AliyunDomainFullAccess`
     - VPC（首次开通 ECS 需要建 VPC）：`AliyunVPCFullAccess`
  5. 官方文档：https://help.aliyun.com/zh/ram/user-guide/create-an-accesskey-pair
- **偷懒路径（主账号 AK）**：能用但明确告知风险，不推荐。
- **安全纪律（agent 必须遵守）**：Secret 拿到后只用于配置 CLI，**不在聊天/文件中明文复述**；任务完成后建议用户到 RAM 控制台禁用或删除该 AK。

## 步骤 4：安装并配置阿里云 CLI 【自动】

- **自动化**：✅ agent 在自己可用的机器上安装配置。
- **自动路径（agent 执行）**：
  - Windows：从 GitHub Releases 下载 `aliyun-cli-windows-amd64.zip` 解压到 PATH，或 `winget install Alibaba.AlibabaCloudCLI`
  - macOS：`brew install aliyun-cli`
  - Linux：`curl -sL https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz | tar xz && sudo mv aliyun /usr/local/bin/`
  - 配置（非交互，避免把 Secret 打进聊天记录，让用户私下提供或走安全通道）：
    ```
    aliyun configure set --profile default --mode AK --region cn-hangzhou --access-key-id <ID> --access-key-secret <SECRET>
    ```
  - 官方安装文档：https://help.aliyun.com/zh/cli/install-update-alibaba-cloud-cli
- **验证（agent 执行）**：
  ```
  aliyun ecs DescribeRegions
  ```
  返回地域列表即配置成功。报 `InvalidAccessKeyId` → AK 抄错；报 `Forbidden`/`NoPermission` → 该 RAM 用户缺权限，回步骤 3 补授权。
- **手动兜底**：若 agent 环境装不了 CLI → 指引用户在自己电脑装（同上命令），由用户复制粘贴 agent 给出的每条命令并回报输出；或全流程退化为纯控制台手动指引模式（各环节手动路径）。

## 步骤 5（可选）：确认账户余额 【自动查询/人工充值】

- API 下单（RunInstances 包年包月）从账户余额/优惠券扣款，余额不足会下单失败。
- **自动查询**（需 BSS 只读权限）：`aliyun bssopenapi QueryAccountBalance`
- **充值**：❌ 只能用户本人 → https://usercenter2.aliyun.com/finance/fund-management/recharge
- 报价确定后提醒用户先充够钱，或选择走控制台手动下单（支付宝直接付，见环节三）。

## 完成标志

- [ ] 账号可登录、已实名（类型与建站主体一致）
- [ ] CLI 配置完成，`aliyun ecs DescribeRegions` 正常返回
- [ ] 用户已知晓 AK 的风险与禁用方法
