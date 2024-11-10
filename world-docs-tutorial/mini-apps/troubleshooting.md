# Mini Apps 故障排除指南

## 常见问题

### 1. 初始化错误

#### 问题描述
```javascript
Error: Failed to initialize World App: Invalid configuration
```

#### 解决方案
1. 检查配置文件
```javascript
// world.config.js
module.exports = {
  appId: process.env.WORLD_APP_ID, // 确保环境变量已设置
  version: '1.0.0',
  debug: true // 开发时启用调试模式
};
```

2. 验证环境变量
```bash
# .env 文件
WORLD_APP_ID=your_app_id
WORLD_API_KEY=your_api_key
```

3. 重新初始化
```javascript
const app = new WorldApp({
  appId: process.env.WORLD_APP_ID,
  debug: true,
  onError: (error) => {
    console.error('初始化错误:', error);
  }
});
```

### 2. 认证问题

#### 问题描述
```javascript
Error: Authentication failed: Invalid credentials
```

#### 解决方案
1. 检查认证流程
```javascript
async function authenticate() {
  try {
    // 确保先初始化
    await WorldAuth.initialize();
    
    // 进行认证
    const result = await WorldAuth.login({
      scope: ['profile', 'payment'],
      prompt: 'consent'
    });
    
    return result;
  } catch (error) {
    console.error('认证错误:', error);
    // 实现重试逻辑
    if (error.code === 'TOKEN_EXPIRED') {
      return await refreshToken();
    }
    throw error;
  }
}
```

2. 验证权限设置
```javascript
// 检查必要权限
const permissions = await WorldAuth.checkPermissions([
  'profile',
  'payment'
]);

if (!permissions.granted) {
  // 请求缺失的权限
  await WorldAuth.requestPermissions(permissions.missing);
}
```

### 3. 支付错误

#### 问题描述
```javascript
Error: Payment failed: Insufficient funds
```

#### 解决方案
1. 实现完整的错误处理
```javascript
async function processPayment(amount) {
  try {
    // 检查余额
    const balance = await WorldPayment.checkBalance();
    if (balance < amount) {
      throw new Error('余额不足');
    }
    
    // 创建支付
    const payment = await WorldPayment.create({
      amount,
      currency: 'USDC',
      description: '商品购买'
    });
    
    // 确认支付
    const result = await WorldPayment.confirm(payment.id);
    return result;
  } catch (error) {
    if (error.code === 'INSUFFICIENT_FUNDS') {
      // 提示用户充值
      WorldUI.Toast.error('余额不足，请充值');
    } else if (error.code === 'NETWORK_ERROR') {
      // 网络错误重试
      return await retryPayment(amount);
    }
    throw error;
  }
}
```

2. 添加支付状态检查
```javascript
async function checkPaymentStatus(paymentId) {
  const status = await WorldPayment.getStatus(paymentId);
  switch (status) {
    case 'pending':
      // 等待确认
      break;
    case 'completed':
      // 支付成功
      break;
    case 'failed':
      // 支付失败
      break;
  }
  return status;
}
```

### 4. 存储问题

#### 问题描述
```javascript
Error: Storage operation failed: Quota exceeded
```

#### 解决方案
1. 实现存储空间管理
```javascript
class StorageManager {
  async checkStorage() {
    const usage = await WorldStorage.getUsage();
    const quota = await WorldStorage.getQuota();
    
    if (usage > quota * 0.9) {
      // 清理旧数据
      await this.cleanOldData();
    }
  }
  
  async cleanOldData() {
    const keys = await WorldStorage.getAllKeys();
    const oldItems = await this.findOldItems(keys);
    
    for (const key of oldItems) {
      await WorldStorage.remove(key);
    }
  }
}
```

2. 实现数据压缩
```javascript
class CompressionService {
  async compressData(data) {
    // 实现数据压缩
    return compressedData;
  }
  
  async decompressData(compressedData) {
    // 实现数据解压
    return originalData;
  }
}
```

### 5. 网络问题

#### 问题描述
```javascript
Error: Network request failed: Timeout
```

#### 解决方案
1. 实现重试机制
```javascript
async function retryRequest(fn, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      return await fn();
    } catch (error) {
      if (i === retries - 1) throw error;
      
      // 等待时间递增
      await new Promise(resolve => 
        setTimeout(resolve, Math.pow(2, i) * 1000)
      );
    }
  }
}
```

2. 添加离线支持
```javascript
class OfflineSupport {
  constructor() {
    this.queue = new WorldQueue();
  }
  
  async handleOffline(operation) {
    // 添加到队列
    await this.queue.add(operation);
    
    // 监听在线状态
    window.addEventListener('online', async () => {
      await this.processQueue();
    });
  }
  
  async processQueue() {
    const operations = await this.queue.getAll();
    for (const op of operations) {
      try {
        await op.execute();
        await this.queue.remove(op.id);
      } catch (error) {
        console.error('队列处理错误:', error);
      }
    }
  }
}
```

### 6. 性能问题

#### 问题描述
```javascript
Warning: Performance degradation detected
```

#### 解决方案
1. 实现性能监控
```javascript
class PerformanceMonitor {
  constructor() {
    this.metrics = new WorldMetrics();
  }
  
  startMonitoring() {
    this.metrics.track({
      fps: true,
      memory: true,
      network: true,
      interval: 1000
    });
    
    this.metrics.onThreshold(metric => {
      if (metric.name === 'fps' && metric.value < 30) {
        this.handleLowFPS();
      }
    });
  }
  
  async handleLowFPS() {
    // 实现性能优化措施
    await this.optimizeRendering();
    await this.cleanupResources();
  }
}
```

2. 实现资源优化
```javascript
class ResourceOptimizer {
  async optimizeImages() {
    const images = document.querySelectorAll('img');
    for (const img of images) {
      if (img.naturalWidth > 1000) {
        await this.compressImage(img);
      }
    }
  }
  
  async cleanupMemory() {
    // 清理未使用的资源
    await WorldRuntime.gc();
  }
}
```

## 调试技巧

### 1. 日志记录
```javascript
class Logger {
  static log(level, message, data) {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] ${level}: ${message}`, data);
    
    // 保存到日志文件
    WorldStorage.append('app.log', {
      timestamp,
      level,
      message,
      data
    });
  }
  
  static error(message, error) {
    this.log('ERROR', message, {
      name: error.name,
      message: error.message,
      stack: error.stack
    });
  }
}
```

### 2. 性能分析
```javascript
class PerformanceAnalyzer {
  static async analyze() {
    const metrics = await WorldMetrics.collect({
      duration: 5000, // 收集5秒数据
      includeMemory: true,
      includeNetwork: true
    });
    
    return {
      fps: metrics.getFPS(),
      memory: metrics.getMemoryUsage(),
      network: metrics.getNetworkStats()
    };
  }
}
```

## 预防措施

### 1. 错误边界
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    // 记录错误
    Logger.error('组件错误', error);
    
    // 发送错误报告
    WorldAnalytics.reportError(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <ErrorFallback />;
    }
    return this.props.children;
  }
}
```

### 2. 健康检查
```javascript
class HealthCheck {
  static async runChecks() {
    const results = await Promise.all([
      this.checkNetwork(),
      this.checkStorage(),
      this.checkAuth(),
      this.checkPermissions()
    ]);
    
    return results.every(result => result.healthy);
  }
  
  static async checkNetwork() {
    try {
      await WorldHttp.ping();
      return { healthy: true };
    } catch (error) {
      return { 
        healthy: false,
        error: error.message
      };
    }
  }
}
```

## 常见错误代码

| 错误代码 | 描述 | 解决方案 |
|---------|------|---------|
| AUTH_001 | 认证失败 | 检查凭证是否有效 |
| PAY_001 | 支付失败 | 验证余额和支付信息 |
| STORAGE_001 | 存储空间不足 | 清理旧数据或增加配额 |
| NET_001 | 网络超时 | 检查网络连接和重试 |
| APP_001 | 初始化失败 | 验证配置和环境变量 |

## 下一步
- 查看[性能优化指南](./performance-optimization.md)
- 了解[安全最佳实践](./security-best-practices.md)
- 探索[开发工具](./development-tools.md)
