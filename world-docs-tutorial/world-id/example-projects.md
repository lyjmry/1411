# World ID 示例项目

## 1. 用户认证系统

这个示例展示如何创建一个完整的用户认证系统，包括注册、登录和身份验证。

### 项目结构
```
auth-system/
├── src/
│   ├── components/
│   │   ├── Login.js
│   │   ├── Register.js
│   │   └── Profile.js
│   ├── services/
│   │   ├── auth.js
│   │   └── api.js
│   └── utils/
│       ├── worldId.js
│       └── storage.js
```

### 核心代码

#### 认证服务
```javascript
// services/auth.js
import { WorldID } from '@worldcoin/idkit';

export class AuthService {
    constructor() {
        this.worldID = new WorldID({
            appId: process.env.WORLD_APP_ID,
            actionId: 'user_auth',
            debug: process.env.NODE_ENV !== 'production'
        });
    }

    async register(userData) {
        try {
            // 验证World ID
            const verification = await this.worldID.verify({
                signal: userData.email
            });

            // 创建用户账户
            const user = await this.createUser({
                ...userData,
                worldIdProof: verification
            });

            return user;
        } catch (error) {
            console.error('注册失败:', error);
            throw error;
        }
    }

    async login(credentials) {
        try {
            // 验证凭证
            const verification = await this.worldID.verify({
                signal: credentials.email
            });

            // 执行登录
            const session = await this.createSession({
                credentials,
                verification
            });

            return session;
        } catch (error) {
            console.error('登录失败:', error);
            throw error;
        }
    }
}
```

#### 登录组件
```javascript
// components/Login.js
import { WorldIDWidget } from '@worldcoin/idkit';
import { useState } from 'react';

export function Login() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleLogin = async (worldIDProof) => {
        try {
            const auth = new AuthService();
            const session = await auth.login({
                email,
                password,
                worldIDProof
            });

            // 处理登录成功
            console.log('登录成功:', session);
        } catch (error) {
            console.error('登录失败:', error);
        }
    };

    return (
        <div className="login-container">
            <h2>登录</h2>
            <form>
                <input
                    type="email"
                    value={email}
                    onChange={(e) => setEmail(e.target.value)}
                    placeholder="邮箱"
                />
                <input
                    type="password"
                    value={password}
                    onChange={(e) => setPassword(e.target.value)}
                    placeholder="密码"
                />
                <WorldIDWidget
                    actionId="login"
                    signal={email}
                    onSuccess={handleLogin}
                    onError={(error) => console.error('World ID验证失败:', error)}
                />
            </form>
        </div>
    );
}
```

## 2. 投票系统

展示如何创建一个使用World ID进行身份验证的投票系统。

### 项目结构
```
voting-system/
├── contracts/
│   └── Voting.sol
├── src/
│   ├── components/
│   │   ├── CreatePoll.js
│   │   ├── VoteForm.js
│   │   └── Results.js
│   └── services/
│       ├── voting.js
│       └── verification.js
```

### 核心代码

#### 智能合约
```solidity
// contracts/Voting.sol
pragma solidity ^0.8.0;

contract VotingSystem {
    struct Poll {
        string question;
        string[] options;
        mapping(bytes32 => bool) hasVoted;
        mapping(uint => uint) votes;
    }

    mapping(uint => Poll) public polls;
    uint public pollCount;

    event VoteCast(uint pollId, uint option);
    event PollCreated(uint pollId, string question);

    function createPoll(string memory question, string[] memory options) public {
        uint pollId = pollCount++;
        Poll storage poll = polls[pollId];
        poll.question = question;
        poll.options = options;

        emit PollCreated(pollId, question);
    }

    function vote(
        uint pollId,
        uint option,
        bytes32 nullifierHash,
        bytes calldata proof
    ) public {
        require(verifyProof(proof), "Invalid proof");
        require(!polls[pollId].hasVoted[nullifierHash], "Already voted");

        polls[pollId].hasVoted[nullifierHash] = true;
        polls[pollId].votes[option]++;

        emit VoteCast(pollId, option);
    }

    function verifyProof(bytes calldata proof) internal returns (bool) {
        // 实现World ID验证逻辑
        return true;
    }
}
```

#### 投票服务
```javascript
// services/voting.js
import { WorldID } from '@worldcoin/idkit';
import { ethers } from 'ethers';

export class VotingService {
    constructor() {
        this.worldID = new WorldID({
            appId: process.env.WORLD_APP_ID,
            actionId: 'cast_vote'
        });
        this.contract = new ethers.Contract(
            VOTING_CONTRACT_ADDRESS,
            VOTING_ABI,
            provider
        );
    }

    async castVote(pollId, option) {
        try {
            // 获取World ID验证
            const verification = await this.worldID.verify({
                signal: `${pollId}-${option}`
            });

            // 提交投票到区块链
            const tx = await this.contract.vote(
                pollId,
                option,
                verification.nullifier_hash,
                verification.proof
            );

            await tx.wait();
            return tx.hash;
        } catch (error) {
            console.error('投票失败:', error);
            throw error;
        }
    }

    async getPollResults(pollId) {
        const poll = await this.contract.polls(pollId);
        const results = await Promise.all(
            poll.options.map(async (_, index) => {
                const votes = await this.contract.getVotes(pollId, index);
                return {
                    option: poll.options[index],
                    votes: votes.toNumber()
                };
            })
        );
        return results;
    }
}
```

#### 投票组件
```javascript
// components/VoteForm.js
import { WorldIDWidget } from '@worldcoin/idkit';
import { useState } from 'react';

export function VoteForm({ pollId, options }) {
    const [selectedOption, setSelectedOption] = useState(null);
    const votingService = new VotingService();

    const handleVote = async (worldIDProof) => {
        try {
            const txHash = await votingService.castVote(
                pollId,
                selectedOption,
                worldIDProof
            );
            console.log('投票成功! 交易哈希:', txHash);
        } catch (error) {
            console.error('投票失败:', error);
        }
    };

    return (
        <div className="vote-form">
            <h3>请选择您的选项</h3>
            {options.map((option, index) => (
                <div key={index} className="option">
                    <input
                        type="radio"
                        name="vote"
                        value={index}
                        onChange={() => setSelectedOption(index)}
                    />
                    <label>{option}</label>
                </div>
            ))}
            <WorldIDWidget
                actionId={`vote_${pollId}`}
                signal={`${pollId}_${selectedOption}`}
                onSuccess={handleVote}
                onError={(error) => console.error('验证失败:', error)}
            />
        </div>
    );
}
```

## 3. 社交媒体验证

展示如何在社交媒体平台中集成World ID验证。

### 项目结构
```
social-verification/
├── src/
│   ├── components/
│   │   ├── Profile.js
│   │   ├── PostForm.js
│   │   └── Comments.js
│   ├── services/
│   │   ├── social.js
│   │   └── verification.js
│   └── utils/
│       ├── worldId.js
│       └── storage.js
```

### 核心代码

#### 社交服务
```javascript
// services/social.js
export class SocialService {
    constructor() {
        this.worldID = new WorldID({
            appId: process.env.WORLD_APP_ID,
            actionId: 'social_verification'
        });
    }

    async verifyProfile(userId) {
        try {
            const verification = await this.worldID.verify({
                signal: userId
            });

            await this.updateProfile(userId, {
                verified: true,
                proof: verification
            });

            return true;
        } catch (error) {
            console.error('验证失败:', error);
            throw error;
        }
    }

    async createPost(content, userId) {
        // 验证用户身份
        const isVerified = await this.checkVerification(userId);
        if (!isVerified) {
            throw new Error('需要验证身份才能发帖');
        }

        // 创建帖子
        const post = await this.savePost({
            content,
            userId,
            timestamp: Date.now()
        });

        return post;
    }
}
```

#### 个人资料组件
```javascript
// components/Profile.js
export function Profile({ userId }) {
    const [isVerified, setIsVerified] = useState(false);
    const socialService = new SocialService();

    const handleVerification = async (worldIDProof) => {
        try {
            await socialService.verifyProfile(userId, worldIDProof);
            setIsVerified(true);
        } catch (error) {
            console.error('验证失败:', error);
        }
    };

    return (
        <div className="profile">
            <h2>个人资料</h2>
            <div className="verification-status">
                {isVerified ? (
                    <span className="verified">已验证</span>
                ) : (
                    <WorldIDWidget
                        actionId="verify_profile"
                        signal={userId}
                        onSuccess={handleVerification}
                        onError={(error) => console.error('验证失败:', error)}
                    />
                )}
            </div>
        </div>
    );
}
```

## 运行示例项目

### 1. 安装依赖
```bash
npm install
```

### 2. 配置环境变量
```bash
# .env
WORLD_APP_ID=your_app_id
WORLD_ACTION_ID=your_action_id
BLOCKCHAIN_RPC_URL=your_rpc_url
```

### 3. 启动项目
```bash
npm run dev
```

## 最佳实践

1. 安全考虑
   - 验证所有用户输入
   - 实施重放攻击保护
   - 安全存储验证数据

2. 用户体验
   - 提供清晰的验证流程
   - 实现优雅的错误处理
   - 添加加载状态提示

3. 性能优化
   - 实施缓存策略
   - 优化验证流程
   - 减少网络请求

4. 代码质量
   - 遵循代码规范
   - 添加适当的注释
   - 编写单元测试

## 下一步
- 查看[API参考](./api-reference.md)
- 了解[故障排除](./troubleshooting.md)
- 探索[高级功能](./advanced-features.md)
