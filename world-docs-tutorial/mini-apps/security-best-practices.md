# Mini Apps 安全最佳实践指南

## 身份认证安全

### 1. 安全的认证实现
```javascript
class SecureAuth {
    constructor() {
        this.auth = new WorldAuth({
            encryption: 'AES-256-GCM',
            tokenStorage: 'secure-enclave'
        });
    }

    // 安全的登录实现
    async login(credentials) {
        try {
            // 验证输入
            this.validateCredentials(credentials);
            
            // 加密敏感数据
            const encryptedCredentials = await this.encryptCredentials(credentials);
            
            // 执行认证
            const result = await this.auth.login(encryptedCredentials);
            
            // 安全存储令牌
            await this.secureTokenStorage(result.token);
            
            return result;
        } catch (error) {
            this.handleAuthError(error);
            throw error;
        }
    }

    // 安全的令牌刷新
    async refreshToken() {
        try {
            const currentToken = await this.getSecureToken();
            if (!this.isTokenValid(currentToken)) {
                throw new Error('Invalid token');
            }
            
            const newToken = await this.auth.refreshToken(currentToken);
            await this.secureTokenStorage(newToken);
            
            return newToken;
        } catch (error) {
            await this.handleTokenError(error);
            throw error;
        }
    }
}
```

### 2. 多因素认证
```javascript
class MFAHandler {
    // 设置MFA
    async setupMFA(userId) {
        // 生成安全的密钥
        const secret = await this.generateSecureSecret();
        
        // 创建加密的备份码
        const backupCodes = await this.generateBackupCodes();
        
        // 安全存储
        await this.secureMFAStorage(userId, {
            secret,
            backupCodes: await this.hashBackupCodes(backupCodes)
        });
        
        return {
            secret,
            backupCodes
        };
    }

    // 验证MFA
    async verifyMFA(userId, code) {
        const mfaData = await this.getMFAData(userId);
        
        // 防止暴力破解
        await this.rateLimiter.checkLimit(userId);
        
        // 验证码检查
        const isValid = await this.verifyCode(code, mfaData.secret);
        
        // 记录验证尝试
        await this.logVerificationAttempt(userId, isValid);
        
        return isValid;
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
        // 生成随机IV
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
            this.handleDecryptionError(error);
            throw error;
        }
    }
}
```

### 2. 安全存储
```javascript
class SecureStorage {
    constructor() {
        this.storage = new WorldStorage({
            encryption: true,
            secureEnclave: true
        });
    }

    // 安全存储数据
    async secureStore(key, data) {
        // 数据验证
        this.validateData(data);
        
        // 加密数据
        const encryptedData = await this.encryptData(data);
        
        // 安全存储
        await this.storage.set(key, encryptedData, {
            secure: true,
            accessControl: {
                biometric: true,
                timeout: 300 // 5分钟超时
            }
        });
    }

    // 安全读取数据
    async secureRetrieve(key) {
        // 验证访问权限
        await this.verifyAccess();
        
        // 获取加密数据
        const encryptedData = await this.storage.get(key);
        
        // 解密数据
        return await this.decryptData(encryptedData);
    }
}
```

## 通信安全

### 1. 安全的API调用
```javascript
class SecureAPI {
    constructor() {
        this.http = new WorldHttp({
            baseURL: 'https://api.example.com',
            timeout: 10000,
            headers: {
                'Content-Type': 'application/json',
                'X-Security-Version': '1.0'
            }
        });
    }

    // 安全的请求处理
    async secureRequest(config) {
        try {
            // 添加安全头
            const secureConfig = await this.addSecurityHeaders(config);
            
            // 请求签名
            const signature = await this.signRequest(secureConfig);
            secureConfig.headers['X-Signature'] = signature;
            
            // 发送请求
            const response = await this.http.request(secureConfig);
            
            // 验证响应
            await this.verifyResponse(response);
            
            return response.data;
        } catch (error) {
            this.handleRequestError(error);
            throw error;
        }
    }

    // 请求签名
    async signRequest(config) {
        const payload = this.prepareSignaturePayload(config);
        return await this.crypto.sign(payload, await this.getSigningKey());
    }
}
```

### 2. WebSocket安全
```javascript
class SecureWebSocket {
    constructor() {
        this.ws = new WorldWebSocket({
            encryption: true,
            heartbeat: true
        });
    }

    // 安全连接
    async secureConnect(url) {
        // 生成连接令牌
        const token = await this.generateConnectionToken();
        
        // 建立加密连接
        this.connection = await this.ws.connect(url, {
            headers: {
                'Authorization': `Bearer ${token}`,
                'X-Client-ID': await this.getClientId()
            },
            protocols: ['wss']
        });
        
        // 设置安全处理器
        this.setupSecurityHandlers();
    }

    // 安全消息处理
    async secureMessage(message) {
        // 消息加密
        const encryptedMessage = await this.encryptMessage(message);
        
        // 添加消息签名
        const signature = await this.signMessage(message);
        
        return {
            data: encryptedMessage,
            signature
        };
    }
}
```

## 代码安全

### 1. 输入验证
```javascript
class InputValidator {
    // 验证用户输入
    validateInput(input, schema) {
        // XSS防护
        input = this.sanitizeInput(input);
        
        // SQL注入防护
        input = this.preventSQLInjection(input);
        
        // 模式验证
        const validation = schema.validate(input);
        if (validation.error) {
            throw new ValidationError(validation.error);
        }
        
        return validation.value;
    }

    // 清理HTML内容
    sanitizeHTML(content) {
        return DOMPurify.sanitize(content, {
            ALLOWED_TAGS: ['b', 'i', 'em', 'strong'],
            ALLOWED_ATTR: []
        });
    }
}
```

### 2. 错误处理
```javascript
class SecureErrorHandler {
    handleError(error) {
        // 清理敏感信息
        const sanitizedError = this.sanitizeError(error);
        
        // 记录错误
        this.logError(sanitizedError);
        
        // 返回安全的错误响应
        return {
            code: sanitizedError.code,
            message: this.getPublicErrorMessage(sanitizedError),
            requestId: this.generateRequestId()
        };
    }

    // 清理敏感信息
    sanitizeError(error) {
        const sanitized = {...error};
        delete sanitized.stack;
        delete sanitized.details;
        return sanitized;
    }
}
```

## 安全配置

### 1. 安全策略配置
```javascript
const securityConfig = {
    // CSP配置
    contentSecurityPolicy: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "'unsafe-inline'"],
        styleSrc: ["'self'", "'unsafe-inline'"],
        imgSrc: ["'self'", "data:", "https:"],
        connectSrc: ["'self'", "wss:", "https:"]
    },
    
    // 安全头配置
    securityHeaders: {
        'Strict-Transport-Security': 'max-age=31536000; includeSubDomains',
        'X-Content-Type-Options': 'nosniff',
        'X-Frame-Options': 'DENY',
        'X-XSS-Protection': '1; mode=block'
    },
    
    // 认证配置
    auth: {
        sessionTimeout: 3600,
        maxAttempts: 5,
        lockoutDuration: 900
    }
};
```

### 2. 权限控制
```javascript
class PermissionManager {
    // 检查权限
    async checkPermission(userId, resource, action) {
        // 获取用户角色
        const roles = await this.getUserRoles(userId);
        
        // 获取权限策略
        const policies = await this.getPermissionPolicies(roles);
        
        // 评估权限
        return this.evaluatePermission(policies, resource, action);
    }

    // 权限评估
    evaluatePermission(policies, resource, action) {
        return policies.some(policy => 
            policy.resources.includes(resource) &&
            policy.actions.includes(action)
        );
    }
}
```

## 安全监控

### 1. 安全日志
```javascript
class SecurityLogger {
    // 记录安全事件
    logSecurityEvent(event) {
        const logEntry = {
            timestamp: new Date().toISOString(),
            type: event.type,
            severity: event.severity,
            details: this.sanitizeLogDetails(event.details),
            userId: event.userId,
            ip: event.ip,
            userAgent: event.userAgent
        };
        
        // 写入安全日志
        this.writeSecurityLog(logEntry);
        
        // 检查是否需要报警
        this.checkAlertThreshold(logEntry);
    }

    // 日志分析
    async analyzeSecurityLogs() {
        const logs = await this.getSecurityLogs();
        return {
            failedLogins: this.analyzeFailedLogins(logs),
            suspiciousActivities: this.analyzeSuspiciousActivities(logs),
            vulnerabilities: this.analyzeVulnerabilities(logs)
        };
    }
}
```

### 2. 安全监控
```javascript
class SecurityMonitor {
    constructor() {
        this.monitor = new WorldSecurityMonitor();
    }

    // 启动安全监控
    startMonitoring() {
        this.monitor.start({
            intervals: {
                basic: 60,    // 基础检查间隔（秒）
                deep: 3600    // 深度检查间隔（秒）
            },
            checks: {
                auth: true,
                network: true,
                storage: true,
                integrity: true
            }
        });
        
        this.setupAlerts();
    }

    // 设置告警
    setupAlerts() {
        this.monitor.onThreat(async (threat) => {
            // 记录威胁
            await this.logThreat(threat);
            
            // 执行响应措施
            await this.respondToThreat(threat);
            
            // 发送告警
            await this.sendAlert(threat);
        });
    }
}
```

## 安全检查清单

### 1. 认证安全
- [ ] 实施强密码策略
- [ ] 启用多因素认证
- [ ] 实现安全的会话管理
- [ ] 防止暴力破解攻击

### 2. 数据安全
- [ ] 加密敏感数据
- [ ] 安全存储密钥
- [ ] 实现安全的数据传输
- [ ] 定期数据备份

### 3. 代码安全
- [ ] 输入验证和清理
- [ ] 安全的错误处理
- [ ] 防止代码注入
- [ ] 定期代码审查

### 4. 配置安全
- [ ] 安全的默认配置
- [ ] 最小权限原则
- [ ] 禁用不必要的功能
- [ ] 定期更新依赖

## 最佳实践

1. 定期安全审计
2. 保持依赖更新
3. 实施安全监控
4. 制定事件响应计划
5. 进行安全培训

## 下一步
- 查看[性能优化指南](./performance-optimization.md)
- 了解[故障排除](./troubleshooting.md)
- 探索[高级功能](./advanced-features.md)
