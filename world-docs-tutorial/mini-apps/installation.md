# MiniKit 安装指南

## 系统要求

### 基本要求
- Node.js 14.0.0+
- npm 6.0.0+ 或 Yarn 1.22.0+
- Git（用于版本控制）

### 推荐的开发工具
- Visual Studio Code
- World App开发者版本
- Chrome DevTools

## 安装方法

### 1. 使用npm安装
```bash
# 安装 MiniKit CLI
npm install -g @world/minikit-cli

# 安装 MiniKit SDK
npm install @world/minikit
```

### 2. 使用Yarn安装
```bash
# 安装 MiniKit CLI
yarn global add @world/minikit-cli

# 安装 MiniKit SDK
yarn add @world/minikit
```

### 3. 使用CDN
```html
<!-- 开发环境 -->
<script src="https://cdn.world.org/minikit/development.js"></script>

<!-- 生产环境 -->
<script src="https://cdn.world.org/minikit/production.min.js"></script>
```

## 项目初始化

### 1. 创建新项目
```bash
# 使用CLI创建项目
world-cli create my-mini-app

# 选择项目模板
? Select a template:
  ❯ React (推荐)
    Vanilla JS
    Next.js
    Vue.js
```

### 2. 项目配置
```javascript
// world.config.js
module.exports = {
  // 基本配置
  name: '我的Mini App',
  version: '1.0.0',
  description: '这是一个World Mini App',
  
  // 开发服务器配置
  devServer: {
    port: 3000,
    host: 'localhost',
    https: true,
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true
      }
    }
  },
  
  // 构建配置
  build: {
    outDir: 'dist',
    assetsDir: 'assets',
    sourcemap: true,
    minify: {
      terser: {
        compress: {
          drop_console: true
        }
      }
    }
  },
  
  // 依赖配置
  dependencies: {
    '@world/minikit': '^1.0.0'
  },
  
  // 环境变量
  env: {
    development: {
      API_URL: 'http://localhost:8080'
    },
    production: {
      API_URL: 'https://api.production.com'
    }
  }
};
```

## 环境配置

### 1. 开发环境设置
```bash
# 创建环境配置文件
touch .env.development

# 配置开发环境变量
WORLD_APP_ID=your_dev_app_id
WORLD_API_KEY=your_dev_api_key
DEBUG_MODE=true
```

### 2. 生产环境设置
```bash
# 创建生产环境配置文件
touch .env.production

# 配置生产环境变量
WORLD_APP_ID=your_prod_app_id
WORLD_API_KEY=your_prod_api_key
DEBUG_MODE=false
```

## SDK配置

### 1. 基础配置
```javascript
// src/config/minikit.js
import { MiniKit } from '@world/minikit';

const minikit = new MiniKit({
  appId: process.env.WORLD_APP_ID,
  apiKey: process.env.WORLD_API_KEY,
  environment: process.env.NODE_ENV,
  debug: process.env.DEBUG_MODE === 'true',
  
  // 功能配置
  features: {
    auth: true,
    payment: true,
    storage: true
  },
  
  // 事件处理
  onReady: () => {
    console.log('MiniKit初始化完成');
  },
  onError: (error) => {
    console.error('MiniKit错误:', error);
  }
});

export default minikit;
```

### 2. 权限配置
```javascript
// 配置应用权限
minikit.configurePermissions({
  required: [
    'basic_profile',
    'storage_read',
    'storage_write'
  ],
  optional: [
    'camera',
    'location',
    'notifications'
  ]
});
```

## 安全配置

### 1. CSP配置
```html
<!-- public/index.html -->
<meta http-equiv="Content-Security-Policy" content="
  default-src 'self';
  script-src 'self' https://cdn.world.org;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  connect-src 'self' https://api.world.org;
">
```

### 2. 证书配置
```bash
# 生成本地开发证书
world-cli cert create

# 信任证书
world-cli cert trust
```

## 依赖管理

### 1. 核心依赖
```json
{
  "dependencies": {
    "@world/minikit": "^1.0.0",
    "@world/minikit-ui": "^1.0.0",
    "@world/minikit-auth": "^1.0.0"
  }
}
```

### 2. 开发依赖
```json
{
  "devDependencies": {
    "@world/minikit-cli": "^1.0.0",
    "@world/minikit-types": "^1.0.0",
    "@world/eslint-config-minikit": "^1.0.0"
  }
}
```

## 验证安装

### 1. 检查安装
```bash
# 验证CLI安装
world-cli --version

# 验证项目设置
world-cli doctor
```

### 2. 运行测试项目
```bash
# 启动开发服务器
npm run dev

# 在新终端运行测试
npm test
```

## 常见问题解决

### 1. 安装错误
- 检查Node.js版本兼容性
- 清除npm缓存：`npm cache clean --force`
- 删除node_modules并重新安装

### 2. 配置问题
- 确保配置文件格式正确
- 验证环境变量设置
- 检查权限设置

### 3. 依赖冲突
- 使用`npm audit`检查依赖问题
- 更新过时的依赖
- 解决版本冲突

## 下一步
- 查看[开发指南](./development-guide.md)
- 了解[API文档](./api-reference.md)
- 探索[示例项目](./example-projects.md)
