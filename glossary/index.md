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
