# World Chain 安全最佳实践指南

## 智能合约安全

### 1. 访问控制
```solidity
// 基础访问控制合约
contract AccessControl {
    mapping(address => bool) public admins;
    mapping(address => mapping(bytes32 => bool)) public permissions;
    
    event AdminAdded(address indexed account);
    event AdminRemoved(address indexed account);
    event PermissionGranted(address indexed account, bytes32 permission);
    event PermissionRevoked(address indexed account, bytes32 permission);
    
    modifier onlyAdmin() {
        require(admins[msg.sender], "Not admin");
        _;
    }
    
    modifier hasPermission(bytes32 permission) {
        require(
            admins[msg.sender] || permissions[msg.sender][permission],
            "Permission denied"
        );
        _;
    }
    
    function addAdmin(address account) external onlyAdmin {
        admins[account] = true;
        emit AdminAdded(account);
    }
    
    function removeAdmin(address account) external onlyAdmin {
        admins[account] = false;
        emit AdminRemoved(account);
    }
    
    function grantPermission(
        address account,
        bytes32 permission
    ) external onlyAdmin {
        permissions[account][permission] = true;
        emit PermissionGranted(account, permission);
    }
}
```

### 2. 重入攻击防护
```solidity
// 防重入合约示例
contract ReentrancyGuard {
    uint256 private constant _NOT_ENTERED = 1;
    uint256 private constant _ENTERED = 2;
    uint256 private _status;
    
    constructor() {
        _status = _NOT_ENTERED;
    }
    
    modifier nonReentrant() {
        require(_status != _ENTERED, "Reentrant call");
        _status = _ENTERED;
        _;
        _status = _NOT_ENTERED;
    }
}

contract SecureContract is ReentrancyGuard {
    mapping(address => uint256) public balances;
    
    function withdraw() external nonReentrant {
        uint256 amount = balances[msg.sender];
        require(amount > 0, "No balance");
        
        balances[msg.sender] = 0;
        
        (bool success, ) = msg.sender.call{value: amount}("");
        require(success, "Transfer failed");
    }
}
```

### 3. 整数溢出保护
```solidity
// 安全数学运算库
library SafeMath {
    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        require(c >= a, "SafeMath: addition overflow");
        return c;
    }
    
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        require(b <= a, "SafeMath: subtraction overflow");
        return a - b;
    }
    
    function mul(uint256 a, uint256 b) internal pure returns (uint256) {
        if (a == 0) return 0;
        uint256 c = a * b;
        require(c / a == b, "SafeMath: multiplication overflow");
        return c;
    }
}
```

## 密钥管理

### 1. 密钥存储
```javascript
class SecureKeyManager {
    constructor() {
        this.keyStore = new SecureKeyStore();
    }

    // 安全存储密钥
    async storeKey(key, metadata) {
        // 加密密钥
        const encryptedKey = await this.encryptKey(key);
        
        // 存储加密的密钥
        await this.keyStore.store(encryptedKey, {
            ...metadata,
            timestamp: Date.now()
        });
    }

    // 密钥恢复
    async recoverKey(id, recoveryData) {
        // 验证恢复数据
        if (!this.validateRecoveryData(recoveryData)) {
            throw new Error('Invalid recovery data');
        }
        
        // 获取加密的密钥
        const encryptedKey = await this.keyStore.get(id);
        
        // 解密密钥
        return await this.decryptKey(encryptedKey, recoveryData);
    }
}
```

### 2. 密钥轮换
```javascript
class KeyRotation {
    constructor() {
        this.keyManager = new SecureKeyManager();
    }

    // 执行密钥轮换
    async rotateKeys() {
        // 生成新密钥
        const newKey = await this.generateKey();
        
        // 获取旧密钥
        const oldKey = await this.keyManager.getCurrentKey();
        
        // 重新加密数据
        await this.reencryptData(oldKey, newKey);
        
        // 更新密钥
        await this.keyManager.updateCurrentKey(newKey);
        
        // 安全销毁旧密钥
        await this.securelyDestroyKey(oldKey);
    }

    // 验证密钥状态
    async verifyKeyStatus() {
        const currentKey = await this.keyManager.getCurrentKey();
        
        return {
            isValid: await this.validateKey(currentKey),
            expiresIn: await this.getKeyExpiration(currentKey),
            needsRotation: await this.checkRotationNeeded(currentKey)
        };
    }
}
```

## 交易安全

### 1. 交易验证
```javascript
class TransactionValidator {
    // 验证交易
    async validateTransaction(tx) {
        // 验证基本参数
        if (!this.validateBasicParams(tx)) {
            throw new Error('Invalid transaction parameters');
        }
        
        // 验证签名
        if (!await this.verifySignature(tx)) {
            throw new Error('Invalid signature');
        }
        
        // 验证nonce
        if (!await this.verifyNonce(tx)) {
            throw new Error('Invalid nonce');
        }
        
        // 验证gas
        if (!await this.verifyGas(tx)) {
            throw new Error('Invalid gas settings');
        }
        
        return true;
    }

    // 验证签名
    async verifySignature(tx) {
        const signer = ethers.utils.verifyMessage(
            this.getMessageHash(tx),
            tx.signature
        );
        
        return signer === tx.from;
    }
}
```

### 2. 交易监控
```javascript
class TransactionMonitor {
    constructor() {
        this.alerts = new AlertSystem();
    }

    // 监控交易
    async monitorTransaction(txHash) {
        const tx = await worldChain.getTransaction(txHash);
        
        // 设置监控
        const subscription = worldChain.on(
            txHash,
            async (transaction) => {
                // 检查交易状态
                await this.checkTransactionStatus(transaction);
                
                // 检查异常模式
                await this.detectAnomalies(transaction);
                
                // 验证最终状态
                await this.verifyFinalState(transaction);
            }
        );
        
        return subscription;
    }

    // 检测异常
    async detectAnomalies(tx) {
        const anomalies = [];
        
        // 检查gas使用
        if (await this.isGasAnomaly(tx)) {
            anomalies.push('Unusual gas usage');
        }
        
        // 检查交易值
        if (await this.isValueAnomaly(tx)) {
            anomalies.push('Unusual transaction value');
        }
        
        // 检查交易模式
        if (await this.isPatternAnomaly(tx)) {
            anomalies.push('Unusual transaction pattern');
        }
        
        if (anomalies.length > 0) {
            await this.alerts.trigger('TRANSACTION_ANOMALY', {
                transaction: tx,
                anomalies
            });
        }
    }
}
```

## 数据安全

### 1. 数据加密
```javascript
class DataEncryption {
    constructor() {
        this.crypto = new WorldCrypto({
            algorithm: 'AES-GCM',
            keySize: 256
        });
    }

    // 加密数据
    async encryptData(data) {
        // 生成IV
        const iv = crypto.getRandomValues(new Uint8Array(12));
        
        // 加密数据
        const encryptedData = await this.crypto.encrypt(
            data,
            await this.getEncryptionKey(),
            iv
        );
        
        return {
            data: encryptedData,
            iv: Buffer.from(iv).toString('base64')
        };
    }

    // 解密数据
    async decryptData(encryptedData, iv) {
        try {
            return await this.crypto.decrypt(
                encryptedData,
                await this.getEncryptionKey(),
                Buffer.from(iv, 'base64')
            );
        } catch (error) {
            console.error('解密失败:', error);
            throw error;
        }
    }
}
```

### 2. 数据验证
```javascript
class DataValidator {
    // 验证输入数据
    validateInput(data, schema) {
        // 类型检查
        if (!this.checkType(data, schema.type)) {
            throw new Error('Invalid data type');
        }
        
        // 范围检查
        if (!this.checkRange(data, schema.range)) {
            throw new Error('Data out of range');
        }
        
        // 格式检查
        if (!this.checkFormat(data, schema.format)) {
            throw new Error('Invalid data format');
        }
        
        return true;
    }

    // 验证状态转换
    async validateStateTransition(from, to, transition) {
        // 验证状态有效性
        if (!this.isValidState(from) || !this.isValidState(to)) {
            throw new Error('Invalid state');
        }
        
        // 验证转换规则
        if (!this.isValidTransition(from, to, transition)) {
            throw new Error('Invalid state transition');
        }
        
        return true;
    }
}
```

## 网络安全

### 1. 节点安全
```javascript
class NodeSecurity {
    constructor() {
        this.firewall = new NetworkFirewall();
        this.monitor = new NetworkMonitor();
    }

    // 配置节点安全
    async configureNodeSecurity() {
        // 配置防火墙规则
        await this.firewall.configure({
            allowedIPs: ['xxx.xxx.xxx.xxx'],
            allowedPorts: [8545, 8546],
            maxConnections: 100
        });
        
        // 设置DDoS保护
        await this.configureDDoSProtection();
        
        // 启用加密通信
        await this.enableEncryptedCommunication();
    }

    // DDoS保护
    async configureDDoSProtection() {
        await this.firewall.setRateLimit({
            windowMs: 15 * 60 * 1000, // 15分钟
            max: 100 // 限制100个请求
        });
        
        await this.firewall.setBlacklist({
            maxFailedAttempts: 5,
            banDuration: 60 * 60 * 1000 // 1小时
        });
    }
}
```

### 2. 通信安全
```javascript
class SecureCommunication {
    constructor() {
        this.transport = new SecureTransport({
            encryption: 'TLS 1.3',
            certificatePinning: true
        });
    }

    // 安全通信
    async secureRequest(endpoint, data) {
        // 生成请求ID
        const requestId = this.generateRequestId();
        
        // 签名请求
        const signature = await this.signRequest(data);
        
        // 发送请求
        const response = await this.transport.send(endpoint, {
            data,
            requestId,
            signature,
            timestamp: Date.now()
        });
        
        // 验证响应
        await this.verifyResponse(response);
        
        return response;
    }

    // 验证响应
    async verifyResponse(response) {
        // 验证签名
        if (!await this.verifySignature(response)) {
            throw new Error('Invalid response signature');
        }
        
        // 验证时间戳
        if (!this.verifyTimestamp(response.timestamp)) {
            throw new Error('Response timeout');
        }
        
        return true;
    }
}
```

## 审计和监控

### 1. 安全审计
```javascript
class SecurityAuditor {
    // 执行安全审计
    async performAudit() {
        const results = await Promise.all([
            this.auditContracts(),
            this.auditTransactions(),
            this.auditPermissions(),
            this.auditNetwork()
        ]);
        
        return {
            timestamp: Date.now(),
            results,
            recommendations: this.generateRecommendations(results)
        };
    }

    // 合约审计
    async auditContracts() {
        return {
            vulnerabilities: await this.findVulnerabilities(),
            gasUsage: await this.analyzeGasUsage(),
            codeQuality: await this.assessCodeQuality()
        };
    }
}
```

### 2. 安全监控
```javascript
class SecurityMonitor {
    constructor() {
        this.alerts = new AlertSystem();
        this.metrics = new MetricsCollector();
    }

    // 启动监控
    startMonitoring() {
        // 监控交易
        this.monitorTransactions();
        
        // 监控合约
        this.monitorContracts();
        
        // 监控网络
        this.monitorNetwork();
        
        // 监控异常
        this.monitorAnomalies();
    }

    // 处理告警
    async handleAlert(alert) {
        // 记录告警
        await this.alerts.log(alert);
        
        // 执行响应措施
        await this.respondToAlert(alert);
        
        // 通知相关人员
        await this.notifyStakeholders(alert);
    }
}
```

## 安全检查清单

### 1. 合约安全
- [ ] 实施访问控制
- [ ] 防止重入攻击
- [ ] 使用安全数学运算
- [ ] 验证输入数据

### 2. 密钥安全
- [ ] 安全存储密钥
- [ ] 定期轮换密钥
- [ ] 实施密钥恢复机制
- [ ] 监控密钥使用

### 3. 网络安全
- [ ] 配置防火墙
- [ ] 启用DDoS保护
- [ ] 使用加密通信
- [ ] 实施证书固定

### 4. 监控审计
- [ ] 实施安全审计
- [ ] 配置监控告警
- [ ] 记录安全事件
- [ ] 定期安全评估

## 最佳实践

1. 开发实践
   - 遵循安全编码规范
   - 实施代码审查
   - 使用自动化测试
   - 保持依赖更新

2. 运维实践
   - 定期安全评估
   - 实施变更管理
   - 保持系统更新
   - 监控系统状态

3. 应急响应
   - 制定应急计划
   - 定期演练
   - 建立响应团队
   - 保持通信畅通

## 下一步
- 查看[性能优化指南](./performance-optimization.md)
- 了解[故障排除](./troubleshooting.md)
- 探索[示例项目](./example-projects.md)
