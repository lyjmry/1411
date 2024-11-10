# World Chain 快速开始指南

## 环境准备

### 1. 系统要求
- Node.js 14.0.0 或更高版本
- npm 6.0.0 或更高版本
- Git
- 支持的操作系统：
  - Windows 10/11
  - macOS 10.15+
  - Ubuntu 20.04+

### 2. 安装World Chain CLI
```bash
# 使用npm安装
npm install -g @world-chain/cli

# 或使用yarn
yarn global add @world-chain/cli

# 验证安装
world-chain --version
```

## 创建第一个项目

### 1. 初始化项目
```bash
# 创建新项目
world-chain init my-first-project
cd my-first-project

# 安装依赖
npm install
```

### 2. 项目结构
```
my-first-project/
├── contracts/           # 智能合约目录
│   └── HelloWorld.sol
├── scripts/            # 部署脚本
│   └── deploy.js
├── test/              # 测试文件
│   └── HelloWorld.test.js
├── world.config.js    # 配置文件
└── package.json
```

### 3. 配置文件
```javascript
// world.config.js
module.exports = {
    // 网络配置
    networks: {
        development: {
            url: 'http://localhost:8545',
            chainId: 1337
        },
        testnet: {
            url: 'https://testnet-rpc.worldchain.org',
            chainId: 2,
            accounts: {
                mnemonic: process.env.MNEMONIC
            }
        }
    },
    
    // 编译器配置
    solidity: {
        version: '0.8.17',
        settings: {
            optimizer: {
                enabled: true,
                runs: 200
            }
        }
    }
};
```

## 开发第一个智能合约

### 1. 创建合约
```solidity
// contracts/HelloWorld.sol
pragma solidity ^0.8.17;

contract HelloWorld {
    // 状态变量
    string public message;
    address public owner;
    
    // 事件
    event MessageUpdated(string newMessage);
    
    // 构造函数
    constructor(string memory initialMessage) {
        message = initialMessage;
        owner = msg.sender;
    }
    
    // 更新消息
    function updateMessage(string memory newMessage) public {
        require(msg.sender == owner, "Only owner can update");
        message = newMessage;
        emit MessageUpdated(newMessage);
    }
    
    // 获取消息
    function getMessage() public view returns (string memory) {
        return message;
    }
}
```

### 2. 编译合约
```bash
# 编译合约
world-chain compile

# 检查编译输出
ls build/contracts
```

### 3. 测试合约
```javascript
// test/HelloWorld.test.js
const { expect } = require('chai');

describe('HelloWorld', function() {
    let helloWorld;
    let owner;
    let addr1;

    beforeEach(async function() {
        [owner, addr1] = await ethers.getSigners();
        
        const HelloWorld = await ethers.getContractFactory('HelloWorld');
        helloWorld = await HelloWorld.deploy('Hello, World Chain!');
        await helloWorld.deployed();
    });

    it('Should return the correct message', async function() {
        expect(await helloWorld.getMessage()).to.equal('Hello, World Chain!');
    });

    it('Should allow owner to update message', async function() {
        await helloWorld.updateMessage('New message');
        expect(await helloWorld.getMessage()).to.equal('New message');
    });

    it('Should not allow non-owner to update message', async function() {
        await expect(
            helloWorld.connect(addr1).updateMessage('Unauthorized')
        ).to.be.revertedWith('Only owner can update');
    });
});
```

### 4. 运行测试
```bash
# 运行测试
world-chain test
```

## 部署合约

### 1. 配置部署脚本
```javascript
// scripts/deploy.js
async function main() {
    // 获取合约工厂
    const HelloWorld = await ethers.getContractFactory('HelloWorld');
    
    // 部署合约
    console.log('Deploying HelloWorld...');
    const helloWorld = await HelloWorld.deploy('Hello, World Chain!');
    await helloWorld.deployed();
    
    console.log('HelloWorld deployed to:', helloWorld.address);
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

### 2. 部署到测试网
```bash
# 设置环境变量
export MNEMONIC="your twelve word mnemonic phrase here"

# 部署到测试网
world-chain deploy --network testnet
```

### 3. 验证合约
```bash
# 验证合约代码
world-chain verify --network testnet --contract HelloWorld
```

## 与合约交互

### 1. 使用CLI交互
```bash
# 读取消息
world-chain call --network testnet --contract HelloWorld --method getMessage

# 更新消息
world-chain send --network testnet --contract HelloWorld --method updateMessage --args "New message"
```

### 2. 使用SDK交互
```javascript
// 初始化SDK
const WorldChain = require('@world-chain/sdk');
const worldChain = new WorldChain({
    network: 'testnet',
    provider: window.ethereum
});

// 连接合约
const contract = await worldChain.getContract('HelloWorld', contractAddress);

// 读取消息
const message = await contract.getMessage();
console.log('Current message:', message);

// 更新消息
const tx = await contract.updateMessage('Hello from SDK!');
await tx.wait();
console.log('Message updated!');
```

### 3. 使用Web3交互
```javascript
// 初始化Web3
const Web3 = require('web3');
const web3 = new Web3('https://testnet-rpc.worldchain.org');

// 加载合约ABI
const HelloWorld = require('./build/contracts/HelloWorld.json');
const contract = new web3.eth.Contract(
    HelloWorld.abi,
    contractAddress
);

// 读取消息
const message = await contract.methods.getMessage().call();
console.log('Current message:', message);

// 更新消息
const accounts = await web3.eth.getAccounts();
await contract.methods.updateMessage('Hello from Web3!')
    .send({ from: accounts[0] });
```

## 集成World ID

### 1. 安装依赖
```bash
npm install @worldcoin/id
```

### 2. 更新合约
```solidity
// contracts/VerifiedHello.sol
pragma solidity ^0.8.17;

import "@worldcoin/id/contracts/interfaces/IWorldID.sol";

contract VerifiedHello {
    IWorldID public worldId;
    string public message;
    
    constructor(address _worldId) {
        worldId = IWorldID(_worldId);
    }
    
    function updateMessage(
        string memory newMessage,
        bytes memory proof
    ) public {
        // 验证World ID证明
        require(worldId.verify(proof), "Invalid World ID proof");
        
        message = newMessage;
    }
}
```

### 3. 前端集成
```javascript
// 初始化World ID
const worldID = new WorldID({
    appId: 'your-app-id',
    actionId: 'update_message'
});

// 更新消息
async function updateMessage(newMessage) {
    try {
        // 获取World ID证明
        const proof = await worldID.verify();
        
        // 调用合约
        const tx = await contract.updateMessage(newMessage, proof);
        await tx.wait();
        
        console.log('Message updated successfully!');
    } catch (error) {
        console.error('Failed to update message:', error);
    }
}
```

## 下一步

1. 探索更多功能
   - 查看[API参考](./api-reference.md)
   - 了解[高级特性](./advanced-features.md)
   - 研究[示例项目](./example-projects.md)

2. 开发建议
   - 遵循[最佳实践](./best-practices.md)
   - 注意[安全考虑](./security.md)
   - 优化[性能](./performance.md)

3. 获取帮助
   - 加入[开发者社区](https://discord.gg/worldchain)
   - 查看[常见问题](./faq.md)
   - 提交[问题反馈](https://github.com/worldcoin/worldchain/issues)
