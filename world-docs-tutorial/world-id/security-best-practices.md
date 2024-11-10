# World ID 安全最佳实践指南

## 身份验证安全

### 1. 验证流程安全
```javascript
class SecureVerification {
    constructor() {
        this.worldID = new WorldID({
            appId: process.env.WORLD_APP_ID,
            security: {
                proofExpiration: 300,  // 5分钟过期
                requireSignal: true,    // 必须提供signal
                strictMode: true        // 启用严格模式
            }
        });
    }

    // 安全的验证实现
    async verifyIdentity(signal) {
        try {
            // 验证输入
            this.validateInput(signal);
            
            // 生成安全的验证参数
            const verificationParams = await this.generateSecureParams(signal);
            
            // 执行验证
            const result = await this.worldID.verify(verificationParams);
            
            // 验证结果完整性
            await this.verifyResultIntegrity(result);
            
            return result;
        } catch (error) {
            this.handleSecurityError(error);
            throw error;
        }
    }

    // 验证参数生成
    async generateSecureParams(signal) {
        return {
            signal: await this.hashSignal(signal),
            action: this.generateSecureAction(),
            timestamp: Date.now()
        };
    }
}
```

### 2. 防重放攻击
```javascript
class ReplayProtection {
    constructor() {
        this.usedNullifiers = new Set();
        this.nullifierExpiration = 24 * 60 * 60 * 1000; // 24小时
    }

    // 检查并记录nullifier
    async checkNullifier(nullifier) {
        // 验证nullifier格式
        if (!this.isValidNullifierFormat(nullifier)) {
            throw new SecurityError('Invalid nullifier format');
        }

        // 检查是否已使用
        if (this.usedNullifiers.has(nullifier)) {
            throw new SecurityError('Nullifier already used');
        }

        // 记录nullifier
        this.usedNullifiers.add(nullifier);
        
        // 设置过期清理
        setTimeout(() => {
            this.usedNullifiers.delete(nullifier);
        }, this.nullifierExpiration);
    }

    // 批量验证nullifiers
    async verifyNullifierBatch(nullifiers) {
        return Promise.all(
            nullifiers.map(nullifier => this.checkNullifier(nullifier))
        );
    }
}
```

## 数据安全

### 1. 敏感数据处理
```javascript
class DataSecurity {
    constructor() {
        this.crypto = new WorldCrypto({
            algorithm: 'AES-GCM',
            keySize: 256
        });
    }

    // 加密敏感数据
    async encryptSensitiveData(data) {
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

    // 安全存储
    async secureStore(key, data) {
        // 加密数据
        const encrypted = await this.encryptSensitiveData(data);
        
        // 安全存储
        await this.storage.set(key, encrypted, {
            secure: true,
            accessControl: {
                biometric: true,
                timeout: 300 // 5分钟超时
            }
        });
    }
}
```

### 2. 密钥管理
```javascript
class KeyManager {
    constructor() {
        this.keyStore = new SecureKeyStore();
    }

    // 生成密钥
    async generateKey() {
        const key = await crypto.subtle.generateKey(
            {
                name: 'AES-GCM',
                length: 256
            },
            true,
            ['encrypt', 'decrypt']
        );

        // 安全存储密钥
        await this.keyStore.storeKey(key);
        
        return key;
    }

    // 密钥轮换
    async rotateKeys() {
        // 生成新密钥
        const newKey = await this.generateKey();
        
        // 重新加密数据
        await this.reencryptData(newKey);
        
        // 废弃旧密钥
        await this.revokeOldKeys();
    }
}
```

## 通信安全

### 1. 安全通信
```javascript
class SecureCommunication {
    constructor() {
        this.transport = new SecureTransport({
            encryption: 'TLS 1.3',
            certificatePinning: true
        });
    }

    // 安全请求
    async secureRequest(endpoint, data) {
        // 添加安全头
        const headers = await this.generateSecureHeaders();
        
        // 签名请求
        const signature = await this.signRequest(data);
        
        // 发送请求
        const response = await this.transport.send(endpoint, {
            data,
            headers,
            signature
        });
        
        // 验证响应
        await this.verifyResponse(response);
        
        return response;
    }

    // 请求签名
    async signRequest(data) {
        const payload = this.prepareSignaturePayload(data);
        return await this.crypto.sign(payload, await this.getSigningKey());
    }
}
```

### 2. 证书验证
```javascript
class CertificateValidator {
    // 验证证书
    async validateCertificate(cert) {
        // 检查证书有效期
        if (!this.isValidPeriod(cert)) {
            throw new SecurityError('Certificate expired');
        }
        
        // 验证证书链
        await this.validateCertChain(cert);
        
        // 检查吊销状态
        await this.checkRevocationStatus(cert);
    }

    // 证书固定
    async pinCertificate(cert) {
        const fingerprint = await this.calculateFingerprint(cert);
        await this.storePinnedCert(fingerprint);
    }
}
```

## 访问控制

### 1. 权限管理
```javascript
class PermissionManager {
    constructor() {
        this.acl = new AccessControlList();
    }

    // 检查权限
    async checkPermission(user, resource, action) {
        // 获取用户角色
        const roles = await this.getUserRoles(user);
        
        // 获取资源策略
        const policies = await this.getResourcePolicies(resource);
        
        // 评估权限
        return this.evaluatePermission(roles, policies, action);
    }

    // 权限评估
    evaluatePermission(roles, policies, action) {
        return roles.some(role => 
            policies.some(policy =>
                policy.roles.includes(role) &&
                policy.actions.includes(action)
            )
        );
    }
}
```

### 2. 会话管理
```javascript
class SessionManager {
    constructor() {
        this.sessionStore = new SecureSessionStore();
    }

    // 创建安全会话
    async createSession(user, verification) {
        const session = {
            id: this.generateSecureId(),
            userId: user.id,
            verification: verification.id,
            created: Date.now(),
            expires: Date.now() + (24 * 60 * 60 * 1000) // 24小时
        };

        // 加密会话数据
        const encryptedSession = await this.encryptSession(session);
        
        // 存储会话
        await this.sessionStore.set(session.id, encryptedSession);
        
        return session.id;
    }

    // 验证会话
    async validateSession(sessionId) {
        const session = await this.getSession(sessionId);
        
        if (!session) {
            throw new SecurityError('Invalid session');
        }

        if (Date.now() > session.expires) {
            throw new SecurityError('Session expired');
        }

        return session;
    }
}
```

## 错误处理

### 1. 安全错误处理
```javascript
class SecurityErrorHandler {
    // 处理安全错误
    handleSecurityError(error) {
        // 清理敏感信息
        const sanitizedError = this.sanitizeError(error);
        
        // 记录安全事件
        this.logSecurityEvent({
            type: 'security_error',
            error: sanitizedError,
            timestamp: Date.now()
        });
        
        // 返回安全的错误响应
        return {
            code: sanitizedError.code,
            message: this.getPublicErrorMessage(sanitizedError)
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

### 2. 审计日志
```javascript
class SecurityAuditor {
    // 记录安全事件
    async logSecurityEvent(event) {
        const auditLog = {
            timestamp: Date.now(),
            type: event.type,
            severity: this.calculateSeverity(event),
            details: this.sanitizeEventDetails(event.details),
            metadata: {
                userId: event.userId,
                ip: event.ip,
                userAgent: event.userAgent
            }
        };

        // 存储审计日志
        await this.auditStore.save(auditLog);
        
        // 检查是否需要报警
        await this.checkAlertThreshold(auditLog);
    }

    // 生成审计报告
    async generateAuditReport(timeRange) {
        const logs = await this.auditStore.query(timeRange);
        
        return {
            summary: this.summarizeLogs(logs),
            details: this.analyzeLogs(logs),
            recommendations: this.generateRecommendations(logs)
        };
    }
}
```

## 安全监控

### 1. 实时监控
```javascript
class SecurityMonitor {
    constructor() {
        this.alerts = new AlertSystem();
        this.metrics = new SecurityMetrics();
    }

    // 启动监控
    startMonitoring() {
        // 监控验证尝试
        this.monitorVerifications();
        
        // 监控异常行为
        this.monitorAnomalies();
        
        // 监控系统健康状况
        this.monitorSystemHealth();
    }

    // 处理安全事件
    async handleSecurityEvent(event) {
        // 评估威胁级别
        const threatLevel = this.assessThreat(event);
        
        // 记录事件
        await this.logSecurityEvent(event);
        
        // 执行响应措施
        await this.respondToThreat(event, threatLevel);
    }
}
```

### 2. 威胁检测
```javascript
class ThreatDetector {
    // 检测异常行为
    async detectAnomalies(data) {
        // 分析行为模式
        const patterns = await this.analyzePatterns(data);
        
        // 检测异常
        const anomalies = this.findAnomalies(patterns);
        
        // 评估风险
        return this.assessRisk(anomalies);
    }

    // 实时警报
    async triggerAlert(threat) {
        // 发送警报
        await this.alerts.send({
            level: threat.severity,
            message: threat.description,
            data: threat.details
        });
        
        // 执行自动响应
        await this.autoRespond(threat);
    }
}
```

## 安全配置

### 1. 安全设置
```javascript
const securityConfig = {
    // 验证配置
    verification: {
        proofExpiration: 300,        // 5分钟
        maxAttempts: 3,              // 最大尝试次数
        lockoutDuration: 900,        // 15分钟锁定
        requireSignal: true,         // 必须提供signal
        strictMode: true             // 严格模式
    },
    
    // 加密配置
    encryption: {
        algorithm: 'AES-GCM',
        keySize: 256,
        keyRotationInterval: 7776000 // 90天
    },
    
    // 会话配置
    session: {
        duration: 3600,              // 1小时
        renewalThreshold: 300,       // 5分钟
        maxConcurrent: 5             // 最大并发会话
    }
};
```

### 2. 安全策略
```javascript
const securityPolicies = {
    // 密码策略
    password: {
        minLength: 12,
        requireNumbers: true,
        requireSymbols: true,
        requireUppercase: true,
        maxAge: 7776000              // 90天过期
    },
    
    // 访问控制
    access: {
        maxFailedAttempts: 5,
        lockoutDuration: 900,        // 15分钟
        requireMFA: true,
        sessionTimeout: 3600         // 1小时
    },
    
    // 网络安全
    network: {
        allowedOrigins: ['https://example.com'],
        allowedMethods: ['GET', 'POST'],
        rateLimiting: {
            windowMs: 900000,        // 15分钟
            maxRequests: 100         // 最大请求数
        }
    }
};
```

## 安全检查清单

### 1. 基础安全
- [ ] 实施强密码策略
- [ ] 启用多因素认证
- [ ] 加密敏感数据
- [ ] 实现安全会话管理

### 2. 通信安全
- [ ] 使用TLS加密
- [ ] 实施证书固定
- [ ] 验证请求签名
- [ ] 保护API端点

### 3. 数据安全
- [ ] 加密存储数据
- [ ] 安全密钥管理
- [ ] 实施数据备份
- [ ] 定期数据清理

### 4. 监控和审计
- [ ] 记录安全事件
- [ ] 监控异常行为
- [ ] 实施入侵检测
- [ ] 定期安全审计

## 最佳实践

1. 安全开发
   - 遵循安全编码规范
   - 实施代码审查
   - 定期安全培训
   - 保持依赖更新

2. 运行时安全
   - 监控系统活动
   - 实施入侵检测
   - 定期安全评估
   - 及时响应事件

3. 数据保护
   - 加密敏感数据
   - 安全密钥管理
   - 实施访问控制
   - 定期数据备份

## 下一步
- 查看[性能优化指南](./performance-optimization.md)
- 了解[故障排除](./troubleshooting.md)
- 探索[示例项目](./example-projects.md)
