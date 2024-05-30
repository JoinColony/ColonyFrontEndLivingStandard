# Code Quality

## Linting / Prettier

In the CDapp we have linting rules enabled. When commiting, a number of pre-commit checks will run to ensure your code is consistent with these.

We also use Prettier to ensure that our code is formatted neatly on the page. Ensure you have the relevant editor extension installed.

## GraphQL codegen

CDapp has a codegen tool set up that automatically generates TS types based on the Amplify schema and client-side queries, mutations and subscriptions. It can be run with `npm run codegen` and produces a `generated.ts` file that contains the following: 

### Amplify schema types

Interfaces and enums for all types defined in the `schema.graphql` (and additional types added during Amplify schema compilation). Note those interfaces will contain ALL possible fields of each object type. Apart from enums, entire object types imported from `~gql` should almost never be used (use fragments instead).

### Client-side operations and fragments from `graphql/**/*.graphql`

Interfaces for return types and variables, individual Apollo hooks for each operation as well as `Document` objects which can be useful when using Apollo client directly or refetching queries. Those return types will contain the exact subset of fields that the operations are fetching. If you expect to use the generated type of a particular operation or field, consider using fragments.

Each fragment will also generate a corresponding interface. **Please make sure to re-export all fragments from `graphql/types.ts` to avoid confusion between fragments and entire object types from schema**

## GraphQL best practices
- Use fragments to generate the types you need in the CDapp. Remember to re-export fragments from `graphql/types.ts`.
- Consider the query size. The more fields you add, the longer it will take to fetch - maybe not all fields are needed at the same time and some could be fetched separately? This is especially true for relationship fields (linking to other models) or fields resolved by lambdas.
- Relationship fields should be nullable unless you're certain the related object will exist. Otherwise, the entire operation will fail.
- In the schema, the convention is to use the word "ID" whenever it means a unique database ID of a model vs. "native ID" when it refers to an on-chain ID (which might not be unique).

## Refactoring

Set aside time / issues for refactoring… but not too early (nor too late). Generally when you start to notice copy/pasted code or that files are becoming too long, it's a good sign that you could start refactoring. How long is too long? Anything over 100-150 lines is a good estimate.

## Types

We generally try to be as specific as possible with our types. That means avoiding `any` and type-casting.

Whenever we create a component with props, we always create an `interface` to describe the shape of the props object. If these props are being passed through multiple components, please reuse the interfaces higher up either by importing them as is, or by extending them in the child component.

### Defining types in components

If you're defining a type inside of a component file, ensure you define it at the top of the file and outside of the component.

## React best practices

We generally follow guidance from the [new react docs](https://react.dev/) when assessing react-specific best practices.

Answers to the following questions and more can be found therein:

- When to use useMemo/useCallback (hint: not unless you have to)
- When (and when not!) to use useEffect
- How to correctly implement useContext

If a question can't be resolved by the react docs (or other authoritative source), then we default to sound programming practices (DRY/SOLID etc.). Finally, if neither of the previous stages produce a definitive answer, the preference of the person committing takes precedence.

With that said, here are some other things we like to see.

### Using useContext

We generally prefer to use `useContext` than prop drill to pass state down multiple levels of the component hierarchy. We prefer this pattern for the majority of data acquisition inside components.

If you see a set of components not following this pattern, consider refactoring the data flow so data can be accessed via a context hook.

With that said, it's not perfect for every use case. For example:
-> State that's needed in multiple parts of the app. If it doesn't belong in the `AppContext` provider, consider using `redux`.
-> State that should persist across sessions: use local storage or the db
-> When introducing an additional context provider is awkward/impractical or clutters the existing set of providers. In cases such as this, prop drilling is acceptable. Exercise judgement!

### Passing props

We prefer to pass props explicitly:

```jsx
const name = "Tim"
const age = 100
<MyComponent name={name} age={age} />
```

As opposed to

```jsx
<MyComponent {...{ name, age }} />
```

### Boolean logic

Sometimes we render components conditionally:

```jsx
const itsMyBirthday = true;

{
  itsMyBirthday && <Card />;
}
```

What we don't like to see is the following chaining of booleans:

```jsx
const todayIsSunday = true;
const itsMyBirthday = true;
const monthIsMarch = true;

{
  todayIsSunday && itsMyBirthday && monthIsMarch && <Card />;
}
```

Wrap up your booleans into a single boolean:

```jsx
const displayCard = todayIsSunday && itsMyBirthday && monthIsMarch;

{
  displayCard && <Card />;
}
```

Much nicer.

### Object destructuring

When destructuring an object that may be undefined, employ the following syntax:

```jsx
const { nativeToken } = colony || {};
```

### Event handler naming

To make it easier to distinguish between event handlers defined within the component and ones passed down as props, the former should be prefixed with `handle`, and the latter with `on`, e.g.:

```jsx
const handleClick = () => {};

<Button onClick={handleClick} />
```

## Javascript best practices

### Explicitly scoping `if` statements

Prefer brackets when writing `if` statements

```js
// Yes
if (true) {
  doSomething();
}

// No
if (true) doSomething();
```

### Type coercion

Prefer coercion via the constructor:

```js
const a = "1";
const b = 0;

// Yes
const c = Number(a);
const d = String(b);
const e = Boolean(b);
```

But be sure to leave off the `new` keyword in these cases.

`!!` for boolean coercion and template literals for string coercion is also fine:

```js
const num = 5;
const c = somethingThatMightBeFalsy;

// Yes
const sentence = `I can count up to ${num} on one hand.`;
const d = !!c;
```

Avoid other coercion methods, such as:

```js
// No
const c = +a;
const d = Number(0).toString();
```

## Forms best practices

We use forms all the time to take user input and do something with it (usually pass it on to a saga). All new forms should be created using the dedicated `react-hook-form` components. If you see any legacy code using `formik`, please open an issue and refactor.

In general, we aim to:

### Validation

Handle all validation in the validation schema. There’s generally no need for “custom” errors, since the validation schema can be generated dynamically, either by defining it inside the component’s scope or wrapping it in a function that is called with the required data as arguments.

We use the (yup)[https://github.com/jquense/yup] validation library for our validation needs.

### Size

Ensure forms follow all the ordinary component rules regarding file size and component encapsulation. No single part should be more than 100-150 lines.

### Managing complexity

As forms grow they have a tendency to become complex, unwieldy, and fragile.

Bear in mind that there is usually always something you can do better if you feel like your form is getting out of hand.

Signs you have an unruly form:

1. You're using multipe `useEffects` to keep state in sync across multiple components at different nesting levels. You find it hard to reason about the values of variables because of this.

2. Small changes to one part of the form break other parts unexpectedly. Could the form be redesigned to make the data flow more explicit and easier to reason about?

## i18n

We use the [react-intl](https://formatjs.io/docs/react-intl/) library for localization.

### Where to put localized strings

We try to keep all strings used on a specific page or a component within that component's scope.

Define an object called `MSG` with the `defineMessages` helper from `react-intl` and prefix the messages with the component's `displayName` if applicable, otherwise prefix it with something that semantically makes sense.

Then format those values with the `formatMessage` function from `react-intl`.

React component example:

```jsx
import { FC } from 'react';
import { defineMessages, formatMessage } from 'react-intl';

const displayName = 'MyComponent.InputWidget';

const MSG = defineMessages({
  title: {
      id: `${displayName}.title`,
      defaultMessage: 'Lorem ipsum',
    },
    inputLabel: {
        id: `${displayName}.inputLabel`,
        defaultMessage: 'Please input something',
    }.
});

const InputWidget: FC = () => {
  return (
    <div>
        <h1>{formatMessage(MSG.title)}</h1>
        <label>{formatMessage(MSG.inputLabel)}</label>
        <input />
    </div>
  );
}

InputWidget.displayName = displayName;

export default InputWidget;
```

A helper file example:
```jsx
import { defineMessages, formatMessage } from 'react-intl';
    
const MSG = defineMessages({
    aboutLink: {
        id: 'FooterLinks.about',
        defaultMessage: 'About',
    },
    tokenLink: {
        id: 'FooterLinks.token',
        defaultMessage: 'CLNY',
    },
});
    
const footerLinks = [{
    text: formatMessage(MSG.aboutLink),
    url: 'https://colony.io/about-us',
}, {
    text: formatMessage(MSG.tokenLink),
    url: 'https://colony.io/clny',
}];
```

## Further Reading

- (Google's Typescript style guide)[https://google.github.io/styleguide/tsguide.html]: An excellent explanation of the style decisions adopted in Google's engineering teams. We don't implement everything in here, but it's nevertheless an informative read.
