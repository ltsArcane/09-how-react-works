# Theory:

Components = function Tab({}) { ... }
A blueprint or template.
Instance: \<Tab />
A specific instance of a component. Object.
React Element: Big immutable JavaScript object that contains the info necessary to create DOM elements. const element = React.createElement('button', {className:'primary'}, 'Click');
DOM Element: Stuff like \<h1>, \<p>, \<div>, \<button>, etc.

We want react to see the component instance (\<BlahBlahBlah />), and not the raw output of the component (\<div className="primary">Click</div>). When you call a component directly, then React no longer sees it as a component instance, and state is moved up to the parent component.

## How rendering works:

When JSX runs it becomes `React.createElement` calls that make React elements. React uses those elements to decide what the DOM should look like. React does this in two phases:
<br>**Render phase** (React-only, no DOM changes): React calls component functions and builds/compares React elements to figure out what needs to change. It does not touch the DOM or display anything. <br>**Commit phase** (actual DOM updates): React applies the determined changes to the DOM (insert/update/remove nodes). After the DOM updates, the browser repaints and the user sees the result.

For triggers and notes, rendering happens on initial load and on state/props updates (re-renders). React walks the whole component tree during rendering, but only the necessary DOM updates are applied in the commit phase — the entire DOM isn’t necessarily rewritten.

## Rendering deep dive (rendering):

[Link (18m:28s)](www.udemy.com/course/the-ultimate-react-course/learn/lecture/37350774)

The render phase is React calling component functions to produce updated React elements (the “virtual DOM”). When a state change triggers a render, React traverses the component tree, re-invokes the functions for the components that need re-rendering, and builds a new tree of React elements. Note: when a component is rendered, all its children are rendered too — React can’t know ahead which children are affected, so it plays it safe.

The virtual DOM is just a cheap JavaScript object tree of React elements (not the browser’s shadow DOM). Creating it is fast; recreating it on each render is inexpensive for typical apps. But recreating the virtual DOM does not touch the real DOM.

Reconciliation is the process that compares the new virtual DOM to the current UI representation (the Fiber tree) to compute the minimal set of DOM changes. Fiber is React’s reconciler and work unit: it builds and maintains a mutable tree of Fibers (one per component/DOM node) that stores state, props, hooks, side effects, and update queues. Fibers are linked so children and siblings are easy to traverse.

Fiber supports asynchronous rendering: work can be split into chunks, paused, prioritized, resumed, or thrown away — powering features like Suspense and transitions and preventing long renders from blocking the main thread.

After reconciliation, React collects the necessary DOM operations (insertions, updates, deletions) into a list of effects. These effects are applied later during the commit phase, which is when the real DOM actually changes and the browser repaints. For example, toggling a modal typically only inserts/removes modal nodes; React reuses the rest of the DOM.

Key points:

- Render phase = call components and build a new virtual DOM (no DOM updates).
- Virtual DOM is a lightweight JS object tree, not the shadow DOM.
- Re-rendering a component also re-renders its children.
- Fiber reconciles the new virtual DOM with the current Fiber tree to compute minimal DOM changes.
- Fiber is mutable, stores state/effects, and enables async, interruptible work.
- Reconciliation produces an effects list; actual DOM updates happen in the commit phase.

## Rendering deep dive (commit):

[Link (11m:27s)](www.udemy.com/course/the-ultimate-react-course/learn/lecture/37350776)

When the render phase finishes, it produces a list of DOM updates (effects) and a work-in-progress Fiber tree. The commit phase is when those effects are actually applied to the host (for web apps, React DOM): inserts, updates, and deletions happen synchronously so the DOM never shows a partial state. Because the commit is synchronous it cannot be paused or interrupted — this guarantees UI consistency. After the commit, the work-in-progress Fiber tree becomes the current tree and is reused for future updates.

React itself only does the render phase; committing is done by a renderer (React DOM for the web). That’s why React can target many hosts — the render phase outputs React Elements and the renderer commits those results to whatever host (DOM, native UI, video frames, Figma, etc.). The term “renderer” is a bit misleading: it’s really the committer.

Reconciliation (the reconciler/Fiber) compares the new virtual DOM (React element tree) with the current Fiber tree to compute the minimal set of changes. Fiber is a mutable tree of units of work that stores state, props, hooks, side effects, and update queues. The render phase can be asynchronous and interruptible (work is chunked, prioritized, paused, resumed, or discarded), which enables features like Suspense and transitions and prevents long renders from blocking the main thread. But all resulting changes are collected as effects and applied in one synchronous commit.

After the commit, the browser repaints when it’s able, making the visual update visible. In short: render = compute changes (React-only, possibly async); reconcile = diff against Fiber and produce effects; commit = synchronously apply effects to the host (React DOM or other renderer); paint = browser/host displays the result.

## Diffing and Keys:

| Key (status) | Props change | Location change |  →  | State preserved |
| :----------: | :----------: | :-------------: | :-: | :-------------: |
|   ❌ (N/A)   |      ✅      |       ❌        |  →  |       ✅        |
|   ❌ (N/A)   |      ❌      |       ✅        |  →  |       ❌        |
| ✅ (stable)  |      ❔      |       ❔        |  →  |       ✅        |
| ✅ (changes) |      ❔      |       ❔        |  →  |       ❌        |

❔ = Does not affect state outcome.

## Render Logic and Rules:

Render logic should consist of pure functional level coding. It should not contain any side effects, such as DOM manipulation, timers, or asynchronous calls.
Only Event Handler functions should contain side effects, such as mutating outsdie variables, calling APIs, dispatching actions, relying on non-predictable events, etcetera.

## Batching and stale state:

React batches state updates such that only one render and commit is performed per event handler.
This occurs at the very end of the event handler. What this means, is that any later calls within the event handler to use state that has been marked for a change, will not use the new state, but rather a stale version of it stored within the Fiber tree, until AFTER the re-render. **Updating state in React is asynchronous**. This does not change when using updater form (callback function) either, as updater form merely ensures that the state's new value is applied correctly, rather than running the state update immediately.

## Event Propagation, Event Delegation, and how React handles Events:

When an event occurs (or is dispatched), the browser creates an event object at the root of the document. The event travels down the DOM tree (capturing phase) until it reaches the target element which triggered the event, and runs it's event handler. After that, it goes up the tree (bubbling phase) and triggers the event handlers on each element along the way (event propagation). If there were thousands of buttons, this would be problematic for the app's performance and memory, so people use event delegation, which is a technique of attaching a single event listener to a parent element and then using `e.target` to determine the event's origin.

Behind the scenes, React actually performs event delegation for all events in our applications, by delegating them all to the root DOM container.

React also wraps up events in a synthetic event object, which is a JavaScript object that mimics the browser's native event object, in such a way that all of the important methods still work, but the wrapper also fixes some browser inconsistencies, making it so that the events work the same way in all browsers. All events are wrapped in the SyntheticEvent, and so all of them bubble (except for the scroll event).

Second lastly, you can use preventDefault to prevent the browser's default behavior (normally you'd need to return faults from the event handler function). Very useful for forms where you want to prevent the page from refreshing.

Lastly, you can simply attach "Capture" (onClickCapture) to the event handler name, and it'll handle the event within the capturing phase instead of the bubbling phase (where all synthetic events are usually handled).

# React is a library, not a framework:

Angular, Vue and Svelte are all frameworks. These are groups of "ingredients" which all come together when creating a web app, but you cannot choose which "ingredient" within the kit you want to use. However, React is a library, which means that you can choose the best ingredients,a t the costo f having to need to do research and having to acquire each ingredient separately.

A framework is a complete structure that includes everything you'd need in order to build a complete large scale application. Sometimes, they even include stuff like routing, styling, HTTP requests for management, and more all out of the box. The problem is that you're stuck with the framework's tools and conventions, even if you don't like or agree with them. On the other hand, JavaScript libraries are basically pieces of code that developers share for other developers to use. A prime example here is of course ,React, which is what we call a view library, because all React does is draw components onto a user interface / view. Luckily, React has a huge third party library ecosystem, as it's popularity has led to a really, really large ecosystem of libraries that we can include in our React projects for different needs (routing for SPA, http reqs, managing remote server state, global application state, styling, forms, animations, transitions, even entire UI componenet libraries, etc). React Router, React Query, Redux, styled components, Tailwind, etcetera.

# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can't go back!**

If you aren't satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you're on your own.

You don't have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn't feel obligated to use this feature. However we understand that this tool wouldn't be useful if you couldn't customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)

### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)
