# World Chain 示例项目

## 1. 去中心化身份验证系统

这个示例展示如何创建一个基于World Chain和World ID的身份验证系统。

### 项目结构
```
identity-system/
├── contracts/
│   ├── IdentityRegistry.sol
│   └── VerificationManager.sol
├── src/
│   ├── components/
│   │   ├── IdentityVerifier.js
│   │   └── UserProfile.js
│   └── services/
│       ├── identity.js
│       └── verification.js
```

### 核心代码

#### 身份注册合约
```solidity
// contracts/IdentityRegistry.sol
pragma solidity ^0.8.17;

import "@worldcoin/id/contracts/interfaces/IWorldID.sol";

contract IdentityRegistry {
    IWorldID public worldId;
    mapping(address => bool) public verifiedUsers;
    mapping(address => bytes32) public userProfiles;
    
    event UserVerified(address indexed user);
    event ProfileUpdated(address indexed user, bytes32 profileHash);
    
    constructor(address _worldId) {
        worldId = IWorldID(_worldId);
    }
    
    function verifyIdentity(bytes memory proof) public {
        require(!verifiedUsers[msg.sender], "Already verified");
        require(worldId.verify(proof), "Invalid proof");
        
        verifiedUsers[msg.sender] = true;
        emit UserVerified(msg.sender);
    }
    
    function updateProfile(bytes32 profileHash) public {
        require(verifiedUsers[msg.sender], "Not verified");
        userProfiles[msg.sender] = profileHash;
        emit ProfileUpdated(msg.sender, profileHash);
    }
}
```

#### 身份服务
```javascript
// services/identity.js
class IdentityService {
    constructor(provider) {
        this.worldChain = new WorldChain({
            network: 'mainnet',
            provider
        });
        this.contract = new Contract(
            REGISTRY_ADDRESS,
            REGISTRY_ABI,
            this.worldChain.getSigner()
        );
    }

    async verifyIdentity() {
        try {
            // 获取World ID证明
            const worldID = new WorldID({
                appId: 'identity_system',
                actionId: 'verify_identity'
            });
            const proof = await worldID.verify();
            
            // 提交验证
            const tx = await this.contract.verifyIdentity(proof);
            await tx.wait();
            
            return true;
        } catch (error) {
            console.error('身份验证失败:', error);
            throw error;
        }
    }

    async updateProfile(profile) {
        try {
            const profileHash = ethers.utils.keccak256(
                ethers.utils.toUtf8Bytes(JSON.stringify(profile))
            );
            
            const tx = await this.contract.updateProfile(profileHash);
            await tx.wait();
            
            return true;
        } catch (error) {
            console.error('更新档案失败:', error);
            throw error;
        }
    }
}
```

## 2. 去中心化投票系统

展示如何创建一个安全的投票系统，确保每个用户只能投票一次。

### 项目结构
```
voting-system/
├── contracts/
│   ├── VotingSystem.sol
│   └── ProposalManager.sol
├── src/
│   ├── components/
│   │   ├── ProposalList.js
│   │   ├── VoteForm.js
│   │   └── Results.js
│   └── services/
│       ├── voting.js
│       └── proposals.js
```

### 核心代码

#### 投票合约
```solidity
// contracts/VotingSystem.sol
pragma solidity ^0.8.17;

import "@worldcoin/id/contracts/interfaces/IWorldID.sol";

contract VotingSystem {
    struct Proposal {
        bytes32 id;
        string description;
        uint256 votesFor;
        uint256 votesAgainst;
        uint256 endTime;
        bool executed;
    }
    
    IWorldID public worldId;
    mapping(bytes32 => Proposal) public proposals;
    mapping(bytes32 => mapping(address => bool)) public hasVoted;
    
    event ProposalCreated(bytes32 indexed id, string description);
    event VoteCast(bytes32 indexed proposalId, address indexed voter, bool support);
    
    constructor(address _worldId) {
        worldId = IWorldID(_worldId);
    }
    
    function createProposal(
        string memory description,
        uint256 duration
    ) public returns (bytes32) {
        bytes32 proposalId = keccak256(
            abi.encodePacked(description, block.timestamp)
        );
        
        proposals[proposalId] = Proposal({
            id: proposalId,
            description: description,
            votesFor: 0,
            votesAgainst: 0,
            endTime: block.timestamp + duration,
            executed: false
        });
        
        emit ProposalCreated(proposalId, description);
        return proposalId;
    }
    
    function vote(
        bytes32 proposalId,
        bool support,
        bytes memory proof
    ) public {
        require(worldId.verify(proof), "Invalid proof");
        require(!hasVoted[proposalId][msg.sender], "Already voted");
        require(block.timestamp < proposals[proposalId].endTime, "Voting ended");
        
        if (support) {
            proposals[proposalId].votesFor++;
        } else {
            proposals[proposalId].votesAgainst++;
        }
        
        hasVoted[proposalId][msg.sender] = true;
        emit VoteCast(proposalId, msg.sender, support);
    }
}
```

#### 投票服务
```javascript
// services/voting.js
class VotingService {
    constructor(provider) {
        this.contract = new Contract(
            VOTING_ADDRESS,
            VOTING_ABI,
            provider.getSigner()
        );
        this.worldID = new WorldID({
            appId: 'voting_system',
            actionId: 'cast_vote'
        });
    }

    async createProposal(description, duration) {
        try {
            const tx = await this.contract.createProposal(
                description,
                duration
            );
            const receipt = await tx.wait();
            
            // 解析事件获取提案ID
            const event = receipt.events.find(
                e => e.event === 'ProposalCreated'
            );
            return event.args.id;
        } catch (error) {
            console.error('创建提案失败:', error);
            throw error;
        }
    }

    async castVote(proposalId, support) {
        try {
            // 获取World ID证明
            const proof = await this.worldID.verify({
                signal: ethers.utils.hexlify(proposalId)
            });
            
            // 提交投票
            const tx = await this.contract.vote(
                proposalId,
                support,
                proof
            );
            await tx.wait();
            
            return true;
        } catch (error) {
            console.error('投票失败:', error);
            throw error;
        }
    }
}
```

## 3. 去中心化社交网络

展示如何创建一个基于区块链的社交网络平台。

### 项目结构
```
social-network/
├── contracts/
│   ├── SocialNetwork.sol
│   └── ContentManager.sol
├── src/
│   ├── components/
│   │   ├── Profile.js
│   │   ├── PostForm.js
│   │   └── Feed.js
│   └── services/
│       ├── social.js
│       └── content.js
```

### 核心代码

#### 社交网络合约
```solidity
// contracts/SocialNetwork.sol
pragma solidity ^0.8.17;

import "@worldcoin/id/contracts/interfaces/IWorldID.sol";

contract SocialNetwork {
    struct Post {
        bytes32 id;
        address author;
        string content;
        uint256 timestamp;
        uint256 likes;
    }
    
    IWorldID public worldId;
    mapping(address => bool) public verifiedUsers;
    mapping(bytes32 => Post) public posts;
    mapping(address => bytes32[]) public userPosts;
    
    event PostCreated(bytes32 indexed id, address indexed author);
    event PostLiked(bytes32 indexed id, address indexed liker);
    
    constructor(address _worldId) {
        worldId = IWorldID(_worldId);
    }
    
    function verifyUser(bytes memory proof) public {
        require(!verifiedUsers[msg.sender], "Already verified");
        require(worldId.verify(proof), "Invalid proof");
        
        verifiedUsers[msg.sender] = true;
    }
    
    function createPost(string memory content) public {
        require(verifiedUsers[msg.sender], "Not verified");
        
        bytes32 postId = keccak256(
            abi.encodePacked(msg.sender, content, block.timestamp)
        );
        
        posts[postId] = Post({
            id: postId,
            author: msg.sender,
            content: content,
            timestamp: block.timestamp,
            likes: 0
        });
        
        userPosts[msg.sender].push(postId);
        emit PostCreated(postId, msg.sender);
    }
    
    function likePost(bytes32 postId) public {
        require(verifiedUsers[msg.sender], "Not verified");
        require(posts[postId].author != address(0), "Post not found");
        
        posts[postId].likes++;
        emit PostLiked(postId, msg.sender);
    }
}
```

#### 社交服务
```javascript
// services/social.js
class SocialService {
    constructor(provider) {
        this.contract = new Contract(
            SOCIAL_NETWORK_ADDRESS,
            SOCIAL_NETWORK_ABI,
            provider.getSigner()
        );
        this.worldID = new WorldID({
            appId: 'social_network',
            actionId: 'verify_user'
        });
    }

    async verifyUser() {
        try {
            const proof = await this.worldID.verify();
            const tx = await this.contract.verifyUser(proof);
            await tx.wait();
            return true;
        } catch (error) {
            console.error('用户验证失败:', error);
            throw error;
        }
    }

    async createPost(content) {
        try {
            const tx = await this.contract.createPost(content);
            const receipt = await tx.wait();
            
            const event = receipt.events.find(
                e => e.event === 'PostCreated'
            );
            return event.args.id;
        } catch (error) {
            console.error('创建帖子失败:', error);
            throw error;
        }
    }

    async likePost(postId) {
        try {
            const tx = await this.contract.likePost(postId);
            await tx.wait();
            return true;
        } catch (error) {
            console.error('点赞失败:', error);
            throw error;
        }
    }
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
WORLD_CHAIN_RPC_URL=your_rpc_url
WORLD_ID_APP_ID=your_app_id
PRIVATE_KEY=your_private_key
```

### 3. 部署合约
```bash
npx hardhat run scripts/deploy.js --network worldchain
```

## 最佳实践

1. 安全考虑
   - 验证所有用户输入
   - 实施访问控制
   - 保护敏感数据

2. 性能优化
   - 实施事件索引
   - 优化存储结构
   - 批量处理交易

3. 用户体验
   - 提供友好的错误信息
   - 实现平滑的交互流程
   - 优化加载时间

## 下一步
- 查看[API参考](./api-reference.md)
- 了解[性能优化](./performance-optimization.md)
- 探索[安全最佳实践](./security-best-practices.md)
