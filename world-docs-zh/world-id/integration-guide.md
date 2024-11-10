# World ID 集成指南

## 概述

本指南将帮助您将World ID集成到您的应用程序中。World ID提供了简单且安全的方式来验证用户的真实身份，同时保护他们的隐私。

## 前置条件

1. 开发环境要求
   - Node.js 12.x 或更高版本
   - 支持的框架：React, Next.js, Vue.js等
   - 基本的Web开发知识

2. 账户准备
   - 创建World ID开发者账户
   - 获取必要的API密钥
   - 了解基本的安全实践

## 集成步骤

### 1. 安装SDK

```bash
# 使用npm
npm install @worldcoin/idkit

# 使用yarn
yarn add @worldcoin/idkit
```

### 2. 基本配置

```javascript
// React示例
import { IDKitWidget } from '@worldcoin/idkit'

const YourComponent = () => {
  const onSuccess = (proof) => {
    // 处理验证成功
    console.log(proof)
  }

  return (
    <IDKitWidget
      app_id="your-app-id"
      action="your-action"
      onSuccess={onSuccess}
    />
  )
}
```

### 3. 实现验证流程

#### 前端实现

```javascript
// 验证按钮组件
<IDKitWidget
  app_id="your-app-id"
  action="your-action"
  signal="user_value" // 可选
  onSuccess={handleSuccess}
  handleVerify={handleVerify}
  credential_types={['orb', 'phone']}
/>

// 处理成功回调
const handleSuccess = (proof) => {
  // 验证成功后的处理
  verifyProof(proof)
}

// 验证处理
const handleVerify = async (proof) => {
  const response = await fetch('/api/verify', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({ proof })
  })
  return response.ok
}
```

#### 后端验证

```javascript
// Node.js/Express示例
app.post('/api/verify', async (req, res) => {
  try {
    const { proof } = req.body
    const response = await verifyProof(proof)
    
    if (response.ok) {
      res.status(200).json({ success: true })
    } else {
      res.status(400).json({ success: false })
    }
  } catch (error) {
    res.status(500).json({ error: error.message })
  }
})

// 验证函数
async function verifyProof(proof) {
  const response = await fetch('https://developer.worldcoin.org/api/v1/verify', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${process.env.WORLD_ID_SECRET}`
    },
    body: JSON.stringify({ proof })
  })
  return response
}
```

## 安全最佳实践

### 1. API密钥管理
- 使用环境变量存储敏感信息
- 定期轮换密钥
- 永远不要在客户端代码中暴露密钥

### 2. 验证安全
- 始终在服务器端验证proof
- 实施请求速率限制
- 记录并监控验证尝试

### 3. 错误处理
- 实现全面的错误处理机制
- 提供用户友好的错误提示
- 记录详细的错误日志

## 测试和调试

### 1. 开发环境配置
```javascript
const config = {
  app_id: 'test_app_id',
  action: 'test_action',
  development: true // 启用测试模式
}
```

### 2. 测试场景
- 正常验证流程
- 验证失败情况
- 网络错误处理
- 用户取消操作

### 3. 常见问题排查
- 验证失败：检查API密钥和proof格式
- 集成问题：确认SDK版本和配置
- 网络问题：检查连接和请求设置

## 生产环境部署

### 1. 上线准备
- 更换为生产环境API密钥
- 配置错误监控系统
- 设置日志记录

### 2. 性能优化
- 实施缓存策略
- 优化API调用
- 监控系统性能

### 3. 维护和更新
- 定期更新SDK版本
- 监控安全公告
- 及时应用补丁

## 用户体验优化

### 1. 界面设计
- 提供清晰的验证流程指引
- 添加加载状态提示
- 优化错误提示信息

### 2. 性能考虑
- 异步加载组件
- 优化验证响应时间
- 实现平滑的状态转换

### 3. 可访问性
- 支持键盘导航
- 添加ARIA标签
- 确保颜色对比度

## 故障排除指南

### 常见问题解决

1. 验证失败
   - 检查API密钥配置
   - 验证proof格式
   - 检查网络连接

2. 集成问题
   - 确认SDK版本兼容性
   - 检查配置参数
   - 查看控制台错误

3. 性能问题
   - 优化API调用
   - 检查资源加载
   - 监控响应时间

## 支持资源

- [官方文档](https://docs.world.org/)
- [API参考](https://docs.world.org/api)
- [开发者社区](https://developers.world.org/)
- [示例代码库](https://github.com/worldcoin)

---

本指南提供了World ID集成的基本框架。随着平台的发展，我们会持续更新文档内容。建议定期查看官方文档以获取最新信息。
