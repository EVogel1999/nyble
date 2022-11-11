---
title: "How to Programmatically get Candy Machine NFTs"
date: "2022-10-29"
draft: False
tags: ["solana", "javascript"]
showEdit: False
showWordCount: False
summary: "Solana is a very popular blockchain that is a lot more efficient that Ethereum.  The lower gas fees and higher transaction throughput made it ideal for NFTs and innovations surrounding them.  In this post, we'll learn how to programmatically retrieve NFT data using node and @solana/web3.js."
---

## Introduction

[Solana](https://solana.com/) is another very popular blockchain that is drastically different from EVM-based blockchains.  Instead of Proof of Stake, Solana uses Proof of History.  This has made Solana, in comparison with Ethereum, faster and have lower gas fees.  Additionally, Solana has a dedicated team working on NFT related programs called [Metaplex](https://www.metaplex.com/).  This team has created many different tools such as CLIs, customizable storefronts, and [no-code](https://studio.metaplex.com/) solutions.

Solana has become integral to the NFT community and innovation, therefore it's important to learn how to work with Solana smart contract programs.  A simple way to start is retrieving all the NFTs (minted or not) from a specific Candy Machine V2 address.  [Candy Machine](https://docs.metaplex.com/programs/candy-machine/overview) is a type of program that operates similarly to a gumball machine.  When people buy an NFT from the machine, it mints them a random NFT from the collection instead of a specific one.

### Technology Requirements

To follow along with this post you need the following installed on your machine:

- [Node.js](https://nodejs.org/en/download/)
- [Visual Studio Code](https://code.visualstudio.com/Download)

### Prerequisites

To fully follow along with this post it's assumed that you know how to work with ```node.js``` and ```javascript```.  Also, you will need to understand some key javascript concepts like ```async/await```.

Here are some resources if you need to learn these concepts:

- [Net Ninja: Node.js Crash Course Tutorial](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jsz4LDYc6kv3ymONOKxwBU)
- [Net Ninja: Asynchronous Javascript Tutorial](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jx2TTZk3IGWKSbtugYdrlu)

## Project Setup

To setup the project, create a new directory that we will be working in.  After that, init a new node project using the following command:

```
npm init -y
```

You will also need to install the [@solana/web3.js](https://www.npmjs.com/package/@solana/web3.js) dependency in order to work with the Candy Machine:

```
npm i @solana/web3.js
```

For the sake of simplicity, we'll be coding everything in one javascript function.  So the following setup will be used:

```javascript
async function main() {
    // TODO: Write code
}

main();
```

## Creating a Blockchain Connection

It's rather easy to connect to the Solana blockchain.  The @solana/web3.js package handles this part almost entirely by itself.  To connect to the Solana mainnet, you code the following in javascript:

```javascript
import { Connection, clusterApiUrl } from '@solana/web3.js';

async function main() {
    const connection = new Connection(clusterApiUrl('mainnet-beta'));
}
```

This connection is used to interact with the Solana blockchain.  In order to write data to the chain you will need your own public and private key; however, for reading data from the chain, you don't need your own key.

## Getting the Account Info

Getting a specific key's information requires knowing the public key of a Candy Machine.  For this post, lets use the public key ```9tQLFyLeaUwQ1PN2YDiFztZDxu4KT6px8CBYEapkshAD```.  Any specific Candy Machine V2 address will work though, not just this address.

To get started, we need to get the Account Info from the Candy Machine.  Depending on the program type, this account information can look different and hold different values.  The code for getting the Account Info is as follows:

```javascript
import { Connection, clusterApiUrl, PublicKey } from '@solana/web3.js';

async function main() {
    // Generate public key
    const key = new PublicKey("9tQLFyLeaUwQ1PN2YDiFztZDxu4KT6px8CBYEapkshAD");

    // Get account info with key
    const info = await connection.getAccountInfo(key);
}
```

We have to ensure that the string public key conforms to the cryptographic public key that is an acceptable parameter when getting the Account Info for the key.  From this info, we can get some basic information about the account such as it's ```Owner``` and ```Lamports``` (a lamport is a fraction of a SOL, the cryptocurrency used to pay gas fees).  You can also pass this information to other Metaplex or Solana packages to get more information out of them (such as the ```CandyMachine``` class from [@metaplex-foundation/mpl-candy-machine](https://www.npmjs.com/package/@metaplex-foundation/mpl-candy-machine)).

## Parsing the Data Buffer

The information that we need about the NFTs are located in ```info.data```.  However, there is one small problem that we need to work through first: ```data``` is a very long buffer that we need to parse through.  Thankfully, the information is chunked in such a way that it's easy to parse through and get specific information about the Candy Machine.

Each NFT has a 4 byte separator, name length of 32 bytes, followed by another 4 byte separator, then followed by 200 bytes for the URL of the metadata.  The NFT metadata starts at byte 717.  So to get the first NFT from the buffer, the code would look like the following:

```javascript
async function main() {
    info.data.slice(717, 717 + 4 + 32 + 4 + 200);
}
```

In order to get just the first URL of the NFT, it'd look like the following:

```javascript
async function main() {
    let url = info.data.slice(717 + 4 + 32 + 4, 717 + 4 + 32 + 4 + 200).toString();
    url = url.substring(0, url.indexOf('\x00'));
}
```

The code for getting just the URL looks a little different because we are cleaning up the URL string to remove the empty buffer values of ```\x00```.

From this, you can abstract out the code so it loops to the end of the buffer to get all the NFT URLs in the Candy Machine.  You can also setup pagination so you can only show 5 NFTs per page to speed up performance.  To get the NFT metadata programmatically, all you will need to do is make a network request using [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) or [Axios](https://www.npmjs.com/package/axios).

## Conclusion

In conclusion, Solana is a very popular blockchain for NFTs because of its low gas fees and official programs dedicated to innovating and working with the NFT protocol.  A simple way to get started working with Solana is by working with the @solana/web3.js dependency to view the NFTs of a Candy Machine V2 address.  Candy Machine is a type of program maintained by Metaplex.

Additionally, there are other packages that are maintained by Metaplex that will do similar things to this post such as their new [Javascript SDK](https://github.com/metaplex-foundation/js).  However, the point of this post was to work closer with the protocols, programs, and packages rather then it be abstracted out by the SDK which is why it was avoided in this post.

## Full Code

```javascript
import { Connection, clusterApiUrl, PublicKey } from '@solana/web3.js';

async function main() {
    const connection = new Connection(clusterApiUrl('mainnet-beta'));

    // Generate public key
    const key = new PublicKey("9tQLFyLeaUwQ1PN2YDiFztZDxu4KT6px8CBYEapkshAD");

    // Get account info with key
    const info = await connection.getAccountInfo(key);

    info.data.slice(717, 717 + 4 + 32 + 4 + 200);
    let url = info.data.slice(717 + 4 + 32 + 4, 717 + 4 + 32 + 4 + 200).toString();
    url = url.substring(0, url.indexOf('\x00'));
}

main();
```