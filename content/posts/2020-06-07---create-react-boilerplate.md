---
title: Create React Boilerplate 
date: "2020-06-07T22:40:32.169Z"
template: "post"
draft: false
slug: "create-react-starter-boilerplate"
category: "React"
tags:
  - "React"
  - "Boilerplate"
  - "Lazy Loading"
description: "Steps to create a React Boilerplate"
---

[Full source code](https://github.com/arunghosh/create-react-starter-boilerplate)

There is a story of a hungry man who went begging for food. One generous person provided him fish and another taught him to fish. Sometimes it is better to provide means to earn food rather than food. Yeah... it depends of how hungry they are and how difficult is the means to earn food. So let's have both, boilerplate and steps to create boilerplate.

So why steps to create boilerplate rather than a boilerplate. With ready made boilerplate you may face the following: 
- Not all the features are required
- Some packages are outdated
- You don't what is happening

For above reasons this article aims to detail the steps to create the boilerplate.

The boilerplate aims to have the following
1. TypeScript
2. [Folder Structure](#Folder-Structure)
3. [Routing](#Routing)
4. [Styling](#Styling)
5. [Manage Document Head](#Manage-Document-Head)
6. [Lazy Loading of Pages](#Lazy-Loading)
7. [Setting Error Boundary](#Setting-Error-Boundary)
8. [Testing](#Testing)
9. [Precommit Hooks](#Precommit-Hooks)
10. [Storybook](#Storybook)
11. [State Management?](#State-Management)

##  Lets us start

The starting point will be with the `create-react-app`

Here you have a choice to go with or without **TypeScript**.

Without TypeScript
```bash
npx create-react-app your-application-name 
``` 

With TypeScript
```bash
npx create-react-app your-application-name --template typescript
```


## Folder Structure
There is not hard and fast rule for the folder structure. The document follows the below structure:
```bash
+-- components
|   +-- generics
|   |   +-- Button
|   |     |-- Button.tsx
|   |     |-- Button.stories.tsx
|   |     |-- Button.test.tsx
|   +-- domain
|   |   +-- UserCart
|   |     |-- UserCart.tsx
|   |     |-- UserCart.stories.tsx
|   |     |-- UserCart.test.tsx
|   +-- layout
|   |   +-- NavBar
|   |     |-- NavBar.tsx
|   |     |-- NavBar.stories.tsx
|   |     |-- NavBar.test.tsx
|   +-- pages
|   |   +-- HomePage
|   |     |-- HomePage.tsx
|   |     |-- HomePage.stories.tsx
|   |     |-- HomePage.test.tsx
+-- config
|   |-- pages.ts
|   |-- theme.ts
+-- apis
|   |-- auth.ts
|   |-- cart.ts
+-- utils
    |-- tax.ts 
```

## Routing
In case you have a more than one page, you need to have a router. For routing we are using [React Router](https://github.com/ReactTraining/react-router)

First install react-router
```bash
 yarn add react-router-dom
```

And if you are using TypeScript add types
```bash
yarn add @types/react-router
```

It is good to have a `config/pages.ts` file where you define all the pages related information. And you can define all your pages in a `components/pages` folder.

```javascript
import { HomePage, AboutPage } from "../components/pages";

export const pages = {
  home: {
    title: "Home",
    path: "/",
    Component: HomePage
  },
  about: {
    title: "About",
    path: "/about",
    Component: AboutPage
  }
};
```

This is in the form of a directory so that we can refer `pages.about.path`. To get as an array use `Object.values(pages)`

We will also have simple navigation bar `components/layouts/NavBar.tsx`

```javascript
import React from "react";
import { Link } from "react-router-dom";
import { pages } from "../../config/pages";

export default function TopNav() {
  return (
    <ul>
      <li>
        <Link to={pages.home.path}>Home</Link>
      </li>
      <li>
        <Link to={pages.about.path}>About</Link>
      </li>
    </ul>
  );
}
```

And in the `App.tsx` 

```javascript
import React from "react";
import { BrowserRouter as Router, Switch, Route } from "react-router-dom";
import { pages } from "./config/pages";
import "./App.css";

function App() {
  return (
    <Router>
      <div>
        <Switch>
          {Object.values(pages).map((page, index) => (
            <Route
              key={index}
              exact
              path={page.path}
              render={() => 
                <>
                  <TopNav />
                  <page.Component />
                </>
              }
            />
          ))}
        </Switch>
      </div>
    </Router>
  );
}

export default App;

```

Once done you will have `/` pointing to the `Home Page` and `/about` pointing to the `About Page`.

## Manage Document Head 
We will use [react-helment-async](https://www.npmjs.com/package/react-helmet-async) to manage document head.
Ther is npm package `react-helment` which is used manage changes to the document head like `title`, `metadata`, etc. But then why `react-helment-async` instead of `react-helmet`?
> The `react-helmet` relies on `react-side-effect`, which is not **thread-safe**. If you are doing anything asynchronous on the server, you need Helmet to encapsulate data on a per-request basis, this package does just that.

Install 
```bash
yarn add react-helmet-async
```

Now to change `title` in relation with the page loaded, the `App.tsx` will be 
```javascript
import { Helmet, HelmetProvider } from "react-helmet-async";
// ....
// ....
function App() {
  return (
    <HelmetProvider>
      <Router>
// ....
// ....
                <>
                  <Helmet>
                    <title>{page.title}</title>
                  </Helmet>
                  <TopNav />
                  <page.Component />
                </>
// ....
// ....
      </Router>
    </HelmetProvider>
  );
}
```


## Styling
When it comes to styling there are many choices: 
  - Pre-processors like `sass` or `less`
  - UI libraries like [Bootstrap](https://getbootstrap.com/), [Ant Design](https://ant.design/docs/react/introduce)
  - Minimal utility library like [Tailwind](https://www.tailwindtoolbox.com/)
  - Or [styled-components](https://styled-components.com/)

Here we will be using `styled-components`. The choice depends on the application requirement and the team. If the requirement is to create custom theme with you should go with something like `sass` or `styled-components`. But when you are running short of time and the design is a cliche (like an admin dashboard) go with `bootstrap` or `ant design`.

Install
```bash
yarn add styled-components
```

And if you are using `TypeScript` add types
```bash
yarn add @types/styled-components
```

So Let's make the NavBar look better
```javascript
import React from "react";
import { Link } from "react-router-dom";
import styled from "styled-components";
import { pages } from "../../config/pages";

const NavCtnr = styled.aside`
  width: 100%;
  background: #555;
`;

const NavList = styled.ul`
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
`;

const NavItem = styled.li`
  a {
    text-decoration: none;
    padding: 1rem;
    display: inline-block;
    color: #eee;
  }
`;

export default function TopNav() {
  return (
    <NavCtnr>
      <NavList>
        <NavItem>
          <Link to={pages.home.path}>Home</Link>
        </NavItem>
        <NavItem>
          <Link to={pages.about.path}>About</Link>
        </NavItem>
      </NavList>
    </NavCtnr>
  );
}
```

It is not a good practice to hardcode the style values like above. The [ThemeProvider](https://styled-components.com/docs/advanced#theming) from `styled-component` should be used.

So let us define our theme in `config/theme.ts`
```javascript
export default {
  light : {
    primary: "#19f",
    background: "#eee",
    foreground: "#444",
  },
  dark : {
    primary: "#19f",
    background: "#444",
    foreground: "#eee",
  },
};
```
 
 And the components need to wrapped with the `ThemeProvider` in `App.tsx`
 ```javascript
// ....
// ....
import { ThemeProvider } from "styled-components";
import theme from "./config/theme";
// ....
// ....
      <Router>
        <ThemeProvider theme={theme.dark}>
          <Switch>
              // ....
              // ....
            </Switch>
        </ThemeProvider>
      </Router>
// ....
// ....
 ```

Now in the `Navbar` you can
```javascript
const NavItem = styled.li`
  a {
    text-decoration: none;
    padding: 1rem;
    display: inline-block;
    color: ${props => props.theme.primary};
  }
`;
```

## Lazy Loading
Consider an application having 10 pages. Normally when the first page loads, it loads JS required for all the pages, not just the current page.  This is normal loading (eager loading). Via lazy loading the application can be made to load only the JS required for the current page. The rest of the resources will be loaded when requested for. It is on-demand loading(lazy) of the resources rather than eager loading all of them. This helps to improve the initial loading time.

To enable lazy loading we are making a few changes to the `config/pages.ts` lazy load the pages. Instead of loading pages directly we need to lazy load it.
```javascript
import { lazy } from "react";

const HomePage = lazy(() =>
  import("../components/pages/HomePage" /* webpackChunkName: "HomePage" */)
);

const AboutPage = lazy(() =>
  import("../components/pages/AboutPage" /* webpackChunkName: "AboutPage" */)
);

// ....
// ....
```

Now we will define a simple loading indicator in `pages/generics/LoadingIndicator.tsx`. You can make it fancier as you wish
```javascript
import React from "react";

export default function LoadingIndicator() {
  return <div>Loading...  </div>;
}
```

And in the `App.tsx` we will wrap the page component with `Suspense`
```javascript
import React, { Suspense } from "react";
import { LoadingIndicator } from "./components/generics";

// .....
// .....
                  <Suspense fallback={<LoadingIndicator />}>
                    <page.Component />
                  </Suspense>
// .....
// .....
```

## Setting Error Boundary
Since we are lazy loading the component, what if the component fails to load. Rather than making the whole screen go blank, we can provide better user experience by using [Error Boundaries](https://reactjs.org/docs/error-boundaries.html). Error boundary will help to replace component having exception with a fallback component. 

We will be using [react-error-boundary](https://github.com/bvaughn/react-error-boundary).

Install
```bash
yarn add react-error-boundary
```

Add `components/generics/ErrorFallback/ErrorFallback.tsx`
```javascript
import React from 'react'

export default function ErrorFallback() {
  return (
    <div>
      Sorry!!! Failed to load.   
    </div>
  )
}
```

And in the `App.tsx` page add the following
```javascript
//....
//....
import { BrowserRouter as Router, Switch, Route } from "react-router-dom";
import { LoadingIndicator, ErrorFallback } from "./components/generics";
//....
//....
                    <ErrorBoundary FallbackComponent={ErrorFallback}>
                      <Suspense fallback={<LoadingIndicator />}>
                        <page.Component />
                      </Suspense>
                    </ErrorBoundary>
//....
//....
```

**References**
  - [Egghead.io Video](https://egghead.io/lessons/react-using-react-error-boundaries-to-handle-errors-in-react-components)
  - [Official React Documentation](https://reactjs.org/docs/error-boundaries.html)


## Testing

The [`react-testing-library`](https://testing-library.com/) will be used for testing. This comes bundled with `create-react-app`.

Let's write a test for `ErrorFallback` component in `components/generics/ErrorFallback/ErrorFallback.test.tsx`
```javascript
import React from "react";
import { render } from "@testing-library/react";
import ErrorFallback from "./ErrorFallback";

test("renders learn react link", () => {
  const { getByText } = render(<ErrorFallback />);
  const linkElement = getByText(/Failed to load/i);
  expect(linkElement).toBeInTheDocument();
});
```

To run test
```
yarn test
```

## Precommit Hooks
>[Git hooks](https://githooks.com/) are scripts that Git executes before or after events such as: commit, push, and receive. Git hooks are a built-in feature - no need to download anything. Git hooks are run locally.

Before commiting changes to the repository it is good practice to check if:
- All tests are passing
- There are no linting errors

To enable precommit hook, add [pre-commit](https://www.npmjs.com/package/pre-commit) as dev dependency
```
yarn add --dev pre-commit
```

Now make the following changes to `packages.json`.
```diff
     "test": "react-scripts test",
+    "ci-test": "CI=true react-scripts test",
     "eject": "react-scripts eject",
     "storybook": "start-storybook -p 9009 -s public",
-    "build-storybook": "build-storybook -s public"
+    "build-storybook": "build-storybook -s public",
+    "lint": "eslint --ext js,ts,tsx src"
   },
+  "pre-commit": [
+    "lint",
+    "ci-test"
+  ],
   "eslintConfig": {
     "extends": "react-app"
   },
```
The `ci-test` is added so that test can be executed in the CI mode rather than in the watch mode.

If in case you want to commit without verification you can use `--no-verify`.

## Storybook
>[Storybook](https://storybook.js.org/docs/guides/guide-react/) is a user interface development environment and playground for UI components. The tool enables developers to create components independently and showcase components interactively in an isolated development environment.

>Storybook runs outside of the main app so users can develop UI components in isolation without worrying about app specific dependencies and requirements.

Install (since we are using `create-react-app`)
```bash
npx -p @storybook/cli sb init --type react_scripts
```

This will create 2 folders
  1. `.storybook` containing the config
  2. `stories` having the stories where we can add more stories

The stories location can be edited in the `.storybook/main.js` file. Since we are using `tsx` files we will update the config as 
```diff
module.exports = {
-  stories: ['../src/**/*.stories.js'],
+  stories: ['../src/**/*.stories.tsx'],
  addons: [
    '@storybook/preset-create-react-app',
    '@storybook/addon-actions',
    '@storybook/addon-links',
  ],
};
```

Now add a story for `ErrorFallback` component in `components/generics/ErrorFallback/ErrorFallback.stories.tsx`
```javascript
import React from "react";
import { ErrorFallback } from "../components/generics";

export default {
  title: "Error Fallback",
  component: ErrorFallback
};

export const Error = () => <ErrorFallback />;
```

Now to run the storybook
```
yarn storybook
```

## State Management
This is not a mandatory component. When the components have lot of shared state you will have to go for a [state management](https://kentcdodds.com/blog/application-state-management-with-react) to avoid [props drilling](https://kentcdodds.com/blog/prop-drilling/).
Do read the article mentioned here. It will give you a direction on State Management. It mentions about state management methods like
  1. [`useState`](https://reactjs.org/docs/hooks-reference.html#usestate)
  2. [`useReducer`](https://reactjs.org/docs/hooks-reference.html#usereducer)
  3. [`useContext`](https://reactjs.org/docs/hooks-reference.html#usecontext)
  4. [Redux](https://react-redux.js.org/)

Other than the onces mentioned above there is new child in state management ~ [Recoil](https://github.com/facebookexperimental/Recoil) by Facebook. You can also checkout the [egghead.io video tutorial on Recoil](https://egghead.io/playlists/getting-started-with-recoil-in-react-1fca)