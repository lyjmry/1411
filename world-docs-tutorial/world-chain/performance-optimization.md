# World Chain 性能优化指南

## 智能合约优化

### 1. 存储优化
```solidity
// 优化前
contract Unoptimized {
    mapping(address => uint256) public balances;
    mapping(address => bool) public isRegistered;
    mapping(address => string) public userNames;
    
    function register(string memory name) public {
        isRegistered[msg.sender] = true;
        userNames[msg.sender] = name;
        balances[msg.sender] = 0;
    }
}

// 优化后
contract Optimized {
    struct UserData {
        bool isRegistered;
        string name;
        uint256 balance;
    }
    
    mapping(address => UserData) public users;
    
    function register(string memory name) public {
        users[msg.sender] = UserData({
            isRegistered: true,
            name: name,
            balance: 0
        });
    }
}
```

### 2. Gas优化
```solidity
contract GasOptimized {
    // 使用uint256代替小数值类型
    uint256 private constant MULTIPLIER = 1e18;
    
    // 使用不可变变量
    address immutable public owner;
    
    // 批量操作优化
    function batchTransfer(
        address[] calldata recipients,
        uint256[] calldata amounts
    ) public {
        uint256 length = recipients.length;
        for (uint256 i = 0; i < length;) {
            _transfer(recipients[i], amounts[i]);
            unchecked { ++i; }
        }
    }
    
    // 使用短路评估
    modifier onlyAuthorized() {
        require(
            msg.sender == owner || isOperator[msg.sender],
            "Not authorized"
        );
        _;
    }
}
```

## 交易优化

### 1. 批量处理
```javascript
class BatchProcessor {
    constructor() {
        this.queue = [];
        this.batchSize = 50;
    }

    // 添加到队列
    async addToQueue(tx) {
        this.queue.push(tx);
        
        if (this.queue.length >= this.batchSize) {
            return this.processBatch();
        }
    }

    // 处理批次
    async processBatch() {
        const batch = this.queue.splice(0, this.batchSize);
        
        // 创建多重调用交易
        const multicall = new Contract(
            MULTICALL_ADDRESS,
            MULTICALL_ABI,
            provider
        );
        
        const calls = batch.map(tx => ({
            target: tx.to,
            callData: tx.data
        }));
        
        // 执行批量调用
        const tx = await multicall.aggregate(calls);
        return await tx.wait();
    }
}
```

### 2. Nonce管理
```javascript
class NonceManager {
    constructor() {
        this.nonceMap = new Map();
    }

    // 获取下一个nonce
    async getNextNonce(address) {
        if (!this.nonceMap.has(address)) {
            const nonce = await worldChain.getTransactionCount(address);
            this.nonceMap.set(address, nonce);
        }
        
        const currentNonce = this.nonceMap.get(address);
        this.nonceMap.set(address, currentNonce + 1);
        return currentNonce;
    }

    // 重置nonce
    async resetNonce(address) {
        const nonce = await worldChain.getTransactionCount(address);
        this.nonceMap.set(address, nonce);
    }
}
```

## 数据优化

### 1. 缓存策略
```javascript
class CacheManager {
    constructor() {
        this.cache = new Map();
        this.maxAge = 3600 * 1000; // 1小时
    }

    // 设置缓存
    async set(key, value, ttl = this.maxAge) {
        this.cache.set(key, {
            value,
            timestamp: Date.now(),
            ttl
        });
    }

    // 获取缓存
    async get(key) {
        const cached = this.cache.get(key);
        if (!cached) return null;
        
        if (Date.now() - cached.timestamp > cached.ttl) {
            this.cache.delete(key);
            return null;
        }
        
        return cached.value;
    }

    // 批量预热
    async warmup(keys) {
        const promises = keys.map(async key => {
            const value = await this.fetchData(key);
            await this.set(key, value);
        });
        
        await Promise.all(promises);
    }
}
```

### 2. 数据索引
```javascript
class DataIndexer {
    constructor() {
        this.indexes = new Map();
    }

    // 创建索引
    async createIndex(field) {
        const data = await this.fetchAllData();
        const index = new Map();
        
        for (const item of data) {
            const key = item[field];
            if (!index.has(key)) {
                index.set(key, []);
            }
            index.get(key).push(item);
        }
        
        this.indexes.set(field, index);
    }

    // 使用索引查询
    async queryByIndex(field, value) {
        const index = this.indexes.get(field);
        if (!index) {
            throw new Error('Index not found');
        }
        
        return index.get(value) || [];
    }
}
```

## 网络优化

### 1. 连接池管理
```javascript
class ConnectionPool {
    constructor(config) {
        this.pool = [];
        this.maxSize = config.maxSize || 10;
        this.minSize = config.minSize || 2;
    }

    // 获取连接
    async getConnection() {
        if (this.pool.length === 0) {
            return this.createConnection();
        }
        
        return this.pool.pop();
    }

    // 释放连接
    async releaseConnection(connection) {
        if (this.pool.length < this.maxSize) {
            this.pool.push(connection);
        } else {
            await connection.close();
        }
    }

    // 维护连接池
    async maintain() {
        // 清理空闲连接
        while (this.pool.length > this.minSize) {
            const connection = this.pool.pop();
            await connection.close();
        }
        
        // 补充连接
        while (this.pool.length < this.minSize) {
            const connection = await this.createConnection();
            this.pool.push(connection);
        }
    }
}
```

### 2. 请求优化
```javascript
class RequestOptimizer {
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

    // 批量请求
    async batchRequest(requests) {
        const results = await Promise.all(
            requests.map(req => this.makeRequest(req.key, req.fn))
        );
        
        return results;
    }
}
```

## 事件处理优化

### 1. 事件过滤
```javascript
class EventFilter {
    constructor() {
        this.filters = new Map();
    }

    // 添加过滤器
    addFilter(eventName, filter) {
        this.filters.set(eventName, filter);
    }

    // 过滤事件
    async filterEvents(events) {
        const filtered = [];
        
        for (const event of events) {
            const filter = this.filters.get(event.name);
            if (!filter || await filter(event)) {
                filtered.push(event);
            }
        }
        
        return filtered;
    }
}
```

### 2. 事件订阅优化
```javascript
class EventSubscriber {
    constructor() {
        this.subscriptions = new Map();
        this.batchSize = 10;
        this.batchTimeout = 100;
    }

    // 批量处理事件
    async handleEvents(events) {
        const batches = this.createBatches(events);
        
        for (const batch of batches) {
            await this.processBatch(batch);
        }
    }

    // 创建批次
    createBatches(events) {
        const batches = [];
        for (let i = 0; i < events.length; i += this.batchSize) {
            batches.push(events.slice(i, i + this.batchSize));
        }
        return batches;
    }
}
```

## 监控和分析

### 1. 性能监控
```javascript
class PerformanceMonitor {
    constructor() {
        this.metrics = new Map();
    }

    // 记录指标
    trackMetric(name, value) {
        if (!this.metrics.has(name)) {
            this.metrics.set(name, []);
        }
        
        this.metrics.get(name).push({
            value,
            timestamp: Date.now()
        });
    }

    // 生成报告
    generateReport() {
        const report = {};
        
        for (const [name, values] of this.metrics) {
            report[name] = {
                average: this.calculateAverage(values),
                max: this.calculateMax(values),
                min: this.calculateMin(values)
            };
        }
        
        return report;
    }
}
```

### 2. 性能分析
```javascript
class PerformanceAnalyzer {
    // 分析交易性能
    async analyzeTransaction(txHash) {
        const tx = await worldChain.getTransaction(txHash);
        const receipt = await tx.wait();
        
        return {
            gasUsed: receipt.gasUsed.toString(),
            blockTime: await this.getBlockTime(receipt.blockNumber),
            confirmationTime: await this.getConfirmationTime(tx),
            status: receipt.status
        };
    }

    // 分析合约性能
    async analyzeContract(address) {
        const code = await worldChain.getCode(address);
        
        return {
            codeSize: code.length,
            storageSize: await this.getStorageSize(address),
            callCount: await this.getCallCount(address)
        };
    }
}
```

## 优化清单

### 1. 合约优化
- [ ] 优化存储布局
- [ ] 减少gas消耗
- [ ] 实现批量操作
- [ ] 优化循环结构

### 2. 交易优化
- [ ] 实施批处理
- [ ] 优化nonce管理
- [ ] 实现交易队列
- [ ] 优化gas估算

### 3. 数据优化
- [ ] 实施缓存策略
- [ ] 创建数据索引
- [ ] 优化查询性能
- [ ] 实现数据压缩

### 4. 网络优化
- [ ] 管理连接池
- [ ] 优化请求策略
- [ ] 实现请求重试
- [ ] 优化数据传输

## 最佳实践

1. 性能监控
   - 持续监控关键指标
   - 设置性能基准
   - 定期性能审查
   - 优化瓶颈

2. 资源管理
   - 优化内存使用
   - 管理连接资源
   - 实施清理策略
   - 控制并发数量

3. 错误处理
   - 实现优雅降级
   - 添加重试机制
   - 记录性能问题
   - 监控错误率

## 下一步
- 查看[安全最佳实践](./security-best-practices.md)
- 了解[故障排除](./troubleshooting.md)
- 探索[示例项目](./example-projects.md)
