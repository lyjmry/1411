# World ID API 参考文档

## IDKit API

### WorldIDWidget
React组件，用于集成World ID验证。

```typescript
interface WorldIDWidgetProps {
    // 必需参数
    actionId: string;          // 动作标识符
    signal?: string;           // 验证信号
    
    // 回调函数
    onSuccess: (result: VerificationResult) => void;
    onError?: (error: Error) => void;
    onInitSuccess?: () => void;
    
    // 可选配置
    enableTelemetry?: boolean;    // 是否启用遥测
    theme?: 'light' | 'dark';     // 主题设置
    debug?: boolean;              // 是否启用调试模式
    
    // 自定义选项
    walletConnectProjectId?: string;
    appName?: string;
    signalDescription?: string;
}

// 使用示例
import { WorldIDWidget } from '@worldcoin/idkit';

function App() {
    return (
        <WorldIDWidget
            actionId="verify_user"
            signal="user_login"
            onSuccess={(result) => {
                console.log('验证成功:', result);
            }}
            onError={(error) => {
                console.error('验证失败:', error);
            }}
            theme="light"
            debug={true}
        />
    );
}
```

### VerificationResult
验证结果接口。

```typescript
interface VerificationResult {
    merkle_root: string;          // Merkle树根
    nullifier_hash: string;       // Nullifier哈希
    proof: string;                // 零知识证明
    credential_type: string;      // 凭证类型
    verification_level: string;   // 验证级别
    timestamp: number;            // 时间戳
}
```

## Core API

### WorldID
核心API类，提供底层功能。

```typescript
class WorldID {
    constructor(options: WorldIDOptions);
    
    // 验证方法
    async verify(params: VerifyParams): Promise<VerificationResult>;
    
    // 配置方法
    setOptions(options: Partial<WorldIDOptions>): void;
    
    // 工具方法
    static isValidProof(proof: string): boolean;
    static hashSignal(signal: string): string;
}

interface WorldIDOptions {
    appId: string;
    actionId: string;
    signal?: string;
    debug?: boolean;
    enableTelemetry?: boolean;
}

interface VerifyParams {
    signal: string;
    proof: string;
    nullifier_hash: string;
    merkle_root: string;
}
```

## Verification API

### Verifier
验证器类，用于处理验证逻辑。

```typescript
class Verifier {
    constructor(options: VerifierOptions);
    
    // 验证方法
    async verifyProof(proof: string): Promise<boolean>;
    async verifyCredential(credential: Credential): Promise<boolean>;
    
    // 高级验证
    async verifyWithRules(proof: string, rules: VerificationRule[]): Promise<boolean>;
    
    // 批量验证
    async verifyBatch(proofs: string[]): Promise<boolean[]>;
}

interface VerifierOptions {
    verificationLevel: 'orb' | 'device';
    maxAge?: number;  // 最大有效期（秒）
    strictMode?: boolean;
}

interface VerificationRule {
    type: string;
    params: any;
}
```

## Credential API

### CredentialManager
凭证管理类。

```typescript
class CredentialManager {
    // 凭证创建
    async createCredential(params: CredentialParams): Promise<Credential>;
    
    // 凭证验证
    async verifyCredential(credential: Credential): Promise<boolean>;
    
    // 凭证撤销
    async revokeCredential(credentialId: string): Promise<void>;
    
    // 凭证状态查询
    async checkCredentialStatus(credentialId: string): Promise<CredentialStatus>;
}

interface Credential {
    id: string;
    type: string;
    holder: string;
    issuanceDate: string;
    expirationDate?: string;
    proof: CredentialProof;
}

interface CredentialParams {
    type: string;
    holder: string;
    expiresIn?: number;
    attributes?: Record<string, any>;
}
```

## Storage API

### StorageManager
安全存储管理类。

```typescript
class StorageManager {
    // 存储方法
    async store(key: string, data: any): Promise<void>;
    async retrieve(key: string): Promise<any>;
    async remove(key: string): Promise<void>;
    
    // 批量操作
    async storeMany(items: Record<string, any>): Promise<void>;
    async retrieveMany(keys: string[]): Promise<Record<string, any>>;
    
    // 存储空间管理
    async clear(): Promise<void>;
    async getSize(): Promise<number>;
}
```

## Event API

### EventEmitter
事件管理类。

```typescript
class WorldIDEventEmitter {
    // 事件监听
    on(event: string, callback: EventCallback): void;
    once(event: string, callback: EventCallback): void;
    off(event: string, callback?: EventCallback): void;
    
    // 事件触发
    emit(event: string, data?: any): void;
}

type EventCallback = (data: any) => void;

// 预定义事件
const WorldIDEvents = {
    VERIFICATION_SUCCESS: 'verification:success',
    VERIFICATION_FAILURE: 'verification:failure',
    CREDENTIAL_CREATED: 'credential:created',
    CREDENTIAL_REVOKED: 'credential:revoked'
};
```

## Error Handling

### WorldIDError
错误处理类。

```typescript
class WorldIDError extends Error {
    constructor(code: string, message: string, details?: any);
    
    readonly code: string;
    readonly details?: any;
}

// 错误代码
const ErrorCodes = {
    INVALID_PROOF: 'INVALID_PROOF',
    EXPIRED_PROOF: 'EXPIRED_PROOF',
    INVALID_CREDENTIAL: 'INVALID_CREDENTIAL',
    NETWORK_ERROR: 'NETWORK_ERROR',
    VERIFICATION_FAILED: 'VERIFICATION_FAILED'
};
```

## Utility API

### WorldIDUtils
工具类。

```typescript
class WorldIDUtils {
    // 哈希计算
    static hashSignal(signal: string): string;
    static hashCredential(credential: Credential): string;
    
    // 验证工具
    static isValidProof(proof: string): boolean;
    static isValidCredential(credential: Credential): boolean;
    
    // 格式转换
    static formatProof(proof: string): FormattedProof;
    static parseCredential(raw: string): Credential;
}
```

## 常量定义

```typescript
// 验证级别
export const VerificationLevels = {
    DEVICE: 'device',
    ORB: 'orb'
} as const;

// 凭证类型
export const CredentialTypes = {
    IDENTITY: 'identity',
    VERIFICATION: 'verification',
    CUSTOM: 'custom'
} as const;

// 事件类型
export const EventTypes = {
    VERIFICATION_SUCCESS: 'verification:success',
    VERIFICATION_FAILURE: 'verification:failure',
    CREDENTIAL_CREATED: 'credential:created',
    CREDENTIAL_REVOKED: 'credential:revoked'
} as const;
```

## 配置选项

```typescript
interface WorldIDConfig {
    // 基础配置
    appId: string;
    actionId: string;
    version: string;
    
    // 功能配置
    features: {
        verification: boolean;
        credentials: boolean;
        storage: boolean;
    };
    
    // 安全配置
    security: {
        proofExpiration: number;
        strictMode: boolean;
        requireSignal: boolean;
    };
    
    // 网络配置
    network: {
        endpoint: string;
        timeout: number;
        retries: number;
    };
}
```

## 使用示例

### 基础验证流程
```javascript
// 初始化
const worldID = new WorldID({
    appId: 'your-app-id',
    actionId: 'verify_user'
});

// 执行验证
try {
    const result = await worldID.verify({
        signal: 'user_verification',
        proof: proofData
    });
    
    console.log('验证成功:', result);
} catch (error) {
    console.error('验证失败:', error);
}
```

### 自定义验证规则
```javascript
const verifier = new Verifier({
    verificationLevel: 'orb',
    strictMode: true
});

const rules = [
    {
        type: 'age',
        params: { minimum: 18 }
    },
    {
        type: 'location',
        params: { countries: ['US', 'CA'] }
    }
];

const isValid = await verifier.verifyWithRules(proof, rules);
```

### 事件处理
```javascript
const events = new WorldIDEventEmitter();

events.on(WorldIDEvents.VERIFICATION_SUCCESS, (result) => {
    console.log('验证成功:', result);
});

events.on(WorldIDEvents.VERIFICATION_FAILURE, (error) => {
    console.error('验证失败:', error);
});
```

## 注意事项

1. 安全考虑
   - 始终验证proof的有效性
   - 实施重放攻击保护
   - 安全存储凭证

2. 性能优化
   - 使用批量验证
   - 实施缓存策略
   - 优化网络请求

3. 错误处理
   - 实现完整的错误处理
   - 提供用户友好的错误信息
   - 记录错误日志

4. 最佳实践
   - 遵循API使用规范
   - 定期更新依赖
   - 保持文档更新
