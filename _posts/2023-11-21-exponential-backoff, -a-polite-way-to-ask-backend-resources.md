When building a solution related to RPC calls between frontend and backend,
developers need to consider handling errors due to unreliable networking or because
the backend is unavailable at some point. One of the strategies is using a **retry mechanism**.

Looking at this problem from the customer's perspective, having this retry pattern just makes sense. 
**Retry pattern** provides a fallback when there is an error in the downstream call by _just retrying_ 
the same call until it succeeds.

## Retry Pattern
![retry pattern_faisalmorensya.dev](https://github.com/lloistborn/lloistborn.github.io/assets/4990180/7b1028ce-b9cf-4eed-937a-7ea2e8434ead)

The image demonstrates how Frontend is calling Backend; notice that when Frontend calls receive an error, **it retries immediately**. 

However, another issue arose when using this type of retry, and the effect on our Backend is the same as sending a DDos type of attack.

## Polite Way of Retrying using Exponential Backoff
Below is the updated scenario based on the diagram above. I added Exponential Backoff into concern for each retry except the first.

![faisalmorensyadotdev-resources drawio (1)](https://github.com/lloistborn/lloistborn.github.io/assets/4990180/648e6b73-9382-4dec-95f0-5f2ec32fea8c)

<span>**Let's** start with the first request that leads to an error response.</span><br>
<span>**Instead of immediately** retrying the request, the call waits 2 seconds.</span><br>
<span>**The system then** retries the same call after waiting 2 seconds.</span><br>
<span>**Notice** that the wait time is increased to 4 seconds before it retries</span><br>
<span>**and eventually** gets a successful response.</span><br>
<span>This is how **Exponential Backoff** works. </span>


## Implementing it using JavaScript
Here is how we can implement a combination of Retry Pattern with Exponential Backoff
```javascript
function waitFor(milliseconds) {
  return new Promise((resolve) => setTimeout(resolve, milliseconds));
}

function retry(promise, onRetry, maxRetries) {
  async function retryWithBackoff(retries) {
    try {
      // Make sure we don't wait on the first attempt
      if (retries > 0) {
        // Here is where the magic happens.
        // on every retry, we exponentially increase the time to wait.
        // Here is how it looks for a `maxRetries` = 4
        // (2 ** 1) * 100 = 200 ms
        // (2 ** 2) * 100 = 400 ms
        // (2 ** 3) * 100 = 800 ms
        const timeToWait = 2 ** retries * 100;
        console.log(`waiting for ${timeToWait}ms...`);
        await waitFor(timeToWait);
      }
      return await promise();
    } catch (e) {
      if (retries < maxRetries) {
        onRetry();
        return retryWithBackoff(retries + 1);
      } else {
        console.warn("Max retries reached");
        throw e;
      }
    }
  }

  return retryWithBackoff(0);
}
```

Now, let's test the implementation.
```javascript
function generateFailableAPICall() {
  let counter = 0;
  return function () {
    if (counter < 3) {
      counter++;
      return Promise.reject(new Error("Simulated error"));
    } else {
      return Promise.resolve({ status: "ok" });
    }
  };
}

/*** Testing our Retry with Exponential Backoff */
async function test() {
  const apiCall = generateFailableAPICall();
  const result = await retry(
    apiCall,
    () => {
      console.log("onRetry called...");
    },
    4
  );

  assert(result.status === "ok");
}

test();
```

_code is based on this [reference](https://bpaulino.com/entries/retrying-api-calls-with-exponential-backoff)_
