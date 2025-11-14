# API密钥轮询功能配置指南

本文档提供了配置Cloudflare D1数据库、MASTER_KEY和各服务商API密钥以支持API密钥轮询功能的详细说明。

## 1. 表名配置

在 `_worker.js` 文件中，表名已设置为常量，您可以根据实际数据库中的表名进行修改：

```javascript
// 表名常量，方便修改
const ROTATION_STATE_TABLE = 'rotation_state'; // 用户可以根据实际数据库表名修改
```

默认表名为 `rotation_state`，您可以根据自己的需求修改这个值。

## 2. 通过Cloudflare仪表板创建D1数据库（可选，和下一步二选一，建议）

以下是通过Cloudflare仪表板创建D1数据库的详细步骤：

1. **创建D1数据库**
   - 在Workers & Pages页面中，点击左侧菜单中的 "D1"
   - 点击 "创建数据库" 按钮
   - 输入数据库名称（例如：`api-rotation-db`）
   - 选择数据库区域（建议选择离您的目标用户较近的区域）
   - 点击 "创建" 按钮完成数据库创建

2. **创建表结构**
   - 在新创建的数据库页面中，点击 "查询" 标签
   - 在查询编辑器中粘贴以下SQL语句：
     ```sql
     CREATE TABLE IF NOT EXISTS rotation_state (
       service_name TEXT PRIMARY KEY,
       next_index INTEGER NOT NULL DEFAULT 1,
       last_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
     );
     ```
   - 点击 "运行" 按钮执行SQL语句，创建表格

4. **查看数据库ID**
   - 表格创建成功后，点击数据库页面上的 "设置" 标签
   - 找到并复制 "数据库ID"，后续配置会用到

## 3. 使用Wrangler CLI初始化数据库(可选，和上一步二选一，不建议)

如果您偏好使用命令行工具，也可以使用Wrangler CLI执行上述操作：

```bash
# 安装Wrangler CLI（如果尚未安装）
npm install -g wrangler

# 登录Cloudflare账户
wrangler login

# 创建新数据库
wrangler d1 create api-rotation-db

# 执行初始化SQL到远程数据库
wrangler d1 execute api-rotation-db --remote --command="CREATE TABLE IF NOT EXISTS rotation_state (service_name TEXT PRIMARY KEY, next_index INTEGER NOT NULL DEFAULT 1, last_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP);"

# 或执行初始化SQL到本地开发数据库
wrangler d1 execute api-rotation-db --local --command="CREATE TABLE IF NOT EXISTS rotation_state (service_name TEXT PRIMARY KEY, next_index INTEGER NOT NULL DEFAULT 1, last_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP);"
```

## 4. 通过Cloudflare仪表板配置Worker与数据库绑定（可选，和下一步二选一，不建议）

不建议使用该方法，因为如果绑定了github部署，每次代码更改需要重新绑定。

1. **返回Worker页面**
   - 在左侧导航菜单中，点击 "Workers & Pages"，然后选择您的Worker

2. **添加D1绑定**
   - 在Worker页面中，点击 "设置"或者"绑定" 标签
   - 点击 "添加绑定" 按钮
   - 在 "变量名" 字段输入 `DB`（必须与代码中的 `env.DB` 保持一致）
   - 在 "数据库" 下拉菜单中选择您刚刚创建的数据库
   - 点击 "保存" 按钮完成绑定

## 5. 配置wrangler.toml(可选，和上一步二选一，建议)

确保您的 `wrangler.toml` 文件中已正确配置D1数据库绑定：

```toml
[[d1_databases]]
binding = "DB"                  # 这是您代码中 `env.DB` 的名字，必须保持一致
database_name = "api-rotation-db" # 数据库的名称
database_id = "您的数据库ID"      # 您的数据库ID
```

## 6. 配置MASTER_KEY主密钥

MASTER_KEY是启用轮询模式的控制开关，用于验证请求是否使用轮询功能。

### 配置步骤

1. 在Cloudflare Workers仪表板中，进入您的Worker设置页面
2. 导航到 "变量" 或 "Secrets" 部分
3. 添加一个名为 `MASTER_KEY` 的变量密钥
4. 设置一个强密码作为其值，建议使用随机生成的长字符串以提高安全性

### 注意事项

- MASTER_KEY是敏感信息，请妥善保管
- 所有使用轮询功能的请求必须使用MASTER_KEY作为其API密钥

## 7. 配置各服务商API密钥

为每个您想要使用轮询功能的服务提供商配置API密钥列表。

### 配置步骤

1. 在Cloudflare Workers仪表板中，进入您的Worker设置页面
2. 导航到 "变量" 或 "Secrets" 部分
3. 为想要使用轮询功能的服务提供商添加对应的密钥变量：
   - `GEMINI_KEYS`：用于Gemini API的密钥列表
   - `OPENAI_KEYS`：用于OpenAI API的密钥列表
   - `CLAUDE_KEYS`：用于Claude API的密钥列表
   - `GROQ_KEYS`：用于Groq API的密钥列表
   - `CEREBRAS_KEYS`：用于Cerebras API的密钥列表

### 密钥格式要求

每个服务商的密钥列表必须是有效的JSON数组格式，例如：

```json
["api_key_1", "api_key_2", "api_key_3"]
```

### 注意事项

- 严格遵循格式，任何不想使用轮询的服务商可以不配，但不能配置为空或者不完全
- 建议为每个服务提供商配置多个密钥以充分利用轮询功能
- 密钥是敏感信息，请妥善保管

## 8. 配置ROTATION_LIMIT轮询限制（可选）

ROTATION_LIMIT用于自定义轮询模式下的最大重试次数，当服务商返回429错误时，系统将尝试的最大API密钥数量。

### 配置步骤

1. 在Cloudflare Workers仪表板中，进入您的Worker设置页面
2. 导航到 "变量" 或 "Secrets" 部分
3. 添加一个名为 `ROTATION_LIMIT` 的变量
4. 设置一个整数值作为其值（例如：`10`）

### 注意事项

- 如果未配置ROTATION_LIMIT，系统将使用默认值 `5`
- 实际重试次数将取ROTATION_LIMIT和可用密钥数量的较小值

## 9. 使用轮询功能

配置完成后，您可以通过以下方式使用轮询功能：

1. 将请求中的原始API密钥替换为您配置的 `MASTER_KEY`
2. 发送请求到Worker，Worker会自动使用轮询算法选择一个可用的API密钥
3. 系统会自动记录每个服务提供商的密钥使用状态，并在下次请求时使用下一个可用密钥

## 10. 常见问题解决

如果遇到数据库错误或轮询功能不工作，请检查以下几点：

1. 确保您的D1数据库已正确创建并绑定到Worker
2. 确保表名在代码中和数据库中保持一致
3. 确保表结构包含必要的字段：`service_name`, `next_index`, `last_updated`
4. 确保 `service_name` 字段已设置为主键，以便 `ON CONFLICT` 子句正常工作
5. 确保已正确配置 `MASTER_KEY` 环境变量
6. 确保已为相应服务提供商配置了有效的密钥列表，且格式为JSON数组
7. 确保请求中使用的API密钥与配置的 `MASTER_KEY` 匹配

## 11. 测试数据库连接

您可以在Worker代码中添加简单的测试逻辑，以确保数据库连接正常工作：

```javascript
async function testDatabaseConnection(env) {
  if (!env.DB) {
    console.error('D1数据库未配置');
    return false;
  }
  
  try {
    const result = await env.DB.prepare('SELECT 1 as test').all();
    console.log('数据库连接成功:', result);
    return true;
  } catch (e) {
    console.error('数据库连接错误:', e);
    return false;
  }
}
```