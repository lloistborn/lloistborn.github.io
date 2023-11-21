# Overview
When it comes to building a solution related to RPC calls between frontend and backend, 
developers need to think about handling errors due to unreliable networking or just because 
the backend is not available at some point. One of the strategies is using a **retry mechanism**.

Looking at this problem from the customer's perspective, it just makes sense to have this
retry pattern. It provides a fallback when there is an error in the downstream call by _just retrying_ the same call
until it succeeds.

