## Overview
When building a solution related to RPC calls between frontend and backend,
developers need to think about handling errors due to unreliable networking or just because
the backend is unavailable at some point. One of the strategies is using a **retry mechanism**.

Looking at this problem from the customer's perspective, having this retry pattern just makes sense. 
**Retry pattern** provides a fallback when there is an error in the downstream call by _just retrying_ 
the same call until it succeeds.

## Retry Pattern
![retry pattern drawio (2)](https://github.com/lloistborn/lloistborn.github.io/assets/4990180/7b1028ce-b9cf-4eed-937a-7ea2e8434ead)

The image demonstrates how Frontend is calling Backend, notice that when Frontend calls receive an error, **it retries immediately**. Another issue arose when using this type of retry and the effect on our Backend is the same as sending a DDos type of attack.
