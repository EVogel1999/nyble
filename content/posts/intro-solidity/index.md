---
title: "Intro to Solidity"
date: "2022-08-15"
draft: False
tags: ["solidity", "ethereum", "javascript"]
showEdit: False
showWordCount: False
summary: "Solidity is a new and upcoming language that is used for EVM-based blockchain programming.  Learn some of the fundamentals behind the language by setting up a new hardhat project and coding a simple smart contract!"
---

## Introduction

Blockchain technology is a new emerging technology revolutionizing many different industries including finance (DeFi), governance (DAOs), art (NFTs), and more.  There are a few languages that are popular for coding projects on the blockchain, ```solidity``` is used for EVM-based (ethereum virtual machine) blockchains.

One of the best ways to learn is by doing so in this tutorial we will focus on introducing the key concepts and fundamentals of solidity blockchain development with ```hardhat```.

All the code (with comments) can be found at this [GitHub repository](https://github.com/EVogel1999/solidity-workshop).

### Technology Requirements

To run this tutorial you need the following installed on your machine:

- [Node.js](https://nodejs.org/en/download/)
- [Visual Studio Code](https://code.visualstudio.com/Download)

### Prerequisites

To fully follow along with this tutorial it's assumed that you know how to work with ```node.js``` and ```javascript```.  Also, you will need to understand some key javascript concepts like ```async/await```.

Here are some resources if you need to learn these concepts:

- [Net Ninja: Node.js Crash Course Tutorial](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jsz4LDYc6kv3ymONOKxwBU)
- [Net Ninja: Asynchronous Javascript Tutorial](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jx2TTZk3IGWKSbtugYdrlu)

## Project Setup

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

If you are prompted to choose a between ```javascript``` or ```typescript```, choose **javascript**.  Select the option for setting up a empty hardhat project, you can use the other options too if you want to see a boilerplate example of a hardhat project.

## Developing the Smart Contract

The smart contract that we will be developing is influenced by the genre of cyberpunk.  What we will be coding is a simple ```Job``` smart contract that will do the following:

- Allow the job's client to deposit ETH into the contract
- Allow a team of people to accept the job
- Allow the client to mark the job as finished
- Auto pay the team who accepted the job

Create a new ```Job.sol``` file in the ```contracts``` folder and copy the following code into the file:

```solidity
// SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

contract Job {
    // TODO: Write code
}
```

To create an empty smart contract, you need the following:

- A software license identifier (for now, leave it unlicensed)
- The version of solidity you want to run
- The contract name

### Registering the Client and Depositing ETH

Before the client can deposit ETH into the smart contract, we first need to register who the client is.  To do that, we set a client variable in the constructor.

```solidity
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
```

```msg``` is a global variable that can be accessed in any ```function``` or ```modifier```.  The ```sender``` attribute is the address (a wallet or contract address) that last called the contract.  In this particular case, the person who *deploys* the smart contract is the ```msg.sender```.

Now that we know who the job's client is, we can write the ```deposit()``` function.

```solidity
// Contract events
event PaymentReceived(address from, uint256 amount);

/**
 * Deposits ether to the contract
 */
function deposit() payable public isClient {
    // Emit an event saying the contract received the deposit
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
```

To be able to deposit ETH into a contract the function has to be of type ```payable```.  This is a modifier (similar to public/private visibility modifiers in other languages) that allows the function to accept token payments.  We emit an ```event``` when the client deposits ETH in order to acknowledge the deposit (we don't need to emit an event for this but it's good practice).

To make sure that only the client is depositing ETH to the contract, we need to create ```modifiers```.  The two that are created above are ```notClient``` and ```isClient```; what these two functions do is check that the person calling a particular function (such as ```deposit()```) is the client we defined earlier.

Like ```msg```, ```tx``` is another global variable; ```origin``` is an attribute on ```tx``` that is similar to the ```msg.sender``` attribute.  The difference between ```msg.sender``` and ```tx.origin``` is who the "origin" wallet address that called the function.  To better visualize this, a table below is provided.  In the first scenario, contract A would register ```msg.sender``` and ```tx.origin``` as the same.  However, in the second scenario contract B would register them as different.

| Contract Call | ```msg.sender``` | ```tx.origin``` |
| --- | --- | --- |
| Wallet -> A | Wallet | Wallet |
| Wallet -> A -> B | A | Wallet |

```solidity
/**
 * Returns the total payout for the job
 */
function getPayout() public view returns (uint256) {
    return address(this).balance;
}
```

Last but not least we also need to provide a way to easily get the total payout for the job.  This isn't strictly necessary as you can do this with frontend javascript libraries but contains another modifier we haven't come across yet.  The ```view``` modifier is a modifier that allows the address to read a value from the blockchain **gas-free**.

In blockchain applications, every transaction onto the chain costs ETH (or the blockchain's native token).  So when you declare something as ```view``` or ```pure``` then that particular action isn't changing the chain's state, only reading from it and doesn't count as a transaction.

### Accepting a Job

Now that the client can deposit ETH for the job into the smart contract, we need to write the functionality for a group of people to accept the job.  We need to include the ability to record the whole percentages of the cut for each team member.

```solidity
// Team information
address[] private members;
uint8[] private shares;

/**
 * Accepts a job with a given team array and the shares for each team member
 */
function acceptJob(address[] memory m, uint8[] memory s) external notClient {
    // Check members and shares are defined and same length
    require(m.length == s.length, "Team members and shares must be same length");
    require(m.length > 0, "Team must have at least one member");
    require (state == JobState.WAITING, "Job must be awaiting team");

    // Check the total shares is equal to 100 (100%)
    uint8 total = 0;
    for (uint i = 0; i < s.length; i++) {
        total += s[i];
    }
    require (total == 100, "Shares must total to 100");

    // Set team members
    members = m;
    shares = s;

    // Set contract state to accepted
    state = JobState.ACCEPTED;
}
```

The first three ```require``` statements check that the team members and shares match in length, that there is at least one member in the team, and that the job is waiting for someone to accept it.

The for loop checks that the shares array total to 100 (or 100%).  If it passes those checks then the members and their shares are recorded and the job's state moves to "accepted", no other team can accept the job now.

### Completing a Job

The only thing that's left for this contract is for the client to complete and close out the job, automating the payments to each team member.  To do the payments safely and securely, we will be using another contract from a company called ```Open Zeppelin```.  Open Zeppelin is a blockchain security company that offers a library of secure solidity contracts for the community to use.

We install these contracts through ```npm```:

```
npm i @openzeppelin/contracts
```

Then we import it using the ```import``` statement:

```solidity
//SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Address.sol";

contract Job {
    // ...
}
```

```solidity
/**
 * Completes a job and sends out the payment to the team
 */
function complete() external isClient {
    require(state == JobState.ACCEPTED, "Job must be accepted to complete");

    // Set job state
    state = JobState.COMPLETED;

    // Payout each team member
    uint256 totalPayout = this.getPayout();
    for (uint i = 0; i < members.length; i++) {
        uint256 cut = (totalPayout * shares[i]) / 100;
        Address.sendValue(payable(members[i]), cut);
    }
}
```

The ```complete()``` function only needs to check that the job is in the accepted state so it can later be moved to completed.  Then we iterate through each team member and automate payments to them using Open Zeppelin's ```Address``` contract.

What's important to note here is that **solidity doesn't have decimals**, so the ```cut``` needs to be a whole number.  This is why the member's share is multiplied by the total payout and divided by 100.

## Creating a Deploy and Test Script

One of the great features about blockchain development environments like hardhat is they provide the setup needed to run and test on a local ethereum node on your computer.  This means you don't need to deploy to the ```testnet``` or ```mainnet``` in order to test out your smart contracts.

### Deploying the Contract

To create a simple deploy/test script, create a new ```run.js``` file in the ```scripts``` folder:

```javascript
async main () => {
    // TODO: Write code
}

main()
```

In order to test the functions we have written, the first step is to deploy the contract to our local ethereum node.  To do this, we use the ```ethers``` library installed earlier:

```javascript
// Init contract
const [client, addr1, addr2] = await hre.ethers.getSigners();
const JobFactory = await hre.ethers.getContractFactory('Job');
let jobContract = await JobFactory.deploy();
await jobContract.deployed();

console.log('Contract deployed to:', jobContract.address);
```

When you start a node, 10 random wallets pre-loaded with test ETH are created onto the node.  To test our contract, we want to save 3 of these.  The first one in the array is always the default wallet that is connected to the node (hence why it's named ```client```).

Ethers does most of the work for us when deploying a smart contract.  What we need to do first is get the ```Job``` factory, then we need to deploy it (using the provided ethers method).  The last step we need to do is wait for the contract to be ```deployed```.  With blockchain, **just because a transaction is added to the chain doesn't mean that it was successful**.  The purpose of waiting for deployed is to ensure the transaction went through and that the contract is live on our local node.

### Executing Functions from Javascript

Now that the contract has been deployed to our local ethereum node, we can interact with it using the ```ethers``` library.  To test this, lets deposit some ETH into the smart contract:

```javascript
// Constants
const weth = 1000000000000000000;

// Deposit ETH to the contract
await jobContract.deposit({value: ethers.utils.parseEther('1.3')});
const payout = await jobContract.getPayout();
console.log('Payout (ETH): ', payout / weth);
```

If you recall while coding ```Job.sol```, it was mentioned that solidity does not support decimals and the only way to work with fractions and decimals is to mimic the fraction using whole numbers.  ```1 ETH``` is equivalent to ```1,000,000,000,000,000,000```, anything less then this number is called ```WETH```.  We can manually parse the WETH returned using the ```getPayout()``` method or use an ethers library.

To deposit ETH into the contract programmatically, we need to pass a ```value``` as a javascript object in the WETH equivalent number.  When the ```deposit()``` function is called, the amount of WETH will be automatically withdrawn from the client's wallet.

```javascript
// Accept job
jobContract = jobContract.connect(addr1);
await jobContract.acceptJob([addr1.address, addr2.address], [75, 25]);
```

To accept the job, we can pass in arrays to the ```acceptJob()``` function that represent the team and each member's share.  In the example above ```addr1``` is receiving ```75%``` of the total payout while ```addr2``` is receiving ```25%```.

In order for this function to work, we need to simulate another wallet connecting and interacting with the contract (not just the client).  To do that, we use an ethers function called ```connect()```.

```javascript
// Complete job, check address balances
jobContract = jobContract.connect(client);
let preBal = await addr1.getBalance();
console.log('Pre-Job Value: ', preBal / weth);

await jobContract.complete();

let postBal = await addr1.getBalance();
console.log('Post-Job Value: ', postBal / weth);
console.log('Value Difference: ', (postBal - preBal) / weth);
```

The last function we need to check is ```complete()```.  After we connect the client address back to the contract, we can call the ```complete()``` function.  To see if the team members were adequately paid we check their wallet's ```balances``` before and after completing the job.

## Conclusion

That's it!  In this tutorial we created a new hardhat project, smart contract, and deploy script.  This covers most of the fundamentals of solidity development however there are more advanced concepts to learn that include but are not limited to the following:

- Design patterns
- Unit testing
- Cybersecurity
- Frontend + smart contract integrations

Thank you for reading and I hope this was helpful!

## Full Code

[GitHub Repository](https://github.com/EVogel1999/solidity-workshop)

```solidity
// SPDX-License-Identifier: Unlicense
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/utils/Address.sol";

contract Job {
    // Enums
    enum JobState {
        WAITING,
        ACCEPTED,
        COMPLETED
    }

    // Contract events
    event PaymentReceived(address from, uint256 amount);

    // Job information
    address private client;
    JobState private state = JobState.WAITING;

    // Team information
    address[] private members;
    uint8[] private shares;

    constructor() {
        // Sends the job client to the deployer
        client = msg.sender;
    }

    /**
     * Returns the total payout for the job
     */
    function getPayout() public view returns (uint256) {
        return address(this).balance;
    }

    /**
     * Gets the current job state
     */
    function getJobState() external view returns (string memory) {
        if (state == JobState.WAITING) {
            return "Waiting";
        } else if (state == JobState.ACCEPTED) {
            return "Accepted";
        } else {
            return "Completed";
        }
    }

    /**
     * Deposits ether to the contract
     */
    function deposit() payable public isClient {
        // Emit an event saying the contract received the deposit
        emit PaymentReceived(msg.sender, msg.value);
    }

    /**
     * Accepts a job with a given team array and the shares for each team member
     */
    function acceptJob(address[] memory m, uint8[] memory s) external notClient {
        // Check members and shares are defined and same length
        require(m.length == s.length, "Team members and shares must be same length");
        require(m.length > 0, "Team must have at least one member");
        require (state == JobState.WAITING, "Job must be awaiting team");

        // Check the total shares is equal to 100 (100%)
        uint8 total = 0;
        for (uint i = 0; i < s.length; i++) {
            total += s[i];
        }
        require (total == 100, "Shares must total to 100");

        // Set team members
        members = m;
        shares = s;

        // Set contract state to accepted
        state = JobState.ACCEPTED;
    }

    /**
     * Completes a job and sends out the payment to the team
     */
    function complete() external isClient {
        require(state == JobState.ACCEPTED, "Job must be accepted to complete");

        // Set job state
        state = JobState.COMPLETED;

        // Payout each team member
        uint256 totalPayout = this.getPayout();
        for (uint i = 0; i < members.length; i++) {
            uint256 cut = (totalPayout * shares[i]) / 100;
            Address.sendValue(payable(members[i]), cut);
        }
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
}
```

```javascript
async main () => {
    // Constants
    const weth = 1000000000000000000;

    // Init contract
    const [client, addr1, addr2] = await hre.ethers.getSigners();
    const JobFactory = await hre.ethers.getContractFactory('Job');
    let jobContract = await JobFactory.deploy();
    await jobContract.deployed();
    console.log('Contract deployed to:', jobContract.address);
    console.log();


    // Deposit ETH to the contract
    await jobContract.deposit({value: ethers.utils.parseEther('1.3')});
    const payout = await jobContract.getPayout();
    console.log('Payout (ETH): ', payout / weth);
    console.log();

    // Accept job
    jobContract = jobContract.connect(addr1);
    await jobContract.acceptJob([addr1.address, addr2.address], [75, 25]);
    const state = await jobContract.getJobState();
    console.log('Current Job State: ', state);
    console.log();

    // Complete job, check address balances
    jobContract = jobContract.connect(client);
    let preBal = await addr1.getBalance();
    console.log('Pre-Job Value: ', preBal / weth);
    await jobContract.complete();
    let postBal = await addr1.getBalance();
    console.log('Post-Job Value: ', postBal / weth);
    console.log('Value Difference: ', (postBal - preBal) / weth);
}
    
main();
```