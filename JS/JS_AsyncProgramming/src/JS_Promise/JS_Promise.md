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

### Error handling

The catch block at the end of a Promise chain will only catch the first error that occurs in the chain. It won’t accumulate errors from each then block. If an error occurs at any point in the chain, the catch block will handle that single error and the chain will stop executing further then blocks.

Nesting is a control structure to limit the scope of catch statements. Specifically, a nested catch only catches failures in its scope and below, not errors higher up in the chain outside the nested scope. When used correctly, this gives greater precision in error recovery:

```
doSomethingCritical()
  .then((result) =>
    doSomethingOptional(result)
      .then((optionalResult) => doSomethingExtraNice(optionalResult))
      .catch((e) => {}),
  ) // Ignore if optional stuff fails; proceed.
  .then(() => moreCriticalStuff())
  .catch((e) => console.error(`Critical failure: ${e.message}`));
```

### Chaining after a catch

It's possible to chain after a failure, i.e. a catch, which is useful to accomplish new actions even after an action failed in the chain:

```
doSomething()
  .then(() => {
    throw new Error("Something failed");

    console.log("Do this");
  })
  .catch(() => {
    console.error("Do that");
  })
  .then(() => {
    console.log("Do this, no matter what happened before");
  });
```

This will output the following text:

```
Initial
Do that
Do this, no matter what happened before
```

### Promises and Rejections

When you create a Promise in JavaScript, it can either resolve (success) or reject (failure). If a Promise is rejected and there is no .catch() handler to handle the rejection, the error will “bubble up” the call stack.

“Bubbling up” means that the error will propagate up through the chain of function calls until it reaches the top level of the call stack. If no handler is found to catch the error, it will eventually reach the global context.

The “host” refers to the environment in which your JavaScript code is running, such as a web browser or Node.js. When an unhandled Promise rejection reaches the top of the call stack, the host environment needs to surface it, meaning it needs to notify you about the unhandled error.

```
function doSomething() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('Error in doSomething');
    }, 1000);
  });
}

doSomething();

// No .catch() handler here
```

In this example, the Promise returned by doSomething is rejected, but there is no .catch() handler to catch the error. As a result, the rejection bubbles up to the top of the call stack. You can see
it in dev-console.

Surfacing the Error In a web browser, an unhandled Promise rejection will typically result in an error message being logged to the console. In Node.js, it will emit an unhandledRejection event, which you can listen for and handle if needed.

To avoid unhandled Promise rejections, always include a .catch() handler at the end of your Promise chains:

```
doSomething()
  .catch(error => {
    console.error('Caught error:', error);
  });
```

This ensures that any errors are properly handled and do not bubble up to the top of the call stack.

On the web, whenever a promise is rejected, one of two events is sent to the global scope (generally, this is either the window or, if being used in a web worker, it's the Worker or other worker-based interface). The two events are:

_unhandledrejection_
Sent when a promise is rejected but there is no rejection handler available.

_rejectionhandled_
Sent when a handler is attached to a rejected promise that has already caused an unhandledrejection event.

In both cases, the event (of type PromiseRejectionEvent) has as members a promise property indicating the promise that was rejected, and a reason property that provides the reason given for the promise to be rejected.

In Node.js, handling promise rejection is slightly different. You capture unhandled rejections by adding a handler for the Node.js unhandledRejection event (notice the difference in capitalization of the name), like this:

```
process.on("unhandledRejection", (reason, promise) => {
  // Add code here to examine the "promise" and "reason" values
});
```

For Node.js, to prevent the error from being logged to the console (the default action that would otherwise occur), adding that process.on() listener is all that's necessary; there's no need for an equivalent of the browser runtime's preventDefault() method.

However, if you add that process.on listener but don't also have code within it to handle rejected promises, they will just be dropped on the floor and silently ignored. So ideally, you should add code within that listener to examine each rejected promise and make sure it was not caused by an actual code bug.

### Handling Unhandled Rejections in the Browser

You can add an event listener for the unhandledrejection event on the window object. This event is fired whenever a Promise is rejected and the rejection is not handled by a .catch() block.

```
function doSomething() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject('Error in doSomething');
    }, 1000);
  });
}

// No .catch() handler here
doSomething();

window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled rejection:', event.reason);
  // You can add additional handling logic here
});
```

In this example:

The doSomething function returns a Promise that is rejected after 1 second.
Since there is no .catch() handler for the Promise, the unhandledrejection event is fired.
The event listener logs the error to the console.

You cannot access the Promise properties state and result.

Tracking Promise State

```
function getPromiseState(promise) {
  const t = {};
  return Promise.race([promise, t])
    .then(v => (v === t) ? 'pending' : 'fulfilled', () => 'rejected');
}

// Example usage
const promise1 = new Promise((resolve, reject) => {
  setTimeout(resolve, 1000);
});

const promise2 = new Promise((resolve, reject) => {
  setTimeout(reject, 1000);
});

const promise3 = new Promise((resolve, reject) => {
  // This promise will remain pending
});

getPromiseState(promise1).then(state => console.log(`promise1 is ${state}`)); // pending, then fulfilled
getPromiseState(promise2).then(state => console.log(`promise2 is ${state}`)); // pending, then rejected
getPromiseState(promise3).then(state => console.log(`promise3 is ${state}`)); // pending
```
