# Metatransactions in the CDapp

Metatransactions (MetaTX) present a paradigm shift in the blockchain world, especially in enhancing user experience by removing the need for users to pay gas fees. The CDapp leverages this concept by introducing a two-pronged approach involving the broadcaster service and specially designed contract methods.

## What are Metatransactions?

Metatransactions encapsulate the original transaction and relay it via a third-party entity which takes responsibility for the gas fee. This enables seamless interactions, particularly beneficial for new users unfamiliar with the concept of gas or those unwilling to spend on every transaction.

### Benefits

- User Friendliness: Users without ETH can still perform transactions.
- Flexibility: Allows dApps like CDapp to decide when and how to bear the gas costs, possibly leading to promotional or free transaction periods.
- Adoption: Removes a major hurdle for new users by simplifying interactions with blockchain-based applications.

## Broadcaster Service Middleware

The Broadcaster Service is not just an intermediary, but a critical component in the Metatransaction architecture.

This is a node service that exposes a REST api that accepts encoded data _(see below)_ and based on it broadcasts new transactions to the network using it's locally available wallet.

As with maintenance this is most an ops task, however any changes and issues should be reported to the Network team.

### Workflow
- The user initiates a transaction on the CDapp.
- Instead of sending the transaction directly to the blockchain, it's encoded and packaged.
- The encoding is actually a plain object in the format that the Broadcaster Service expects. While there isn't any specific documentation I can point to, here's a link to [the spot in the file where this happens](https://github.com/JoinColony/colonyCDapp/blob/master/src/redux/sagas/transactions/getMetatransactionPromise.ts#L209-L234)
- This encoded package is then sent to the Broadcaster Service.
- The Broadcaster Service deciphers the package, recreating the original transaction.
- The service then transmits the transaction to the blockchain, paying the required gas fees, using a locally available wallet.

```
 CDapp
   |
   | (Encoded Transaction Data)
   v
Broadcaster Service (Decodes & Broadcasts Transaction)
   |
   | (Reconstructed Transaction)
   v
Blockchain
```

### Security & Trust

By design, the broadcaster does not have access to the user's private key. Instead, it merely acts on the user's behalf, using its funds to process the transaction. This ensures that while the transaction integrity is maintained, the user's security is not compromised.

### Maintenance

The Broadcaster Service's wallet requires periodic top-ups. It's crucial to monitor the balance to prevent service disruptions. Automated alerts are set up to notify us in the case of the wallet balance getting low.

This is a ops task and does not require dev intervention, but for security and privacy reasons, I'll not be sharing more details here.

## The Contracts

To facilitate metatransactions, the Colony Network introduced specialized methods after v9 in their contracts.

While most of the logic for metatransactions is internal to contracts, the exposed method that you'll most likely to be using is `getMetatransactionNonce(address)`, which upon being called with the sender's address, returns the nonce for the next metatransaction. This is needed as one of the values that gets encoded and sent to the broadcaster is the nonce.

The method is available on all contract interfaces in the Colony Network: colony, tokens, extensions, etc

### Authorization Mechanism

For the blockchain to recognize and accept these third-party transactions, there's a need for an authorization mechanism. The contracts allow transactions to be "relayed" by a broadcaster (or relayer) on behalf of the original sender, ensuring the state changes reflect as if the original sender initiated them.

## CDapp's "Dual" Transaction System

### Detection Mechanism

The CDapp checks the user's profile settings to determine whether to use regular transactions or metatransactions. This check ensures that the user's preference is always respected.

### Saga Development

One of CDapp's design strengths is the invariance of sagas across both transaction methods. This ensures that developers can code without constant concern over the transaction pathway, enhancing development efficiency.

Using metatransactions requires a departure from specific colony-js helper methods, particularly those labeled "withProofs". Developers must directly use on-chain methods. This is because encapsulating and relaying transactions using helper methods that don't exist on-chain will lead to transaction failures.

---

Metatransactions represent a significant evolution in the blockchain space. By effectively addressing the gas fee barrier, they make blockchain applications, like CDapp, more user-friendly. However, their implementation requires a nuanced understanding, both at the infrastructure and development levels, to ensure they deliver on their promise without compromising security or functionality.
