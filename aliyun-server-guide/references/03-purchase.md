# 环节三：购买 ECS

目标：以最合适的价格拿到一台「运行中、有公网 IP、安全组正确、能登录」的服务器。

**两条购买路线，先做路线决策：**

| 路线 | 适用 | 价格 | 自动化程度 |
|------|------|------|-----------|
| A. 活动页手动购买 | 新用户（首购特惠通常比 API 标准价便宜一半以上） | 活动价（如历史上的 99 元/年 2核2G） | ❌ 手动（活动套餐不走 API），agent 逐步指引 |
| B. API 自动下单 | 老用户/无合适活动/用户嫌麻烦愿付标准价 | 标准价（自动用可用优惠券） | ✅ 全自动 |

**决策规则**：新注册账号 → 先带用户看活动页（路线 A），价格差距大时如实告知；活动页无合适套餐或用户明确要 agent 代办 → 路线 B。

---

## 路线 A：活动页手动购买（agent 指引，用户操作）

1. 【agent】让用户打开活动页：https://www.aliyun.com/minisite/goods （云小站/特惠专区）或 aliyun.com 首页「最新活动」。活动入口常变，找不到就在官网顶部搜「新人特惠」。
2. 【用户】找与选型结论匹配的套餐（如 2核2G e 实例 / 2核4G u1），核对：地域可选、带宽、系统盘、**续费价格**（活动价常是首年价）。
3. 【用户】选好地域、镜像（Ubuntu 24.04 64位）、时长（1年）→ 下单支付（支付宝等）。**支付必须本人**。
4. 【agent 验证】支付完成后自动核查：
   ```
   aliyun ecs DescribeInstances --RegionId <region> --output cols=InstanceId,InstanceName,Status,PublicIpAddress rows=Instances.Instance[]
   ```
   看到新实例 `Running` + 公网 IP 即成功。
5. 活动机通常用密码登录：提醒用户在控制台「重置实例密码」设置强密码（或后续 agent 用云助手免密操作，见环节四）。
6. 跳到本文「购买后统一收尾」。

## 路线 B：API 自动下单（agent 执行）

### B0. 前置检查 【自动】

```
# 该地域可用的目标规格与库存
aliyun ecs DescribeAvailableResource --RegionId cn-hangzhou --DestinationResource InstanceType --InstanceChargeType PrePaid --InstanceType ecs.e-c1m1.large

# 当前系统镜像（取最新 Ubuntu LTS 的 ImageId）
aliyun ecs DescribeImages --RegionId cn-hangzhou --OSType linux --ImageOwnerAlias system --output cols=ImageId,OSName rows=Images.Image[]

# 是否已有默认 VPC/交换机/安全组
aliyun ecs DescribeSecurityGroups --RegionId cn-hangzhou
aliyun vpc DescribeVpcs --RegionId cn-hangzhou
```
- 新账号无 VPC：RunInstances 不指定 VSwitchId 时部分地域可自动创建默认 VPC；保险起见可先建：`aliyun vpc CreateDefaultVpc --RegionId cn-hangzhou`（需 VPC 权限）。
- **手动兜底**：查询失败 → 指引用户在 ECS 售卖页 https://ecs-buy.aliyun.com/ 按选型逐项选择（页面会实时显示价格），走到支付页即等同路线 A 的 3~5 步。

### B1. 询价并报价 【自动查询 → 需同意】

```
aliyun ecs DescribePrice --RegionId cn-hangzhou --ResourceType instance --InstanceType ecs.e-c1m1.large --ImageId <ImageId> --InstanceNetworkType vpc --InternetChargeType PayByBandwidth --InternetMaxBandwidthOut 3 --SystemDisk.Category cloud_essd_entry --SystemDisk.Size 40 --PriceUnit Year --Period 1
```
向用户报价话术：
> 「这套配置 API 标准价是一年 X 元（可能自动叠加你账户里的优惠券）。注意：如果你是新用户，活动页可能更便宜，要不要先看看活动页？确认要我直接下单吗？下单会从你的阿里云账户余额扣款。」

**用户明确同意 + 余额充足，才进入 B2。**

### B2. 创建密钥对与安全组 【自动，安全组开端口属敏感操作已含在下单同意内】

```
# SSH 密钥对（PrivateKeyBody 只返回一次，立即存为 .pem 文件并提醒用户备份）
aliyun ecs CreateKeyPair --RegionId cn-hangzhou --KeyPairName web-server-key

# 安全组 + 放行 22/80/443
aliyun ecs CreateSecurityGroup --RegionId cn-hangzhou --VpcId <VpcId> --SecurityGroupName web-sg
aliyun ecs AuthorizeSecurityGroup --RegionId cn-hangzhou --SecurityGroupId <sg-id> --IpProtocol tcp --PortRange 22/22 --SourceCidrIp 0.0.0.0/0
aliyun ecs AuthorizeSecurityGroup --RegionId cn-hangzhou --SecurityGroupId <sg-id> --IpProtocol tcp --PortRange 80/80 --SourceCidrIp 0.0.0.0/0
aliyun ecs AuthorizeSecurityGroup --RegionId cn-hangzhou --SecurityGroupId <sg-id> --IpProtocol tcp --PortRange 443/443 --SourceCidrIp 0.0.0.0/0
```
数据库端口（3306 等）**绝不**对公网开放。
- **手动兜底**：控制台 → ECS → 网络与安全 → 密钥对/安全组，页面操作路径逐步告知。

### B3. 下单 【自动，已获同意】

```
aliyun ecs RunInstances --RegionId cn-hangzhou --ImageId <ImageId> --InstanceType ecs.e-c1m1.large --SecurityGroupId <sg-id> --VSwitchId <vsw-id> --InstanceChargeType PrePaid --Period 1 --PeriodUnit Year --InternetChargeType PayByBandwidth --InternetMaxBandwidthOut 3 --SystemDisk.Category cloud_essd_entry --SystemDisk.Size 40 --KeyPairName web-server-key --InstanceName my-web-server
```
要点（已核实官方文档）：
- RunInstances 支持 `InstanceChargeType=PrePaid`（包年包月），付款自动使用可用优惠券，从账户余额扣款；余额不足报 `InvalidPayMethod`/欠费类错误 → 让用户充值（环节二步骤 5）或改走路线 A。
- `InternetMaxBandwidthOut > 0` 自动分配公网 IP。
- 异步接口：返回 InstanceId 后轮询状态：
  ```
  aliyun ecs DescribeInstanceStatus --RegionId cn-hangzhou --InstanceId.1 <id>
  ```
  `Running` 即创建成功。
- **自动续费**：默认不开。问用户是否要开（`--AutoRenew true`），如实说明"到期自动扣款"的含义。
- **手动兜底**：下单失败 2 次 → 指引用户走 https://ecs-buy.aliyun.com/ 控制台购买页，按选型结论逐项选，agent 陪跑核对每一项，用户支付。

### B4. 获取公网 IP 【自动】

```
aliyun ecs DescribeInstances --RegionId cn-hangzhou --InstanceIds '["<id>"]'
```
取 `PublicIpAddress` 记录备用。

---

## 购买后统一收尾（两条路线共用）

- [ ] 实例 `Running`、有公网 IP —— agent 用 DescribeInstances 验证
- [ ] 安全组已放行 22/80/443 —— agent 用 `aliyun ecs DescribeSecurityGroupAttribute --SecurityGroupId <id> --RegionId <region>` 验证；缺则补（AuthorizeSecurityGroup 或指引控制台）
- [ ] 登录凭证已妥存（.pem 或密码），提醒用户自己也备份一份
- [ ] 【建议，涉及告警非花钱】指引用户设置费用预警：https://usercenter2.aliyun.com/finance/expense-report → 费用预警
- [ ] 内地地域 + 要绑域名 → **当天启动环节五（买域名）+ 环节六（备案）**，审核期间并行做环节四部署

播报模板：
> 「服务器买好了 ✅ 公网 IP 是 x.x.x.x，22/80/443 端口已放行。接下来我可以直接帮你把网站环境装好——这一步不花钱，但会在服务器上安装软件，可以开始吗？」
