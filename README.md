# 寄件服务MCP Server

提供寄件订单管理、地址簿查询等功能的Model Context Protocol (MCP) Server，支持通过HTTP API对接远程寄件服务。

## 功能特性

### 🏠 地址簿管理
- 查询用户保存的地址簿数据
- 支持关键词搜索（姓名、电话、地址）
- 标识默认地址

### 📦 寄件订单管理
- 创建新的寄件订单
- 分页查询订单列表
- 查询订单详细信息
- 支持多种服务类型（标准/快速/经济）
- 订单状态追踪

### 🔍 高级查询
- 按日期范围过滤订单
- 按订单状态过滤
- 分页浏览订单列表
- 完整物流追踪记录

### 🌐 API集成
- 支持HTTP API对接真实寄件服务
- 灵活的配置管理（命令行参数/环境变量）
- Mock模式用于开发和测试
- 完善的错误处理和重试机制

## 安装

### 使用 uvx（推荐）
```bash
# 直接运行，无需安装
uvx guoguo-mcp-server --help

# 如果遇到包未找到的问题，可以指定索引
uvx --index-url https://pypi.org/simple/ guoguo-mcp-server --help
```

### 使用 pip
```bash
pip install guoguo-mcp-server
```

> **注意**: 新发布的包可能需要几分钟时间在各个 PyPI 镜像中同步。如果遇到包未找到的错误，请稍等片刻再试，或使用 pip 安装后直接使用命令。

## 使用方法

### 配置API参数

#### 方法1: 通过命令行参数
```bash
# 使用真实API
guoguo-mcp-server --api-url https://api.sf-express.com --token your_access_token --no-mock

# 或使用 uvx
uvx guoguo-mcp-server --api-url https://api.sf-express.com --token your_access_token --no-mock

# 使用模拟数据（开发测试）
guoguo-mcp-server --use-mock
```

#### 方法2: 通过环境变量
```bash
export SHIPPING_API_BASE_URL="https://api.sf-express.com"
export SHIPPING_API_TOKEN="your_access_token"
export SHIPPING_USE_MOCK="false"
guoguo-mcp-server
```

#### 方法3: 在Claude Desktop中配置

**使用 uvx（推荐，无需预安装）:**
```json
{
  "mcpServers": {
    "guoguo-mcp-server": {
      "command": "uvx",
      "args": [
        "guoguo-mcp-server",
        "--api-url", "https://api.sf-express.com",
        "--token", "your_access_token",
        "--no-mock"
      ]
    }
  }
}
```

**使用已安装的包:**
```json
{
  "mcpServers": {
    "guoguo-mcp-server": {
      "command": "guoguo-mcp-server",
      "args": [
        "--api-url", "https://api.sf-express.com",
        "--token", "your_access_token",
        "--no-mock"
      ]
    }
  }
}
```

**使用模拟模式（开发测试）:**
```json
{
  "mcpServers": {
    "guoguo-mcp-server": {
      "command": "uvx",
      "args": [
        "guoguo-mcp-server",
        "--use-mock"
      ]
    }
  }
}
```

### 开发模式运行
```bash
# 使用模拟数据（默认）
guoguo-mcp-server --use-mock

# 使用 uvx（无需安装）
uvx guoguo-mcp-server --use-mock

# 使用 mcp 开发工具
mcp dev guoguo-mcp-server -- --use-mock
```

### 生产模式运行
```bash
# 连接真实API
guoguo-mcp-server \
  --api-url https://api.your-shipping-provider.com \
  --token your_production_token \
  --no-mock

# 或使用 uvx
uvx guoguo-mcp-server \
  --api-url https://api.your-shipping-provider.com \
  --token your_production_token \
  --no-mock
```

## 工具列表

### 1. 地址簿查询 (query_address_book)
查询用户的地址簿信息

**参数:**
- `search_key` (可选): 搜索关键词，可以搜索姓名、电话或地址

**返回:**
```json
{
  "status": "success",
  "total_count": 4,
  "addresses": [
    {
      "name": "张三",
      "phone": "13800138000",
      "address": "北京市朝阳区中关村大街1号",
      "is_default": true
    }
  ]
}
```

### 2. 创建寄件订单 (create_shipping_order)
创建新的寄件订单

**参数:**
- `sender_name`: 寄件人姓名
- `sender_phone`: 寄件人电话
- `sender_address`: 寄件人地址
- `receiver_name`: 收件人姓名
- `receiver_phone`: 收件人电话
- `receiver_address`: 收件人地址
- `package_type`: 物品类型
- `package_weight`: 重量(kg)
- `package_size`: 尺寸(长x宽x高,cm)
- `service_type`: 服务类型(标准/快速/经济)

**返回:**
```json
{
  "status": "success",
  "order_id": "SF2024003",
  "estimated_cost": 18.4,
  "estimated_delivery": "2024-01-15 14:30:00",
  "pickup_time": "24小时内安排取件"
}
```

### 3. 查询订单列表 (list_shipping_orders)
分页查询寄件订单列表

**参数:**
- `page_size`: 每页数量（默认10）
- `page_number`: 页码，从1开始（默认1）
- `start_date`: 开始日期(YYYY-MM-DD格式)
- `end_date`: 结束日期(YYYY-MM-DD格式)
- `status`: 订单状态过滤

**返回:**
```json
{
  "status": "success",
  "orders": [...],
  "pagination": {
    "total_count": 2,
    "total_pages": 1,
    "current_page": 1,
    "page_size": 10,
    "has_next": false,
    "has_prev": false
  }
}
```

### 4. 查询订单详情 (get_order_detail)
获取订单详细信息

**参数:**
- `order_id`: 订单号
- `detail_level`: 详情级别(basic/full)

**返回:**
```json
{
  "status": "success",
  "order_info": {
    "order_id": "SF2024001",
    "status": "已签收",
    "create_time": "2024-01-13 10:30:00",
    "service_type": "快速"
  },
  "sender_info": {...},
  "receiver_info": {...},
  "package_info": {...},
  "tracking_history": [...]
}
```

## 订单状态说明

- **待取件**: 订单已创建，等待快递员取件
- **运输中**: 包裹正在运输途中
- **派送中**: 包裹已到达目的地，正在派送
- **已签收**: 包裹已成功签收
- **已退回**: 包裹已退回寄件人
- **已取消**: 订单已取消

## 服务类型说明

- **标准**: 48小时送达，费用标准
- **快速**: 24小时送达，费用1.5倍
- **经济**: 72小时送达，费用0.8倍

## 开发

### 项目结构
```
guoguo-mcp-server/
├── src/
│   └── guoguo_mcp/
│       └── __init__.py      # 主服务器代码
├── requirements.txt         # Python依赖
├── pyproject.toml          # 项目配置
├── test_client.py          # 测试客户端
├── run_server.py           # 启动脚本
└── README.md               # 说明文档
```

### 代码说明

服务器使用了以下主要组件：
- **FastMCP**: MCP协议的Python实现
- **数据模型**: 使用dataclass定义订单、地址等数据结构
- **枚举类型**: 定义订单状态和服务类型
- **模拟数据**: 内存中存储示例数据，实际应用中可替换为数据库

### 扩展功能

可以进一步扩展的功能：
- 集成真实的快递API
- 添加数据库持久化
- 实现用户认证
- 添加订单状态更新功能
- 集成支付功能
- 添加物流追踪API

## 许可证

MIT License

## 贡献

欢迎提交Issue和Pull Request来改进这个项目。
