# React Guide

A minimalist, actionable React pocket guide covering core concepts, patterns, and code snippets.

## Index

- [Bootstrap](#bootstrap)
- [Functional Components](#functional-components)
- [Props](#props)
  - [Data Props](#data-props)
  - [Children Prop](#children-prop)
  - [Event Props (Callbacks)](#event-props-callbacks)
- [Fragments](#fragments)
- [Memoization](#memoization)
- [Forward Ref](#forward-ref)
- [Portals](#portals)
- [Suspense](#suspense)
- [Hooks](#hooks)
  - [Summary](#summary)
  - [State](#state)
    - [useState](#usestate)
    - [useReducer](#usereducer)
  - [Context](#context)
    - [useContext](#usecontext)
    - [use](#use)
  - [Effect](#effect)
    - [useEffect](#useeffect)
    - [useLayoutEffect](#uselayouteffect)
    - [useInsertionEffect](#useinsertioneffect)
  - [Ref](#ref)
    - [useRef](#useref)
    - [useImperativeHandle](#useimperativehandle)
  - [Performance](#performance)
    - [useMemo](#usememo)
    - [useCallback](#usecallback)
  - [Concurrent](#concurrent)
    - [useTransition](#usetransition)
    - [useDeferredValue](#usedeferredvalue)
    - [useOptimistic](#useoptimistic)
  - [Form](#form)
    - [useActionState](#useactionstate)
    - [useFormStatus](#useformstatus)
  - [Utility](#utility)
    - [useId](#useid)
    - [useSyncExternalStore](#usesyncexternalstore)
    - [useDebugValue](#usedebugvalue)
  - [Custom Hooks](#custom-hooks)

## Bootstrap

React applications require a single entry point file (typically `index.js` or `main.jsx`) to bind the root React component to an existing HTML DOM element (usually `<div id="root"></div>`).

> **Note:** Always wrap your root component with `<StrictMode>` during development. It activates additional checks, double-invokes render functions to catch side-effects, and highlights deprecated APIs without affecting the production build.

```jsx
import { createRoot } from 'react-dom/client';
import { StrictMode } from 'react';

createRoot(document.getElementById('root')).render(
    <StrictMode>
        <!-- Root Component Here -->
    </StrictMode>
);
```

## Functional Components

Functional components are plain JavaScript functions that return React elements (JSX) describing the UI. They serve as the primary building blocks of modern React applications.

On every re-render, the entire function executes top-to-bottom. React provides Hooks to attach behaviors, hold state, and control when specific logic needs to be processed across executions.

```jsx
const Component = () => {
    return (
        <h1>Hello World!</h1>
    );
};
```

## Props

Props (short for "properties") are read-only inputs passed from a parent component to a child component. They allow components to be dynamic, reusable, and form a top-down (unidirectional) data flow.

On every re-render, a child component receives updated props from its parent. **Props are immutable: a component must never modify its own props directly**. By default, when a parent component re-renders, all of its child components re-render as well, even if their props have not changed.

Props fall into three main categories based on their usage:

### Data Props

Values passed top-down (**parent to child**) for the child to render or consume. They carry primitive values, objects, arrays, or configurations.

```jsx
// Child component receiving data props
const Title = ({ content }) => {
    return (
        <h1>{content}</h1>
    );
};

// Parent passing data props
const Page = () => {
    return (
        <Title content="Newsletter" />
    );
};
```

### Children Prop

A built-in prop (`children`) automatically provided by React to pass JSX elements or components directly inside another component's opening and closing tags. Used for component composition and wrapper layouts.

```jsx
// Child component wrapping nested content
const Section = ({ children }) => {
    return (
        <section>{children}</section>
    );
};

// Parent passing nested elements via children
const Page = () => {
    return (
        <Section>
            <h1>Newsletter</h1>
            <p>Lorem ipsum dolor sit amet, consectetur adipiscing elit.</p>
        </Section>
    );
};
```

### Event Props (Callbacks)

Functions passed down to a child component, allowing it to communicate back to the parent when an action or event occurs. By convention, event props start with `on` (e.g., `onClick`, `onChange`), while the handler functions in the parent start with `handle` (e.g., `handleClick`, `handleChange`).

```jsx
// Child component triggering the callback
const Button = ({ onClick }) => {
    return (
        <button onClick={onClick}>Click!</button>
    );
};

// Parent component passing the event handler function
const Page = () => {
    const handleClick = (e) => {
        console.log(e.target.innerHTML); // Output: Click!
    };

    return (
        <Button onClick={handleClick} />
    );
};
```

## Fragments

Fragments `<></>` allow you to group a list of children elements without adding extra nodes (like a `<div>`) to the DOM. This keeps the DOM tree clean and avoids breaking layouts like tables, lists, etc.

> **Note:** When mapping over an array in a loop, you must use the explicit `<React.Fragment key="{id}">` syntax instead of `<></>`, as the short syntax does not support keys or attributes.

```jsx
// Child component returning multiple list items without a wrapping div
const Items = () => {
    return (
        <>
            <li>Item 1</li>
            <li>Item 2</li>
            <li>Item 3</li>
        </>
    );
};

// Parent component rendering a valid HTML list structure
const List = () => {
    return (
        <ul>
            <Items />
        </ul>
    );
};
```

## Memoization

In React, component-level memoization is achieved using `React.memo`, a Higher-Order Component (HOC) that acts as a **component decorator**. It wraps a functional component and **skips re-rendering** if its props are shallowly equal to the previous ones.

> **Note:** Memoization carries memory overhead for comparison. Use it intentionally, typically when profiling identifies actual performance bottlenecks or when components re-render frequently with identical props.

```jsx
import { memo } from 'react';

// Wraps the component to skip re-renders if props don't change
const CustomCard = memo(({ title }) => {
    return (
        <div className="card">
            <h2>{title}</h2>
        </div>
    );
});
```

## Forward Ref

By default, React components do not expose their underlying DOM nodes to parent components because `ref` is not passed like a standard prop. `React.forwardRef` allows a functional component to receive a `ref` from its parent and forward (pass) it down to a child DOM element or component.

```jsx
import { forwardRef, useRef, useEffect } from 'react';

// Child component accepting ref as a second argument
const Input = forwardRef(({ label }, ref) => {
    return (
        <>
            <label>{label}</label>
            <input ref={ref} />
        </>
    );
});

// Parent component creating and passing the ref
const Form = () => {
    const inputRef = useRef(null);

    useEffect(() => {
        inputRef.current?.focus();
    }, []);

    return (
        <form>
            <Input ref={inputRef} label="Name" />
        </form>
    );
}
```

## Portals

Portals provide a way to render a child component into a DOM node that exists outside the parent component's DOM hierarchy (i.e. `<div id="portal-root"></div>`). This is particularly useful for UI overlays like modals, tooltips, slide-overs, or notifications that need to break out of parent container styles (`overflow: hidden` or `z-index` stacking contexts).

> **Note**: Despite being rendered into a different location in the DOM tree, portals behave like standard React children in every other way, preserving event bubbling and context access.

```jsx
import { createPortal } from 'react-dom';

// The second argument defines where the JSX should be rendered in the DOM
const Modal = ({ message }) => {
    return createPortal(
        <div className="modal">{message}</div>,
        document.getElementById('portal-root')
    );
};
```

## Suspense

`<Suspense>` lets you display a fallback UI (like a loading indicator or skeleton) while its child components are waiting for an asynchronous operation to complete (such as lazy-loading code or fetching data).

> **Note:** Works seamlessly with `React.lazy()` for code-splitting and the `use` hook for inline promise resolution.

```jsx
import { lazy, Suspense, use } from 'react';

// Scenario A: Lazy load a component on demand
const HeavyComponent = lazy(() => import('./HeavyComponent'));
const Page = () => {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <HeavyComponent />
        </Suspense>
    );
};

// Scenario B: Suspend rendering until 'dataPromise' resolves
const Chart = ({ dataPromise }) => {
    const data = use(dataPromise);
    return <div className="chart">{data.title}</div>;
};
const Container = ({ dataPromise }) => {
    return (
        <Suspense fallback={<div>Loading...</div>}>
            <Chart dataPromise={dataPromise} />
        </Suspense>
    );
};
```

## Hooks

### Summary

| Group | Hook | Description | Used When |
|---|---|---|---|
| **State** | `useState` | Manages local component state. | Handling simple, local values like toggles or text inputs. |
| **State** | `useReducer` | Manages state via actions and a reducer function. | Handling complex state logic or multiple sub-values. |
| **Context** | `useContext` | Reads a React context. | Accessing global/shared data (themes, auth) without prop drilling. |
| **Context** | `use` | Reads the value of a Promise or Context conditionally. | Fetching asynchronous data inside components or conditional Context reads. |
| **Ref** | `useRef` | Holds a mutable value or DOM element reference across renders. | Accessing DOM nodes directly or storing persistent values without re-rendering. |
| **Ref** | `useImperativeHandle` | Customizes the ref exposed by a child component to its parent. | Exposing imperative methods (e.g., `focus()`, `scroll()`) to parent components. |
| **Effect** | `useEffect` | Runs side effects after rendering. | Fetching data, setting subscriptions, or updating DOM after paint. |
| **Effect** | `useLayoutEffect` | Runs synchronously before the browser repaints the screen. | Measuring DOM elements or preventing flickering UI layouts. |
| **Effect** | `useInsertionEffect` | Runs before DOM mutations happen. | Injecting dynamic styles (e.g., CSS-in-JS libraries) before DOM reads. |
| **Performance** | `useMemo` | Caches the result of a calculation between renders. | Optimizing expensive computational functions. |
| **Performance** | `useCallback` | Caches a function definition between renders. | Preventing unnecessary re-renders of child components receiving callbacks. |
| **Concurrent** | `useTransition` | Marks state updates as non-blocking transitions. | Keeping UI responsive during heavy visual updates or tab switches. |
| **Concurrent** | `useDeferredValue` | Defers updating a non-critical part of the UI. | Lag-free user input response while waiting for heavy re-renders. |
| **Concurrent** | `useOptimistic` | Optimistically updates UI while async actions run. | Providing instant UI feedback before server requests finish. |
| **Form** | `useActionState` | Updates state based on the result of a form action. | Handling form submissions, pending statuses, and action responses. |
| **Form** | `useFormStatus` | Provides status info of the nearest parent form. | Building reusable submit buttons and inline form loading indicators. |
| **Utility** | `useId` | Generates unique IDs stable across Client and SSR. | Linking accessibility attributes (`htmlFor`, `aria-describedby`). |
| **Utility** | `useSyncExternalStore` | Reads and subscribes to external data stores. | Synchronizing state with external libraries, `localStorage`, or browser APIs. |
| **Utility** | `useDebugValue` | Adds custom labels in React DevTools. | Inspecting internals and state of custom Hooks. |

### State

#### `useState`

Declares a reactive state variable. When updated, React **re-renders** the component. State must be treated as immutable. Always return a new object or array reference (e.g., using the spread operator) so React can detect changes.

```jsx
import { useState } from 'react';

const Component = () => {
    // Initialization
    const [count, setCount] = useState(0);

    // Lazy initialization (synchronous)
    const [count, setCount] = useState(() => calculateCount());

    // Direct update
    setCount(5);

    // Update based on previous state
    setCount(prev => prev + 1);

    // Object example
    const [user, setUser] = useState({ id: 1, name: 'John' });
    setUser(prev => ({ ...prev, name: 'Jane' }));

    // Array example
    const [items, setItems] = useState(['Computer', 'Keyboard']);
    setItems(prev => [...prev, 'Screen']);
}
```

#### `useReducer`

An alternative to useState for managing **complex state logic** or multiple sub-values through dispatched actions. State must be treated as immutable, and the reducer function must be pure `(state, action) => newState`.

> **Note:** Unlike `useState`, `useReducer` receives the 2nd argument (`arg`) and passes it directly to the 3rd argument function `init(arg)`. This allows `init` to be declared outside the component as a pure, reusable function. 

```jsx
import { useReducer } from 'react';

// Reducer function (outside component)
function reducer(state, action) {
    switch (action.type) {
        case 'increment':
            return { ...state, count: state.count + 1 };
        case 'set_name':
            return { ...state, name: action.payload };
        default:
            return state;
    }
}

const Component = () => {
    // Initialization
    const [state, dispatch] = useReducer(reducer, { 
        count: 0, 
        name: 'John' 
    });

    // Lazy initialization (synchronous)
    const [state, dispatch] = useReducer(reducer, 0, arg => ({ 
        count: arg, 
        name: 'John' 
    }));

    // Dispatching actions
    dispatch({ type: 'increment' });
    dispatch({ type: 'set_name', payload: 'Jane' });
}
```

### Context

#### `useContext`

Allows components to subscribe and read data from a nearest matching `<Context>` without passing props manually down through every level of the component tree (prop drilling). The `use` hook can also be used to read contexts.

> **Note:** Whenever a Context `value` changes reference, **all** components consuming that Context will re-render. To prevent unnecessary re-renders across the tree, split unrelated data into smaller, domain-specific Contexts (e.g., `AuthContext`, `ThemeContext`) rather than using a single monolithic global object.

```jsx
import { createContext, useContext } from 'react';

// Create context
export const ThemeContext = createContext('light');

// Provide context
const Parent = () => {
    return (
        <ThemeContext value="dark">
            <MainContent />
        </ThemeContext>
    );
}

// Consume context
const Child = () => {
    const theme = useContext(ThemeContext);
}
```

#### `use`

Passes a promise or context resource directly inside render. It integrates with `<Suspense>` to suspend component rendering until promises resolve, and unlike standard Hooks, it can be called conditionally (inside `if` statements) and within loops.

```jsx
import { use } from 'react';

// Scenario A: Reading a promise (suspends component until resolved)
const Component1 = ({ commentsPromise }) => {
    const comments = use(commentsPromise);
};

// Scenario B: Reading context
const Component2 = () => {
    const theme = use(ThemeContext);
};

// Allowed inside conditional blocks or loops
const Component3 = ({ commentsPromise }) => {
    if (commentsPromise) {
        const comments = use(commentsPromise);
    }
};
```

### Effect

#### `useEffect`

Synchronizes a component with an external system (e.g., browser APIs, networks, timers). It runs asynchronously **after the component has rendered** to the screen.

```jsx
import { useEffect } from 'react';

const Component = () => {
    // Runs on every render
    useEffect(() => {
        console.log('Component rendered or updated');
    });

    // Runs only once on mount
    useEffect(() => {
        fetch('/api').then(...);
    }, []);

    // Runs on mount and when dependencies change
    useEffect(() => {
        fetch('/api').then(...);
    }, [dep1, dep2, ...]);

    // Returns a cleanup function
    useEffect(() => {
        const socket = ...
        return () => socket.close();
    }, []);
}
```

This table maps traditional Class Component lifecycle methods to their modern Functional Component equivalents using useEffect (and React.memo for update checks).

|Class Component|Functional Component|
|---|---|
|`componentDidMount`|`useEffect(() => { ... }, [])`|
|`componentDidUpdate`|`useEffect(() => { ... }, [dep1, dep2])`|
|`componentWillUnmount`|`useEffect(() => { return () => { ... } }, [])`|
|`shouldComponentUpdate`|`React.memo(Component, arePropsEqual)`|

#### `useLayoutEffect`

A version of `useEffect` that fires synchronously **before the browser repaints the screen**. Use it to read layout from the DOM and synchronously re-render before the user sees the visual update.

> **Note:** Has the exact same signature as `useEffect` (accepts a dependency array and returns a cleanup function).

```jsx
import { useRef, useLayoutEffect } from 'react';

const Component = () => {
    const boxRef = useRef(null);

    useLayoutEffect(() => {
        const width = boxRef.current?.offsetWidth;
        // ...
    }, []);
};
```

#### `useInsertionEffect`

A version of `useEffect` that fires **synchronously before any DOM mutations occur**. It is designed specifically for CSS-in-JS libraries to inject `<style>` tags into the DOM before layout is read or elements are painted.

```jsx
import { useInsertionEffect } from 'react';

const Component = () => {
    useInsertionEffect(() => {
        const style = document.createElement('style');
        // ...
    }, []);
};
```

### Ref

#### `useRef`

Returns a mutable ref object whose `.current` property is initialized to the passed argument. It persists across renders and updating it **does not trigger a re-render**.

```jsx
import { useRef } from 'react';

// Scenario A: Store mutable data (does not trigger re-render)
const Component1 = () => {
    const countRef = useRef(0);
    return (
        <button onClick={() => { countRef.current += 1 }}>Increment</button>
    )
};

// Scenario B: Store reference to an element
const Component2 = () => {
    const inputRef = useRef(null);
    useEffect(() => { inputRef.current?.focus(); }, []);
    return (
        <input ref={inputRef} />
    )
};
```

#### `useImperativeHandle`

Customizes the handle exposed as a ref to a parent component. It allows a child component to expose specific imperative methods rather than the raw DOM node.

```jsx
import { forwardRef, useRef, useEffect, useImperativeHandle } from 'react';

// Custom element with forwardRef
const CustomInput = forwardRef((props, ref) => {
    const inputRef = useRef(null);
    useImperativeHandle(ref, () => ({
        customFocus: () => inputRef.current?.focus()
    }), []);
    return (
        <input ref={inputRef} />
    );
});

// Only the exposed properties (customFocus) will be available on parent ref
const Form = () => {
    const inputRef = useRef(null);
    useEffect(() => {
        inputRef.current?.customFocus(); // OK
        inputRef.current?.focus(); // Error: undefined
    }, []);
    return (
        <CustomInput ref={inputRef} />
    );
};
```

### Performance

#### `useMemo`

Caches the result of a calculation between renders. It will only recompute the memoized value when one of the dependencies has changed.

> **Note:** For **lightweight operations**, simply compute the value directly in the function component.

```jsx
import { useMemo } from 'react';

const Component = () => {
    // Lightweight operation (performed for every render)
    const sortedProducts1 = lightSort(products);

    // Heavy operation (performed only when dependencies change)
    const sortedProducts2 = useMemo(() => heavySort(products), [products]);
};
```

#### `useCallback`

By default, React re-creates function definitions on every render. `useCallback` caches a function definition between renders, returning the exact same function instance as long as its dependencies do not change.

> **Note:** Typically used to pass stable function references to `React.memo` components or as `useEffect` dependencies.

```jsx
import { useEffect, useCallback } from 'react';

const Component = () => {
    // Preserved function reference across re-renders
    const fetchData = useCallback(() => {
        fetch('/api').then(...);
    }, []);

    // Prevents infinite re-triggering since fetchData maintains a stable reference
    useEffect(() => {
        fetchData();
    }, [fetchData]);
};
```

### Concurrent

#### `useTransition`

Allows you to mark a **state update** as a non-blocking transition. React will prioritize urgent updates (like typing or clicking) and let background renders yield to user input.

```jsx
import { useState, useTransition } from 'react';

const Component = () => {
    const [tab, setTab] = useState('tab 1');
    const [isPending, startTransition] = useTransition();

    const nextTab = () => {
        // Marks state update as low-priority
        startTransition(() => {
            setTab('tab 2');
        });
    };

    return (
        <>
            <button onClick={nextTab}>Next Tab</button>
            {isPending && <span>Loading...</span>}
        </>
    );
};
```

#### `useDeferredValue`

Defers updating a part of the UI. During rapid updates (e.g., typing `a` -> `ab` -> `abc`), it keeps the deferred value on its previous state, skipping intermediate re-renders so the user input remains responsive. Once input pauses, React renders the UI with the latest value.

```jsx
import { useState, useDeferredValue } from 'react';

const Component = () => {
    const [query, setQuery] = useState('');

    // Lagging copy that updates in background when main thread is clear
    const deferredQuery = useDeferredValue(query);

    return (
        <>
            <input value={query} onChange={(e) => setQuery(e.target.value)} />
            <HeavyList query={deferredQuery} />
        </>
    );
};
```

#### `useOptimistic`

Allows you to optimistically update the UI while an async action is in progress. It immediately displays a temporary expected state to the user and automatically reverts if the operation fails or completes.

```jsx
import { useState, useOptimistic } from 'react';

const Component = () => {
    const [messages, setMessages] = useState([]);

    // Intermediate state updated before server responds
    const [optimisticMessages, addOptimisticMessage] = useOptimistic(
        messages,
        (state, newMessage) => [
            ...state,
            { message: newMessage, sending: true }
        ]
    );

    const action = async (formData) => {
        const message = formData.get('message');
        addOptimisticMessage(message); // Immediately update UI
        await save(message); // Perform actual server request
        setMessages(prev => [...prev, message]);
    };

    return (
        <form action={action}>
            <Messages messages={optimisticMessages} />
            <input name="message" />
            <button type="submit">Send</button>
        </form>
    );
};
```

### Form

#### `useActionState`

Allows to update state based on the result of a form action. It receives an **async or sync action function** and an initial state, returning the current state, a wrapped action handler, and a pending status indicator.

```jsx
import { useActionState } from 'react';

// Action receives previous state and form data
const action = async (previousState, formData) => {
    const name = formData.get('name');
    if (!name) return 'Name is required';
    await save(name);
    return 'Saved successfully!';
};

const Component = () => {
    const [message, formAction, isPending] = useActionState(action, '');

    return (
        <form action={formAction}>
            <input name="name" type="text" />
            <button type="submit" disabled={isPending}>Save</button>
            {message && <p>{message}</p>}
        </form>
    );
};
```

#### `useFormStatus`

Provides status information about the nearest parent `<form>` submission (such as pending, data, method, and action). It must be called from a child component rendered inside the `<form>`, eliminating the need to pass state or props down for loading indicators.

```jsx
import { useFormStatus } from 'react';

// Simple form component
const Form = () => {
    return (
        <form action="...">
            <input name="..." />
            <SubmitButton />
        </form>
    );
};

// The child component can access form metadata from the nearest form
const SubmitButton = () => {
    const { pending, data, method, action } = useFormStatus();
    
    return (
        <button type="submit" disabled={pending}>
            {pending ? 'Submitting...' : 'Submit'}
        </button>
    );
};
```

### Utility

#### `useId`

Generates unique IDs that are stable across server and client renders. It avoids hydration mismatches in SSR applications and is primarily used for linking accessibility attributes (`aria-describedby`, `htmlFor`, etc.) to HTML elements.

```jsx
import { useId } from 'react';

// The generated ID is always deterministic and safe
const Component = () => {
    const id = useId();

    return (
        <>
            <label htmlFor={id}>Name</label>
            <input id={id} />
        </>
    );
};
```

#### `useSyncExternalStore`

Subscribes to an external data store (such as state libraries or browser APIs like `localStorage`) and reads its current value. It guarantees synchronous, collision-free updates across concurrent rendering features in React. Requires two functions: `subscribe` (registers a callback when the store changes) and `getSnapshot` (returns the current value from the store).
 
```jsx
import { useSyncExternalStore } from 'react';

// Registers a callback when the store changes
const subscribe = (callback) => {
    window.addEventListener('storage', callback);
    return () => window.removeEventListener('storage', callback);
}

// Returns the current value from the store
const getSnapshot = () => localStorage.getItem('theme') ?? 'light';

// Use the functions above to define an external store
const Component = () => {
    const theme = useSyncExternalStore(subscribe, getSnapshot);
};
```

#### `useDebugValue`

Adds a custom label to custom Hooks in React Developer Tools to simplify debugging. It should only be called inside custom Hooks and has zero performance overhead in production.

```jsx
import { useState, useDebugValue } from 'react';

const useAuth = () => {
    const [isAuthenticated, setAuthenticated] = useState(false);

    // Displays formatted label next to useAuth in React DevTools
    useDebugValue(isAuthenticated ? 'Authenticated' : 'Not Authenticated');

    return [isAuthenticated, setAuthenticated];
};
```

### Custom Hooks

JavaScript functions starting with `use` that encapsulate and share reusable stateful logic between components. **Whenever you need to reuse logic that depends on React Hooks, you must create a custom Hook**. Every call creates a completely isolated state instance for that component.

> **Note:** Must follow the `use` prefix naming rule so React's linter can enforce the Rules of Hooks inside the function.

```jsx
import { useState, useEffect } from 'react';

// Custom hook
const useDocumentTitle = (initialValue) => {
    const [title, setTitle] = useState(initialValue);

    useEffect(() => {
        document.title = title;
    }, [title]);

    return [title, setTitle];
};

// Consumer
const Component = () => {
    const [title, setTitle] = useDocumentTitle('Home');

    return (
        <button onClick={() => setTitle('Contact')}>Contact Page</button>
    );
};
```