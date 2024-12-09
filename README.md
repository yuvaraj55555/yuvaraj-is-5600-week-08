# Lab 08 | Fullstack Prints Part - Integration Testing

## Table of Contents

1. [Lab 08 | Fullstack Prints Part - Integration Testing](#lab-08--integration-testing)
   - [Overview](#overview)
   - [Instructions](#instructions)
2. [Guidance and Testing](#guidance-and-testing)
3. [Submission](#submission)
4. [Getting Started with GitHub and Codespaces](#getting-started-with-github-and-codespaces)
   - [Step 1: Fork the Repository](#step-1-fork-the-repository)
   - [Step 2: Open the Repository in Codespaces](#step-2-open-the-repository-in-codespaces)
   - [Step 3: Complete the Lab Assignment](#step-3-complete-the-lab-assignment)
   - [Step 4: Submit Your Work via Pull Request](#step-4-submit-your-work-via-pull-request)

## Overview

For this lab, we will be adding unit tests to our Fullstack Prints application using Jest. We will be using NPM to install Jest and write tests for our models, routes, and other application logic. This will be based on our work from the previous Fullstack Prints labs, specifically Week 5. 


## Instructions

1. For this lab, we will return to our NodeJS codebase. Take a moment to quickly review the code from the previous labs. In this codebase, you'll see the following key files:

- `app.js` - Main application entry point. It is responsible for setting up the express application and starting the server.
- `api.js` - Contains route definitions for our API endpoints.
- `products.js` - Contains the Products model and business logic for CRUD operations on products.
- `orders.js` - Contains the Orders model and business logic for CRUD operations on orders.

2. Let's begin the lab by installing Jest. In previous labs we have had all of the dependencies available to you at the start of the lab. But for this exercise, we want to install Jest ourselves. Open a terminal in your Codespace, and type the following:

```
$ npm install --save-dev jest supertest
```

Here, the `--save-dev` flag indicates that this is dependency that should only be available during the development build of the application. We are telling NPM that this is not a dependency that will be required by the production or final build of the application. 

You'll also note that we are installing a second dependency, `supertest`. This package is often used for making HTTP test assertions. 

To learn more about NPM, see our course videos, or review the [NPM documentation](https://docs.npmjs.com/).

3. Next, we need to create our first test. Our first test will be simply to establish that the server has booted up successfully. We'll add tests to verify our Orders and Products modules later. Begin by creating a file called `app.test.js`. This file will reside in the `tests` directory. Where to place test files is often up to the project maintainer. Some developers choose to place their tests alongside the domain code, others choose to place them in a separate directory. For the simplicity of this lab, we will place them in a separate directory.

Place the following code in your `app.test.js` file:

```js
// tests/app.test.js
const request = require('supertest');
const app = require('../app.js');

describe('The Express Server', () => {
  beforeAll(done => {
    done()
  })

  test('should return response', async () => {
    const res = await request(app)
      .get('/');
    expect(res.statusCode).toEqual(200);
  });
});
```

This performs a very simple test, it will simply check that the server has booted up, and ensure that the HTTP status returned by the server is `200`. If an error is encountered, that error will be notified to the console.

The `beforeAll` callback is run before each test in the test suite is run, this ensures that the previous job is done, and any dangling services have been shut down.

4. Great! We have our first test, but now we need to run it. We will continue to use the terminal to test our app. Open a new terminal in your Codespace, and type the following:

```
$ npm run test
```
This will run your Jest tests in the terminal window. If everything is configured correctly, you should see a successful status!

5. Now, we'll want to verify that all of the required routes are registered. This is what is called "smoke testing". Let's write our first smoke test:

```js
// tests/app.test.js
describe('The Express Server', () => {
  beforeAll(done => {
    done()
  })

  test('should return response', async () => {
    const res = await request(app)
      .get('/');
    expect(res.statusCode).toEqual(200);

  });

  test('should respond at /products', async () => {
    const res = await request(app)
      .get('/products')
    expect(res.statusCode).toEqual(200);
  });
});
```

Now, on your own, add a test for the `/orders` route. It should look very similar to the `/products` route test we just wrote. We want to check that the route exists, and that it returns a 200 response.

Excellent, now we could continue to write smoke tests for the API, and there is value in that, but if we were to disconnect the `Products` or `Orders` modules from the API controller, our tests would fail. We'd have no way to verify that our `Products` or `Orders` modules worked correctly. So instead, let's begin to write some tests for the Products module.

6. Create a new test file to contain our test suite: `tests/products.test.js`. Now, we want to first make sure we actually have some data to work with. We have a helper utility already in place to load test data into the database. Let's use that instead of writing out our own test data. You can view the helper in the `test-utils` directory.

This helper does a few things:
- It loads the test data into the database
- It provides a function to clean up the test data after all tests have completed

Now we can use this helper in our test suite. Create `tests/products.test.js`:

```js
// tests/products.test.js
const productTestHelper = require('./test-utils/productTestHelper');
const { list } = require('../products');

describe('Product Module', () => {
  // Set up and clean up test data
  beforeAll(async () => {
    await productTestHelper.setupTestData();
  });

  afterAll(async () => {
    await productTestHelper.cleanupTestData();
  });

  // your tests go here
});
```

This setup provides several benefits:
1. Separates test data management into a reusable helper
2. Properly cleans up test data after tests complete
3. Makes the test file cleaner and more focused on test cases
4. Allows other test files to reuse the same data setup/cleanup logic

The `beforeAll` and `afterAll` hooks ensure that:
- Test data is loaded once before any tests run
- All test data is cleaned up after tests complete
- The database is left in a clean state for the next test run

So now when you run `npm run test`, Jest will call `setupTestData()` before running any tests, and call `cleanupTestData()` after all tests complete. Now open your terminal and run `npm run test` again to verify that your test suite works. 

Next, we'll add our first test. This test will verify that we can retireve a list of products from the database:

```js
// tests/product.test.js

// This test goes in the greater `describe('Product Module')` function
describe('list', () => {
  it('should list all products', async () => {
    const products = await list();
    expect(products.length).toBeGreaterThan(0);
  });
});
```

Now when we review the test results, we should see something like this:

```
Test Suites: 2 passed, 2 total
Tests:       3 passed, 3 total
Snapshots:   0 total
Time:        5.341 s
```

Now this is fine, but things will get tricky if we start to test getting, creating or deleting individual products. This is because our test are actually integration tests, using the actual MongoDB database to run requests against. If we were to do something like this below: 

```js
describe('get', () => {
  it('should retrieve a product by id', async () => {
    // Assume there is a product with id 'abc123'
    const product = await get('abc123');
    expect(product).not.toBeNull();
  });
});
```

We'd run into an issue because we'd need to know what the product ID was before running the test. This gets further complicated when we start testing the delete method, because
the product would be deleted after the test. That would mean we would never be able to run the test again, it would not be sustainable or deterministic.

```js
// This would delete the product after the first test, and fail every other test afterwards.
 describe('destroy', () => {
  it('should delete a product', async () => {
    // Assume there is a product with id 'abc123'
    await destroy('abc123');
    const product = await get('abc123');
    expect(product).toBeNull();
  });
});
```

So in our next step, let's setup some mocking. Using mocks we can isolate the unit of work we are testing, ensuring that tests run quickly and are not affected by external dependencies (such as deletes on our database).

** Note: ** In this lab, we are actually setting up and tearing down the database to ensure that all tests are run with a clean state. However, we do not always have the option to do this, especially if we are using a shared database or an external database. In those cases, we can use mocks to simulate the behavior of the database. The rest of this lab will be focused on using mocks to simulate the behavior of the database.

7. To configure our test to use mocks, we'll need to begin by defining the mock callback. At a high level, a mock is a simulated object or function that mimics the behavior of real objects in controlled ways. It's a kind of test double, which is a generic term for any case where you replace a production object for testing purposes.

So let's configure our mocks. First we'll need to create a new file to hold the mocks: `tests/db.mock.js`. Next add the following to the file:

```js
// tests/db.mock.js

/**
 * Mock data to be returned by our mock database queries.
 * This simulates the documents we'd typically get from MongoDB.
 */
const mockProducts = [
    { description: 'Product 1' },
    { description: 'Product 2' }
];

/**
 * Mock Mongoose Query object.
 * This simulates Mongoose's chainable query interface.
 * For example, in real Mongoose you can do: Model.find().sort().skip().limit()
 * 
 * mockReturnThis() is used to make methods chainable by returning 'this'
 * exec() and then() both resolve with our mockProducts to simulate a DB response
 */
const mockQuery = {
    sort: jest.fn().mockReturnThis(),  // Returns 'this' to allow chaining
    skip: jest.fn().mockReturnThis(),  // Returns 'this' to allow chaining
    limit: jest.fn().mockReturnThis(), // Returns 'this' to allow chaining
    exec: jest.fn().mockResolvedValue(mockProducts),  // Simulates DB query execution
    then: function(resolve) { resolve(mockProducts) }  // Makes the query thenable (Promise-like)
};

/**
 * Mock Mongoose Model object.
 * This simulates the methods available on a Mongoose model (e.g., Product model).
 * The find() method returns our mockQuery to allow for method chaining.
 */
const mockModel = {
    find: jest.fn().mockReturnValue(mockQuery)
};

/**
 * Mock DB object that simulates the mongoose db interface.
 * In real code, we use db.model('Product') to get the Product model.
 * Here, we return our mockModel whenever model() is called.
 */
const mockDb = {
    model: jest.fn().mockReturnValue(mockModel)
};

module.exports = {
    mockDb,        
    mockProducts, 
    mockModel,     
    mockQuery     
};

```

Next, let's update our `product.test.js` file so that it uses our mocks instead of the actual database module.

```js
// tests/product.test.js
const { mockDb, mockProducts } = require('./db.mock');
const { list } = require('../products');

// Mock the db module to use our mockDb
jest.mock('../db', () => mockDb);

describe('Product Module', () => {
    beforeEach(() => {
        jest.clearAllMocks();
    });

    // your tests go here
});
```

Lastly, we'll need to modify our test that checks for the list of products to be returned:

```js
// tests/products.test.js


// replace your current list test with this below:
  describe('list', () => {
      it('should list products', async () => {
          const products = await list();
          expect(products.length).toBe(2);
          expect(products[0].description).toBe('Product 1');
          expect(products[1].description).toBe('Product 2');
      });
  });
```

Alright - open your terminal and run `npm run test`. You should see all tests passing! What is happening here is instead of using the `mongoose` database module that we normally use, we are replacing it with our jest mock objects that exist in the `tests/db.mock.js` file. So when we call `.sort()` or `.find()` on the `db` module, instead of the "real" mongoose methods being called, we are using the ones we configured in `db.mock.js`. This allows us to produce stateless deterministic test results, and not worry about our delete or create tests modifying the database.

8. Mocking our tests is a great solution, but can lead down a rabbit hole, where our tests are more complicated than our actual application code. For instance, if we were to mock out the create and delete methods for our tests, we'd almost have to rebuild our entire application code. Luckily, Mongodb has a Jest package that we can use which actually creates an in memory lite weight version of the database. This is similar to how Java test suites use H2 or SQL Lite. We will use this to build out our test suite for the `orders`. 

Let's begin by installing the package, open your terminal and run `npm install @shelf/jest-mongodb --save-dev`. This will install the Jest MongoDB package. 

Now we need to edit a config file, to enable the in-memory databse. Open the `jest.config.js` file and uncomment or add the following:

```js
module.exports = {
  preset: '@shelf/jest-mongodb', // uncomment this line!
  testEnvironment: 'node',
};
```

Next, let's create a new test suite file for our orders: `tests/orders.test.js`. Because orders require products, we'll need to do a bit of setup first.  

```js
// tests/orders.test.js
const { create, get, list, edit } = require('../orders');
const orderData = require('../data/order1.json');
const productTestHelper = require('../test-utils/productTestHelper');

describe('Orders Module', () => {
 
  let createdProduct;
  let createdOrder;

  // Populate the database with dummy data
  beforeAll(async () => {
    await productTestHelper.setupTestData();
    await productTestHelper.createTestOrders(5);
  });

  afterAll(async () => {
    await productTestHelper.cleanupTestData();
  });

  describe('list', () => {
    it('should list orders', async () => {
      const orders = await list();
      expect(orders.length).toBeGreaterThan(4);
    });
  });
});
```

*Note there is some configuration that is occuring behind the scenes for this to work. You can watch the course video for some explanation*

Now, if you run this test using `npm run test` you'll see it fail. That is expected! We've not created any orders in the databse, we're requesting orders,
but nothing has been created. Let's remedy this with a quick tests to create an order:

```js
// tests/orders.test.js

describe('create', () => {
    it('should create an order', async () => {
      createdOrder = await create(orderData);
      expect(createdOrder).toBeDefined();
      expect(createdOrder.buyerEmail).toBe(orderData.buyerEmail);
    });
  });
```

Run our `npm run test` again and we should see a successful pass of all our tests!


## Your Task

Now we have a fully functioning test suite for orders, products and our core app. Let's improve it a bit by adding a few more tests. This will be your responsiblity.

1.  ### Add "get" test to orders

  For this task you'll need to create a new test on the `orders.test.js` suite. This task will require you calling the get method, using the `createdOrder._id`
  
  ```js
  const order = await get(createdOrder._id);
  ```

  You'll then want to assert that the returned order id matches, and that the returned order is defined:
  
  ```js
  expect(order).toBeDefined();
  ```

2. ### Add "edit" test to orders

  For this task you'll need to create a new test for the `orders.test.js` suite. You'll want to create a change on the order, and call that change:

  ```js
    const change = {} // implement your change here
    const editedOrder = await edit(createdOrder._id, change)
  ```
  
  You'll then want to assert that the `editedOrder` exists or is defined, and that the change which was made exists on the new `editedOrder` object.

3. ### Add "get" test to products

  For this task you'll be creating a new test for the `products.test.js` suite. You'll want to create a mock response for the get method:

  ```js
  describe('get', () => {
    it('should get a product by id', async () => {
      // Mock the Product.findById method to return a specific product
      mockModel.findById = jest.fn().mockResolvedValue({ description: 'Product 1' });

      // call to get the product using the `get` method
      // your assertions
    });
  });
  ```

  You'll then make assertions to verify that the product description is correct.

4. ### Add "destroy" test to products

  For this task you'll be creating a new test for the `products.test.js` suite. You'll want to create a mock response for the delete method. Use the template provided above:

  ```js
  mockModel.deleteOne = jest.fn().mockResolvedValue({ deletedCount: 1 });
  ```

  Your assertion can then be to verify that the "deleteCount" is correct. 

While writing out the mocks, it should be apparent one of the downsides of mocking - our tests making lots of assumptions about our application code. If we were to change how the application code handled the delete method for instance, our tests would still pass until we updated the way we are mocking the test. Mocks are not _bad_ necessarily, it is just important to understand the trade offs.

## Guidance and Testing

1. You can re-run your tests using the `npm run test` command. Or you can run `npm run test --watch` to run tests as you make changes.
2. For this lab you will be using the terminal in your Codespace. This is not to be confused with the Console. Please make sure you are calling commands from the terminal.

## Submission

Once you have completed the lab, please submit your lab by committing the code and creating a pull request against the `main` branch of your forked repository.

Once you have a URL for your Pull Request, submit that URL with a brief message in Canvas against the Assignment.

# Getting Started with GitHub and Codespaces

Welcome to the course! In this guide, you’ll learn how to set up your coding environment using GitHub and Codespaces. By following these steps, you’ll be able to work on your lab assignments, write and test your code, and submit your work for review. Let's get started!

## Step 1: Fork the Repository

Forking a repository means making a copy of it under your GitHub account. This allows you to make changes without affecting the original project.

1. **Open the Repository**: Start by navigating to the GitHub repository link provided by your instructor.
2. **Click "Fork"**: In the top-right corner, find the “Fork” button and click it.
3. **Select Your Account**: Choose your GitHub account as the destination for the fork. Once done, you’ll be redirected to your forked copy of the repository.

   > **Tip**: Make sure you’re logged into your GitHub account, or you won’t see the option to fork!

## Step 2: Open the Repository in Codespaces

With your forked repository ready, you can now set up a development environment using Codespaces. This setup provides a pre-configured environment for you to code in, with everything you need to complete the lab.

1. **Open the Codespaces Menu**:
   - In your forked repository, click the green "Code" button, then switch to the "Codespaces" tab.
2. **Create a Codespace**:
   - Click on "Create codespace on main" to start the setup.
3. **Wait for Codespaces to Load**:
   - It may take a few minutes for Codespaces to create and configure your environment. Be patient, as it’s setting up all the tools you’ll need.
4. **Start Coding**:
   - Once the setup is complete, Codespaces will automatically open a new browser tab where your code will be ready to run. You’ll be able to see the code and any outputs as you go through the lab assignment.

## Step 3: Complete the Lab Assignment

Inside the Codespaces environment, you’ll find all the files and instructions you need. Follow the steps outlined in the README file to complete your assignment.

1. **Read the Instructions**: Carefully go through the README file to understand the tasks you need to complete.
2. **Edit the Code**: Make the necessary changes to the code files as instructed.
3. **Run and Test Your Code**: Use the terminal and editor within Codespaces to run your code and make sure everything works as expected.

   > **Hint**: If you’re stuck, try reviewing the README file again or refer to any resources provided by your instructor.

## Step 4: Submit Your Work via Pull Request

Once you’ve completed the assignment, it’s time to submit your work. You’ll do this by creating a pull request, which is a way to propose your changes to the original repository.

1. **Commit Your Changes**:
   - Save your work by committing your changes. In Codespaces, go to the Source Control panel, write a commit message, and click "Commit" to save your changes.
2. **Push to Your Fork**:
   - After committing, click "Push" to upload your changes to your forked repository on GitHub.
3. **Create a Pull Request**:
   - Go back to your GitHub repository, and you’ll see an option to “Compare & pull request.” Click it to start your pull request.
   - Include your name in the pull request description so your instructor knows who submitted it.
4. **Submit the Pull Request**:
   - Click "Create pull request" to submit your work for review. Your instructor will be notified and can review your work.

And that’s it! You’ve now completed your first lab assignment using GitHub and Codespaces. Well done!

### Additional Steps

1. Open the terminal in Codespaces.
2. Run the following commands to install dependencies and start the development server:

    ```sh
    npm install
    npm run dev
    ```

3. You can now view the project in the browser by clicking the "Application" port in the Ports panel.

Follow the instructions in the previous sections to complete the lab.