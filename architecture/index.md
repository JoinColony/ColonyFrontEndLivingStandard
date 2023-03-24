# Architecture overiew

## General overview

![image](https://github.com/JoinColony/ColonyFrontEndLivingStandard/assets/64402732/7084329d-8499-4be7-9448-964a1c349d7f)

A typical user interaction involves updating state, e.g. by minting some new tokens. In this case, a user is using the Colony UI to manipulate state held in the Colony Network smart contracts, and this process is managed in the client by Redux Saga. Sagas asynchronously handle side-effects, such as making calls to on-chain contract methods, and keep the UI updated as to the state of a given chain transaction.

Our smart contracts generally emit events after their methods execute successfully, which we're then able to listen to by a service we call the block ingestor.

At Colony, we're using chain state as the single source of truth, and a database as a faster, caching layer for reads from the client. To keep these in sync, we use the block ingestor to listen for events emitted by the chain, and then update database state accordingly.

All CRUD requests to the database go through our authentication server, which ensures access to our database is secure and all requests are correctly permissioned.

## Saga Overview

[Redux Saga](https://redux-saga.js.org/) allows us to manage asynchronous side effects based on changes to Redux state (such as dispatching actions). We use Sagas to manage the asynchronicity of interacting with the chain, and also to update the Redux state with the transaction status, so the UI can display the correct information to users ("Transaction Loading / Failed / Succeeded" etc).

Each saga listens to a specific action, and will fire when it's dispatched to the store. It then handles calling the on-chain contract method (e.g. `mintTokens`). Furthermore, it advances the transaction through its lifecycle ("Created", "Succeeded" / "Error" etc.), so the UI can respond appropriately.

We also persist a user's on-chain transactions in the database. To do this, we have a separate saga listening for all `TRANSACTION_CREATED` Redux actions, which then independently persists them in the database.

## Lambda Overview

Sometimes we want to directly interact with the database from the client, such as for updating state that we don't store on chain (e.g. a user profile).

Generally, and particularly for interactions that involve complex logic, we wrap the work up into an AWS Lambda function. This is so the client has less work to do, which should result in a smoother user experience. Lambdas are AWS's serverless function implementation, and are interacted with through the same graphql api as other database operations. The difference is that when triggered, a Node instance will spin up somewhere in AWS land and execute the corresponding function, instead of manipulating the database directly.

In addition to manipulating database state, we also use lambdas to asynchronously read chain state, and return it to the client, such as in the case of getting the latest motion state for a particular motion. We do this in cases where chain state cannot reliably/easily be cached, such as motion state or a user's reputation, and so instead of reading from the database, we go direct to the chain.

## Block ingestor overview

The block ingestor is a Node service that listens to events emitted by our smart contracts. We register listeners for events we care about, and whenever they're emitted, the ingestor will add them to a queue. It will then process events one-by-one in the order they were emitted. For each event, we have a handler that takes the event and then performs the corresponding database updates.

## Network overview

The Colony Network is our suite of smart contracts, written in Solidity. They contain the business logic of the Colony Network, and is where we store the state of the network (single source of truth).

## AWS overview

Aside from Lambda functions mentioned above, we're also using the following AWS services:

- Amplify: A hosted backend that provides GraphQL-based database access and cloud-based CI/CD tooling
- DynamoDB: the no-sql database underpinning Amplify
- AppSync: The GraphQL API Amplify uses to interact with the DB.

## Reputation Monitor (Dev) overview

The reputation monitor is a service used in local development that ensures reputation updates propagate correctly.

For info on how to use it, see: https://www.notion.so/colony/Reputation-in-development-c05b7a1f79fc4465a220c2806792e4a7

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
