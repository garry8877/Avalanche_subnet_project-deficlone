# Avalanche Subnet ERC20 & Vault Contract Project

## Overview

This project consists of two Solidity contracts: an ERC20 token and a Vault contract. The contracts are designed to run on an Avalanche Subnet, enabling users to create and manage tokens within a customized blockchain environment. Using the ERC20 contract, tokens can be minted, transferred, and burned. The Vault contract allows users to deposit and withdraw these tokens, receiving Vault shares proportional to their stake.

### What is an Avalanche Subnet?

Avalanche Subnets are customizable blockchains operating within the Avalanche ecosystem. They allow developers to define specific consensus mechanisms, economic models, and rules for their application. By deploying smart contracts on a Subnet, you can leverage the high performance and scalability of Avalanche while tailoring the blockchain to your specific needs.

## Contracts

### 1. ERC20 Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ERC20 {
    uint public totalSupply;
    mapping(address => uint) public balanceOf;
    mapping(address => mapping(address => uint)) public allowance;
    string public name = "MRG Construction";
    string public symbol = "MRG";
    uint8 public decimals = 18;

	event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);

    function transfer(address receiver, uint amount) external returns (bool) {
        balanceOf[msg.sender] -= amount;
        balanceOf[receiver] += amount;
        emit Transfer(msg.sender, receiver, amount);
        return true;
    }

    function approve(address spender, uint amount) external returns (bool) {
        allowance[msg.sender][spender] = amount;
        emit Approval(msg.sender, spender, amount);
        return true;
    }

    function transferFrom(
        address sender,
        address receiver,
        uint amount
    ) external returns (bool) {
        allowance[sender][msg.sender] -= amount;
        balanceOf[sender] -= amount;
        balanceOf[receiver] += amount;
        emit Transfer(sender, receiver, amount);
        return true;
    }

    function mint(uint amount) external {
        balanceOf[msg.sender] += amount;
        totalSupply += amount;
        emit Transfer(address(0), msg.sender, amount);
    }

    function burn(uint amount) external {
        balanceOf[msg.sender] -= amount;
        totalSupply -= amount;
        emit Transfer(msg.sender, address(0), amount);
    }
}
```

This contract implements the ERC20 standard, providing basic token functionality such as transfers, approvals, minting, and burning. The contract runs on the Avalanche Subnet, ensuring fast, low-cost transactions.

#### Token Details:
- **Name**: `MRG Construction`
- **Symbol**: `MRG`
- **Decimals**: `18`

#### Key Functions:
- **transfer(address receiver, uint amount)**: Allows token transfers between users.
- **approve(address spender, uint amount)**: Approves a spender to transfer tokens on behalf of the user.
- **transferFrom(address sender, address receiver, uint amount)**: Facilitates transfers using approved allowances.
- **mint(uint amount)**: Mints new tokens, increasing the total supply.
- **burn(uint amount)**: Burns tokens, reducing the total supply.

### 2. Vault Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

interface IERC20 {
    function totalSupply() external view returns (uint);

    function balanceOf(address account) external view returns (uint);

    function transfer(address recipient, uint amount) external returns (bool);

    function allowance(address owner, address spender) external view returns (uint);

    function approve(address spender, uint amount) external returns (bool);

    function transferFrom(
        address sender,
        address recipient,
        uint amount
    ) external returns (bool);

    event Transfer(address indexed from, address indexed to, uint value);
    event Approval(address indexed owner, address indexed spender, uint value);
}

contract Vault {
    IERC20 public immutable token;

    uint public totalSupply;
    mapping(address => uint) public balanceOf;

    constructor(address _token) {
        token = IERC20(_token);
    }

    function _mint(address _to, uint _shares) private {
        totalSupply += _shares;
        balanceOf[_to] += _shares;
    }

    function _burn(address _from, uint _shares) private {
        totalSupply -= _shares;
        balanceOf[_from] -= _shares;
    }

    function deposit(uint _amount) external {
        /*
        a = amount
        B = balance of token before deposit
        T = total supply
        s = shares to mint

        (T + s) / T = (a + B) / B 

        s = aT / B
        */
        uint shares;
        if (totalSupply == 0) {
            shares = _amount;
        } else {
            shares = (_amount * totalSupply) / token.balanceOf(address(this));
        }

        _mint(msg.sender, shares);
        token.transferFrom(msg.sender, address(this), _amount);
    }

    function withdraw(uint _shares) external {
        /*
        a = amount
        B = balance of token before withdraw
        T = total supply
        s = shares to burn

        (T - s) / T = (B - a) / B 

        a = sB / T
        */
        uint amount = (_shares * token.balanceOf(address(this))) / totalSupply;
        _burn(msg.sender, _shares);
        token.transfer(msg.sender, amount);
    }
}
```

The Vault contract enables users to deposit tokens and receive shares that represent their portion of the Vault’s total balance. When withdrawing, users burn their shares to receive tokens. This system is ideal for staking or liquidity pools where users contribute to a shared resource.

#### Key Functions:
- **deposit(uint amount)**: Deposits ERC20 tokens into the Vault in exchange for shares.
- **withdraw(uint shares)**: Withdraws tokens by burning shares, with the amount received proportional to the user's stake.

## Using Remix for Compilation & Deployment

You can compile and deploy both contracts using [Remix](https://remix.ethereum.org/), an online Solidity IDE that simplifies the development process for smart contracts.

### Steps for Deployment Using Remix:

1. **Open Remix**: Navigate to [Remix IDE](https://remix.ethereum.org/).
2. **Create the Contracts**:
   - In the Remix file explorer, create two new files: `ERC20.sol` and `Vault.sol`.
   - Copy and paste the ERC20 code into `ERC20.sol` and the Vault code into `Vault.sol`.
3. **Compile**:
   - Go to the "Solidity Compiler" tab in Remix.
   - Select the appropriate Solidity version (`^0.8.17`) and click **Compile**.
4. **Deploy**:
   - Switch to the "Deploy & Run Transactions" tab.
   - Ensure you're connected to the correct Avalanche Subnet by configuring MetaMask.
   - First, deploy the `ERC20` contract.
   - Then, deploy the `Vault` contract, passing the address of the deployed `ERC20` contract to the constructor.
   
### Interacting with the Contracts:
- Once deployed, you can interact with the contracts directly from the Remix interface.
- **Mint Tokens**: Call the `mint` function on the ERC20 contract.
- **Deposit Tokens**: Use the `deposit` function in the Vault contract to deposit ERC20 tokens and receive shares.
- **Withdraw Tokens**: Use the `withdraw` function to burn Vault shares and retrieve your tokens.

## License

This project is licensed under the MIT License
