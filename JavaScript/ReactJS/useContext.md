# useContext

`useContext` is a React Hook that lets you read and subscribe to context from your component.

```JavaScript
const value = useContext(SomeContext)
```

Call useContext at the top level of your component to read and subscribe to context.

```JavaScript
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...

```

### Parameters

`SomeContext`: The context that you’ve previously created with `createContext`. The context itself does not hold the information, it only represents the kind of information you can provide or read from components.

### Returns

`useContext` returns the context value for the calling component. It is determined as the value passed to the closest SomeContext.Provider above the calling component in the tree. If there is no such provider, then the returned value will be the defaultValue you have passed to `createContext` for that context. The returned value is always up-to-date. React automatically re-renders components that read some context if it changes.

### Caveats

- `useContext()` call in a component is not affected by providers returned from the same component. The corresponding `<Context.Provider>` needs to be above the component doing the `useContext()` call.

- React <strong>automatically re-renders</strong> all the children that use a particular context starting from the provider that receives a different value. The previous and the next values are compared with the `Object.is` comparison. Skipping re-renders with `memo` does not prevent the children receiving fresh context values.

- If your build system produces duplicates modules in the output (which can happen with symlinks), this can break context. Passing something via context only works if 'SomeContext' that you use to provide context and 'SomeContext' that you use to read it are exactly the same object, as determined by a '===' comparison.

## Usage

### Passing data deeply into the tree

Call useContext at the top level of your component to read and subscribe to context.

```JavaScript
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...

```

`useContext` returns the context value for the context you passed. To determine the context value, React searches the component tree and finds the closest context provider above for that particular context.

To pass context to a Button, wrap it or one of its parent components into the corresponding context provider:

```JavaScript
function MyPage() {
return (
  <ThemeContext.Provider value="dark">
    <Form />
  </ThemeContext.Provider>
);
}

function Form() {
// ... renders buttons inside ...
}
```

It doesn’t matter how many layers of components there are between the provider and the Button. When a Button anywhere inside of Form calls useContext(ThemeContext), it will receive "dark" as the value.

```JavaScript
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  )
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```
