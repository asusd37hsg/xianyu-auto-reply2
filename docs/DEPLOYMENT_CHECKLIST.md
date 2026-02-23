# 商品发货配置功能 - 部署检查清单

## ✅ 已完成的工作

### 1. 数据库层面
- [x] 在 `item_info` 表添加 `delivery_card_id` 字段
- [x] 添加 `update_item_delivery_card()` 方法
- [x] 添加 `get_item_delivery_card()` 方法
- [x] 数据库迁移会在首次启动时自动执行

### 2. 后端API
- [x] 添加 `GET /items/{cookie_id}/{item_id}/delivery-config` 接口
- [x] 添加 `PUT /items/{cookie_id}/{item_id}/delivery-config` 接口
- [x] 修改自动发货逻辑，支持商品级别配置优先

### 3. 前端界面
- [x] 在商品管理表格添加"发货配置"列
- [x] 实现发货配置弹窗
- [x] 添加卡券选择功能
- [x] 显示配置状态
- [x] 前端已构建并部署到 `static` 目录

### 4. 类型定义
- [x] 在 `Item` 接口添加 `delivery_card_id` 字段
- [x] 添加前端API函数类型定义

## 📋 部署步骤

### 1. 备份数据库（重要！）
```bash
# 备份数据库文件
cp xianyu_auto_reply.db xianyu_auto_reply.db.backup_$(date +%Y%m%d_%H%M%S)
```

### 2. 停止服务
```bash
# 停止正在运行的服务
# 根据你的部署方式选择合适的命令
```

### 3. 更新代码
```bash
# 拉取最新代码
git pull

# 或者手动复制更新的文件
```

### 4. 安装依赖（如果需要）
```bash
# 后端依赖
pip install -r requirements.txt

# 前端依赖（如果需要重新构建）
cd frontend
npm install
npm run build
cd ..
```

### 5. 启动服务
```bash
# 启动服务
python reply_server.py
```

### 6. 验证部署

#### 6.1 检查数据库字段
```python
import sqlite3
conn = sqlite3.connect('xianyu_auto_reply.db')
cursor = conn.cursor()
cursor.execute("PRAGMA table_info(item_info)")
columns = cursor.fetchall()
print([col[1] for col in columns])
# 应该包含 'delivery_card_id'
```

#### 6.2 检查API接口
```bash
# 测试获取发货配置接口
curl -X GET "http://localhost:8000/items/{cookie_id}/{item_id}/delivery-config" \
  -H "Authorization: Bearer YOUR_TOKEN"

# 测试更新发货配置接口
curl -X PUT "http://localhost:8000/items/{cookie_id}/{item_id}/delivery-config" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"card_id": 1}'
```

#### 6.3 检查前端界面
1. 访问 `http://localhost:8000`
2. 登录系统
3. 进入"商品管理"页面
4. 检查是否有"发货配置"列
5. 点击"配置"按钮，检查弹窗是否正常显示
6. 选择卡券并保存，检查是否成功

#### 6.4 检查自动发货功能
1. 创建测试订单
2. 查看日志，确认是否使用商品配置的卡券
3. 验证发货内容是否正确

## 🔍 故障排查

### 问题1：数据库字段未添加
**症状**：启动时报错 `no such column: delivery_card_id`

**解决方案**：
```python
# 手动添加字段
import sqlite3
conn = sqlite3.connect('xianyu_auto_reply.db')
cursor = conn.cursor()
cursor.execute("ALTER TABLE item_info ADD COLUMN delivery_card_id INTEGER")
conn.commit()
conn.close()
```

### 问题2：前端界面未更新
**症状**：商品管理页面没有"发货配置"列

**解决方案**：
```bash
# 清除浏览器缓存
# 或者强制刷新（Ctrl+F5）

# 重新构建前端
cd frontend
npm run build
cd ..
rm -rf static
cp -r frontend/dist static
```

### 问题3：API接口404
**症状**：调用发货配置接口返回404

**解决方案**：
- 检查 `reply_server.py` 是否包含新增的API接口
- 重启服务
- 检查日志输出

### 问题4：自动发货未使用商品配置
**症状**：配置了商品卡券但仍使用发货规则

**解决方案**：
- 检查卡券是否已启用
- 检查日志输出，查看是否有错误信息
- 验证 `XianyuAutoAsync.py` 中的自动发货逻辑是否正确

## 📊 监控指标

### 1. 数据库监控
- 检查 `delivery_card_id` 字段的使用情况
- 统计配置了发货卡券的商品数量

### 2. 日志监控
关键日志关键词：
- `✅ 检测到商品级别的发货配置`
- `✅ 使用商品配置的卡券`
- `🎯 使用商品级别发货配置，跳过发货规则匹配`
- `⚠️ 商品配置的卡券不存在或已禁用`

### 3. 性能监控
- API响应时间
- 自动发货处理时间
- 数据库查询性能

## 🎯 测试用例

### 测试用例1：基础配置
1. 创建一个卡券（文本类型）
2. 为商品配置该卡券
3. 验证配置保存成功
4. 验证界面显示"已配置"

### 测试用例2：自动发货
1. 配置商品发货卡券
2. 创建测试订单
3. 验证自动发货使用商品配置的卡券
4. 验证发货内容正确

### 测试用例3：取消配置
1. 为商品配置卡券
2. 取消配置（选择"不配置"）
3. 验证配置已清除
4. 验证自动发货使用发货规则

### 测试用例4：多规格商品
1. 创建多规格卡券
2. 为多规格商品配置该卡券
3. 创建不同规格的订单
4. 验证规格匹配逻辑

### 测试用例5：回退机制
1. 配置一个已禁用的卡券
2. 创建订单
3. 验证自动回退到发货规则
4. 检查日志输出

## 📝 回滚计划

如果部署后出现严重问题，可以按以下步骤回滚：

### 1. 停止服务
```bash
# 停止服务
```

### 2. 恢复数据库
```bash
# 恢复数据库备份
cp xianyu_auto_reply.db.backup_YYYYMMDD_HHMMSS xianyu_auto_reply.db
```

### 3. 恢复代码
```bash
# 回滚到上一个版本
git checkout <previous_commit>

# 或者恢复备份的文件
```

### 4. 重启服务
```bash
# 启动服务
python reply_server.py
```

## ✅ 部署完成确认

- [ ] 数据库字段已添加
- [ ] API接口正常工作
- [ ] 前端界面显示正常
- [ ] 自动发货功能正常
- [ ] 日志输出正确
- [ ] 测试用例全部通过
- [ ] 性能指标正常
- [ ] 备份已完成

## 📞 联系方式

如有问题，请联系开发团队或提交 Issue。
