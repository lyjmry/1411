# World ID 性能优化指南

## 验证性能优化

### 1. 验证缓存策略
```javascript
class VerificationCache {
    constructor() {
        this.cache = new Map();
        this.maxAge = 3600 * 1000; // 1小时缓存
    }

    // 缓存验证结果
    async cacheVerification(key, verification) {
        this.cache.set(key, {
            data: verification,
            timestamp: Date.now()
        });
    }

    // 获取缓存的验证结果
    async getCachedVerification(key) {
        const cached = this.cache.get(key);
        if (!cached) return null;

        if (Date.now() - cached.timestamp > this.maxAge) {
            this.cache.delete(key);
            return null;
        }

        return cached.data;
    }

    // 实现示例
    async verify(signal) {
        const cacheKey = this.generateCacheKey(signal);
        
        // 检查缓存
        const cached = await this.getCachedVerification(cacheKey);
        if (cached) {
            return cached;
        }

        // 执行验证
        const verification = await worldID.verify({ signal });
        
        // 缓存结果
        await this.cacheVerification(cacheKey, verification);
        
        return verification;
    }
}
```

### 2. 批量验证优化
```javascript
class BatchVerifier {
    constructor() {
        this.batchSize = 10;
        this.queue = [];
    }

    // 添加到验证队列
    async addToQueue(verification) {
        this.queue.push(verification);
        
        if (this.queue.length >= this.batchSize) {
            return this.processBatch();
        }
    }

    // 处理批量验证
    async processBatch() {
        const batch = this.queue.splice(0, this.batchSize);
        
        const results = await Promise.all(
            batch.map(async (item) => {
                try {
                    return await worldID.verify(item);
                } catch (error) {
                    return { error };
                }
            })
        );

        return results;
    }
}
```

## 网络优化

### 1. 请求优化
```javascript
class NetworkOptimizer {
    constructor() {
        this.pendingRequests = new Map();
    }

    // 请求去重
    async makeRequest(key, requestFn) {
        if (this.pendingRequests.has(key)) {
            return this.pendingRequests.get(key);
        }

        const promise = requestFn().finally(() => {
            this.pendingRequests.delete(key);
        });

        this.pendingRequests.set(key, promise);
        return promise;
    }

    // 实现预加载
    async preload(resources) {
        const preloadPromises = resources.map(resource => {
            if (resource.type === 'script') {
                return this.preloadScript(resource.url);
            } else if (resource.type === 'data') {
                return this.preloadData(resource.url);
            }
        });

        await Promise.all(preloadPromises);
    }
}
```

### 2. 连接优化
```javascript
class ConnectionManager {
    constructor() {
        this.retryConfig = {
            maxRetries: 3,
            baseDelay: 1000,
            maxDelay: 5000
        };
    }

    // 实现连接池
    async getConnection() {
        if (!this.pool) {
            this.pool = await this.createConnectionPool();
        }
        return this.pool.acquire();
    }

    // 实现重试机制
    async executeWithRetry(operation) {
        let lastError;
        
        for (let i = 0; i < this.retryConfig.maxRetries; i++) {
            try {
                return await operation();
            } catch (error) {
                lastError = error;
                
                const delay = Math.min(
                    this.retryConfig.baseDelay * Math.pow(2, i),
                    this.retryConfig.maxDelay
                );
                
                await new Promise(resolve => setTimeout(resolve, delay));
            }
        }
        
        throw lastError;
    }
}
```

## 资源优化

### 1. 懒加载策略
```javascript
class ResourceLoader {
    // 组件懒加载
    static loadComponent(componentName) {
        return React.lazy(() => {
            return import(`./components/${componentName}`).catch(error => {
                console.error(`加载组件失败: ${componentName}`, error);
                return { default: ErrorComponent };
            });
        });
    }

    // 资源懒加载
    static async loadResource(resource) {
        if (typeof window === 'undefined') return null;

        const loader = {
            script: this.loadScript,
            style: this.loadStyle,
            image: this.loadImage
        }[resource.type];

        if (!loader) {
            throw new Error(`不支持的资源类型: ${resource.type}`);
        }

        return loader(resource.url);
    }
}
```

### 2. 内存优化
```javascript
class MemoryOptimizer {
    constructor() {
        this.maxCacheSize = 100;
        this.cache = new LRUCache(this.maxCacheSize);
    }

    // 内存使用监控
    monitorMemoryUsage() {
        if (performance.memory) {
            const usage = performance.memory.usedJSHeapSize;
            const limit = performance.memory.jsHeapSizeLimit;
            
            if (usage / limit > 0.9) {
                this.cleanup();
            }
        }
    }

    // 清理未使用的资源
    async cleanup() {
        // 清理缓存
        this.cache.prune();
        
        // 清理弱引用
        this.cleanupWeakRefs();
        
        // 建议垃圾回收
        if (global.gc) {
            global.gc();
        }
    }
}
```

## 渲染优化

### 1. 组件优化
```javascript
// 使用memo优化组件
const OptimizedWorldIDButton = React.memo(function WorldIDButton({ 
    onSuccess, 
    onError 
}) {
    const handleVerification = useCallback(async () => {
        try {
            const result = await worldID.verify();
            onSuccess?.(result);
        } catch (error) {
            onError?.(error);
        }
    }, [onSuccess, onError]);

    return (
        <button onClick={handleVerification}>
            验证身份
        </button>
    );
});

// 使用useMemo优化计算
function VerificationStatus({ verification }) {
    const status = useMemo(() => {
        return computeStatus(verification);
    }, [verification]);

    return <div>{status}</div>;
}
```

### 2. 状态管理优化
```javascript
class StateManager {
    constructor() {
        this.subscribers = new Map();
        this.state = new Proxy({}, {
            set: (target, key, value) => {
                const oldValue = target[key];
                target[key] = value;
                
                if (oldValue !== value) {
                    this.notifySubscribers(key, value);
                }
                
                return true;
            }
        });
    }

    // 优化状态更新
    batchUpdate(updates) {
        const batch = new Set();
        
        Object.entries(updates).forEach(([key, value]) => {
            if (this.state[key] !== value) {
                this.state[key] = value;
                batch.add(key);
            }
        });

        if (batch.size > 0) {
            this.notifyBatchUpdate(batch);
        }
    }
}
```

## 监控和分析

### 1. 性能监控
```javascript
class PerformanceMonitor {
    constructor() {
        this.metrics = new Map();
        this.thresholds = {
            verificationTime: 2000,  // 2秒
            networkLatency: 1000,    // 1秒
            memoryUsage: 0.8         // 80%
        };
    }

    // 监控验证性能
    async trackVerification(verification) {
        const start = performance.now();
        
        try {
            const result = await verification();
            
            const duration = performance.now() - start;
            this.recordMetric('verificationTime', duration);
            
            return result;
        } catch (error) {
            this.recordError('verification', error);
            throw error;
        }
    }

    // 生成性能报告
    generateReport() {
        return {
            metrics: Object.fromEntries(this.metrics),
            thresholdViolations: this.checkThresholds(),
            recommendations: this.generateRecommendations()
        };
    }
}
```

### 2. 性能分析
```javascript
class PerformanceAnalyzer {
    // 分析验证性能
    analyzeVerification(metrics) {
        const analysis = {
            averageTime: this.calculateAverage(metrics.verificationTimes),
            p95Time: this.calculatePercentile(metrics.verificationTimes, 95),
            successRate: this.calculateSuccessRate(metrics.results)
        };

        return {
            ...analysis,
            recommendations: this.generateRecommendations(analysis)
        };
    }

    // 生成优化建议
    generateRecommendations(analysis) {
        const recommendations = [];

        if (analysis.averageTime > 2000) {
            recommendations.push({
                type: 'performance',
                message: '验证时间过长，建议实施缓存策略'
            });
        }

        if (analysis.successRate < 0.95) {
            recommendations.push({
                type: 'reliability',
                message: '验证成功率过低，建议检查错误处理'
            });
        }

        return recommendations;
    }
}
```

## 优化清单

### 1. 性能检查
- [ ] 实施验证缓存
- [ ] 优化网络请求
- [ ] 实现资源懒加载
- [ ] 优化组件渲染
- [ ] 监控性能指标

### 2. 资源优化
- [ ] 实施代码分割
- [ ] 优化资源加载
- [ ] 管理内存使用
- [ ] 清理未使用资源

### 3. 网络优化
- [ ] 实现请求去重
- [ ] 优化连接管理
- [ ] 实施重试策略
- [ ] 预加载关键资源

## 最佳实践

1. 性能监控
   - 持续监控关键指标
   - 设置性能预警
   - 定期性能分析

2. 优化策略
   - 实施分层缓存
   - 优化关键路径
   - 按需加载资源

3. 维护优化
   - 定期清理缓存
   - 更新优化策略
   - 响应性能问题

## 下一步
- 查看[安全最佳实践](./security-best-practices.md)
- 了解[高级功能](./advanced-features.md)
- 探索[示例项目](./example-projects.md)
