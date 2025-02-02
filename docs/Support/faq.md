---
sidebar_position: 1
---

# FAQ

## General Questions

### 1. Does the SDK work with Typescript?

Yes, you can find the library [here](https://github.com/LIT-Protocol/js-sdk). Lit will be deprecating the older JavaScript library soon.

### 2. Are there fees for using Lit? What about rate limits?

Currently access control conditions aren’t very expensive in terms of compute & storage, so we’ve been working under the premise that the “artificial” rate limiting of web3 (ie, rpc endpoints aren’t lightning fast right now) will provide all users with an equal opportunity of excellent network performance. Our payment model, when it becomes active is predicated on payment being made to store access control conditions - not to read & evaluate them.

So while we don’t have plans for access control rate limiting yet, it could help with scaling up; Lit could envision a project that stores a single access control conditions, and then effectively attempts to “read” it a thousand times a second … obviously this starts to add up, and takes from other projects.   This could create a rate limiting scenario - though we’d prefer to scale up first!   

In the end, rate limiting (if applied correctly and in good faith!) is a reasonable economic measure that falls into the web3 ethos of paying for what gets used - and only what gets used.  🙂  

### 3. Why don’t I get a Metamask popup for signing?

The signature is stored in your browser local storage for convenience. You can call the LitJsSdk.disconnectWeb3() method to delete it from the local storage.

### 4. How would this work if we wanted to use a custodial wallet instead?

With a custodial wallet, there is no need to store the signature. the reason it's stored is to prevent the user from having to sign the Metamask popup a dozen times if they're doing a dozen operations. but with a custodial wallet, there is no Metamask popup, so you can just create the signature fresh each time.

## PKPs / Lit Actions

### 1. What is the difference between authorization and authentication?
An authentication method refers to the specific credential (i.e a wallet address, Google oAuth, or Discord account) that is programmatically tied to the PKP and used to control the underlying key-pair. 

Authorization is through auth signatures - an auth sig is always required when making a request to Lit, whether it be decrypting some piece of content or sending a transaction with a PKP.  

## Access Control

### 1. Why do I need to call saveSigningCondition() before getSignedToken()? Shouldn’t the former just return a JWT?

The reason behind this separation is that saveSigningCondition() is a function that an admin/developer of a website will most likely call only once to set up the token-gating mechanism. The function getSignedToken() can then be called for every website visitor. You can view this as a 1-to-many relationship.

### 2. Can I deploy my signing condition without including it in my codebase?

Yes, by accessing the dev console on https://litgateway.com/ (right click, choose “inspect”, go to console tab), you can access the globally exposed LitJsSdk and paste your saveSigningCondition() code. This will automatically publish your resourceId to Lit nodes.

### 3. If I call saveSigningCondition() twice with the exact same conditions, do two copies get created and stored by nodes?

If the resourceId is exactly the same, the nodes will not hold a second copy. However, to deploy iterations of the resourceID, the “extraData” field is available to provide versioning (e.g., "extraData": "v2”). Any change in the resourceId object will create a new unique resourceId for the nodes to store.

### 4. If I already have a connect/sign mechanism in place, can I just create my own authSig object and pass it to saveSigningCondition() instead of using checkAndSignAuthMessage()?

Yes, you most certainly can. The following link provides an example of how this can be done: https://github.com/LIT-Protocol/hotwallet-signing-example/blob/main/sign.js. You MUST use an EIP 4361 compliant signature (aka Sign in with Ethereum)

### 5. Can more than one condition be added for access control?

Yes! See https://developer.litprotocol.com/docs/AccessControlConditions/booleanLogic for examples.

### 6. Can I delete or edit a published resourceId?

No, you cannot. If the condition was saved with `"permanent": false` then you can edit the condition associated with the resourceId, but you cannot ever edit the resourceId itself. If you wanted to edit a resourceId, you could create a new one with the same conditions, and as long as the old condition was stored with `"permanent": false` then you could remove all the conditions and make that old resourceId impossible to use.


### 7. Are encryptedSymmetricKey and encryptedString unique to a user or global for a piece of content that we've encrypted?

Global for the piece of content. The only exception, where something is scoped to a user, is if you want to update the access control conditions, because you set permanent: false
when you stored the access control condition.  In that case, then the wallet address that stored the condition is able to update it, and only that wallet address can update it.

## Security and trust implications

### 1. What encryption algorithm are you using? AES?

Yes. AES-GCM webcrypto for the symmetric encryption. then that key is encrypted to the Lit network's BLS public key. the BLS private key shares are used by the nodes to decrypt.


### 2. How does Lit handles key management?

There is only one key, created with distributed key generation. The nodes all know the public key but nobody knows the whole private key. 

### 3.  What's to prevent a Lit node operator from discovering all the symmetric keys stored on the network and being able to decrypt anything?
**(cont'd) It says that it uses BLS threshold signatures so that the decryption key is split into multiple pieces, but that doesn't really totally explain how the key management works? How nodes learn about the various keys being managed by the network? How you prevent one node operator from accumulating all the component keys needed to reconstruct any of the symmetric decryption keys?**

Each node only holds a private key share. When a user wants to decrypt something, he presents the thing to decrypt, and proof that he meets the conditions (a wallet signature). Each node independently checks that the user meets the condition with a RPC call to an ETH node. If the user meets the condition, the node uses its private key share to create a decryption share. The user collects the decryption shares and accumulates them above the threshold, and is then able to decrypt the content. 

So, you can see, the nodes don't talk to each other when decrypting the content. Each node's private key share never leaves the node.

### 4. How do new nodes that come online discover the key shares they need to help decrypt previously-encrypted data? Or is each key fragment tied to a single node lifetime?
**(cont'd) If the latter, then doesn't that mean that there's no redundancy on the key fragments and you're very susceptible to nodes going offline? Like if too many nodes go offline then you might no longer have enough key fragments in the online nodes to decrypt some pieces of content?**

Right now, we're running all the nodes, so the nodes and shares don't change. soon, we will run a federated network with named nodes, and the ultimate goal is to run a permissionless one. we use a process called proactive secret sharing to share the private key shares with new nodes as they come online. the shares given to new nodes are incompatible with any nodes that have left the network. we use threshold encryption with a 2/3 threshold so redundancy is built in.

### 5. What's to prevent one person from running many Lit Protocol nodes so as to acquire sufficient key fragments across their nodes to be able to reconstitute the decryption key for some pieces of content?

In the federated network with named nodes, we know who the operators are, so a sybil attack is pretty hard. In the permissionless network, node operators must stake to run a node. so, you can do the math there to figure out the cryptoeconomic guarantees depending on the number of nodes and the staking cost, which are parameters we will tune. We also intend to use probabilistic guarantees that make it difficult for a given node operator to actually know which private key shares they have and which public keys they correspond to. Meaning, a node operator doesn't actually know what stuff the key share they are holding will unlock. the goal here is to make it difficult to amass 2/3 of the private key shares for a given private key.

### 6. Orphaned and unreachable data due to nodes rotating
**(cont'd) Even with the redundancy that 2/3 threshold encryption gives you, that's 2/3 of the number of shares at the time of the encryption right? So over a long enough time horizon, isn't it fairly likely that after a few years you'll have had enough nodes rotate off the network that more than a third of the shares for some early content encrypted by the network are now lost, leaving that data orphaned and unable to ever be decrypted?**

Nope! you're probably thinking of shares in the context of shamir's secret sharing? We use threshold encryption which is different. Nodes all share one big private key generated via a distributed key generation operation. Nobody knows the whole private key. As nodes join and leave the network, through a process called proactive secret sharing, the private key shares are regenerated and each node gets a new private key share. But the shares, together, still represent the original private key.

So the entire network could turnover (all nodes that were there when you encrypted content are gone) but you can still decrypt the content, becuause the private key itself is persisted

### 7. So you need 2/3 of the entire network to decrypt content?  not 2/3 of some fixed constant number of key fragments?

Yup! those are the default params we are launching with. Over time, we want to let users launch their own "subnets" with their own parameters.

### 8. As the number of nodes on the network grows, it gets more secure, but also slower to decrypt content?

Yeah that's correct. we plan to tune the network to have the max number of nodes while still remaining within some performance bounds. like if we can have 100 nodes and it takes less than 2 seconds to unlock something, that would be acceptable. when the network grows beyond that size, we support the automatic creation of subnets, which are basically just parallel networks. and then when someone goes to store some content, automatically load balance between those subnets

### 9. So long as that an attack on the network remains impractical then the system is pretty robust?
**(cont'd) One operator runs enough nodes to gather sufficient fragments to decrypt stuff - is sufficiently difficult and costly to execute so as to be impractical.  That still seems like the most likely attack vector if things aren't designed just right. **

Yeah you nailed it - there are lots of little tricks we can use there.  like suppose there are 10 subnets each with their own private key, each with 100 nodes.  if you wanted to break one of those private keys, you would need to run 66 of 100 of those nodes.  but, when you join the network, the subnet you are assigned to is uniformly random.  so now, you need to run 666 of 1000 nodes to amass those 66 shares for a single given subnet (maybe technically less due to the birthday problem).  we could give out fake shares.  we could make it difficult to discover which public key goes with which private key share, so as a node operator can't even tell which shares will unlock a given piece of data.  It's a hard problem to solve but we have a lot of tools.

## I have a question that isn't answered here. Where can I get help?

Join our discord: https://litgateway.com/discord
