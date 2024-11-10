# Mini Apps 快速开始指南

## MiniKit-JS SDK 简介
MiniKit-JS是World官方提供的SDK，用于创建与World App完美集成的mini apps。它提供了丰富的功能和类型定义，使开发过程变得简单高效。

## 开发环境要求
- Node.js 14.0.0 或更高版本
- npm 6.0.0 或更高版本
- 现代浏览器（用于本地开发）
- World App（用于测试）

## 项目模板
World提供了多个官方项目模板，帮助您快速开始开发：

### 1. React模板
```bash
# 使用React模板创建项目
npx create-world-app my-app --template react

# 或使用yarn
yarn create world-app my-app --template react
```
特点：
- 包含简单的后端验证系统
- React最佳实践
- TypeScript支持
- 预配置的开发环境

### 2. Vanilla JS模板
```bash
# 使用Vanilla JS模板创建项目
npx create-world-app my-app --template vanilla

# 或使用CDN方式
<script src="https://cdn.world.org/minikit-js/latest/minikit.min.js"></script>
```
特点：
- 轻量级实现
- 无框架依赖
- 简单的后端验证系统
- 易于定制

### 3. NextJS模板
```bash
# 使用NextJS模板创建项目
npx create-world-app my-app --template nextjs
```
特点：
- 服务端渲染支持
- 优化的性能
- 完整的TypeScript支持
- 集成的路由系统

## 项目结构
```
my-world-app/
├── src/
│   ├── components/     # UI组件
│   ├── pages/         # 页面组件
│   ├── services/      # 业务逻辑
│   ├── utils/         # 工具函数
│   └── config.ts      # 配置文件
├── public/            # 静态资源
├── tests/            # 测试文件
├── package.json      # 项目配置
└── world.config.js   # World应用配置
```

## 基础配置
### world.config.js
```javascript
module.exports = {
  // 应用基本信息
  name: "我的World应用",
  version: "1.0.0",
  description: "一个示例World mini app",
  
  // 开发配置
  development: {
    port: 3000,
    host: "localhost"
  },
  
  // 构建配置
  build: {
    outDir: "dist",
    minify: true
  },
  
  // World ID集成
  worldId: {
    required: true,
    verificationLevel: "device"
  },
  
  // 权限配置
  permissions: [
    "storage",
    "camera",
    "location"
  ]
};
```

## 快速开发流程

### 1. 创建新项目
```bash
# 选择合适的模板创建项目
npx create-world-app my-app --template react

# 进入项目目录
cd my-app

# 安装依赖
npm install
```

### 2. 启动开发服务器
```bash
# 启动开发服务器
npm run dev
```

### 3. 连接World App
```javascript
// src/index.js
import { WorldApp } from '@world/minikit';

const app = new WorldApp({
  appId: 'your-app-id',
  debug: true
});

app.onReady(() => {
  console.log('World App connected!');
});
```

### 4. 实现基本功能
```javascript
// 用户认证
app.auth.login().then(user => {
  console.log('User logged in:', user);
});

// 支付功能
app.payment.request({
  amount: '10.00',
  currency: 'USDC'
}).then(result => {
  console.log('Payment completed:', result);
});

// 存储数据
app.storage.set('user-preferences', {
  theme: 'dark',
  language: 'zh-CN'
});
```

## 调试技巧

### 1. 开发者工具
```javascript
// 启用详细日志
app.setDebugLevel('verbose');

// 监听事件
app.on('error', (error) => {
  console.error('App error:', error);
});
```

### 2. 模拟器使用
- 使用World App模拟器进行本地测试
- 支持热重载
- 可模拟各种设备和条件

### 3. 错误处理
```javascript
try {
  await app.someAction();
} catch (error) {
  if (error.code === 'NOT_AUTHORIZED') {
    // 处理授权错误
  } else if (error.code === 'NETWORK_ERROR') {
    // 处理网络错误
  }
}
```

## 部署准备

### 1. 构建应用
```bash
# 构建生产版本
npm run build
```

### 2. 测试检查
```bash
# 运行测试
npm test

# 检查代码质量
npm run lint
```

### 3. 发布应用
```bash
# 发布到World App Store
world-cli deploy
```

## 下一步
- 查看[完整API文档](./api-reference.md)
- 探索[高级功能](./advanced-features.md)
- 了解[最佳实践](./best-practices.md)
- 参考[示例项目](./example-projects.md)
