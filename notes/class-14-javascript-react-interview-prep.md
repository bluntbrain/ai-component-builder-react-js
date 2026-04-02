# Class 14: JavaScript & React Interview Prep

## 30 Questions You Will Be Asked

**Part A -- JavaScript:** Event Loop, Closures, Hoisting, var/let/const, ==vs===, this, call/apply/bind, Prototypes, Promises, Event Delegation, Debounce/Throttle, map/forEach/reduce, Shallow vs Deep Copy, Spread/Rest, Destructuring, Currying

**Part B -- React:** How React Works, Virtual DOM, Reconciliation, Fiber, Keys, useState vs useReducer, useEffect, Controlled vs Uncontrolled, Context, Error Boundaries, Synthetic Events, Batching, Server Components, memo/useMemo/useCallback

**Part C -- React Native Under the Hood:** Architecture, Bridge vs JSI, Fabric, TurboModules

---

# Part A: JavaScript

---

### Q1. What is the Event Loop?

JavaScript is **single-threaded** -- it has one call stack and can do one thing at a time. The event loop is the mechanism that handles async operations.

```
  CALL STACK              WEB APIs              TASK QUEUES
  (executes code)         (browser handles)     (waiting to run)

  +----------+            +----------+          MICROTASK QUEUE (high priority)
  | func3()  |            | setTimeout|         +-----------------------------+
  | func2()  |            | fetch()   |         | Promise.then()              |
  | func1()  |            | DOM events|         | queueMicrotask()            |
  +----------+            +----------+          +-----------------------------+
       ^                       |
       |                       |                MACROTASK QUEUE (low priority)
       +----- event loop ------+                +-----------------------------+
              checks if                         | setTimeout callback         |
              stack is empty,                   | setInterval callback        |
              then moves next                   | DOM event callbacks         |
              task to stack                     +-----------------------------+

  ORDER: Call Stack -> ALL Microtasks -> ONE Macrotask -> ALL Microtasks -> ...
```

```javascript
console.log('1');                    // call stack (runs immediately)

setTimeout(() => console.log('2'), 0); // macrotask queue

Promise.resolve().then(() => console.log('3')); // microtask queue

console.log('4');                    // call stack (runs immediately)

// output: 1, 4, 3, 2
// why: stack first (1, 4), then microtasks (3), then macrotasks (2)
```

---

### Q2. What is a Closure?

A closure is a function that **remembers variables from its outer scope** even after the outer function has returned.

```javascript
function createCounter() {
  let count = 0; // this variable is "closed over"
  return {
    increment: () => ++count,
    getCount: () => count,
  };
}

const counter = createCounter();
counter.increment(); // 1
counter.increment(); // 2
counter.getCount();  // 2

// count is private -- no way to access it directly
// the returned functions "remember" it via closure
```

**Common interview follow-up -- the loop problem:**

```javascript
// bug: all callbacks print 3
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 3, 3, 3
}

// fix 1: use let (block-scoped, creates new binding per iteration)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // 0, 1, 2
}

// fix 2: closure with IIFE
for (var i = 0; i < 3; i++) {
  ((j) => setTimeout(() => console.log(j), 100))(i); // 0, 1, 2
}
```

---

### Q3. Explain Hoisting

JavaScript moves **declarations** (not assignments) to the top of their scope during compilation.

```javascript
// what you write:                    // what JS sees:
console.log(a);                       var a;           // declaration hoisted
var a = 5;                            console.log(a);  // undefined (not 5!)
                                      a = 5;

// let and const are hoisted too, but in the "Temporal Dead Zone" (TDZ)
console.log(b); // ReferenceError: Cannot access 'b' before initialization
let b = 10;

// functions are fully hoisted (both declaration and body)
greet(); // works! prints "hello"
function greet() { console.log('hello'); }

// function expressions are NOT fully hoisted
sayHi(); // TypeError: sayHi is not a function
var sayHi = function() { console.log('hi'); };
```

---

### Q4. var vs let vs const

| | `var` | `let` | `const` |
|---|---|---|---|
| **Scope** | Function | Block `{}` | Block `{}` |
| **Hoisted?** | Yes (as `undefined`) | Yes (in TDZ) | Yes (in TDZ) |
| **Re-declare?** | Yes | No | No |
| **Re-assign?** | Yes | Yes | No |
| **Use when** | Never (legacy) | Value will change | Value won't change |

```javascript
if (true) {
  var x = 1;   // leaks out of block
  let y = 2;   // stays inside block
  const z = 3; // stays inside block
}
console.log(x); // 1
console.log(y); // ReferenceError
console.log(z); // ReferenceError

// const with objects -- the REFERENCE is constant, not the content
const user = { name: 'Ishan' };
user.name = 'Rahul'; // allowed (mutating content)
user = {};           // TypeError (reassigning reference)
```

---

### Q5. == vs ===

`==` does **type coercion** (converts types before comparing). `===` does **strict comparison** (no conversion).

```javascript
// == (loose -- avoid this)
0 == ''          // true  (both coerce to 0)
0 == false       // true
'' == false      // true
null == undefined // true
'5' == 5         // true  (string converts to number)

// === (strict -- always use this)
0 === ''          // false (number vs string)
0 === false       // false (number vs boolean)
null === undefined // false (different types)
'5' === 5         // false

// rule: always use === unless you have a specific reason for ==
```

---

### Q6. What is `this`?

`this` depends on **how the function is called**, not where it's defined. Four rules:

```javascript
// Rule 1: DEFAULT -- global object (window in browser, undefined in strict mode)
function show() { console.log(this); }
show(); // window (or undefined in strict mode)

// Rule 2: IMPLICIT -- the object before the dot
const user = {
  name: 'Ishan',
  greet() { console.log(this.name); }
};
user.greet(); // 'Ishan' (this = user)

// Rule 3: EXPLICIT -- using call, apply, or bind
function greet() { console.log(this.name); }
greet.call({ name: 'Rahul' }); // 'Rahul'

// Rule 4: ARROW FUNCTIONS -- inherit `this` from parent scope (lexical)
const team = {
  name: 'Alpha',
  members: ['A', 'B'],
  show() {
    this.members.forEach((m) => {
      console.log(`${m} is in ${this.name}`); // arrow inherits `this` from show()
    });
  }
};
team.show(); // "A is in Alpha", "B is in Alpha"

// priority: new > explicit (call/apply/bind) > implicit (dot) > default
```

---

### Q7. call, apply, bind

All three set `this` explicitly. The difference is how they're invoked:

```javascript
function introduce(greeting, punctuation) {
  console.log(`${greeting}, I'm ${this.name}${punctuation}`);
}

const person = { name: 'Ishan' };

// call -- invokes immediately, args passed individually
introduce.call(person, 'Hello', '!');     // "Hello, I'm Ishan!"

// apply -- invokes immediately, args passed as array
introduce.apply(person, ['Hi', '.']);      // "Hi, I'm Ishan."

// bind -- returns NEW function with `this` bound, call it later
const boundFn = introduce.bind(person, 'Hey');
boundFn('?');                              // "Hey, I'm Ishan?"
```

**Memory trick:** **C**all = **C**omma separated. **A**pply = **A**rray. **B**ind = returns **B**ound function.

---

### Q8. What is Prototypal Inheritance?

Every JavaScript object has a hidden `[[Prototype]]` link to another object. When you access a property, JS looks up the **prototype chain**.

```
  PROTOTYPE CHAIN:

  dog --[[Prototype]]--> animal --[[Prototype]]--> Object.prototype --> null
   |                      |                         |
   name: 'Rex'            speak()                   toString()
   breed: 'Lab'           eat()                     hasOwnProperty()
```

```javascript
const animal = {
  speak() { console.log(`${this.name} makes a sound`); },
  eat() { console.log('eating'); }
};

const dog = Object.create(animal); // dog's prototype = animal
dog.name = 'Rex';
dog.breed = 'Lab';

dog.speak(); // "Rex makes a sound" (found on animal, `this` = dog)
dog.eat();   // "eating" (found on animal)
dog.toString(); // "[object Object]" (found on Object.prototype)

// check where property lives
dog.hasOwnProperty('name');  // true (own property)
dog.hasOwnProperty('speak'); // false (inherited from prototype)
```

---

### Q9. What are Promises?

A Promise represents a value that will be available in the future. Three states: **pending**, **fulfilled**, **rejected**.

```
  PROMISE LIFECYCLE:

  new Promise()
       |
    [PENDING]
      / \
     /   \
    v     v
[FULFILLED]  [REJECTED]
  .then()     .catch()
```

```javascript
// creating a promise
const fetchUser = (id) => new Promise((resolve, reject) => {
  setTimeout(() => {
    if (id > 0) resolve({ id, name: 'Ishan' });
    else reject(new Error('invalid id'));
  }, 1000);
});

// consuming with .then/.catch
fetchUser(1)
  .then(user => console.log(user.name))    // "Ishan"
  .catch(err => console.error(err.message));

// consuming with async/await (cleaner)
async function getUser() {
  try {
    const user = await fetchUser(1);
    console.log(user.name); // "Ishan"
  } catch (err) {
    console.error(err.message);
  }
}

// Promise.all -- wait for ALL to resolve (fails if ANY rejects)
const [users, posts] = await Promise.all([fetchUsers(), fetchPosts()]);

// Promise.allSettled -- wait for ALL to finish (never fails)
const results = await Promise.allSettled([fetchA(), fetchB()]);
// [{status: 'fulfilled', value: ...}, {status: 'rejected', reason: ...}]
```

---

### Q10. Event Bubbling, Capturing, and Delegation

```
  EVENT PROPAGATION PHASES:

  CAPTURING (top -> target)       TARGET        BUBBLING (target -> top)

       document                                      document
          |                                              ^
          v                                              |
        <body>                                        <body>
          |                                              ^
          v                                              |
        <div>                                         <div>
          |                                              ^
          v                                              |
        <button>  -------- CLICK HERE ---------->   <button>

  Phase 1: Capturing (rarely used)
  Phase 2: Target (the element you clicked)
  Phase 3: Bubbling (default -- this is what you normally handle)
```

```javascript
// event delegation -- one listener on parent handles all children
document.getElementById('list').addEventListener('click', (e) => {
  if (e.target.tagName === 'LI') {
    console.log('clicked:', e.target.textContent);
  }
});

// works for 10 items or 10,000 items -- one listener instead of thousands
// also handles dynamically added items automatically

// stop bubbling
button.addEventListener('click', (e) => {
  e.stopPropagation(); // parent handlers won't fire
});
```

---

### Q11. Debounce vs Throttle

```
  USER TYPING "hello" (5 keystrokes):

  Events:    h     e     l     l     o
  Time:    100ms 200ms 300ms 400ms 500ms

  DEBOUNCE (wait 300ms after LAST event):
  Actions:  -     -     -     -     fire! (at 800ms)
  (only fires ONCE after user stops typing)

  THROTTLE (fire at most once per 300ms):
  Actions: fire!  -     -    fire!  -     fire!
  (fires at regular intervals while events keep coming)
```

```javascript
// debounce -- fires AFTER user stops
function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
const search = debounce((query) => fetchResults(query), 300);

// throttle -- fires AT MOST once per interval
function throttle(fn, limit) {
  let inThrottle = false;
  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
const onScroll = throttle(() => updatePosition(), 100);
```

**When to use what:** Debounce = search input, resize handler. Throttle = scroll, mouse move, API rate limiting.

---

### Q12. map vs forEach vs reduce

| | `map` | `forEach` | `reduce` |
|---|---|---|---|
| **Returns** | New array | `undefined` | Single value |
| **Mutates original?** | No | No | No |
| **Chainable?** | Yes | No | Yes |
| **Use when** | Transform each item | Side effects (logging, DOM) | Accumulate to one value |

```javascript
const nums = [1, 2, 3, 4, 5];

// map -- transform each element, get new array
const doubled = nums.map(n => n * 2);           // [2, 4, 6, 8, 10]

// forEach -- do something with each element, returns nothing
nums.forEach(n => console.log(n));               // prints 1 2 3 4 5

// reduce -- accumulate into single value
const sum = nums.reduce((acc, n) => acc + n, 0); // 15

// reduce can do anything map can (and more)
const names = users.reduce((acc, u) => {
  acc[u.id] = u.name; // build a lookup object
  return acc;
}, {});
```

---

### Q13. Shallow Copy vs Deep Copy

```
  SHALLOW COPY:                           DEEP COPY:

  original = { a: 1, b: { c: 2 } }      original = { a: 1, b: { c: 2 } }

  copy = { ...original }                  copy = structuredClone(original)

  copy.a = 99;                            copy.a = 99;
  copy.b.c = 99;                          copy.b.c = 99;

  original.a -> 1 (safe)                  original.a -> 1 (safe)
  original.b.c -> 99 (CHANGED!)           original.b.c -> 2 (safe!)

  Shallow: top level is copied,           Deep: everything is copied,
  nested objects still SHARED             completely independent
```

```javascript
const original = { a: 1, b: { c: 2 } };

// shallow copy methods (nested refs shared)
const s1 = { ...original };
const s2 = Object.assign({}, original);

// deep copy methods (fully independent)
const d1 = structuredClone(original);      // modern (best)
const d2 = JSON.parse(JSON.stringify(original)); // old way (breaks functions, dates)
```

---

### Q14. Spread vs Rest Operator

Both use `...` but in opposite directions:

```javascript
// SPREAD -- expands (unpacks) elements
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];           // [1, 2, 3, 4, 5]

const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };         // { a: 1, b: 2, c: 3 }

Math.max(...arr1);                       // 3

// REST -- collects (packs) remaining elements
function sum(...nums) {                  // nums = [1, 2, 3, 4]
  return nums.reduce((a, b) => a + b);
}
sum(1, 2, 3, 4);                         // 10

const { a, ...rest } = { a: 1, b: 2, c: 3 };
// a = 1, rest = { b: 2, c: 3 }
```

**Memory trick:** Spread = right side of `=` (giving). Rest = left side of `=` (receiving).

---

### Q15. What is Destructuring?

Extracting values from objects/arrays into variables in one line:

```javascript
// object destructuring
const user = { name: 'Ishan', age: 25, city: 'Bangalore' };
const { name, age, city } = user;

// with rename and default
const { name: userName, role = 'student' } = user;
// userName = 'Ishan', role = 'student' (default)

// array destructuring
const [first, second, , fourth] = [10, 20, 30, 40];
// first = 10, second = 20, fourth = 40 (skipped 30)

// swap variables without temp
let a = 1, b = 2;
[a, b] = [b, a]; // a = 2, b = 1

// nested destructuring
const { address: { street } } = { address: { street: '5th Ave' } };

// in function parameters (very common in React)
const Greeting = ({ name, age }) => <p>{name} is {age}</p>;
```

---

### Q16. What is Currying?

Currying transforms a function with multiple arguments into a chain of functions, each taking **one argument**.

```javascript
// normal function
function add(a, b, c) { return a + b + c; }
add(1, 2, 3); // 6

// curried version
function curriedAdd(a) {
  return (b) => {
    return (c) => a + b + c;
  };
}
curriedAdd(1)(2)(3); // 6

// practical use: creating reusable specialized functions
const multiply = (a) => (b) => a * b;

const double = multiply(2);  // specialized: always multiplies by 2
const triple = multiply(3);  // specialized: always multiplies by 3

double(5); // 10
triple(5); // 15

// real-world: logger with preset level
const log = (level) => (message) => console.log(`[${level}] ${message}`);
const warn = log('WARN');
const error = log('ERROR');

warn('disk almost full');   // [WARN] disk almost full
error('connection lost');   // [ERROR] connection lost
```

---

# Part B: React

---

### Q17. How Does React Work Under the Hood?

```
  REACT RENDER CYCLE:

  1. STATE CHANGE           2. RENDER PHASE             3. COMMIT PHASE
  (trigger)                 (pure, no side effects)     (mutate the DOM)

  setState()                React calls your             React applies
  or props change           component function           changes to the
       |                    to get new JSX               real DOM
       v                         |                           |
  +----------+              +----v-----+              +------v-------+
  | Schedule |  --------->  | Create   |  --------->  | Update real  |
  | re-render|              | new      |              | DOM (only    |
  +----------+              | Virtual  |              | the parts    |
                            | DOM tree |              | that changed)|
                            +----+-----+              +--------------+
                                 |
                            +----v-----+
                            | DIFF old |
                            | vs new   |
                            | vDOM     |
                            | (recon-  |
                            | ciliation)|
                            +----------+

  Key insight: React NEVER re-renders the entire page.
  It diffs the old and new virtual DOM, finds the minimum
  set of changes, and applies only those to the real DOM.
```

---

### Q18. What is the Virtual DOM?

The Virtual DOM is a **lightweight JavaScript object** that mirrors the real DOM. Manipulating JS objects is much faster than manipulating actual DOM nodes.

```
  VIRTUAL DOM vs REAL DOM:

  Virtual DOM (JS object):              Real DOM (browser):
  {                                     <div id="app">
    type: 'div',                          <h1>Hello</h1>
    props: { id: 'app' },                <p>World</p>
    children: [                         </div>
      { type: 'h1', children: 'Hello' },
      { type: 'p',  children: 'World' }
    ]
  }

  Updating JS object: ~0.01ms           Updating DOM node: ~1-10ms
  (100-1000x faster)                    (triggers layout, paint, composite)


  THE PROCESS:

  Old vDOM          New vDOM           Real DOM
  +------+          +------+           +------+
  | div  |          | div  |           | div  |
  | - h1 |   diff   | - h1 |  patch    | - h1 |
  |  "Hi"|  ----->  |  "Hi"|  ----->   |  "Hi"|
  | - p  |          | - p  |  (only p  | - p  |
  |  "A" |          |  "B" |  changed) |  "B" | <-- only this updated
  +------+          +------+           +------+
```

---

### Q19. What is Reconciliation?

Reconciliation is React's **diffing algorithm** that compares old and new virtual DOM trees to find the minimum changes needed.

**Three rules that make it O(n) instead of O(n^3):**

```
  RULE 1: Different element types = destroy and rebuild entire subtree

  Old:  <div><Counter /></div>
  New:  <span><Counter /></span>
        ^^^^
  Different tag -> destroy <div> and ALL children,
  rebuild <span> from scratch. Counter state is LOST.


  RULE 2: Same element type = update only changed attributes

  Old:  <div className="old" style={{color: 'red'}} />
  New:  <div className="new" style={{color: 'blue'}} />

  Same <div> -> only update className and style. Keep the DOM node.


  RULE 3: Lists use KEYS to match elements

  Old:  [<li key="a">A</li>, <li key="b">B</li>]
  New:  [<li key="b">B</li>, <li key="a">A</li>, <li key="c">C</li>]

  React matches by key:
  - key="a" moved to index 1 (REORDER, don't destroy)
  - key="b" moved to index 0 (REORDER, don't destroy)
  - key="c" is new (CREATE)
```

---

### Q20. What is React Fiber?

Fiber is React's **reconciliation engine** (since React 16). It replaced the old synchronous "stack reconciler."

```
  OLD ARCHITECTURE (Stack):             NEW ARCHITECTURE (Fiber):

  render() called                       render() called
       |                                     |
       v                                     v
  Process entire tree                   Process in CHUNKS
  in ONE go                             (can pause and resume)
       |                                     |
  +----+----+----+----+                 +----+  pause  +----+
  | A  | B  | C  | D  |                | A  | ------> | B  | --> ...
  +----+----+----+----+                +----+  (let    +----+
  |                    |                       browser
  |  BLOCKS main       |                       handle
  |  thread the        |                       user input)
  |  entire time       |
  |  (janky!)          |               Result: smooth 60fps UI
  +--------------------+               even during heavy renders
```

**Key benefits:**
- Can **pause** rendering to handle user input (no more frozen UI)
- Can **prioritize** updates (user typing > background data fetch)
- Enables **concurrent features** (Suspense, transitions, startTransition)

---

### Q21. Why Are Keys Important in Lists?

Keys help React **identify which items changed, moved, or were removed** without re-rendering the entire list.

```javascript
// BAD: using index as key
{items.map((item, index) => (
  <TodoItem key={index} item={item} />
))}

// what happens when you delete item at index 0:
// OLD: [key=0: "Buy milk", key=1: "Walk dog", key=2: "Code"]
// NEW: [key=0: "Walk dog", key=1: "Code"]
//
// React thinks key=0 CHANGED content (milk -> dog)
//                key=1 CHANGED content (dog -> code)
//                key=2 was REMOVED
// Result: re-renders items 0 and 1 unnecessarily. State gets mixed up.

// GOOD: using stable unique id
{items.map((item) => (
  <TodoItem key={item.id} item={item} />
))}

// OLD: [key="a": "Buy milk", key="b": "Walk dog", key="c": "Code"]
// NEW: [key="b": "Walk dog", key="c": "Code"]
//
// React knows: key="a" was REMOVED. keys "b" and "c" are unchanged.
// Result: only removes one DOM node. Efficient!
```

---

### Q22. useState vs useReducer

| | `useState` | `useReducer` |
|---|---|---|
| **Best for** | Simple, independent values | Complex state with related fields |
| **Updates** | Direct setter | Dispatch actions |
| **Logic lives in** | Event handlers | Reducer function (pure) |
| **Debugging** | Harder with many setStates | Easy (log actions) |

```javascript
// useState -- simple counter
const [count, setCount] = useState(0);

// useReducer -- complex form state
const reducer = (state, action) => {
  switch (action.type) {
    case 'SET_NAME':    return { ...state, name: action.payload };
    case 'SET_EMAIL':   return { ...state, email: action.payload };
    case 'RESET':       return { name: '', email: '' };
    default:            return state;
  }
};

const [form, dispatch] = useReducer(reducer, { name: '', email: '' });
dispatch({ type: 'SET_NAME', payload: 'Ishan' });
dispatch({ type: 'RESET' });
```

**Rule of thumb:** If you have 3+ related `useState` calls, switch to `useReducer`.

---

### Q23. useEffect Lifecycle

```
  useEffect TIMELINE:

  MOUNT                  UPDATE                 UNMOUNT
  (component appears)    (state/props change)   (component removed)

  render -> paint ->     render -> paint ->     cleanup runs
  effect runs            cleanup of PREV        (returned function)
                         effect runs,
                         then NEW effect runs

  useEffect(() => {
    // SETUP: runs after render
    const id = setInterval(tick, 1000);

    return () => {
      // CLEANUP: runs before next effect or on unmount
      clearInterval(id);
    };
  }, [dependency]);
  //    ^^^^^^^^^
  //    []: run once on mount
  //    [dep]: run when dep changes
  //    (nothing): run after EVERY render
```

```javascript
// fetch data on mount
useEffect(() => {
  let cancelled = false;
  fetchUser(id).then(data => {
    if (!cancelled) setUser(data); // prevent update after unmount
  });
  return () => { cancelled = true; }; // cleanup
}, [id]);
```

---

### Q24. Controlled vs Uncontrolled Components

```javascript
// CONTROLLED -- React state is the source of truth
const Controlled = () => {
  const [value, setValue] = useState('');
  return (
    <input
      value={value}                          // React controls the value
      onChange={(e) => setValue(e.target.value)} // every keystroke updates state
    />
  );
};

// UNCONTROLLED -- DOM is the source of truth
const Uncontrolled = () => {
  const inputRef = useRef(null);
  const handleSubmit = () => {
    console.log(inputRef.current.value);     // read DOM directly
  };
  return <input ref={inputRef} defaultValue="hello" />;
};
```

| | Controlled | Uncontrolled |
|---|---|---|
| **State lives in** | React (useState) | DOM |
| **Access value** | From state variable | Via ref |
| **Validate on type** | Yes | No |
| **Recommended** | Yes (React way) | Only for simple cases / file inputs |

---

### Q25. What is Context API?

Context avoids **prop drilling** -- passing props through many intermediate components.

```
  WITHOUT CONTEXT (prop drilling):        WITH CONTEXT:

  <App theme="dark">                      <ThemeContext.Provider value="dark">
    <Layout theme="dark">                   <Layout>           (no prop needed)
      <Sidebar theme="dark">                  <Sidebar>        (no prop needed)
        <Button theme="dark" />                 <Button />     (reads from context)
```

```javascript
// 1. create context
const ThemeContext = createContext('light');

// 2. provide value at top
const App = () => (
  <ThemeContext.Provider value="dark">
    <Layout />
  </ThemeContext.Provider>
);

// 3. consume anywhere below (no prop drilling)
const Button = () => {
  const theme = useContext(ThemeContext); // "dark"
  return <button className={theme}>Click</button>;
};
```

---

### Q26. What Are Error Boundaries?

Error boundaries catch JavaScript errors in child components and show a **fallback UI** instead of crashing the whole app. They must be class components.

```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true }; // show fallback on next render
  }

  componentDidCatch(error, errorInfo) {
    console.error('caught:', error, errorInfo); // log for debugging
  }

  render() {
    if (this.state.hasError) {
      return <h2>Something went wrong.</h2>; // fallback UI
    }
    return this.props.children; // normal render
  }
}

// usage: wrap any section that might fail
<ErrorBoundary>
  <RiskyComponent />
</ErrorBoundary>

// error boundaries do NOT catch:
// - event handlers (use try/catch)
// - async code (use .catch())
// - errors in the boundary itself
```

---

### Q27. What Are Synthetic Events?

React wraps native browser events in **SyntheticEvent** objects for cross-browser consistency.

```javascript
// React event (synthetic)
<button onClick={(e) => {
  // e is a SyntheticEvent, not a native MouseEvent
  e.preventDefault();      // works the same
  e.stopPropagation();     // works the same
  e.nativeEvent;           // access the real browser event if needed
}}>

// key differences from native events:
// 1. camelCase: onClick not onclick, onChange not onchange
// 2. pass function: onClick={handleClick} not onclick="handleClick()"
// 3. events are pooled (recycled) for performance in React <17
// 4. all events delegate to root (not individual DOM nodes)
```

---

### Q28. What is Batching in React 18?

**Batching** = React groups multiple state updates into a single re-render for performance.

```javascript
// BEFORE React 18: only batched inside event handlers
function handleClick() {
  setCount(1);   // doesn't re-render yet
  setFlag(true); // doesn't re-render yet
  // React batches: ONE re-render at the end
}

setTimeout(() => {
  setCount(1);   // re-renders!
  setFlag(true); // re-renders AGAIN!
  // two separate re-renders (not batched)
}, 100);

// AFTER React 18: batched EVERYWHERE (automatic batching)
setTimeout(() => {
  setCount(1);   // doesn't re-render yet
  setFlag(true); // doesn't re-render yet
  // React batches: ONE re-render (even inside setTimeout!)
}, 100);

// opt out of batching (rare)
import { flushSync } from 'react-dom';
flushSync(() => setCount(1));   // forces immediate re-render
flushSync(() => setFlag(true)); // forces another re-render
```

---

### Q29. What Are React Server Components?

Server Components run on the **server only** -- their JavaScript is never sent to the browser.

```
  TRADITIONAL REACT:                     SERVER COMPONENTS:

  Server sends JS bundle                 Server renders component,
  to browser                             sends HTML to browser

  Browser:                               Browser:
  +--------------------+                 +--------------------+
  | Download 500KB JS  |                 | Receive HTML       |
  | Parse and execute  |                 | (0 KB JS for       |
  | Render components  |                 |  server components)|
  | Fetch data via API |                 | Data already       |
  +--------------------+                 | included!          |
                                         +--------------------+
  Slower initial load                    Faster initial load
  Larger bundle size                     Smaller bundle size
```

```javascript
// server component (default in Next.js App Router)
// can directly access database, file system, etc.
async function UserList() {
  const users = await db.query('SELECT * FROM users'); // direct DB access!
  return (
    <ul>
      {users.map(u => <li key={u.id}>{u.name}</li>)}
    </ul>
  );
}

// client component (needs interactivity)
'use client'; // this directive makes it a client component
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

**Rule:** Use Server Components by default. Add `'use client'` only when you need: useState, useEffect, onClick, onChange, or browser APIs.

---

### Q30. React.memo vs useMemo vs useCallback

| | `React.memo` | `useMemo` | `useCallback` |
|---|---|---|---|
| **What** | HOC wrapping a component | Hook inside a component | Hook inside a component |
| **Caches** | Component render output | Computed value | Function reference |
| **Prevents** | Re-render when props unchanged | Re-computation when deps unchanged | New function reference when deps unchanged |
| **Use when** | Child re-renders often with same props | Expensive calculation | Passing function as prop to memoized child |

```javascript
// React.memo -- skip re-render if props are same
const ExpensiveList = memo(({ items }) => {
  return items.map(i => <Item key={i.id} data={i} />);
});

// useMemo -- cache expensive computation
const sorted = useMemo(() => items.sort(compareFn), [items]);

// useCallback -- cache function reference
const handleClick = useCallback((id) => {
  deleteItem(id);
}, [deleteItem]);
```

See **Class 11 notes** for detailed examples from our codebase.

---

# Part C: React Native Under the Hood

---

### Q31. How Does React Native Work?

React Native runs your JavaScript on a **separate thread** and communicates with native platform code (iOS/Android) to render **real native UI** -- not a WebView.

```
  REACT NATIVE ARCHITECTURE (New Architecture):

  +-----------------------------------------------------------+
  |                    YOUR APP CODE                           |
  |                (JavaScript / TypeScript)                    |
  +---------------------------+-------------------------------+
                              |
                    +---------v-----------+
                    |   JAVASCRIPT ENGINE  |
                    |   (Hermes / JSC)     |
                    |   Runs your React    |
                    |   code               |
                    +---------+-----------+
                              |
                  +-----------v-----------+
                  |         JSI           |
                  | (JavaScript Interface)|
                  | Synchronous bridge    |
                  | between JS and Native |
                  +-----------+-----------+
                              |
              +---------------+----------------+
              |                                |
    +---------v---------+          +-----------v-----------+
    |      FABRIC       |          |    TURBO MODULES      |
    | (New Rendering    |          | (New Native Modules   |
    |  System)          |          |  System)              |
    |                   |          |                       |
    | Renders native    |          | Camera, GPS,          |
    | UI components     |          | Bluetooth, Storage,   |
    | (UIView, Android  |          | any native API        |
    | View)             |          |                       |
    +-------------------+          +-----------------------+
              |                                |
    +---------v--------------------------------v-----------+
    |                                                      |
    |              NATIVE PLATFORM (iOS / Android)         |
    |              Real native UI components                |
    |              Not a WebView!                           |
    +------------------------------------------------------+
```

**Key point:** Unlike Cordova/Ionic (which render HTML in a WebView), React Native renders **actual native views** -- UIButton on iOS, android.widget.Button on Android. That's why RN apps look and feel native.

---

### Q32. Old Architecture vs New Architecture

```
  OLD ARCHITECTURE (Bridge):            NEW ARCHITECTURE (JSI):

  JS Thread        Native Thread        JS Thread        Native Thread
  +--------+       +--------+           +--------+       +--------+
  | Your   |       | Native |           | Your   |       | Native |
  | React  |       | UI &   |           | React  |       | UI &   |
  | code   |       | modules|           | code   |       | modules|
  +---+----+       +----+---+           +---+----+       +----+---+
      |                 |                    |                 |
      v                 v                    +---------+-------+
  +---+--------+--------+---+                          |
  |       THE BRIDGE         |               +---------v--------+
  |  (async JSON messages)   |               |       JSI        |
  |                          |               | (direct sync     |
  | JS -> serialize to JSON  |               |  function calls) |
  |    -> send over bridge   |               |                  |
  |    -> deserialize        |               | No serialization |
  |    -> execute native     |               | No bridge        |
  |                          |               | Synchronous      |
  | SLOW: every call goes    |               | 10x faster       |
  | through JSON             |               +---------+--------+
  +--------------------------+

  Bridge problems:                       JSI benefits:
  - Async only (can't get sync result)  - Synchronous calls possible
  - JSON serialization is slow           - No serialization overhead
  - Bottleneck for large data            - Direct memory sharing
  - All communication is queued          - Lazy loading of modules
```

| | Old (Bridge) | New (JSI + Fabric + TurboModules) |
|---|---|---|
| **Communication** | Async JSON messages over bridge | Synchronous direct calls via JSI |
| **Rendering** | Paper (old renderer) | Fabric (new renderer) |
| **Native modules** | All loaded at startup | TurboModules: lazy loaded on demand |
| **Startup time** | Slower (loads all modules) | Faster (loads only what's needed) |
| **Performance** | Good | Much better |
| **Available since** | React Native 0.1 | React Native 0.68+ (opt-in), default in 0.76+ |

---

### Q33. What is JSI (JavaScript Interface)?

JSI is the **replacement for the Bridge**. It lets JavaScript directly call C++ functions (and through them, native iOS/Android code) **synchronously**.

```
  OLD WAY (Bridge):

  JS: "I need the screen width"
    -> serialize to JSON: {"method": "getScreenWidth"}
    -> send over bridge (async)
    -> native receives, processes
    -> serialize result to JSON: {"result": 375}
    -> send back over bridge (async)
    -> JS receives, deserializes
  Total: ~5-10ms, two async hops

  NEW WAY (JSI):

  JS: screenWidth = global.nativeModule.getScreenWidth()
    -> direct C++ function call (synchronous)
    -> returns 375 immediately
  Total: ~0.01ms, one synchronous call
```

JSI also enables **shared memory** -- instead of copying large data (like images) across the bridge, JS and native can access the same memory.

---

### Q34. How Does React Native Render UI?

```
  REACT NATIVE RENDER PIPELINE:

  Step 1: REACT TREE              Step 2: SHADOW TREE          Step 3: NATIVE VIEWS
  (your JSX)                      (layout calculation)          (actual UI)

  <View style={flex:1}>           ShadowNode: View              UIView (iOS)
    <Text>Hello</Text>              width: 375, height: 812      +------------------+
    <Image src={...} />             ShadowNode: Text             | Hello            |
  </View>                            x: 0, y: 0                  | [image]          |
       |                              width: 375, height: 20      +------------------+
       |                            ShadowNode: Image            android.view.View
       v                              x: 0, y: 20                (Android)
  React creates a tree              width: 375, height: 200
  of elements (JS objects)               |
       |                                 v
       |                          YOGA LAYOUT ENGINE
       +-------->                 (written in C++)
                                  Calculates flexbox layout
                                  (same algorithm as web CSS flexbox)
                                  Determines exact x, y, width, height
                                  for every element
                                       |
                                       v
                                  Native thread creates
                                  REAL native views at
                                  the calculated positions
```

**Key components:**

| Component | What it does |
|-----------|-------------|
| **React tree** | Your JSX components (JavaScript) |
| **Shadow tree** | Intermediate representation with layout info |
| **Yoga** | C++ layout engine that calculates flexbox (used by RN, Litho, ComponentKit) |
| **Fabric** | New rendering system that manages shadow tree and native views |
| **Native views** | Actual platform UI (UIView on iOS, android.view.View on Android) |

**Why `<View>` not `<div>`:** In React (web), `<div>` maps to an HTML element. In React Native, `<View>` maps to a **native platform view** (UIView/android.view.View). Same React paradigm, different rendering target.

---

## Quick Reference: What to Review Before Your Interview

| Priority | Topic | Key thing to say |
|----------|-------|-----------------|
| Must know | Event Loop | "Microtasks (promises) run before macrotasks (setTimeout)" |
| Must know | Closures | "A function that remembers its outer scope variables" |
| Must know | this | "Depends on how the function is called, not where it's defined" |
| Must know | Promises / async-await | "Three states: pending, fulfilled, rejected" |
| Must know | Virtual DOM | "Lightweight JS copy of DOM, diffed for minimal updates" |
| Must know | Reconciliation | "Compares old vs new vDOM using keys and element types" |
| Must know | useEffect | "Runs after render, cleanup before next run or unmount" |
| Must know | Keys | "Never use index for dynamic lists -- use stable unique IDs" |
| High | Hoisting / TDZ | "var is hoisted as undefined, let/const are in TDZ" |
| High | Fiber | "Incremental rendering -- can pause and prioritize updates" |
| High | Debounce/Throttle | "Debounce waits for silence, throttle limits frequency" |
| High | React Native architecture | "JSI replaced the Bridge for synchronous native calls" |
| Medium | Prototypes | "Property lookup walks the prototype chain" |
| Medium | Currying | "Transform multi-arg function into chain of single-arg functions" |
| Medium | Server Components | "Run on server, zero JS sent to browser" |
