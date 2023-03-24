# Code Quality

## Linting / Prettier

In the CDapp we have linting rules enabled. When commiting, a number of pre-commit checks will run to ensure your code is consistent with these.

Many of us also use Prettier to ensure that our code is formatted neatly on the page

## Refactoring

Set aside time / issues for refactoring… but not too early (nor too late). Generally when you start to notice copy/pasted code or that files are becoming too long, it's a good sign that you could start refactoring.

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

We generally prefer to use `useContext` than prop drill to pass state down multiple levels of the component hierarchy.

Some exceptions to this might include:
-> State that's needed in multiple parts of the app. If it doesn't belong in the `AppContext` provider, consider using redux.
-> State that should persist across sessions: use local storage or the db
-> When introducing an additional context provider is awkward/impractical or clutters the existing set of providers. In cases such as this, prop drilling is acceptable.

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

### Type casting

Prefer casting via the constructor when casting numbers or strings:

```js
const a = "1";
const b = 0;

// Yes
const c = Number(a);
const d = String(b);
// No
const c = +a;
const d = Number(0).toString();
const d = `${b}`;
```

But via the `!!` syntax when casting booleans:

```js
const c = 0;
// Yes
const d = !!0;
// No
const d = Boolean(0);
```

## Forms best practices

We use forms all the time to take user input and do something with it (usually pass it on to a saga).

In general, we aim to:

### Validation

Handle all validation in the validation schema. There’s generally no need for “custom” errors, since the validation schema can be generated dynamically, either by defining it inside the component’s scope or wrapping it in a function that is called with the required data as arguments.

### Size

Ensure forms follow all the ordinary component rules regarding file size and component encapsulation. No single part should be more than 100-150 lines.

### Managing complexity

As forms grow they have a tendency to become complex, unwieldy, and fragile.

Bear in mind that there is usually always something you can do better if you feel like your form is getting out of hand.

Signs you have an unruly form:

1. You're using multipe `useEffects` to keep state in sync across multiple components at different nesting levels. You find it hard to reason about the values of variables because of this.

2. Small changes to one part of the form break other parts unexpectedly. Could the form be redesigned to make the data flow more explicit and easier to reason about?
