# Safe Control

Safe Control is a cross-chain feature that allows users to interact with a Safe (https://safe.global/) via the Colony UI. It supports cross-chain bridging, meaning that you can interact with a Safe even if it's on a different chain to the Colony (supported chains only).

With this feature, they can: Transfer funds and NFTs to a member of the colony, create raw transactions, and use their Safe to interact directly with other smart contracts.

## How Safe Control Works

### Dialogs

This feature comes with 3 dialogs that can be accessed through the UAC:

- "Adding a Safe" dialog
- "Remove Safe" dialog
- "Control Safe" dialog

The first step to controlling a Safe is by linking that safe to the Colony via the "Adding a Safe" dialog. This is done by providing the Safe's contract address. Currently we only support Safes on the Ethereum and Binance networks. Once the user provides the address, they will be instructed to install a module (Zodiac Bridge module) to their safe through the Safe web app. Installing this module is important because: 
- It allows the user to interact with the Safe from a Colony on a different chain (DAI in our case)
- It allows the user to bypass the normal requirement of needing multiple signatures when executing a Safe transaction.

The "Remove Safe" dialog is quite simple. It's used to remove/unlink a safe from the colony. The user should be able to remove 1 or multiple safes at once.

Last but not least, the "Control Safe " dialog is used to interact with a linked safe. In it, the user can choose to send 4 different types of transactions (Transfer funds, transfer NFT, raw transaction, and contract interaction), and decide whether you want to send just 1 transaction or multiple. After they fill up the form with as many transactions as they want, they can review what they entered one final time in a preview screen before sending them to the contracts.

Something to keep in mind is that we use APIs for 3 of the 4 transaction types:
- The Transfer funds and NFT transactions use [Safe Transaction API](https://safe-transaction-mainnet.safe.global/) (this API is used in the case of the Safe being in the Ethereum network, for example) which allows us to fetch the balance and NFTs of a safe.
- The Contract interaction transaction uses the corresponding block explorer API of the selected safe's network. In our case, that would be either [Etherscan](https://etherscan.io/) or [BscScan](https://bscscan.com/). They are used to fetch the verified contract's ABI whenever the user enters a valid contract address.

### Sagas

Both the "adding" and "removing a safe" dialogs use the same saga, in which we store the newly updated colony's metadata in the DB and use the `editColony` method to link/unlink the safes to the colony.

When it comes to the "Control Safe" dialog's saga, the flow is more complicated.

- We take the transactions provided by the user and format the data before encoding it onto an `executeTransaction` function, which is the method used by the Zodiac Bridge Module contract to execute the transactions.

- After encoding the function data, we encode it once more but this time onto a function called `requireToPassMessage`, which is the method used by the Home Bridge to pass along messages to the Foreign Bridge.

- The result of that is then sent to the home bridge contract through the `makeArbitraryTransactions` Colony contract method. In the case that the user is creating a motion, this `makeArbitraryTransactions` would be encoded onto a `createMotion` function, which is the method used by the Colony Network to create motions.

- In the colony contracts, `makeArbitraryTransactions` will emit an `ArbitraryTransaction` event for every safe transaction that the user added in the dialog.

- After sending the transaction to the contracts, we then store the safe transaction data in the DB using 2 models: `SafeTransaction` and `SafeTransactionData`.
  - `SafeTransaction` contains the transaction hash as an id, title, selected safe, and transactions made which is an array of `SafeTransactionData`.
  - `SafeTransactionData` contains most of the data that the user entered in the form, such as the transaction type, the recipient, the amount, the recipient address, the contract method, the contract method params, etc...

- Finally, the user is redirected to the action/motion page where they will finalize the process by following a link that we provide for them to execute their transactions on the other network via a Live Monitor app that keeps track of cross-chain Safe transactions. Meanwhile, the action/motion page will show a tag indicating that the transaction needs action from the user to be completed. Once the transaction is executed by the user in the live monitor app, the tag will be updated to show that the transaction was executed.

- To summarize the process, it would be something like this: Saga > Colony contracts (`makeArbitraryTransactions`) > Home Bridge (`requireToPassMessage`) (Gnosis network) > Foreign Bridge (Ethereum or Binance network) > Transaction is manually executed on the foreign chain by the user via the Live Monitor app > Zodiac Bridge Module (`executeTransaction`) > Selected safe executes the transactions.

### Block ingestor

- We look for `ColonyMetadata` events in the case of adding or removing a safe and then we parse them client-side in order to know whether the event is emitted due to a change of linked safes, colony name, address book, etc.

- For safe transactions, we catch the `ArbitraryTransaction` event and parse it to know whether the transaction was created by the user or not and if it's already present in the DB. In the first case, the transaction might not be created by the user if the arbitrary transaction was emitted by the motion finalizing. In the second case, the transaction might already be present in the DB if the user sent multiple transactions (every transaction included by the user in the dialog will emit its own event).

## Live Monitor App

For Ethereum: https://alm-gnosis-eth.colony.io/
For Binance: https://alm-gnosis-bsc.colony.io/

These services are hosted and maintained by us. It was previously developed and maintained by [TokenBridge](https://docs.tokenbridge.net/) but they seem to have abandoned it, and with this app being essential for the users to execute their transactions, we've decided it to maintain it ourselves.

## Testing Safe Control locally

Testing this feature on the dev env is possible due to the docker container called `safe`, which will be used to deploy a foreign chain, home and foreign bridges, and a safe with tokens and 1 NFT. The addresses to each of the relevant contracts will be printed in the console once the container is up and running.

Once they are printed out, you must include them in the `.env` file as the `.env.example` indicates.

With that, you can start testing the feature as you would in QA or production with the only difference being that you don't need to execute the transactions in the live monitor app since the transactions will be executed automatically by the mock contracts, therefore, locally the transactions will always be shown as `Completed` from the get-go (We don't check if they were executed, like in the QA or prod env, we assume they were). Since we use mock contracts in a mock foreign chain, the Safe (which MUST be an actual deployed Safe in either ETH or BNB) that you may link to your colony in the dev env is not used as the target for these transactions, as there would be no way to reach it, instead we use as a target the mock Safe you just deployed by running the dev env and included in the `.env` file.

### Final considerations

- The action/motion should appear in the list with a custom title provided by the user in the final step of the "Control Safe" dialog that has a `Safe transaction: ` prefix and again, a tag like in the action/motion page showing whether the transactions were executed or not in the other network.

- In the case of a motion, this status tag will not show until the motion has passed and is finalized.
