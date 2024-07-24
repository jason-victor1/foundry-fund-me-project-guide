# Foundry Fund Me Project Guide

This README provides comprehensive instructions on setting up, organizing, refactoring, scripting, testing, and managing a Foundry project using VS Code.

### Table of Contents
- [Foundry Fund Me Project Guide](#foundry-fund-me-project-guide)
  - [1. Setting Up a Foundry Project](#1-setting-up-a-foundry-project)
    - [Installing Foundry](#installing-foundry)
    - [Creating a New Project](#creating-a-new-project)
  - [2. Organizing Files into Folders](#2-organizing-files-into-folders)
    - [Folder Structure](#folder-structure)
    - [Example Structure](#example-structure)
  - [3. Refactoring Smart Contracts to be Modular and Chain-Agnostic](#3-refactoring-smart-contracts-to-be-modular-and-chain-agnostic)
    - [Modularization](#modularization)
    - [Chain-Agnostic Design](#chain-agnostic-design)
  - [4. Writing Scripts for Interaction Commands](#4-writing-scripts-for-interaction-commands)
    - [Create Scripts](#create-scripts)
    - [Run Scripts](#run-scripts)
  - [5. Using Mocks](#5-using-mocks)
    - [Create a Mock Contract](#create-a-mock-contract)
    - [Deploy and Use Mocks in Tests](#deploy-and-use-mocks-in-tests)
  - [6. Writing Unit and Integration Tests](#6-writing-unit-and-integration-tests)
    - [Unit Tests](#unit-tests)
    - [Integration Tests](#integration-tests)
  - [7. Setting Up Makefiles](#7-setting-up-makefiles)
    - [Create a Makefile](#create-a-makefile)
    - [Run Makefile Commands](#run-makefile-commands)
  - [8. Creating a GitHub Repository and Pushing Code](#8-creating-a-github-repository-and-pushing-code)
    - [Initialize Git](#initialize-git)
    - [Create a Repository on GitHub](#create-a-repository-on-github)
  - [9. Cloning a GitHub Repository](#9-cloning-a-github-repository)
    - [Clone the Repository](#clone-the-repository)
  - [Summary](#summary)

## 1. Setting Up a Foundry Project

### Installing Foundry

1. **Open Terminal in VS Code:**
   - Go to `View` > `Terminal` or use the shortcut (`Ctrl + ` on Windows/Linux, `Cmd + ` on macOS).

2. **Install Foundry:**
   ```bash
   curl -L https://foundry.paradigm.xyz | bash
   foundryup
   ```

3. **Verify Installation:**
   ```bash
   forge --version
   ```

### Creating a New Project

1. **Initialize a Foundry Project:**
   ```bash
   forge init MyNewProject
   cd MyNewProject
   ```

## 2. Organizing Files into Folders

### Folder Structure

- `src/`: Solidity source files.
- `script/`: Deployment and interaction scripts.
- `test/`: Test files.
- `lib/`: External libraries and dependencies.
- `cache/`, `out/`: Generated files during build and test processes.

### Example Structure

```plaintext
MyNewProject/
├── src/
│   └── FundMe.sol
├── script/
│   └── DeployFundMe.s.sol
├── test/
│   └── FundMeTest.sol
├── lib/
├── cache/
├── out/
├── .env
├── .gitignore
├── foundry.toml
└── README.md
```

## 3. Refactoring Smart Contracts to be Modular and Chain-Agnostic

### Modularization

Break down contracts into smaller, reusable components. Use inheritance and interfaces to create a modular architecture.

```solidity
// src/PriceFeed.sol
pragma solidity ^0.8.0;

interface PriceFeed {
    function getPrice() external view returns (uint256);
}
```

### Chain-Agnostic Design

Use configuration variables and dependency injection.

```solidity
// src/FundMe.sol
pragma solidity ^0.8.0;

import "./PriceFeed.sol";

contract FundMe {
    address public priceFeed;

    constructor(address _priceFeed) {
        priceFeed = _priceFeed;
    }
}
```

## 4. Writing Scripts for Interaction Commands

### Create Scripts

Place deployment and interaction scripts in the `script/` directory.

```solidity
// script/DeployFundMe.s.sol
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "../src/FundMe.sol";

contract DeployFundMe is Script {
    function run() external {
        vm.startBroadcast();
        new FundMe(0xYourPriceFeedAddress);
        vm.stopBroadcast();
    }
}
```

### Run Scripts

Use `forge` to run scripts.

```bash
forge script script/DeployFundMe.s.sol --rpc-url $RPC_URL --private-key $PRIVATE_KEY --broadcast
```

## 5. Using Mocks

### Create a Mock Contract

```solidity
// src/mock/MockV3Aggregator.sol
pragma solidity ^0.8.0;

contract MockV3Aggregator {
    int256 public answer;

    constructor(int256 _answer) {
        answer = _answer;
    }

    function latestRoundData() external view returns (uint80, int256, uint256, uint256, uint80) {
        return (0, answer, 0, 0, 0);
    }
}
```

### Deploy and Use Mocks in Tests

```solidity
// script/DeployMocks.s.sol
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "../src/mock/MockV3Aggregator.sol";

contract DeployMocks is Script {
    function run() external {
        vm.startBroadcast();
        new MockV3Aggregator(2000e8);
        vm.stopBroadcast();
    }
}
```

## 6. Writing Unit and Integration Tests

### Unit Tests

Test individual components of your contract.

```solidity
// test/FundMeTest.sol
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/FundMe.sol";

contract FundMeTest is Test {
    FundMe fundMe;

    function setUp() public {
        fundMe = new FundMe(address(0));
    }

    function testInitialBalance() public {
        assertEq(fundMe.balance, 0);
    }
}
```

### Integration Tests

Test how different parts of your system work together.

```solidity
// test/IntegrationTest.sol
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/FundMe.sol";
import "../src/mock/MockV3Aggregator.sol";

contract IntegrationTest is Test {
    FundMe fundMe;
    MockV3Aggregator mockPriceFeed;

    function setUp() public {
        mockPriceFeed = new MockV3Aggregator(2000e8);
        fundMe = new FundMe(address(mockPriceFeed));
    }

    function testFundAndWithdraw() public {
        fundMe.fund{value: 1 ether}();
        fundMe.withdraw();
        assertEq(fundMe.balance, 0);
    }
}
```

## 7. Setting Up Makefiles

### Create a Makefile

```makefile
# Makefile
install:
    foundryup

build:
    forge build

test:
    forge test

deploy:
    forge script script/DeployFundMe.s.sol --rpc-url $RPC_URL --private-key $PRIVATE_KEY --broadcast
```

### Run Makefile Commands

```bash
make install
make build
make test
make deploy
```

## 8. Creating a GitHub Repository and Pushing Code

### Initialize Git

```bash
git init
git add .
git commit -m "Initial commit"
```

### Create a Repository on GitHub

1. Go to GitHub and create a new repository.
2. Follow the instructions to push your code:

```bash
git remote add origin https://github.com/yourusername/your-repo.git
git branch -M main
git push -u origin main
```

## 9. Cloning a GitHub Repository

### Clone the Repository

```bash
git clone https://github.com/yourusername/your-repo.git
cd your-repo
```

## Summary

This guide provides detailed steps for setting up a Foundry project using VS Code, organizing files, refactoring smart contracts, writing interaction scripts, using mocks, performing unit and integration tests, setting up Makefiles, managing your project with Git and GitHub, and cloning repositories. By following these steps, you can efficiently manage and develop your Ethereum projects. 

