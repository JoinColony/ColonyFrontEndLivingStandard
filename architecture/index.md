# Architecture overiew

## How data flows through the CDapp

In principle, we want to handle all interactions with the chain outside of the client. We do this using the block ingestor and lambda functions.

### The Block Ingestor

A very common pattern at Colony is taking some data from the user in the ui (usually via a form), and then using that data to call some method in the `colony-js` library, which in turn calls the respective method on the colony network contracts (written in solidity). We coordinate all of this via `redux` actions.

At a glance:

1. Use the ActionForm or ActionButton components to dispatch an action to the store
2. That action will get picked up by the `redux-saga` middleware, and if there's a listener attached, will run the corresponding saga
3. Inside the saga, we interact with our ColonyNetwork contracts using `colony-js`. We also dispatch actions to the store every time the state of a transaction changes, so the `GasStation` can display this information to the user.
4. If the side effect triggers an on-chain event, we listen for that event in a separate service called the [block ingestor](https://github.com/JoinColony/block-ingestor). Generally, we handle all database mutations inside the block-ingestor.
5. The ui will then implement some kind of "optimstic ui" with the intended changes, or poll the database until the block-ingestor's changes show up.

It looks something like:

![CDapp data flow](https://user-images.githubusercontent.com/1193222/213923691-9f7f6512-861d-4e0a-97d3-4997f7e0d0cf.png)

### Lambda functions

Sometimes you need on-chain data that either isn't saved in the db, or doesn't itself trigger an event to be picked up in the block ingestor.

For example, the current state of a motion. If you create a motion, it will start in staking mode. Then if a week goes by, and no-one has staked, the motion will fail (assuming the staking phase duration is less than a week). Because the motion's state can change depending on the time that has elapsed since the last state change, we can't rely on the db to be up-to-date (in this case, after a week, the db would still show that the motion is in its staking phase.)

For cases such as this, we fetch the latest data from the chain in a [lamda function](https://docs.amplify.aws/cli/function/#set-up-a-function), and then handle any relevant database mutations from there.

This means that in the front-end, all we need to do is make a query to the backend like normal, and it will handle running the code inside the lamda function and return its result.
