# CRA Recipe

This is a step-by-step guide to customize CRA for Atolye15 projects. You can review [CRA Starter](https://github.com/atolye15/cra-starter) project to see how your application looks like when all steps followed.

You will get an application which has;

* TypeScript
* Sass
* Linting
* Formatting
* Testing
* CI/CD
* Storybook

## Table of Contents

* [Step 1: Creating a new app](#step-1-creating-a-new-app)
* [Step 2: Removing CRA example files](#step-2-removing-cra-example-files)
* [Step 3: Make TypeScript more strict](#step-3-make-typescript-more-strict)
* [Step 4: Installing Prettier](#step-4-installing-prettier)
* [Step 5: Installing ESLint](#step-5-installing-eslint)
* [Step 6: Enabling Sass](#step-6-enabling-sass)
* [Step 7: Installing stylelint](#step-7-installing-stylelint)
* [Step 8: Setting up our test environment](#step-8-setting-up-our-test-environment)
* [Step 9: Enabling hot reloading](#step-9-enabling-hot-reloading)
* [Step 10: Organizing Folder Structure](#step-10-organizing-folder-structure)
* [Step 11: Adding Storybook](#step-11-adding-storybook)
* [Step 12: Adding React Router](#step-12-adding-react-router)
* [Step 13: Enabling code-splitting](#step-13-enabling-code-splitting)
* [Step 14: Adding CircleCI config](#step-14-adding-circleci-config)
* [Step 15: Auto-deploy to Surge.sh](#step-15-auto-deploy-to-surgesh)
* [Step 16: Github Settings](#step-16-github-settings)
* [Step 17 Final Touches](#step-17-final-touches)
* [Step 18: Starting to Development <g-emoji class="g-emoji" alias="tada" fallback-src="https://github.githubassets.com/images/icons/emoji/unicode/1f389.png">🎉</g-emoji>](#step-18-starting-to-development-)

## Step 1: Creating a new app

First of all, we need to initialize our codebase via CRA command.

```sh
npx create-react-app cra-starter --template typescript
cd cra-starter
yarn start
```

## Step 2: Removing CRA example files

In order to create our stack, we need to remove unnecessary CRA files.

## Step 3: Make TypeScript more strict

We want to keep type safety as strict as possibble. In order to do that, we update `tsconfig.json` with the settings below.

```jsonc
"noImplicitAny": true,
"noImplicitReturns": true,
```

## Step 4: Installing Prettier

We want to format our code automatically. So, we need to install Prettier.

```sh
yarn add prettier --dev
```

`.prettierrc`

```jsonc
{
  "printWidth": 100,
  "singleQuote": true,
  "trailingComma": "all"
}
```

`.prettierignore`

```ignore
build
```

Also, we want to enable format on save on VSCode.

`.vscode/settings.json`

```json
{
  "editor.formatOnSave": true,
}
```

Finally, we update `package.json` with related format scripts.

```json
"format:ts": "prettier --write 'src/**/*.{ts,tsx}'",
"format": "yarn run format:ts",
"format:check": "prettier -c 'src/**/*.{ts,tsx}'"
```

## Step 5: Installing ESLint

We want to have consistency in our codebase and also want to catch mistakes. So, we need to install ESLint.

```sh
yarn add @atolye15/eslint-config --dev
```
`.eslintrc`

```jsonc
{
  "extends": [
    "@atolye15/eslint-config"
  ]
}
```

`.eslintignore`

```ignore
public
build
react-app-env.d.ts
```

`.vscode/settings.json`

```json
{
  "eslint.validate": ["javascript", "javascriptreact", "typescript", "typescriptreact"]
}
```

We need to update `package.json` for ESLint scripts.

```json
"lint": "yarn run lint:ts",
"lint:ts": "tsc && yarn lint:eslint",
"lint:eslint": "eslint 'src/**/*.{ts,tsx}'",
"format:ts": "prettier --write 'src/**/*.{ts,tsx}' && yarn lint:eslint --fix",
```

## Step 6: Enabling Sass

CRA comes with Sass support out of the box. In order to enable it, we only add `node-sass` to our project.

```sh
yarn add node-sass --dev
```

## Step 7: Installing stylelint

We also want a linter for our sass files. We need to install `stylelint`.

```sh
yarn add stylelint stylelint-config-prettier stylelint-prettier @atolye15/stylelint-config --dev
```

`.stylelintrc`

```jsonc
{
  "extends": ["@atolye15/stylelint-config", "stylelint-prettier/recommended"]
}
```

Finally, we need to update `package.json` and `.vscode/settings.json`

`package.json`

```json
"lint:css": "stylelint --syntax scss \"src/**/*.scss\"",
"lint": "yarn run lint:ts && yarn run lint:css",
"format:css": "stylelint --fix --syntax scss \"src/**/*.scss\"",
"format": "yarn run format:ts && yarn run format:css"
```

## Step 8: Setting up our test environment

We'll use `jest` with `enzyme`.

```sh
yarn add enzyme enzyme-adapter-react-16 react-test-renderer --dev
yarn add @types/enzyme @types/enzyme-adapter-react-16 --dev
```

Also we need to install `enzyme-to-json` for simpler snapshosts.

```sh
yarn add enzyme-to-json --dev
```

We update our `package.json` for jest configuration.

```json
"scripts": {
  "coverage": "yarn run test --coverage"
},
"jest": {
  "snapshotSerializers": [
    "enzyme-to-json/serializer"
  ],
  "collectCoverageFrom": [
    "src/**/*.{ts,tsx}",
    "!src/index.tsx",
    "!src/setupTests.ts",
    "!src/components/**/index.{ts,tsx}",
    "!src/components/**/*.stories.{ts,tsx}"
  ],
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  }
}
```

And finally, we need to add the `coverage` to `.gitignore` and `.prettierignore`.

`.eslintignore`

```ignore
# ...
coverage
```

`.prettierignore`

```ignore
# ...
coverage
```

Before the testing, we need to add our setup file to initialize enzyme.

`src/setupTests.ts`

```ts
import { configure } from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

configure({ adapter: new Adapter() });
```

With this config, we are able to run tests with snapshots and create coverage. Let's add a simple test to verify our setup.

`src/App.test.tsx`

```tsx
import React from 'react';
import { shallow } from 'enzyme';
import App from './App';

it('runs correctly', () => {
  const wrapper = shallow(<App />);

  expect(wrapper).toMatchSnapshot();
});
```

Also, verify coverage report with `yarn coverage`.

## Step 9: Enabling hot reloading

We want to take advantage of hot reloading and don't want to lose React's current state. In order to do that we can use react hot loader. Since, we use CRA and don't want to eject it, we need to use `customize-cra` package.

```sh
yarn add react-app-rewired customize-cra @hot-loader/react-dom --dev
```

After the installation we need to update `package.json` scripts to use `react-app-rewired`

```json
"start": "react-app-rewired start",
"build": "react-app-rewired build",
"test": "react-app-rewired test",
"eject": "react-app-rewired eject"
```

Now, we can install `react-hot-loader`.

```sh
yarn add react-hot-loader
```

Also we need to update hot reloader config.

`src/index.tsx`

```tsx
import { setConfig } from 'react-hot-loader';

setConfig({
  ignoreSFC: true,
  pureRender: true,
});
```

In order to update babel config for hot loader, we need to create a `config-overrides.js` file on the root.

```ts
// eslint-disable-next-line @typescript-eslint/no-var-requires
const { override, addBabelPlugin, addWebpackAlias } = require('customize-cra');

module.exports = override(
  addBabelPlugin('react-hot-loader/babel'),
  addWebpackAlias({
    'react-dom': '@hot-loader/react-dom',
  }),
);
```

Lastly, we need to use hot HOC.

`src/App.tsx`

```tsx
import React from 'react';
import { hot } from 'react-hot-loader/root';

export default hot(App);
```

## Step 10: Organizing Folder Structure

Our folder structure should look like this;

```
src/
├── App.test.tsx
├── App.tsx
├── __snapshots__
│   └── App.test.tsx.snap
├── components
│   └── Button
│       ├── Button.scss
│       ├── Button.stories.tsx
│       ├── Button.test.tsx
│       ├── Button.tsx
│       ├── __snapshots__
│       │   └── Button.test.tsx.snap
│       └── index.ts
├── containers
│   └── Like
│       ├── Like.tsx
│       └── index.ts
├── fonts
├── img
├── index.tsx
├── react-app-env.d.ts
├── routes
│   ├── Feed
│   │   ├── Feed.scss
│   │   ├── Feed.test.tsx
│   │   ├── Feed.tsx
│   │   ├── index.ts
│   │   └── tabs
│   │       ├── Discover
│   │       │   ├── Discover.scss
│   │       │   ├── Discover.test.tsx
│   │       │   ├── Discover.tsx
│   │       │   └── index.ts
│   │       └── MostLiked
│   │           ├── MostLiked.test.tsx
│   │           ├── MostLiked.tsx
│   │           └── index.ts
│   ├── Home
│   │   ├── Home.scss
│   │   ├── Home.test.tsx
│   │   ├── Home.tsx
│   │   └── index.ts
│   └── index.ts
├── setupTests.ts
├── styles
│   └── index.scss
└── utils
    ├── location.test.ts
    └── location.ts
```

## Step 11: Adding Storybook

We need to initialize the Storybook on our project.

```sh
npx -p @storybook/cli sb init --type react
```

We also need to add `info` addon and `react-docgen-typescript-loader` package to show component props on our stories (Optional but recommended).

```sh
yarn add @storybook/addon-info react-docgen-typescript-loader --dev
```

We have to use the [custom Webpack config in full control mode, extending default configs](https://storybook.js.org/docs/configurations/custom-webpack-config/#full-control-mode--default) by creating a `webpack.config.js` file in our Storybook configuration directory (by default, it’s `.storybook`):

`.storybook/webpack.config.js`

```js
module.exports = ({ config, mode }) => {
  config.module.rules.push({
    test: /\.(ts|tsx)$/,
    use: [
      {
        loader: require.resolve('babel-loader'),
        options: {
          presets: [require.resolve('babel-preset-react-app')],
        },
      },
      require.resolve('react-docgen-typescript-loader'),
    ],
  });
  config.resolve.extensions.push('.ts', '.tsx');
  return config;
};
```

Since we use `typescript`, we can change the file extensions (`addons` and `config`) to `.ts` in `.storybook` folder. Then we need to update storybook config to register info addon, and stories directory.

`.storybook/config.ts`

```ts
import { configure, addDecorator } from '@storybook/react';
import { withInfo } from '@storybook/addon-info';

// automatically import all files ending in *.stories.tsx
const req = require.context('../src/components', true, /.stories.tsx$/);

function loadStories() {
  addDecorator(withInfo);
  req.keys().forEach(req);
}

configure(loadStories, module);
```

> We will place the stories inside component folders, you can delete the `stories` folder which is created by storybook initialization process.

Let's create a story for our Button component.

`src/components/Button/Button.stories.tsx`

```tsx
import React from 'react';
import { storiesOf } from '@storybook/react';

import Button from './Button';

storiesOf('Button', module)
  .add('Primary', () => <Button primary>Primary Button</Button>)
  .add('Secondary', () => <Button secondary>Secondary Button</Button>);
```

Run storybook

```
yarn storybook
```

## Step 12: Adding React Router

As usual, we want to use `react-router` for routing.

```
yarn add react-router-dom
yarn add @types/react-router-dom --dev
```

Then, we need to encapsulate our root component with `BrowserRouter`.

```tsx
// src/index.tsx

import React, { FunctionComponent } from 'react';
import ReactDOM from 'react-dom';
import { setConfig } from 'react-hot-loader';
import { BrowserRouter } from 'react-router-dom';

import App from './App';

setConfig({
  ignoreSFC: true,
  pureRender: true,
});

const Root: FunctionComponent = () => (
  <BrowserRouter>
    <App />
  </BrowserRouter>
);

ReactDOM.render(<Root />, document.getElementById('root'));
```

```tsx
// src/routes/Routes.tsx

import React, { FunctionComponent } from 'react';
import { Switch, Route } from 'react-router-dom';

import Home from './Home';
import Feed from './Feed';

const Routes: FunctionComponent = () => (
  <Switch>
    <Route path="/" exact component={Home} />
    <Route path="/feed" exact component={Feed} />
  </Switch>
);

export default Routes;
```

```tsx
// src/App.tsx

import React, { FunctionComponent, Fragment } from 'react';
import { hot } from 'react-hot-loader';

import Routes from './routes';

const App: FunctionComponent = () => (
  <Fragment>
    <header>Header</header>
    <Routes />
    <footer>Footer</footer>
  </Fragment>
);

export default hot(module)(App);
```

## Step 13: Enabling code-splitting

We want to make route based code-splitting in order to prevent a huge bundled asset. When we done with this, only relevant assets will be loaded by our application. Let's install `react-loadable`.

```
yarn add react-loadable
yarn add @types/react-loadable --dev
```

Now, let's convert our routes to dynamically loaded.

```tsx
// src/routes/Home/index.ts

import Loadable from 'react-loadable';

import Loading from '../../components/Loading';

const LoadableHome = Loadable({
  loader: () => import('./Home'),
  loading: Loading,
});

export default LoadableHome;
```

```tsx
// src/routes/Feed/index.ts

import Loadable from 'react-loadable';

import Loading from '../../components/Loading';

const LoadableFeed = Loadable({
  loader: () => import('./Feed'),
  loading: Loading,
});

export default LoadableFeed;
```

## Step 14: Adding CircleCI config

We can create a CircleCI pipeline in order to CI / CD.

```yaml
# .circleci/config.yml

version: 2
jobs:
  build_dependencies:
    docker:
      - image: circleci/node:10
    working_directory: ~/repo
    steps:
      - checkout
      - attach_workspace:
          at: ~/repo
      - restore_cache:
          keys:
            - dependencies-{{ checksum "package.json" }}
            - dependencies-
      - run:
          name: Install
          command: yarn install
      - save_cache:
          paths:
            - ~/repo/node_modules
          key: dependencies-{{ checksum "package.json" }}
      - persist_to_workspace:
          root: .
          paths: node_modules

  test_app:
    docker:
      - image: circleci/node:10
    working_directory: ~/repo
    steps:
      - checkout
      - attach_workspace:
          at: ~/repo
      - run:
          name: Lint
          command: yarn lint
      - run:
          name: Format
          command: yarn format:check
      - run:
          name: Test
          command: yarn test
      - run:
          name: Coverage
          command: yarn coverage

workflows:
  version: 2
  build_app:
    jobs:
      - build_dependencies
      - test_app:
          requires:
            - build_dependencies
```

After that we need to enable CircleCI for our repository.

## Step 15: Auto-deploy to Surge.sh

First of all, we need to retrieve our Surge.sh token.

```
surge token
```

After copying the value, we need to add it as a dev dependency.

```
yarn add surge --dev
```

We need to add surge token to CircleCI as an environment variable for our project. Please update project name in the url;

*https://circleci.com/gh/atolye15/{PROJET_NAME}/edit#env-vars*

On the page, we'll add `SURGE_LOGIN` and `SURGE_TOKEN` envs with the email and token we got before. We're almost ready. Let's update our CircleCI config.

```yaml
# .circleci/config.yml

version: 2
jobs:
  build_dependencies:
    docker:
      - image: circleci/node:10
    working_directory: ~/repo
    steps:
      - checkout
      - attach_workspace:
          at: ~/repo
      - restore_cache:
          keys:
            - dependencies-{{ checksum "package.json" }}
            - dependencies-
      - run:
          name: Install
          command: yarn install
      - save_cache:
          paths:
            - ~/repo/node_modules
          key: dependencies-{{ checksum "package.json" }}
      - persist_to_workspace:
          root: .
          paths: node_modules

  test_app:
    docker:
      - image: circleci/node:10
    working_directory: ~/repo
    steps:
      - checkout
      - attach_workspace:
          at: ~/repo
      - run:
          name: Lint
          command: yarn lint
      - run:
          name: Format
          command: yarn format:check
      - run:
          name: Test
          command: yarn test
      - run:
          name: Coverage
          command: yarn coverage

  deploy_app:
    docker:
      - image: circleci/node:10
    working_directory: ~/repo
    steps:
      - checkout
      - attach_workspace:
          at: ~/repo
      - run:
          name: Deploy
          command: yarn deploy

workflows:
  version: 2
  build_app:
    jobs:
      - build_dependencies
      - test_app:
          requires:
            - build_dependencies
      - deploy_app:
          requires:
            - test_app
          filters:
            branches:
              only: master
```

Let's add deploy script to our `package.json`.

```
"deploy": "sh deploy.sh"
```

Finally, we need to create a `deploy.sh` file.

```sh
echo 'Building application...'
yarn build

echo 'Copying index.html as 404.html'
cp build/index.html build/404.html

echo 'Deploying...'
node_modules/.bin/surge --project ./build --domain cra-starter.surge.sh

echo 'Deployed 🚀'
```

> *NOTE: Of course, you can replace Surge.sh with anything else. For this, you only need to update Surge.sh parts.*

## Step 16: Github Settings

We want to protect our `develop` and `master` branches. Also, we want to make sure our test passes and at lest one person reviewed the PR. In order to do that, we need to update branch protection rules like this in GitHub;

![github-branch-settings](https://user-images.githubusercontent.com/1801024/55028896-df2a3300-5019-11e9-9827-a984fcfa29af.png)

## Step 17 Final Touches

We are ready to develop our application. Just a final step, we need to update our `README.md` to explain what we add a script so far.

```md
This project was set up with following [CRA Recipe](https://github.com/atolye15/cra-recipe).

## Available Scripts

In the project directory, you can run:

### `yarn start`

Runs the app in the development mode.<br>
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will hot reload if you make edits.<br>
You will also see any lint errors in the console.

### `yarn test`

Launches the test runner in the interactive watch mode.<br>
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `yarn coverage`

Launches coverage reporter. You can view the details from `coverage` folder.

### `yarn lint`

Runs ESLint and StyleLint. If any lint errors found, then it exits with code 1.

### `yarn format`

Formats TypeScript and Sass files.

### `yarn format:check`

Checkes if any formatting error has been made.

### `yarn storybook`

Launches Storybook application.

### `yarn build`

Builds the app for production to the `build` folder.<br>
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.<br>
Your app is ready to be deployed!

### `yarn deploy`

Deploys app to given platform according to instuctions in `deploy.sh`

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).
```

## Step 18: Starting to Development 🎉

Everything is done! You can start to develop your next awesome React application now on 🚀

## Related

* [crna-recipe](https://github.com/atolye15/crna-recipe) - React Native App Creation Recipe
