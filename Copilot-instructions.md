# Copilot Instructions for Blockchain Development

This document provides comprehensive guidelines for GitHub Copilot to assist with blockchain development using JavaScript, Solidity, and Google Cloud backend infrastructure.

## Project Context

This is a blockchain development project that uses:
- **Frontend/Backend**: JavaScript (Node.js)
- **Smart Contracts**: Solidity
- **Cloud Infrastructure**: Google Cloud Platform (GCP)
- **Architecture**: Full-stack blockchain application with cloud backend

## Technology Stack Overview

### Core Technologies
- **Node.js**: Backend API server and blockchain interaction
- **Solidity**: Smart contract development for Ethereum-compatible blockchains
- **Google Cloud**: Infrastructure, database, and API hosting
- **Web3.js/Ethers.js**: Blockchain interaction libraries
- **Express.js**: Web framework for API endpoints

### Development Tools
- **Hardhat**: Ethereum development environment
- **Truffle**: Alternative development framework
- **Ganache**: Local blockchain for testing
- **MetaMask**: Wallet integration
- **Google Cloud SDK**: Cloud deployment and management

## JavaScript Development Guidelines

### Project Structure
When creating JavaScript/Node.js files, follow this structure:
```
src/
├── contracts/          # Smart contract ABIs and addresses
├── controllers/        # API route controllers
├── middleware/         # Express middleware
├── models/            # Database models
├── routes/            # API route definitions
├── services/          # Business logic and blockchain services
├── utils/             # Utility functions
├── config/            # Configuration files
└── tests/             # Test files
```

### Coding Standards
- Use ES6+ features (async/await, arrow functions, destructuring)
- Implement proper error handling with try-catch blocks
- Use environment variables for sensitive data
- Follow RESTful API design principles
- Implement proper logging for debugging

### Example JavaScript Patterns

#### Blockchain Service Class
```javascript
// When creating blockchain interaction services, use this pattern:
class BlockchainService {
    constructor(providerUrl, contractAddress, abi) {
        this.web3 = new Web3(providerUrl);
        this.contract = new this.web3.eth.Contract(abi, contractAddress);
    }

    async callContractMethod(methodName, params, fromAddress) {
        try {
            // Implementation here
        } catch (error) {
            console.error(`Error calling ${methodName}:`, error);
            throw error;
        }
    }
}
```

#### API Controller Pattern
```javascript
// Use this pattern for API controllers:
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

const createTransaction = asyncHandler(async (req, res) => {
    const { amount, recipient } = req.body;
    // Validation and business logic
    res.status(201).json({ success: true, data: result });
});
```

## Solidity Development Guidelines

### Smart Contract Structure
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;

// Import statements
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract ExampleContract is ReentrancyGuard, Ownable {
    // State variables
    mapping(address => uint256) public balances;
    
    // Events
    event Transfer(address indexed from, address indexed to, uint256 amount);
    
    // Modifiers
    modifier validAddress(address _addr) {
        require(_addr != address(0), "Invalid address");
        _;
    }
    
    // Constructor
    constructor() {}
    
    // External functions
    // Public functions
    // Internal functions
    // Private functions
}
```

### Security Best Practices
- Always use the latest Solidity version
- Implement reentrancy guards for state-changing functions
- Use SafeMath for arithmetic operations (if < 0.8.0)
- Validate all inputs and addresses
- Implement proper access control
- Use events for important state changes
- Follow CEI pattern (Checks-Effects-Interactions)

### Common Patterns

#### Payment Processing
```solidity
function processPayment(address recipient, uint256 amount) 
    external 
    payable 
    nonReentrant 
    validAddress(recipient) 
{
    require(msg.value >= amount, "Insufficient payment");
    require(balances[msg.sender] >= amount, "Insufficient balance");
    
    balances[msg.sender] -= amount;
    balances[recipient] += amount;
    
    emit Transfer(msg.sender, recipient, amount);
    
    // External call last
    (bool success, ) = recipient.call{value: amount}("");
    require(success, "Transfer failed");
}
```

## Google Cloud Backend Setup

### Infrastructure Components
- **Compute Engine**: VM instances for Node.js applications
- **Cloud Functions**: Serverless functions for specific tasks
- **Cloud SQL**: PostgreSQL database for application data
- **Cloud Storage**: File storage for application assets
- **Cloud Pub/Sub**: Message queuing for blockchain events
- **Cloud Monitoring**: Application monitoring and logging

### Project Configuration

#### package.json Dependencies
```json
{
  "dependencies": {
    "express": "^4.18.0",
    "web3": "^4.0.0",
    "ethers": "^6.0.0",
    "@google-cloud/storage": "^6.0.0",
    "@google-cloud/pubsub": "^3.0.0",
    "@google-cloud/logging": "^10.0.0",
    "pg": "^8.8.0",
    "dotenv": "^16.0.0",
    "cors": "^2.8.5",
    "helmet": "^6.0.0"
  },
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "deploy": "gcloud app deploy",
    "test": "jest"
  }
}
```

#### Environment Configuration
```javascript
// config/config.js
module.exports = {
    port: process.env.PORT || 3000,
    nodeEnv: process.env.NODE_ENV || 'development',
    
    // Blockchain Configuration
    blockchain: {
        providerUrl: process.env.BLOCKCHAIN_PROVIDER_URL,
        networkId: process.env.NETWORK_ID || '1',
        contractAddress: process.env.CONTRACT_ADDRESS,
        privateKey: process.env.PRIVATE_KEY
    },
    
    // Google Cloud Configuration
    gcp: {
        projectId: process.env.GCP_PROJECT_ID,
        keyFilename: process.env.GCP_KEY_FILE,
        storageBucket: process.env.GCP_STORAGE_BUCKET
    },
    
    // Database Configuration
    database: {
        host: process.env.DB_HOST,
        port: process.env.DB_PORT || 5432,
        name: process.env.DB_NAME,
        username: process.env.DB_USERNAME,
        password: process.env.DB_PASSWORD
    }
};
```

### Google Cloud Services Integration

#### Cloud Storage Service
```javascript
// services/storageService.js
const { Storage } = require('@google-cloud/storage');

class StorageService {
    constructor() {
        this.storage = new Storage({
            projectId: config.gcp.projectId,
            keyFilename: config.gcp.keyFilename
        });
        this.bucket = this.storage.bucket(config.gcp.storageBucket);
    }

    async uploadFile(fileName, fileBuffer, metadata = {}) {
        const file = this.bucket.file(fileName);
        const stream = file.createWriteStream({
            metadata: {
                contentType: metadata.contentType || 'application/octet-stream'
            }
        });

        return new Promise((resolve, reject) => {
            stream.on('error', reject);
            stream.on('finish', () => resolve(file.publicUrl()));
            stream.end(fileBuffer);
        });
    }
}
```

#### Pub/Sub Event Handler
```javascript
// services/pubsubService.js
const { PubSub } = require('@google-cloud/pubsub');

class PubSubService {
    constructor() {
        this.pubsub = new PubSub({
            projectId: config.gcp.projectId
        });
    }

    async publishBlockchainEvent(topicName, eventData) {
        const topic = this.pubsub.topic(topicName);
        const message = Buffer.from(JSON.stringify(eventData));
        
        try {
            const messageId = await topic.publish(message);
            console.log(`Message ${messageId} published to ${topicName}`);
            return messageId;
        } catch (error) {
            console.error('Error publishing message:', error);
            throw error;
        }
    }
}
```

### Deployment Configuration

#### app.yaml for App Engine
```yaml
runtime: nodejs18

env_variables:
  NODE_ENV: production
  BLOCKCHAIN_PROVIDER_URL: "your-provider-url"
  CONTRACT_ADDRESS: "your-contract-address"
  GCP_PROJECT_ID: "your-project-id"

automatic_scaling:
  min_instances: 1
  max_instances: 10
  target_cpu_utilization: 0.6

resources:
  cpu: 1
  memory_gb: 2
  disk_size_gb: 10
```

#### Cloud Build Configuration
```yaml
# cloudbuild.yaml
steps:
  - name: 'node:18'
    entrypoint: 'npm'
    args: ['install']
  
  - name: 'node:18'
    entrypoint: 'npm'
    args: ['test']
  
  - name: 'gcr.io/cloud-builders/gcloud'
    args: ['app', 'deploy']

timeout: '1200s'
```

## Development Workflow

### Local Development Setup
1. Initialize Node.js project: `npm init -y`
2. Install dependencies: `npm install express web3 ethers dotenv`
3. Set up Hardhat: `npx hardhat init`
4. Configure environment variables in `.env`
5. Start local blockchain: `npx hardhat node`
6. Deploy contracts: `npx hardhat run scripts/deploy.js`

### Testing Strategy
- Unit tests for smart contracts using Hardhat
- Integration tests for API endpoints
- End-to-end tests for blockchain interactions
- Use Jest for JavaScript testing
- Use Waffle/Chai for Solidity testing

### Deployment Process
1. Test locally with Ganache
2. Deploy to testnet (Goerli/Sepolia)
3. Deploy backend to Google Cloud App Engine
4. Configure monitoring and logging
5. Deploy to mainnet when ready

## Common Code Patterns

### Error Handling
```javascript
// Consistent error handling pattern
class APIError extends Error {
    constructor(message, statusCode) {
        super(message);
        this.statusCode = statusCode;
        this.isOperational = true;
    }
}

const errorHandler = (error, req, res, next) => {
    if (error.isOperational) {
        res.status(error.statusCode).json({
            success: false,
            message: error.message
        });
    } else {
        console.error(error);
        res.status(500).json({
            success: false,
            message: 'Internal server error'
        });
    }
};
```

### Transaction Monitoring
```javascript
// Pattern for monitoring blockchain transactions
class TransactionMonitor {
    constructor(web3, contract) {
        this.web3 = web3;
        this.contract = contract;
    }

    async waitForTransaction(txHash, confirmations = 1) {
        let receipt = null;
        while (receipt === null) {
            receipt = await this.web3.eth.getTransactionReceipt(txHash);
            if (receipt === null) {
                await new Promise(resolve => setTimeout(resolve, 5000));
            }
        }

        if (receipt.blockNumber) {
            const currentBlock = await this.web3.eth.getBlockNumber();
            const confirmationCount = currentBlock - receipt.blockNumber;
            
            if (confirmationCount >= confirmations) {
                return receipt;
            }
        }

        return this.waitForTransaction(txHash, confirmations);
    }
}
```

## Security Considerations

### Environment Variables
Never commit sensitive data. Use environment variables for:
- Private keys
- API keys
- Database credentials
- Contract addresses
- Provider URLs

### Input Validation
```javascript
// Always validate inputs
const validateTransactionInput = (req, res, next) => {
    const { amount, recipient } = req.body;
    
    if (!amount || isNaN(amount) || amount <= 0) {
        return res.status(400).json({ error: 'Invalid amount' });
    }
    
    if (!recipient || !web3.utils.isAddress(recipient)) {
        return res.status(400).json({ error: 'Invalid recipient address' });
    }
    
    next();
};
```

### Rate Limiting
```javascript
// Implement rate limiting for API endpoints
const rateLimit = require('express-rate-limit');

const apiLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP'
});
```

## AI Assistant Context Guidelines

When working with this codebase:

1. **Always consider gas optimization** when writing Solidity code
2. **Implement proper error handling** in all JavaScript functions
3. **Use environment variables** for configuration
4. **Follow async/await patterns** consistently
5. **Implement proper logging** for debugging
6. **Consider scalability** when designing Google Cloud architecture
7. **Validate all inputs** before processing
8. **Use established patterns** from this document
9. **Test thoroughly** before deployment
10. **Document complex functions** with clear comments

## Quick Reference Commands

### Hardhat Commands
```bash
npx hardhat compile              # Compile contracts
npx hardhat test                 # Run tests
npx hardhat node                 # Start local node
npx hardhat run scripts/deploy.js --network localhost
```

### Google Cloud Commands
```bash
gcloud auth login                # Authenticate
gcloud config set project PROJECT_ID
gcloud app deploy               # Deploy to App Engine
gcloud functions deploy FUNCTION_NAME
gcloud sql instances create INSTANCE_NAME
```

### Development Commands
```bash
npm run dev                     # Start development server
npm test                        # Run tests
npm run build                   # Build for production
npm run deploy                  # Deploy to cloud
```

This comprehensive guide should provide GitHub Copilot with the necessary context to assist effectively with blockchain development using JavaScript, Solidity, and Google Cloud Platform.