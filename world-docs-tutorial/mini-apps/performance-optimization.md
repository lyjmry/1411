# Mini Apps 性能优化指南

## 加载性能优化

### 1. 资源加载优化
```javascript
// 实现资源预加载
class ResourceLoader {
    constructor() {
        this.resources = new Map();
        this.preloadQueue = new PriorityQueue();
    }

    // 预加载关键资源
    async preloadCriticalResources() {
        const criticalResources = [
            '/assets/fonts/main.woff2',
            '/assets/images/icons.svg',
            '/assets/styles/critical.css'
        ];

        await Promise.all(
            criticalResources.map(resource =>
                this.preload(resource, { priority: 'high' })
            )
        );
    }

    // 懒加载非关键资源
    async lazyLoadResource(resource) {
        if (this.resources.has(resource)) {
            return this.resources.get(resource);
        }

        const result = await this.load(resource);
        this.resources.set(resource, result);
        return result;
    }
}
```

### 2. 代码分割
```javascript
// 路由级别的代码分割
const routes = [
    {
        path: '/',
        component: React.lazy(() => import('./pages/Home'))
    },
    {
        path: '/profile',
        component: React.lazy(() => import('./pages/Profile'))
    },
    {
        path: '/settings',
        component: React.lazy(() => import('./pages/Settings'))
    }
];

// 组件级别的代码分割
const HeavyComponent = React.lazy(() => 
    import('./components/HeavyComponent')
);

function App() {
    return (
        <Suspense fallback={<Loading />}>
            <Router routes={routes} />
        </Suspense>
    );
}
```

## 运行时性能优化

### 1. 虚拟列表
```javascript
class VirtualList {
    constructor(options) {
        this.itemHeight = options.itemHeight;
        this.containerHeight = options.containerHeight;
        this.items = options.items;
        this.visibleItems = Math.ceil(this.containerHeight / this.itemHeight);
    }

    getVisibleRange(scrollTop) {
        const start = Math.floor(scrollTop / this.itemHeight);
        const end = Math.min(
            start + this.visibleItems,
            this.items.length
        );

        return {
            start,
            end,
            items: this.items.slice(start, end)
        };
    }

    renderItems(scrollTop) {
        const { items, start } = this.getVisibleRange(scrollTop);
        
        return (
            <div style={{ 
                height: this.items.length * this.itemHeight,
                position: 'relative'
            }}>
                {items.map((item, index) => (
                    <div key={item.id} style={{
                        position: 'absolute',
                        top: (start + index) * this.itemHeight,
                        height: this.itemHeight
                    }}>
                        {item.content}
                    </div>
                ))}
            </div>
        );
    }
}
```

### 2. 状态管理优化
```javascript
// 实现高效的状态更新
class StateManager {
    constructor() {
        this.state = new Proxy({}, {
            set: (target, key, value) => {
                const oldValue = target[key];
                if (this.shouldUpdate(oldValue, value)) {
                    target[key] = value;
                    this.notifySubscribers(key);
                }
                return true;
            }
        });
        
        this.subscribers = new Map();
    }

    shouldUpdate(oldValue, newValue) {
        return !Object.is(oldValue, newValue);
    }

    subscribe(key, callback) {
        if (!this.subscribers.has(key)) {
            this.subscribers.set(key, new Set());
        }
        this.subscribers.get(key).add(callback);
    }

    notifySubscribers(key) {
        const callbacks = this.subscribers.get(key);
        if (callbacks) {
            callbacks.forEach(callback => callback(this.state[key]));
        }
    }
}
```

## 渲染性能优化

### 1. 组件优化
```javascript
// 使用React.memo优化组件渲染
const OptimizedComponent = React.memo(function Component(props) {
    return (
        <div>
            {/* 组件内容 */}
        </div>
    );
}, (prevProps, nextProps) => {
    // 自定义比较逻辑
    return prevProps.value === nextProps.value;
});

// 使用useMemo优化计算
function ExpensiveComponent({ data }) {
    const processedData = useMemo(() => {
        return expensiveCalculation(data);
    }, [data]);

    return (
        <div>
            {processedData.map(item => (
                <div key={item.id}>{item.content}</div>
            ))}
        </div>
    );
}
```

### 2. DOM操作优化
```javascript
class DOMOptimizer {
    // 批量DOM更新
    batchUpdate(updates) {
        requestAnimationFrame(() => {
            const fragment = document.createDocumentFragment();
            
            updates.forEach(update => {
                const element = this.createElement(update);
                fragment.appendChild(element);
            });
            
            document.getElementById('container').appendChild(fragment);
        });
    }

    // 使用虚拟DOM
    updateVirtualDOM(newVirtualDOM) {
        const patches = this.diff(this.currentVirtualDOM, newVirtualDOM);
        this.patch(patches);
        this.currentVirtualDOM = newVirtualDOM;
    }
}
```

## 网络性能优化

### 1. 请求优化
```javascript
class NetworkOptimizer {
    constructor() {
        this.cache = new Map();
        this.pendingRequests = new Map();
    }

    // 请求去重
    async request(url, options) {
        const key = this.getCacheKey(url, options);
        
        // 检查是否有相同的请求正在进行
        if (this.pendingRequests.has(key)) {
            return this.pendingRequests.get(key);
        }

        // 检查缓存
        if (this.cache.has(key)) {
            return this.cache.get(key);
        }

        // 发起新请求
        const promise = fetch(url, options)
            .then(response => response.json())
            .finally(() => {
                this.pendingRequests.delete(key);
            });

        this.pendingRequests.set(key, promise);
        return promise;
    }

    // 预请求
    prefetch(urls) {
        urls.forEach(url => {
            const link = document.createElement('link');
            link.rel = 'prefetch';
            link.href = url;
            document.head.appendChild(link);
        });
    }
}
```

### 2. 数据缓存
```javascript
class CacheManager {
    constructor() {
        this.storage = new WorldStorage();
    }

    // 多级缓存策略
    async getData(key) {
        // 检查内存缓存
        const memoryData = this.getFromMemory(key);
        if (memoryData) return memoryData;

        // 检查本地存储
        const localData = await this.getFromStorage(key);
        if (localData) {
            this.setToMemory(key, localData);
            return localData;
        }

        // 从网络获取
        const networkData = await this.fetchFromNetwork(key);
        await this.setToStorage(key, networkData);
        this.setToMemory(key, networkData);
        return networkData;
    }

    // 缓存预热
    async warmupCache(keys) {
        await Promise.all(
            keys.map(key => this.getData(key))
        );
    }
}
```

## 内存优化

### 1. 内存泄漏防护
```javascript
class MemoryGuard {
    constructor() {
        this.observers = new WeakMap();
        this.intervals = new Set();
    }

    // 监听组件生命周期
    observe(component) {
        const cleanup = () => {
            // 清理定时器
            this.intervals.forEach(clearInterval);
            this.intervals.clear();

            // 清理事件监听
            const listeners = this.observers.get(component);
            if (listeners) {
                listeners.forEach(({ target, type, listener }) => {
                    target.removeEventListener(type, listener);
                });
            }
        };

        return cleanup;
    }

    // 安全地添加事件监听
    addEventListener(component, target, type, listener) {
        if (!this.observers.has(component)) {
            this.observers.set(component, new Set());
        }

        const listeners = this.observers.get(component);
        listeners.add({ target, type, listener });
        target.addEventListener(type, listener);
    }
}
```

### 2. 资源释放
```javascript
class ResourceManager {
    constructor() {
        this.resources = new WeakMap();
    }

    // 自动资源释放
    async useResource(resource, operation) {
        try {
            const handle = await this.acquire(resource);
            const result = await operation(handle);
            return result;
        } finally {
            await this.release(resource);
        }
    }

    // 定期清理未使用的资源
    async cleanup() {
        const unusedResources = await this.findUnusedResources();
        await Promise.all(
            unusedResources.map(resource => 
                this.release(resource)
            )
        );
    }
}
```

## 监控和分析

### 1. 性能监控
```javascript
class PerformanceMonitor {
    constructor() {
        this.metrics = new WorldMetrics();
    }

    // 监控关键指标
    startMonitoring() {
        this.metrics.track({
            fps: true,
            memory: true,
            network: true,
            timing: true
        });

        // 设置警告阈值
        this.metrics.setThresholds({
            fps: 30,
            memory: 0.8, // 80%内存使用率
            timing: {
                fcp: 2000, // First Contentful Paint
                lcp: 2500  // Largest Contentful Paint
            }
        });

        // 监听性能问题
        this.metrics.onThresholdExceeded(({ metric, value }) => {
            console.warn(`性能警告: ${metric} = ${value}`);
            this.reportPerformanceIssue(metric, value);
        });
    }

    // 生成性能报告
    async generateReport() {
        const data = await this.metrics.collect();
        return {
            fps: this.calculateAverageFPS(data.fps),
            memory: this.analyzeMemoryUsage(data.memory),
            network: this.analyzeNetworkPerformance(data.network),
            timing: this.analyzeTimingMetrics(data.timing)
        };
    }
}
```

### 2. 性能分析
```javascript
class PerformanceAnalyzer {
    // 分析渲染性能
    analyzeRenderPerformance() {
        performance.mark('render-start');
        
        // 执行渲染
        this.render();
        
        performance.mark('render-end');
        performance.measure(
            'render',
            'render-start',
            'render-end'
        );

        const measurements = performance.getEntriesByType('measure');
        return this.analyzeMeasurements(measurements);
    }

    // 分析代码执行时间
    profileFunction(fn) {
        console.profile(fn.name);
        const result = fn();
        console.profileEnd(fn.name);
        return result;
    }
}
```

## 优化清单

### 1. 启动优化
- [ ] 实现资源预加载
- [ ] 优化初始化流程
- [ ] 实现代码分割
- [ ] 优化首屏加载

### 2. 运行时优化
- [ ] 实现虚拟列表
- [ ] 优化状态管理
- [ ] 实现请求优化
- [ ] 优化内存使用

### 3. 持续监控
- [ ] 设置性能监控
- [ ] 实现性能报告
- [ ] 设置警告阈值
- [ ] 建立优化流程

## 最佳实践

1. 定期进行性能审查
2. 建立性能基准
3. 实施持续优化
4. 监控关键指标
5. 及时处理性能问题

## 下一步
- 查看[安全最佳实践](./security-best-practices.md)
- 了解[高级功能](./advanced-features.md)
- 探索[示例项目](./example-projects.md)
