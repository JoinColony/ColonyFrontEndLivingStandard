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

# Glossary: Tech you should be/become familiar with

- [PostCSS](https://postcss.org/): we use to enhance css. Examples include generating a style object for use in components, passing values between css files, modifying the alpha channel of a colour variable using `color-mod` , and using Sass syntax.
- [Yup](https://github.com/jquense/yup): We use yup for form validation. It's a very powerful tool and can generally handle most validation requirements (e.g. type /format validation, async validation, custom test generation...)
- [React-Intl](https://formatjs.io/docs/react-intl/): This is our internationalization library. Anytime you display strings to be read by the user, they should be wrapped by the corresponding React-Intl component or utility to ensure they can be internationalized by us at some stage in the future.
- [React Hook Form](https://react-hook-form.com/): Familiarise yourself with the API since this is our form library (we previously used Formik).
- [Redux](https://redux.js.org/) We use Redux mainly to pass information about on-chain actions we're triggering around the app. We use it heavily in conjunction with [Redux Saga](https://redux-saga.js.org/), which is what we use to manage side-effects (in other words, the place where we use `colony-js` to interact with ColonyNetwork contracts)
- [Apollo](https://www.apollographql.com/docs/react/): Our graphql client and client-side caching layer.
- [AWS Amplify](https://docs.amplify.aws/) We use Amplify for our database and server (via lambda functions). Warning: docs aren't great.
- [Codegen](https://github.com/dotansimha/graphql-code-generator): we use this script to automatically generate graphql hooks and types from those defined in our `schema.graphql` file in the `amplify` folder.
- [ColonyJS](https://github.com/JoinColony/colonyJS): ColonyJS is an abstraction layer on top of our network contracts, designed to make contract-interaction easier. Occasionally you might come into contact with it.
- [Ethers](https://docs.ethers.org/v5/): We use Ethers to handle on-chain interactions (e.g. calling a contract method or listening to an event). You'll probably only need this if you're interacting with the chain in the block ingestor or a lamda function, and there's no existing ColonyJS abstraction for what you're trying to do.

# Other resources

[Google Engineering Practices](https://github.com/google/eng-practices): A nice summary of best practices at Google, with some good advice to both sides of the PR process.
[Colony Dev Notion](https://www.notion.so/colony/Dev-70f1a0965f8b4070b0c749c5378f5f0c): Some more detailed guides are kept here, e.g. on our github workflows, or setting up a lamda function. But bear in mind these may not also be up to date.
