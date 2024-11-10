# Mini Apps 示例项目

## 1. 数字支付应用

这个示例展示如何创建一个完整的数字支付应用，包括用户认证、支付处理和交易历史。

### 项目结构
```
digital-payment/
├── src/
│   ├── components/
│   │   ├── PaymentForm.js
│   │   ├── TransactionHistory.js
│   │   └── UserProfile.js
│   ├── services/
│   │   ├── auth.js
│   │   ├── payment.js
│   │   └── storage.js
│   └── pages/
│       ├── Home.js
│       ├── Payment.js
│       └── History.js
```

### 核心代码示例

#### 支付表单组件
```javascript
// components/PaymentForm.js
import { WorldPayment, WorldUI } from '@world/minikit';

export function PaymentForm() {
    const [amount, setAmount] = useState('');
    const [recipient, setRecipient] = useState('');
    const payment = new WorldPayment();

    async function handlePayment() {
        try {
            const result = await payment.createPayment({
                amount: parseFloat(amount),
                recipient,
                currency: 'USDC'
            });

            if (result.success) {
                WorldUI.Toast.success('支付成功！');
            }
        } catch (error) {
            WorldUI.Toast.error('支付失败：' + error.message);
        }
    }

    return (
        <form onSubmit={handlePayment}>
            <WorldUI.Input
                label="金额"
                value={amount}
                onChange={e => setAmount(e.target.value)}
                type="number"
            />
            <WorldUI.Input
                label="接收方"
                value={recipient}
                onChange={e => setRecipient(e.target.value)}
            />
            <WorldUI.Button type="submit">
                确认支付
            </WorldUI.Button>
        </form>
    );
}
```

#### 交易历史服务
```javascript
// services/payment.js
export class PaymentService {
    constructor() {
        this.payment = new WorldPayment();
        this.storage = new WorldStorage();
    }

    async getTransactionHistory() {
        try {
            const history = await this.payment.getTransactions({
                limit: 50,
                order: 'desc'
            });

            await this.storage.set('transaction_history', history);
            return history;
        } catch (error) {
            console.error('获取交易历史失败:', error);
            throw error;
        }
    }

    async saveTransaction(transaction) {
        try {
            await this.storage.set(
                `transaction_${transaction.id}`,
                transaction
            );
        } catch (error) {
            console.error('保存交易记录失败:', error);
            throw error;
        }
    }
}
```

## 2. 身份验证应用

展示如何实现高级身份验证功能，包括多因素认证和生物识别。

### 项目结构
```
identity-verifier/
├── src/
│   ├── components/
│   │   ├── IdentityVerifier.js
│   │   ├── BiometricAuth.js
│   │   └── MFASetup.js
│   ├── services/
│   │   ├── auth.js
│   │   └── biometric.js
│   └── pages/
│       ├── Login.js
│       ├── Verify.js
│       └── Dashboard.js
```

### 核心代码示例

#### 身份验证组件
```javascript
// components/IdentityVerifier.js
import { WorldAuth, WorldUI } from '@world/minikit';

export function IdentityVerifier() {
    const auth = new WorldAuth();
    const [verificationStep, setVerificationStep] = useState(1);

    async function handleVerification() {
        try {
            // 步骤1：基础身份验证
            const basicAuth = await auth.verifyBasicIdentity();
            setVerificationStep(2);

            // 步骤2：生物识别
            const bioAuth = await auth.verifyBiometric();
            setVerificationStep(3);

            // 步骤3：MFA验证
            const mfaResult = await auth.verifyMFA();

            if (mfaResult.success) {
                WorldUI.Toast.success('身份验证完成！');
            }
        } catch (error) {
            WorldUI.Toast.error('验证失败：' + error.message);
        }
    }

    return (
        <div>
            <WorldUI.ProgressSteps
                current={verificationStep}
                steps={[
                    '基础验证',
                    '生物识别',
                    'MFA验证'
                ]}
            />
            <WorldUI.Button onClick={handleVerification}>
                开始验证
            </WorldUI.Button>
        </div>
    );
}
```

## 3. 智能合约钱包

展示如何创建一个与智能合约交互的钱包应用。

### 项目结构
```
smart-wallet/
├── contracts/
│   ├── Wallet.sol
│   └── Token.sol
├── src/
│   ├── components/
│   │   ├── WalletBalance.js
│   │   ├── TransactionList.js
│   │   └── SendForm.js
│   ├── services/
│   │   ├── wallet.js
│   │   └── contract.js
│   └── pages/
│       ├── Dashboard.js
│       ├── Send.js
│       └── Receive.js
```

### 核心代码示例

#### 智能合约
```solidity
// contracts/Wallet.sol
pragma solidity ^0.8.0;

contract SmartWallet {
    mapping(address => uint256) public balances;
    mapping(address => bool) public verified;

    event Transfer(address indexed from, address indexed to, uint256 amount);
    event Verification(address indexed user, bool status);

    function verifyIdentity(bytes memory proof) public {
        require(WorldID.verify(proof), "Invalid proof");
        verified[msg.sender] = true;
        emit Verification(msg.sender, true);
    }

    function transfer(address to, uint256 amount) public {
        require(verified[msg.sender], "Not verified");
        require(balances[msg.sender] >= amount, "Insufficient balance");

        balances[msg.sender] -= amount;
        balances[to] += amount;

        emit Transfer(msg.sender, to, amount);
    }
}
```

#### 钱包服务
```javascript
// services/wallet.js
export class WalletService {
    constructor() {
        this.contract = new WorldContract({
            address: WALLET_CONTRACT_ADDRESS,
            abi: WALLET_ABI
        });
    }

    async getBalance(address) {
        try {
            const balance = await this.contract.balances(address);
            return balance;
        } catch (error) {
            console.error('获取余额失败:', error);
            throw error;
        }
    }

    async transfer(to, amount) {
        try {
            const tx = await this.contract.transfer(to, amount);
            await tx.wait();
            return tx.hash;
        } catch (error) {
            console.error('转账失败:', error);
            throw error;
        }
    }
}
```

## 4. 社交媒体应用

展示如何创建一个去中心化的社交媒体应用。

### 项目结构
```
social-media/
├── src/
│   ├── components/
│   │   ├── PostCreator.js
│   │   ├── Feed.js
│   │   └── Profile.js
│   ├── services/
│   │   ├── post.js
│   │   ├── profile.js
│   │   └── storage.js
│   └── pages/
│       ├── Home.js
│       ├── Profile.js
│       └── Explore.js
```

### 核心代码示例

#### 发帖组件
```javascript
// components/PostCreator.js
import { WorldStorage, WorldUI } from '@world/minikit';

export function PostCreator() {
    const [content, setContent] = useState('');
    const storage = new WorldStorage();

    async function createPost() {
        try {
            const post = {
                id: Date.now(),
                content,
                author: await WorldAuth.getCurrentUser(),
                timestamp: new Date().toISOString()
            };

            await storage.set(`post_${post.id}`, post);
            WorldUI.Toast.success('发布成功！');
            setContent('');
        } catch (error) {
            WorldUI.Toast.error('发布失败：' + error.message);
        }
    }

    return (
        <div>
            <WorldUI.TextArea
                value={content}
                onChange={e => setContent(e.target.value)}
                placeholder="分享你的想法..."
            />
            <WorldUI.Button onClick={createPost}>
                发布
            </WorldUI.Button>
        </div>
    );
}
```

## 运行示例项目

### 1. 克隆示例仓库
```bash
git clone https://github.com/world/miniapp-examples
cd miniapp-examples
```

### 2. 安装依赖
```bash
npm install
```

### 3. 配置环境变量
```bash
cp .env.example .env
# 编辑.env文件，填入必要的配置信息
```

### 4. 启动开发服务器
```bash
npm run dev
```

## 部署示例项目

### 1. 构建项目
```bash
npm run build
```

### 2. 部署到World App Store
```bash
world-cli deploy
```

## 最佳实践总结

1. 代码组织
   - 使用清晰的项目结构
   - 模块化设计
   - 组件复用

2. 性能优化
   - 实施懒加载
   - 优化资源加载
   - 使用缓存策略

3. 安全考虑
   - 实施身份验证
   - 数据加密
   - 输入验证

4. 用户体验
   - 响应式设计
   - 错误处理
   - 加载状态

## 下一步
- 查看[故障排除](./troubleshooting.md)
- 了解[性能优化](./performance-optimization.md)
- 探索[高级功能](./advanced-features.md)
