# Lab: Unit Tests

In this lab, you will learn how to:

* Run the existing suite of unit tests
* Structure Unit Tests
* Add component unit tests

## Run the Tests

The application is configured to use [jest](https://jestjs.io) to run our tests. The `npm test` script will run all of our tests and then watch for changes to the code.

Type `npm test` and verify that the tests run.

If you would like to run the tests once without the watch, you can do something like this: `(export CI=true; npm test)`

## Test Structure

The basic Jest test structure is a sinlge file with Setup and Teardown code and individual tests cases. Jest also supports grouping test cases together in nested blocks, which allows you to group test together by functionality.

### Setup and Teardown

Often when writing tests there is some initialization that needs to occur before each test is run and some cleanup that needs to occur after each test has been run. Jest provides the following methods to do this:

- `beforeAll` - run once before any test in the file or group
- `beforeEach` - run before each test in the file or group
- `afterAll` - run once at the completion of all tests in the file or group
- `afterEach` - run after the completion of each test in the file or group

### Grouping Tests

Sometimes test logically belong grouped together. For example, tests that exercise a particular method. Often these are tests that also need to share specific setup or and teardown code. Tests are grouped together using the `describe()` method. Some important aspects of a `describe()` group are:

- they can be nested inside of another group
- they can have their own setup and teardown routines which are run in addition to the setup and teardown of the file or enclosing groups

## Refactor `App.test.tsx`

### Add Setup and Teardown Code

All of our tests will need to boostrap the component. When the tests are completed, they should unmount the container for the component and remove it from the DOM. The code to do that looks like this:

```TypeScript
let container: Element;

beforeEach(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
  act(() => {
    render(<App />, container);
  });
});

afterEach(() => {
  unmountComponentAtNode(container);
  container.remove();
});
```

**Note:**  In order for this to compile, the `import` statements will need some adjustment. We will deal with that next.

### Simplify the "renders without crashing" Test

With the setup and teardown in place, we can greatly simplify the "renders without crashing" test:

```TypeScript
it('renders without crashing', () => {
  expect(container.innerHTML).toBeTruthy();
})
```

Along with the required changes to our `import` statements, the full code for the test should now look like this:

```TypeScript
import React from 'react';
import { render, unmountComponentAtNode } from 'react-dom';
import { act } from 'react-dom/test-utils';
import App from './App';

let container: Element;

beforeEach(() => {
  container = document.createElement('div');
  document.body.appendChild(container);
  act(() => {
    render(<App />, container);
  });
});

afterEach(() => {
  unmountComponentAtNode(container);
  container.remove();
});

it('renders without crashing', () => {
  expect(container.innerHTML).toBeTruthy();
});
```

### Test the Tabs

## Conclusion

In this lab we learned the basics of unit testing. We will apply what we learned here and expand upon it as we develop our application. Next we will have to add some Cordova platforms so we can run the application on devices.
