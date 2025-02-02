---
sidebar_position: 4
---

# Lit Actions

:::note

Lit Actions are still heavily in development and things may change.

To start developing with Lit Actions, check out examples [here](/SDK/Explanation/litActions.md). For an in-depth review of the functionality provided by the Lit Actions SDK, take a look at our [API docs](https://actions-docs.litprotocol.com/).

:::

## What are Lit Actions?

Lit Actions are JavaScript functions that can use the threshold cryptography that powers the Lit network. They are basically JavaScript smart contracts, only much more powerful.

Lit Actions are immutable, blockchain agnostic, and can make HTTP requests. This means that they can utilize on-chain and off-chain data in their computation. You can write some code and ask the Lit nodes to execute it, for example asking the network to calculate the difference between two random integers, x and y. 

The result of this function can be used to pass some functionality, like provisioning signature or decryption shares to a user (*if the difference between x and y is greater than 10, sign the result*).

## What can I use Lit Actions for?

Lit Actions are essentially decentralized serverless functions. You can use Lit Actions in conjunction with PKPs to sign and decrypt data, functionally creating a smart contract with its own key pair. 

A toy example would be a Lit Action and corresponding PKP that checks if a number is prime, and only signs it if it is prime.  Think of it as a sort of “prime number” certification service.  Since the Lit Action is immutable, and since you can permanently assign a PKP to a Lit Action, there is a provable chain of trust.  This means you could present the signature and a number to someone, and they could simply check the signature against the public key of the PKP to see if the number is actually prime, instead of having to do all the math to ensure that the number is actually prime.  The signature acts as a proof that the number is prime.  

## Signatures vs. Proofs

It is important to breakdown the distinction between a "signature" and what can be considered a "proof".

### Signing

In cryptography, a digital signature proves that a user controls the private key associated with a particular public address. This is commonly used in the context of *authorization*. Apps can use this signature to verify a user has control over the digital identity provided by the address, and provide authentication to do both web3 and arbitrary, non-web3 actions. For example, changing a profile on OpenSea requires signing in order to confirm the association between the profile owner the the wallet associated with it.

In working with the Lit network, someone will sign with their wallet when they want to mint a PKP. That is a transaction signature — where someone is sending a transaction (minting). This signature is than used as the method of authentication over the PKP and associated Lit Actions.

### Proofs

A proof is a particular application for a digital signature. For example, using a signature to *prove* that a particular interaction took place. 

Signing through Lit Actions opens up the possibilities of verifying information from external sources, such as from a Weather API. Within the Lit ecosystem this signed information is called a proof. 

## Permissionless Execution and Working With External Smart Contracts

Right now, all Lit Actions have to be executed manually by a user. In order to enable the truly permissionless, "condition-based" execution of Lit Actions, we are working on developing our own native [event listening](https://litprotocol.notion.site/Event-based-triggers-for-PKPs-0b22532c571f4dffb08d6e84e9c5dd04) solution. 

Right now, you could use something like [Tenderly Web3 Actions](https://docs.tenderly.co/web3-actions/intro-to-web3-actions) to listen for an on-chain event and then call a Lit Action as a result.

## Do I need a PKP to run a Lit Action? 

No, but without it, Lit Actions lose their "magic" capabilities of signing and decryption. An Action without a PKP is essentially just a serverless function that **can’t sign** or decrypt arbitrary data.  

## Test your knowledge of Lit Actions on 101.xyz
Go through this [Lit Actions](https://101.xyz/course/claift8fc415408l116dbsqpl) course on 101.xyz and receive an NFT credential to show your proof of knowledge. 

## Getting started with Lit Actions

Get started with Lit Actions [here](/SDK/Explanation/LitActions/helloWorld).

More [examples](/coreConcepts/usecases#programmatic-signing-lit-actions-and-pkps).
