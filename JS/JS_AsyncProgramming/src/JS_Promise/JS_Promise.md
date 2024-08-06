# Promise

## Chaining Promises

When we need subsequent operations starts when the previous operation succeeds,
with result from the previous step. In the old days, doing several async operations
in a row would lead to the classic callback hell:

```
doFirstLevel(result => {
  doSecondLevel((result, newResult) => {
    doThirdLevel((newResult, finalResult) => {
      console.log('Finish' + finalResult)
    }, failureCallback),
  }, failureCallback)
}, failureCallback);
```

To not see that hell, we going to use promise chain:

```
function doSomething() {
  return new Promise((resolve, reject) => {
    // Simulate an asynchronous operation
    setTimeout(() => {
      const success = Math.random() > 0.5; // Simulate success or failure
      if (success) {
        resolve('Operation was successful!');
      } else {
        reject('Operation failed.');
      }
    }, 1000);
  });
}

function successCallback(result) {
  console.log(`Success: ${result}`);
}

function failureCallback(error) {
  console.error(`Error: ${error}`);
}

const promise = doSomething();
const promise2 = promise.then(successCallback, failureCallback);
```

_promise2_ - represents the completion not just of _doSomething()_, but also of the _successCallback_
and _failureCallback_ you passed in- which can be other asynchronous functions returning a promise.

```
function doSomething() {
  return new Promise((resolve, reject) => {
    // Simulate an asynchronous operation
    setTimeout(() => {
      const success = Math.random() > 0.5; // Simulate success or failure
      if (success) {
        resolve('Operation was successful!');
      } else {
        reject('Operation failed.');
      }
    }, 1000);
  });
}

function successCallback(result) {
  return new Promise((resolve) => {
    console.log(`Success: ${result}`);
    // Simulate another asynchronous operation
    setTimeout(() => {
      resolve('Success callback completed.');
    }, 1000);
  });
}

function failureCallback(error) {
  return new Promise((resolve) => {
    console.error(`Error: ${error}`);
    // Simulate another asynchronous operation
    setTimeout(() => {
      resolve('Failure callback completed.');
    }, 1000);
  });
}

const promise = doSomething();
const promise2 = promise.then(successCallback, failureCallback);

promise2.then((message) => {
  console.log(message); // This will log the message from either successCallback or failureCallback
});
```

Lets define the order of execution for more clearence:
promise\[doSomething()\] ----resolve('Operation was successful!')----> then\[successCallback(Operation was successful!)\] ---
--> return promise('Operation was successful!') -----> assign to promise2 -----> run promise2 ---> successCallback(resolved value : Success callback completed.)

Starting doSomething
doSomething started
doSomething resolved
successCallback started
Success: Operation was successful!
successCallback resolved
Final message: Success callback completed.

With this pattern, you can create longer chains of processing, where each promise represents the completion of one asynchronous step in the chain.
In addition, the arguments to then are optional, and catch(failureCallback) is short for then(null, failureCallback) — so if your error handling
code is the same for all steps, you can attach it to the end of the chain:

```
doSomething()
.then((result) => doSecondLevel(result))
.then((newResult) => doThirdLevel(newResult))
.then((finalResult) => console.log(`${finalResult}`))
.catch(failureCallback);
```

doSecondLevel, doThirdLevel - can return any value — if they return promises, that promise is first waited until it settles, and the next callback
receives the fulfillment value, not the promise itself. It is important to always return promises from then callbacks, even if the promise always
resolves to undefined. If the previous handler started a promise but did not return it, there's no way to track its settlement anymore, and the promise
is said to be "floating":

```
doSomething()
  .then((url) => {
    // Missing `return` keyword in front of fetch(url).
    fetch(url);
  })
  .then((result) => {
    // result is undefined, because nothing is returned from the previous
    // handler. There's no way to know the return value of the fetch()
    // call anymore, or whether it succeeded at all.
  });
```

By returning the result of the fetch call (which is a promise), we can both track its completion and receive its value when it completes.

```
doSomething()
  .then((url) => {
    // `return` keyword added
    return fetch(url);
  })
  .then((result) => {
    // result is a Response object
  });
```

Floating promises could be worse if you have race conditions — if the promise from the last handler is not returned, the next then handler
will be called early, and any value it reads may be incomplete.

```
const listOfIngredients = [];

doSomething()
  .then((url) => {

    // Missing `return` keyword in front of fetch(url).

    fetch(url)
      .then((res) => res.json())
      .then((data) => {
        listOfIngredients.push(data);
      });
  })
  .then(() => {
    console.log(listOfIngredients);
    // listOfIngredients will always be [], because the fetch request hasn't completed yet.
  });
```

Therefore, as a rule of thumb, whenever your operation encounters a promise, return it and defer its handling to the next then handler.

```
const listOfIngredients = [];

doSomething()
  .then((url) => {
    // `return` keyword now included in front of fetch call.
    return fetch(url)
      .then((res) => res.json())
      .then((data) => {
        listOfIngredients.push(data);
      });
  })
  .then(() => {
    console.log(listOfIngredients);
    // listOfIngredients will now contain data from fetch call.
  });
```

Even better, you can flatten the nested chain into a single chain, which is simpler and makes error handling easier. So then can return
not only Promise but also a simple value _return 'Value'_ when it return anything that value moves to the first argument of the next
then(). In example below:

1. doSomething() is a promise. When promise resolved in url will be value defined in resolve promise method.
2. Then value passed to fetch - which is also a promise, so the next then((res)...) will wait until the first one finish completly.
3. In success fetch will return the res and we will sequently return it to next then((data)...) in data variable. Data will be assigned
   to the list.
4. We didn't return anything from 3-d then and wrote the forth then(). It's okay because then() will execute consequntly until it encounter
   the error. Return required only for the promises, in arrow functions one row logic not require return statement.

```
doSomething()
  .then((url) => fetch(url))
  .then((res) => res.json())
  .then((data) => {
    listOfIngredients.push(data);
  })
  .then(() => {
    console.log(listOfIngredients);
  });
```
