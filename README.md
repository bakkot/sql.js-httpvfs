# A fork of sql.js-httpvfs which bounds the number of requests

This is a fork of https://github.com/phiresky/sql.js-httpvfs/ which includes a hack which allows you to bound the number of a messages a query or set of queries makes, at the cost of rendering the in-memory database unuseable once the bound is reached.

## Usage

I assume you're already familiar with the usage of the original, so I'm only going to highlight relevant differences:

```js
// As usual, create the db worker.
const worker = await createDbWorker(...);

// The returned object has a new "port" property.
// This will receive the below message when the bound is exceeded.
// At this point the in-memory DB is corrupt.
// At is your responsibility to surface this to the user
// and to avoid performing any more queries.
worker.port.onmessage = e => {
  if (e.data === 'too many requests') {
    alert('too many requests');
  }
};

// By default there is no bound on the number of requests.
// You can set the bound by sending a message like below
worker.port.postMessage({ type: 'set-bound', bound: 50 });

// You can reset the counter like below.
// It will keep incrementing until you reset it, including across queries.
// E.g., you might reasonably do this before each query.
// Note that this is only meaningful before the bound is hit,
// since once the bound is hit, the in-memory DB is corrupt.
worker.port.postMessage({ type: 'reset-count' });

// Perform your queries like usual.
// This will throw if it causes the bound to be hit.
let results = await worker.db.query(cmd);
```
