

## Overview
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
<span> This is how** Exponential Backoff** works. </span>


## Implementing it using TypeScript
<div>
  <img src="https://github.com/lloistborn/lloistborn.github.io/assets/4990180/d0cf1250-bf8c-45bc-bf21-f548dde402a8" alt="drawing" width="25" height="25"/>
  <img src="https://github.com/lloistborn/lloistborn.github.io/assets/4990180/c238cd29-b0d4-4fa1-8945-15662edd13bf" alt="drawing" width="25" height="25"/>
  <img src="https://github.com/lloistborn/lloistborn.github.io/assets/4990180/edf7231d-2122-4871-821a-deeff2a32116" alt="drawing" width="25" height="25"/>
</div> <be>

I love TypeScript, so lets implement using it

Here is how we can implement a combination of Retry Pattern with Exponential Backoff
```
// code here 
```
