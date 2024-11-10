# World Chain API 参考文档

## 核心API

### WorldChain
主要的区块链交互类。

```typescript
class WorldChain {
    constructor(config: WorldChainConfig);
    
    // 连接方法
    async connect(): Promise<void>;
    async disconnect(): Promise<void>;
    
    // 账户管理
    async getAccounts(): Promise<string[]>;
    async getBalance(address: string): Promise<BigNumber>;
    
    // 交易方法
    async sendTransaction(tx: TransactionRequest): Promise<TransactionResponse>;
    async call(tx: CallRequest): Promise<string>;
}

interface WorldChainConfig {
    network: 'mainnet' | 'testnet' | 'local';
    provider?: Provider;
    signer?: Signer;
    apiKey?: string;
}
```

### Contract
智能合约交互类。

```typescript
class Contract {
    constructor(address: string, abi: ContractInterface, signer?: Signer);
    
    // 合约方法
    async deploy(args?: any[]): Promise<Contract>;
    async attach(address: string): Promise<Contract>;
    
    // 事件监听
    on(event: string, listener: Listener): Contract;
    once(event: string, listener: Listener): Contract;
    
    // 状态查询
    async estimateGas(method: string, ...args: any[]): Promise<BigNumber>;
    async queryFilter(event: string, fromBlock?: number, toBlock?: number): Promise<Event[]>;
}
```

## 交易API

### TransactionManager
交易管理类。

```typescript
class TransactionManager {
    // 交易创建
    async createTransaction(params: TransactionParams): Promise<Transaction>;
    
    // 交易签名
    async signTransaction(tx: Transaction): Promise<string>;
    
    // 交易发送
    async sendTransaction(signedTx: string): Promise<TransactionResponse>;
    
    // 交易状态
    async getTransactionStatus(hash: string): Promise<TransactionStatus>;
    
    // 交易确认
    async waitForConfirmation(hash: string, confirmations?: number): Promise<TransactionReceipt>;
}

interface TransactionParams {
    to: string;
    value?: BigNumberish;
    data?: string;
    nonce?: number;
    gasLimit?: BigNumberish;
    gasPrice?: BigNumberish;
}
```

## 账户API

### AccountManager
账户管理类。

```typescript
class AccountManager {
    // 账户创建
    async createAccount(): Promise<Account>;
    
    // 账户导入
    async importAccount(privateKey: string): Promise<Account>;
    async importMnemonic(mnemonic: string): Promise<Account>;
    
    // 账户验证
    async verifyAccount(address: string): Promise<boolean>;
    
    // 账户状态
    async getAccountStatus(address: string): Promise<AccountStatus>;
}

interface Account {
    address: string;
    privateKey: string;
    publicKey: string;
}
```

## 节点API

### NodeManager
节点管理类。

```typescript
class NodeManager {
    // 节点连接
    async connect(endpoint: string): Promise<void>;
    
    // 节点状态
    async getNodeStatus(): Promise<NodeStatus>;
    
    // 节点同步
    async syncNode(): Promise<void>;
    
    // 节点配置
    async updateNodeConfig(config: NodeConfig): Promise<void>;
}

interface NodeConfig {
    rpcUrl: string;
    chainId: number;
    networkId: number;
}
```

## 验证API

### WorldIDVerifier
World ID验证类。

```typescript
class WorldIDVerifier {
    constructor(config: VerifierConfig);
    
    // 验证方法
    async verifyProof(proof: Proof): Promise<boolean>;
    
    // 验证状态
    async checkVerificationStatus(id: string): Promise<VerificationStatus>;
    
    // 批量验证
    async verifyBatch(proofs: Proof[]): Promise<boolean[]>;
}

interface Proof {
    merkleRoot: string;
    nullifierHash: string;
    proof: string;
}
```

## 存储API

### StorageManager
链上存储管理类。

```typescript
class StorageManager {
    // 数据存储
    async store(key: string, value: any): Promise<string>;
    
    // 数据读取
    async retrieve(key: string): Promise<any>;
    
    // 数据删除
    async remove(key: string): Promise<void>;
    
    // 批量操作
    async batchStore(items: StorageItem[]): Promise<string[]>;
    async batchRetrieve(keys: string[]): Promise<any[]>;
}
```

## 事件API

### EventManager
事件管理类。

```typescript
class EventManager {
    // 事件订阅
    subscribe(event: string, callback: EventCallback): Subscription;
    
    // 事件过滤
    async getEvents(filter: EventFilter): Promise<Event[]>;
    
    // 事件监听
    async watchEvents(filter: EventFilter, callback: EventCallback): Promise<void>;
}

interface EventFilter {
    address?: string;
    topics?: string[];
    fromBlock?: number;
    toBlock?: number;
}
```

## 工具类

### WorldChainUtils
工具函数集合。

```typescript
class WorldChainUtils {
    // 地址相关
    static isValidAddress(address: string): boolean;
    static toCheckSumAddress(address: string): string;
    
    // 数值转换
    static toWei(value: string, unit?: string): BigNumber;
    static fromWei(value: BigNumber, unit?: string): string;
    
    // 哈希计算
    static keccak256(value: string): string;
    static sha256(value: string): string;
}
```

## 类型定义

### 基础类型
```typescript
type Address = string;
type Hash = string;
type BigNumberish = number | string | BigNumber;

interface Block {
    hash: Hash;
    parentHash: Hash;
    number: number;
    timestamp: number;
    transactions: Transaction[];
}

interface Transaction {
    hash: Hash;
    from: Address;
    to: Address;
    value: BigNumber;
    data?: string;
    nonce: number;
}
```

### 响应类型
```typescript
interface TransactionResponse {
    hash: Hash;
    confirmations: number;
    from: Address;
    wait(confirmations?: number): Promise<TransactionReceipt>;
}

interface TransactionReceipt {
    to: Address;
    from: Address;
    contractAddress?: Address;
    status: number;
    logs: Log[];
}
```

## 常量定义

```typescript
// 网络类型
export const Networks = {
    MAINNET: 'mainnet',
    TESTNET: 'testnet',
    LOCAL: 'local'
} as const;

// 交易状态
export const TransactionStatus = {
    PENDING: 'pending',
    CONFIRMED: 'confirmed',
    FAILED: 'failed'
} as const;

// 事件类型
export const EventTypes = {
    TRANSFER: 'Transfer',
    APPROVAL: 'Approval',
    VERIFICATION: 'Verification'
} as const;
```

## 配置选项

```typescript
interface WorldChainConfig {
    // 网络配置
    network: {
        name: string;
        chainId: number;
        rpcUrl: string;
    };
    
    // 账户配置
    account?: {
        address?: string;
        privateKey?: string;
    };
    
    // API配置
    api?: {
        key?: string;
        endpoint?: string;
        timeout?: number;
    };
    
    // 验证配置
    verification?: {
        required: boolean;
        level: 'basic' | 'advanced';
    };
}
```

## 使用示例

### 基础连接
```javascript
// 初始化
const worldChain = new WorldChain({
    network: 'testnet',
    provider: window.ethereum
});

// 连接
await worldChain.connect();

// 获取账户
const accounts = await worldChain.getAccounts();
console.log('Connected accounts:', accounts);
```

### 合约交互
```javascript
// 加载合约
const contract = new Contract(
    contractAddress,
    contractABI,
    signer
);

// 调用方法
const result = await contract.someMethod();

// 监听事件
contract.on('SomeEvent', (data) => {
    console.log('Event received:', data);
});
```

### 交易处理
```javascript
// 创建交易
const tx = await transactionManager.createTransaction({
    to: recipient,
    value: utils.parseEther('1.0')
});

// 发送交易
const response = await transactionManager.sendTransaction(tx);

// 等待确认
const receipt = await response.wait(2); // 等待2个确认
```

## 注意事项

1. 安全考虑
   - 保护私钥安全
   - 验证所有输入
   - 使用安全的RPC端点

2. 性能优化
   - 实施适当的缓存策略
   - 批量处理交易
   - 优化事件监听

3. 错误处理
   - 实现完整的错误处理
   - 提供用户友好的错误信息
   - 记录错误日志

4. 最佳实践
   - 遵循API使用规范
   - 实施重试机制
   - 保持文档更新
