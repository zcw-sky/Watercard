# 数据模型草案

此文档用于确定 MVP 业务结构，不是最终数据库迁移脚本。字段类型和索引在编码前按实际实现细化。

## 核心实体

### users

- `id`
- `openid`（微信用户标识，唯一）
- `nickname`（可空）
- `phone`（可空，按最终登录/下单规则确认）
- `created_at`, `updated_at`

### admins

- 管理员登录标识、密码哈希、启用状态及时间戳。
- 如采用 Filament 自带用户模型，可与普通小程序用户分表或通过角色区分，避免混用身份。

### service_categories / services

- 服务分类：名称、排序、启用状态。
- 社区服务：名称、摘要、详情、封面图、联系方式、展示状态、排序。

### announcements

- 标题、正文、封面图（可空）、发布时间、置顶标记、展示状态。

### product_categories

- 分类名称、排序、启用状态。

### products

- 商品名称、摘要、详情、封面图、价格、单位、上下架状态、排序。
- 可选字段：库存数量、配送/自提说明（是否加入待确认）。

### orders

- 订单号（唯一）。
- 用户 ID（小程序订单可关联；后台录单可空或关联客户）。
- 来源：`mini_program` / `admin`。
- 客户姓名、电话、收货地址、订单备注。
- 商品金额、订单总金额。
- 履约状态、支付状态。
- 管理员备注、创建管理员 ID（后台录入时）。
- `created_at`, `updated_at`。

### order_items

- 订单 ID、商品 ID（可空以保留商品删除后的快照）。
- 商品名称快照、单价快照、数量、行金额。

## 订单状态字段建议

- `fulfillment_status`: `pending_confirmation`, `ready`, `delivering`, `completed`, `cancelled`。
- `payment_status`: `unpaid`, `offline_paid`。
- 状态变更记录可在 MVP 后续阶段加入 `order_status_logs`；若需要审计所有后台改动，则第一版一并实现。

## 数据约束

- 金额使用整数分存储，展示时换算为元，避免浮点精度误差。
- 下单时复制商品名称、价格等快照到订单明细。
- 删除商品不级联删除历史订单。
- 订单号由服务端生成并保证唯一。
- 商品上下架和下单校验由服务端完成。
