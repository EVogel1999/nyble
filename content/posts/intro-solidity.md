+++
title = "Intro to Solidity"
date = "2022-08-15"
cover = "img/intro-solidity.jpeg"
author = "Emily Vogel"
draft = "False"
description = "Solidity is a new and upcoming language that is used for EVM-based blockchain programming.  Learn some of the fundamentals behind the language by setting up a new hardhat project and coding a simple smart contract!"
+++

# Introduction

Blockchain technology is a new emerging technology revolutionizing many different industries including finance (DeFi), governance (DAOs), art (NFTs), and more.  There are a few languages that are popular for coding projects on the blockchain, ```solidity``` is used for EVM-based (ethereum virtual machine) blockchains.

One of the best ways to learn is by doing so this tutorial will focus on introducing the key concepts and fundamentals of solidity blockchain development with ```hardhat```.

All the code with comments can be found at this [github repo](https://github.com/EVogel1999/solidity-workshop).

## Technology Requirements

To run this tutorial you need the following installed on your machine:

- [Node.js](https://nodejs.org/en/download/)
- [Visual Studio Code](https://code.visualstudio.com/Download)

## Prerequisites

To fully follow along with this tutorial it's assumed that you know how to work with ```node.js``` and ```javascript```.  Also you will need to understand some key javascript concepts like ```async/await```.

Here are some resources if you need to learn these concepts:

- [Net Ninja: Node.js Crash Course Tutorial](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jsz4LDYc6kv3ymONOKxwBU)
- [Net Ninja: Asynchronous Javascript Tutorial](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jx2TTZk3IGWKSbtugYdrlu)

# Project Setup

To write solidity, we will be using the [hardhat](https://hardhat.org/) development environment.  This environment contains everything that is needed to develop, test, and deploy solidity smart contracts.

To create a new hardhat project, first we need to initialize a folder using ```npm```:

```
npm init -y
```

After that, we need to install a few dependencies and then the hardhat package will take care of the rest of the setup:

```
npm install chai ethereum-waffle ethers @nomiclabs/hardhat-ethers @nomiclabs/hardhat-waffle @nomicfoundation/hardhat-toolbox
npx hardhat
```

If you are prompted to choose a between ```javascript``` or ```typescript```, choose **javascript**.

# Developing the Smart Contract

The smart contract that we will be developing is influenced by the genre of cyberpunk.  What we will be coding is a simple ```Job``` contract that will do the following:

- Allow the job client to deposit ETH into the contract
- Allow a team of people to accept the job
- Allow the client to mark the job as finished
- Auto pay the team who accepted the job

Create a new ```Job.sol``` file in the ```contracts``` folder and copy the following code into the file:

{{< code language="solidity" title="Job.sol" id="1" expand="Show" collapse="Hide" isCollapsed="false" >}}
// SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

contract Job {
    // TODO: Write code
}
{{< /code >}}

To create an empty smart contract, you need the following:

- A software license identifier (for now, leave it unlicensed)
- The version of solidity you want to run
- The contract name

## Registering the Client and Depositing ETH

Before the client can deposit ETH into the smart contract we first need to register who the client is.  To do that, we set a client variable in the constructor.

{{< code language="solidity" title="Job.sol" id="2" expand="Show" collapse="Hide" isCollapsed="false" >}}
// Enums
enum JobState {
    WAITING,
    ACCEPTED,
    COMPLETED
}

// Job information
address private client;
JobState private state = JobState.WAITING;

constructor() {
    // Sends the job client to the deployer
    client = msg.sender;
}
{{< /code >}}

```msg``` is a global variable that can be accessed in any ```function``` or ```modifier```.  The ```sender``` attribute in particular is the address (a wallet or contract address) that last called the contract.  So in this particular case, the person who *deploys* the smart contract is the ```msg.sender```.

Now that we know who the client is for the Job, we can write the ```deposit()``` function.

{{< code language="solidity" title="Job.sol" id="3" expand="Show" collapse="Hide" isCollapsed="false" >}}
// Contract events
event PaymentReceived(address from, uint256 amount);

/**
 * Deposits ether to the contract
 */
function deposit() payable public isClient {
    // Emit an event saying the contract recieved the deposit
    emit PaymentReceived(msg.sender, msg.value);
}

/**
 * Check that the client isn't making the transaction
 */
modifier notClient() {
    require(tx.origin != client, "Client can't perform action");
    _;
}

/**
 * Check that the client is making the transaction
 */
modifier isClient() {
    require(tx.origin == client, "Only client can perform action");
    _;
}
{{< /code >}}

To be able to deposit ETH into a contract the function has to be of type ```payable```.  This is a modifier (similar to public/private visibility modifiers) that allows the function to accept token payments.  We emit an ```event``` when the client deposits ETH in order to acknowledge the deposit (we don't need to do this but it's good practice).

To make sure that only the client is depositing ETH to the contract, we need to create ```modifier``` functions.  The two that are created above is ```notClient``` and ```isClient```; what these two functions do is check that the person calling a particular function (such as ```deposit()```) is the client we defined earlier.

Like ```msg```, ```tx``` is another global variable.  The difference between ```msg.sender``` and ```tx.origin``` is who the original wallet address that called the function.  To better visualize this, the table below is provided.  In the first one, contract A would register ```msg.sender``` and ```tx.origin``` as the same.  In the second one contract B would register them as different.

| Contract Call | ```msg.sender``` | ```tx.origin``` |
| --- | --- | --- |
| Wallet -> A | Wallet | Wallet |
| Wallet -> A -> B | A | Wallet |

{{< code language="solidity" title="Job.sol" id="4" expand="Show" collapse="Hide" isCollapsed="false" >}}
/**
 * Returns the total payout for the job
 */
function getPayout() public view returns (uint256) {
    return address(this).balance;
}
{{< /code >}}

Last but not least, we also need to provide a way to easily get the total payout for the Job.  This isn't strictly necessary as you can do this with frontend javascript libraries but contains another interesting modifier we haven't come across yet.  The ```view``` modifier is a modifier that allows the address to read the value from the blockchain **gas-free**.

In blockchain applications, every transaction onto the chain costs ETH (or the blockchain's native token).  So when you declare something as ```view``` or ```pure``` then that particular action isn't changing the chain's state, only reading from it and doesn't count as a transaction.