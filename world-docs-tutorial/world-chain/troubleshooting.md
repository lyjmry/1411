# World Chain 故障排除指南

## 常见问题

### 1. 连接问题

#### 问题描述
```javascript
Error: Could not connect to World Chain network
```

#### 解决方案
1. 检查网络配置
```javascript
// 正确的网络配置
const worldChain = new WorldChain({
    network: 'mainnet',
    rpcUrl: process.env.WORLD_CHAIN_RPC_URL,
    chainId: 1  // 确保chainId正确
});

// 验证连接
async function verifyConnection() {
    try {
        await worldChain.provider.getNetwork();
        console.log('连接成功');
    } catch (error) {
        console.error('连接失败:', error);
        // 实现重连逻辑
        await reconnect();
    }
}
```

2. 实现重连机制
```javascript
class ConnectionManager {
    constructor() {
        this.maxRetries = 3;
        this.retryDelay = 1000;
    }

    async reconnect() {
        for (let i = 0; i < this.maxRetries; i++) {
            try {
                await worldChain.connect();
                console.log('重连成功');
                return;
            } catch (error) {
                console.error(`重连尝试 ${i + 1} 失败:`, error);
                await new Promise(resolve => 
                    setTimeout(resolve, this.retryDelay * Math.pow(2, i))
                );
            }
        }
        throw new Error('重连失败，请检查网络设置');
    }
}
```

### 2. 交易问题

#### 问题描述
```javascript
Error: Transaction failed: insufficient funds
```

#### 解决方案
1. 检查交易参数
```javascript
class TransactionChecker {
    async validateTransaction(tx) {
        // 检查余额
        const balance = await worldChain.getBalance(tx.from);
        const totalCost = tx.value.add(
            tx.gasLimit.mul(tx.gasPrice)
        );
        
        if (balance.lt(totalCost)) {
            throw new Error('余额不足');
        }
        
        // 检查nonce
        const nonce = await worldChain.getTransactionCount(tx.from);
        if (tx.nonce < nonce) {
            throw new Error('Nonce过低');
        }
        
        return true;
    }
}
```

2. 实现交易重试
```javascript
class TransactionManager {
    async sendWithRetry(tx) {
        const maxAttempts = 3;
        
        for (let i = 0; i < maxAttempts; i++) {
            try {
                // 更新gas价格
                const gasPrice = await this.getOptimalGasPrice();
                tx.gasPrice = gasPrice;
                
                // 发送交易
                const response = await worldChain.sendTransaction(tx);
                return await response.wait();
            } catch (error) {
                if (i === maxAttempts - 1) throw error;
                
                // 等待后重试
                await new Promise(resolve => 
                    setTimeout(resolve, 1000 * Math.pow(2, i))
                );
            }
        }
    }
}
```

### 3. 合约问题

#### 问题描述
```javascript
Error: Contract deployment failed: out of gas
```

#### 解决方案
1. 优化合约部署
```javascript
class ContractDeployer {
    async estimateDeployment(contract, args) {
        // 估算gas
        const factory = new ContractFactory(
            contract.abi,
            contract.bytecode,
            signer
        );
        
        const estimatedGas = await factory.estimateGas.deploy(...args);
        
        // 添加安全边际
        return estimatedGas.mul(120).div(100); // 增加20%
    }

    async deployWithOptimization(contract, args) {
        const gasLimit = await this.estimateDeployment(contract, args);
        
        try {
            const deployment = await contract.deploy(...args, {
                gasLimit,
                gasPrice: await this.getOptimalGasPrice()
            });
            
            return await deployment.deployed();
        } catch (error) {
            console.error('部署失败:', error);
            throw error;
        }
    }
}
```

2. 合约验证
```javascript
class ContractValidator {
    async validateContract(address) {
        try {
            // 验证合约代码
            const code = await worldChain.getCode(address);
            if (code === '0x') {
                throw new Error('合约不存在');
            }
            
            // 验证接口
            const contract = new Contract(address, abi, provider);
            await contract.deployed();
            
            return true;
        } catch (error) {
            console.error('合约验证失败:', error);
            throw error;
        }
    }
}
```

### 4. 节点同步问题

#### 问题描述
```javascript
Error: Node is not synchronized
```

#### 解决方案
1. 检查节点状态
```javascript
class NodeChecker {
    async checkNodeStatus() {
        // 获取最新区块
        const latestBlock = await worldChain.getBlockNumber();
        
        // 获取节点同步状态
        const syncStatus = await worldChain.provider.send(
            'eth_syncing',
            []
        );
        
        if (syncStatus) {
            console.log('节点正在同步:', {
                currentBlock: syncStatus.currentBlock,
                highestBlock: syncStatus.highestBlock
            });
            return false;
        }
        
        return true;
    }
}
```

2. 实现节点切换
```javascript
class NodeManager {
    constructor() {
        this.nodes = [
            'https://rpc1.worldchain.org',
            'https://rpc2.worldchain.org',
            'https://rpc3.worldchain.org'
        ];
    }

    async switchToHealthyNode() {
        for (const node of this.nodes) {
            try {
                const provider = new JsonRpcProvider(node);
                await provider.getBlockNumber();
                
                // 切换到健康节点
                worldChain.setProvider(provider);
                return true;
            } catch (error) {
                console.error(`节点 ${node} 不可用:`, error);
            }
        }
        throw new Error('没有可用的节点');
    }
}
```

### 5. World ID验证问题

#### 问题描述
```javascript
Error: World ID verification failed
```

#### 解决方案
1. 验证流程检查
```javascript
class WorldIDVerifier {
    async verifyWithDebug(proof) {
        try {
            // 验证proof格式
            if (!this.isValidProofFormat(proof)) {
                throw new Error('Invalid proof format');
            }
            
            // 验证nullifier
            await this.checkNullifier(proof.nullifierHash);
            
            // 执行验证
            const result = await worldId.verify(proof);
            return result;
        } catch (error) {
            console.error('验证失败:', error);
            this.logVerificationError(error);
            throw error;
        }
    }

    logVerificationError(error) {
        // 记录详细错误信息
        const errorLog = {
            timestamp: Date.now(),
            error: error.message,
            stack: error.stack,
            context: {
                network: worldChain.network,
                blockNumber: worldChain.blockNumber
            }
        };
        
        console.error('验证错误日志:', errorLog);
    }
}
```

2. 重试机制
```javascript
class VerificationRetrier {
    async verifyWithRetry(proof) {
        const maxAttempts = 3;
        let lastError;
        
        for (let i = 0; i < maxAttempts; i++) {
            try {
                return await worldId.verify(proof);
            } catch (error) {
                lastError = error;
                
                if (this.isRetryableError(error)) {
                    await new Promise(resolve => 
                        setTimeout(resolve, 1000 * Math.pow(2, i))
                    );
                    continue;
                }
                
                throw error;
            }
        }
        
        throw lastError;
    }
}
```

## 调试技巧

### 1. 日志记录
```javascript
class Logger {
    constructor() {
        this.logs = [];
    }

    log(level, message, data) {
        const logEntry = {
            timestamp: Date.now(),
            level,
            message,
            data
        };
        
        this.logs.push(logEntry);
        console.log(`[${level}] ${message}`, data);
        
        // 保存到持久存储
        this.persistLogs();
    }

    async getLogs(filter) {
        return this.logs.filter(log => {
            return filter.level ? log.level === filter.level : true;
        });
    }
}
```

### 2. 交易追踪
```javascript
class TransactionTracer {
    async traceTransaction(txHash) {
        try {
            // 获取交易详情
            const tx = await worldChain.getTransaction(txHash);
            const receipt = await tx.wait();
            
            // 分析交易
            return {
                status: receipt.status,
                gasUsed: receipt.gasUsed.toString(),
                events: receipt.events,
                logs: receipt.logs
            };
        } catch (error) {
            console.error('交易追踪失败:', error);
            throw error;
        }
    }
}
```

## 预防措施

### 1. 健康检查
```javascript
class HealthChecker {
    async runHealthCheck() {
        const results = await Promise.all([
            this.checkNetwork(),
            this.checkNode(),
            this.checkContract(),
            this.checkWorldID()
        ]);
        
        return results.every(result => result.healthy);
    }

    async checkNetwork() {
        try {
            await worldChain.getNetwork();
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

### 2. 监控系统
```javascript
class Monitor {
    constructor() {
        this.metrics = new MetricsCollector();
        this.alerts = new AlertSystem();
    }

    startMonitoring() {
        // 监控交易
        this.monitorTransactions();
        
        // 监控节点
        this.monitorNode();
        
        // 监控合约
        this.monitorContracts();
    }

    async handleAlert(alert) {
        // 记录告警
        await this.alerts.log(alert);
        
        // 执行响应措施
        await this.handleAlertAction(alert);
    }
}
```

## 错误代码参考

| 错误代码 | 描述 | 解决方案 |
|---------|------|---------|
| CONN_001 | 连接失败 | 检查网络配置 |
| TX_001 | 交易失败 | 验证交易参数 |
| CONTRACT_001 | 合约错误 | 检查合约代码 |
| NODE_001 | 节点同步 | 等待同步完成 |
| WORLDID_001 | 验证失败 | 检查验证流程 |

## 下一步
- 查看[性能优化指南](./performance-optimization.md)
- 了解[安全最佳实践](./security-best-practices.md)
- 探索[示例项目](./example-projects.md)
