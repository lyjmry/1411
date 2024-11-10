# World ID 故障排除指南

## 常见问题

### 1. 初始化错误

#### 问题描述
```javascript
Error: Failed to initialize World ID: Invalid configuration
```

#### 解决方案
1. 检查配置参数
```javascript
// 正确的配置示例
const worldID = new WorldID({
    appId: process.env.WORLD_APP_ID,        // 确保环境变量已设置
    actionId: 'your_action_id',             // 使用有效的actionId
    debug: process.env.NODE_ENV !== 'production'
});
```

2. 验证环境变量
```bash
# .env文件
WORLD_APP_ID=app_staging_xxx
WORLD_ACTION_ID=wid_staging_xxx
```

3. 检查网络连接
```javascript
// 添加网络状态检查
async function checkConnection() {
    try {
        await worldID.ping();
        console.log('连接正常');
    } catch (error) {
        console.error('网络连接失败:', error);
    }
}
```

### 2. 验证失败

#### 问题描述
```javascript
Error: Verification failed: Invalid proof
```

#### 解决方案
1. 检查验证流程
```javascript
async function verifyIdentity() {
    try {
        // 确保signal正确
        const signal = generateSignal();
        
        // 添加错误处理
        const verification = await worldID.verify({
            signal,
            proofType: 'orb',  // 或 'device'
            onError: (error) => {
                console.error('验证过程错误:', error);
            }
        });
        
        return verification;
    } catch (error) {
        if (error.code === 'PROOF_INVALID') {
            console.error('无效的证明');
        } else if (error.code === 'VERIFICATION_REJECTED') {
            console.error('用户拒绝验证');
        }
        throw error;
    }
}
```

2. 实现重试机制
```javascript
async function verifyWithRetry(maxAttempts = 3) {
    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
        try {
            return await worldID.verify({
                signal: 'your_signal'
            });
        } catch (error) {
            if (attempt === maxAttempts) throw error;
            
            // 等待后重试
            await new Promise(resolve => 
                setTimeout(resolve, attempt * 1000)
            );
        }
    }
}
```

### 3. 会话错误

#### 问题描述
```javascript
Error: Session expired or invalid
```

#### 解决方案
1. 会话管理
```javascript
class SessionManager {
    constructor() {
        this.worldID = new WorldID();
    }
    
    async refreshSession() {
        try {
            const currentSession = await this.getSession();
            if (this.isSessionExpired(currentSession)) {
                // 重新验证
                await this.revalidate();
            }
        } catch (error) {
            console.error('会话刷新失败:', error);
            // 重新登录
            await this.handleRelogin();
        }
    }
    
    isSessionExpired(session) {
        return session.expiresAt < Date.now();
    }
}
```

2. 自动重新验证
```javascript
const sessionManager = new SessionManager();

// 添加请求拦截器
axios.interceptors.response.use(
    response => response,
    async error => {
        if (error.response?.status === 401) {
            await sessionManager.refreshSession();
            // 重试原始请求
            return axios(error.config);
        }
        return Promise.reject(error);
    }
);
```

### 4. 集成问题

#### 问题描述
```javascript
Error: Widget initialization failed
```

#### 解决方案
1. 检查组件集成
```javascript
// 正确的组件使用方式
function App() {
    return (
        <WorldIDWidget
            actionId="your_action_id"
            signal="your_signal"
            onSuccess={(result) => {
                console.log('验证成功:', result);
            }}
            onError={(error) => {
                console.error('验证失败:', error);
            }}
            onInitSuccess={() => {
                console.log('组件初始化成功');
            }}
            onInitError={(error) => {
                console.error('组件初始化失败:', error);
            }}
        />
    );
}
```

2. 样式问题修复
```css
/* 确保组件正确显示 */
.worldid-widget {
    position: relative;
    z-index: 1000;
    min-height: 44px;
}

/* 修复移动端样式 */
@media (max-width: 768px) {
    .worldid-widget {
        width: 100%;
        margin: 10px 0;
    }
}
```

### 5. 性能问题

#### 问题描述
```javascript
Warning: Verification process is taking too long
```

#### 解决方案
1. 实现性能监控
```javascript
class PerformanceMonitor {
    constructor() {
        this.metrics = new Map();
    }
    
    startTimer(operation) {
        this.metrics.set(operation, performance.now());
    }
    
    endTimer(operation) {
        const startTime = this.metrics.get(operation);
        const duration = performance.now() - startTime;
        
        if (duration > 5000) {  // 5秒阈值
            console.warn(`操作${operation}耗时过长: ${duration}ms`);
        }
        
        return duration;
    }
}
```

2. 优化验证流程
```javascript
class OptimizedVerifier {
    constructor() {
        this.cache = new Map();
    }
    
    async verify(signal) {
        // 检查缓存
        const cached = this.cache.get(signal);
        if (cached && !this.isExpired(cached)) {
            return cached.result;
        }
        
        // 执行验证
        const result = await worldID.verify({ signal });
        
        // 更新缓存
        this.cache.set(signal, {
            result,
            timestamp: Date.now()
        });
        
        return result;
    }
}
```

### 6. 网络问题

#### 问题描述
```javascript
Error: Network request failed
```

#### 解决方案
1. 网络状态检查
```javascript
class NetworkManager {
    async checkConnectivity() {
        try {
            await fetch('https://api.worldcoin.org/ping');
            return true;
        } catch (error) {
            console.error('网络连接失败:', error);
            return false;
        }
    }
    
    async executeWithRetry(operation, maxRetries = 3) {
        for (let i = 0; i < maxRetries; i++) {
            try {
                return await operation();
            } catch (error) {
                if (!this.isRetryable(error) || i === maxRetries - 1) {
                    throw error;
                }
                await this.wait(Math.pow(2, i) * 1000);
            }
        }
    }
}
```

2. 离线支持
```javascript
class OfflineSupport {
    constructor() {
        this.queue = [];
    }
    
    async addToQueue(operation) {
        this.queue.push(operation);
        await this.persistQueue();
    }
    
    async processQueue() {
        while (this.queue.length > 0) {
            const operation = this.queue[0];
            try {
                await operation();
                this.queue.shift();
                await this.persistQueue();
            } catch (error) {
                console.error('队列处理错误:', error);
                break;
            }
        }
    }
}
```

## 调试技巧

### 1. 启用调试模式
```javascript
const worldID = new WorldID({
    debug: true,
    logLevel: 'verbose',
    onLog: (level, message) => {
        console.log(`[WorldID][${level}] ${message}`);
    }
});
```

### 2. 使用开发者工具
```javascript
class Debugger {
    static async analyzeVerification(proof) {
        console.log('验证数据:', {
            merkleRoot: proof.merkle_root,
            nullifierHash: proof.nullifier_hash,
            proof: proof.proof
        });
        
        // 验证完整性
        const isValid = await WorldIDUtils.verifyProof(proof);
        console.log('证明是否有效:', isValid);
    }
}
```

## 错误代码参考

| 错误代码 | 描述 | 解决方案 |
|---------|------|---------|
| INIT_001 | 初始化失败 | 检查配置参数 |
| VERIFY_001 | 验证失败 | 检查proof格式 |
| NETWORK_001 | 网络错误 | 检查网络连接 |
| SESSION_001 | 会话过期 | 刷新会话 |
| WIDGET_001 | 组件错误 | 检查集成代码 |

## 预防措施

### 1. 错误监控
```javascript
class ErrorMonitor {
    static init() {
        window.addEventListener('unhandledrejection', (event) => {
            if (event.reason.name === 'WorldIDError') {
                this.handleWorldIDError(event.reason);
            }
        });
    }
    
    static handleWorldIDError(error) {
        // 记录错误
        console.error('World ID错误:', error);
        
        // 发送错误报告
        this.sendErrorReport(error);
    }
}
```

### 2. 健康检查
```javascript
class HealthCheck {
    static async runDiagnostics() {
        const results = await Promise.all([
            this.checkNetwork(),
            this.checkConfiguration(),
            this.checkPermissions()
        ]);
        
        return results.every(result => result.healthy);
    }
}
```

## 最佳实践

1. 错误处理
   - 实现完整的错误处理
   - 提供用户友好的错误信息
   - 记录错误日志

2. 性能优化
   - 实施缓存策略
   - 优化验证流程
   - 监控性能指标

3. 用户体验
   - 提供清晰的错误提示
   - 实现平滑的重试机制
   - 添加加载状态

4. 维护
   - 定期检查系统健康状况
   - 监控错误率
   - 更新依赖包

## 下一步
- 查看[性能优化指南](./performance-optimization.md)
- 了解[安全最佳实践](./security-best-practices.md)
- 探索[高级功能](./advanced-features.md)
