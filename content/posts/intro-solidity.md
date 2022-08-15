+++
title = "Intro to Solidity"
date = "2022-08-15"
cover = "img/intro-solidity.jpeg"
author = "Emily Vogel"
description = "Solidity is a new and upcoming language that is used for EVM-based blockchain programming.  Learn some of the fundamentals behind the language by setting up a new hardhat project and coding a simple smart contract!"
+++

# Introduction

Blockchain technology is a new emerging technology revolutionizing many different industries including finance (DeFi), governance (DAOs), art (NFTs), and more.  There are a few languages that are popular for coding projects on the blockchain, ```solidity``` is used for EVM-based (ethereum virtual machine) blockchains.

One of the best ways to learn is by doing so this tutorial will focus on introducing the key concepts and fundamentals of solidity blockchain development with ```hardhat```.

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