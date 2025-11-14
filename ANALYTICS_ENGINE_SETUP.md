# Analytics Engine 设置指南

本文档提供了配置Cloudflare Analytics Engine以支持API请求日志记录功能的说明。

## 1. 创建Analytics Engine数据集

首先，您需要在Cloudflare仪表板中创建一个Analytics Engine数据集：

1. 登录到 [Cloudflare仪表板](https://dash.cloudflare.com/)
2. 导航到 **Analytics** > **Analytics Engine**
3. 点击 **Create Dataset** 按钮
4. 为数据集输入一个名称（例如：`ai_gateway_logs`）
5. 点击 **Create** 完成创建

## 2. 通过Cloudflare仪表板配置Worker与Analytics Engine绑定（可选，和下一步二选一，不建议）

不建议使用该方法，因为如果绑定了github部署，每次代码更改需要重新绑定。

1. **返回Worker页面**
   - 在左侧导航菜单中，点击 "Workers & Pages"，然后选择您的Worker

2. **添加Analytics Engine绑定**
   - 在Worker页面中，点击 "设置" 或 "绑定" 标签
   - 点击 "Analytics Engine绑定" 按钮
   - 点击 "添加绑定" 按钮
   - 在 "变量名" 字段输入 `LOGS`（必须与代码中的 `env.LOGS` 保持一致）
   - 在 "数据集" 下拉菜单中选择您刚刚创建的数据集
   - 点击 "保存" 按钮完成绑定

## 3. 配置wrangler.toml(可选，和上一步二选一，建议)

确保您的 `wrangler.toml` 文件中已正确配置Analytics Engine数据集绑定：

```toml
[[analytics_engine_datasets]]
binding = "LOGS"                # 这是您代码中 `env.LOGS` 的名字，必须与要求一致
dataset = "ai_gateway_logs"     # 数据集的名称，与您在步骤1中创建的名称一致
```

## 4. 了解日志记录格式

在 `_worker.js` 文件中，日志记录功能会自动收集以下信息：

```javascript
const dataPoint = {
  // 按照要求配置数据点
  // index1存储服务商
  indexes: [
    service
  ],
  // blob1存储使用服务商，blob2存储模型，blob3存储报错信息
  blobs: [
    service,      // blob1: 服务商
    model,        // blob2: 模型
    errorMessage  // blob3: 报错信息
  ],
  // double1存储状态码，double2存储耗时
  doubles: [
    response.status, // double1: HTTP状态码
    latencyMs        // double2: 耗时（毫秒）
  ],
};
```

## 5. 使用Analytics Engine查询日志

创建并绑定数据集后，您可以使用Cloudflare的SQL查询功能来分析日志数据：

1. 登录到 [Cloudflare仪表板](https://dash.cloudflare.com/)
2. 导航到 **Analytics** > **Analytics Engine**
3. 选择您创建的数据集
4. 使用SQL查询编辑器编写查询，例如：

```sql
SELECT * FROM ai_gateway_logs
WHERE time > now() - 1d
ORDER BY time DESC
LIMIT 100;
```

## 6. 常见问题解决

如果遇到Analytics Engine相关错误，请检查以下几点：

1. 确保您的Analytics Engine数据集已正确创建
2. 确保绑定名称和数据集名称与实际配置一致
3. 确保您的Cloudflare账户有足够的权限使用Analytics Engine服务
4. 免费账户有一定的日志容量限制，如果日志数据量超过限制，部分日志可能不会被记录

## 7. 禁用日志记录

如果您不想使用日志记录功能，只需确保没有在Cloudflare仪表板中配置Analytics Engine绑定，或在wrangler.toml中删除或注释掉相关配置。Worker代码中已经包含了对 `LOGS` 绑定不存在时的处理逻辑，不会影响主要功能。
