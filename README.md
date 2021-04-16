# Nimiq Svelte Stores

This library provides [SvelteJS](https://svelte.dev) stores for a [Nimiq](https://nimiq.com) Blockchain Client.

Client initialization is already handled for you (mainnet, pico client).
You simply import the stores that you need.

- [Example](#example)
- [Setup](#setup)
- [Start](#start)
- [Stores](#stores)
- [Writable Stores](#writable-stores)
- [Client](#client)
- [Config](#config)

## Example

- [Example app on Netlify](https://nimiq-svelte-stores.netlify.com)
- [Example code](https://github.com/sisou/nimiq-svelte-stores/blob/master/example/App.svelte)

## Setup

1. Install this library from [NPM](https://www.npmjs.com/package/nimiq-svelte-stores):
    ```bash
    yarn add --dev nimiq-svelte-stores
    ```
2. Add the Nimiq script _before_ the bundle in your `public/index.html`:
   ```html
   <script defer src="https://cdn.nimiq-testnet.com/v1.5.2/web.js"></script>
   ```
3. Import Nimiq stores into your components and start the client, see next sections.

## Start

To start the Nimiq Client, call the exported `start` function:

```js
import { start } from 'nimiq-svelte-stores'

start()
```

Advanced: [Learn how to configure the Nimiq Client](#config).

## Stores

```js
import {
    ready,
    consensus,
    established,
    headHash,
    head,
    height,
    networkStatistics,
    peerCount,
    accounts,
    accountsRefreshing,
    newTransaction,
    transactions,
    transactionsRefreshing,
} from 'nimiq-svelte-stores'
```

| Store | Type | Initial value |
|-------|------|---------------|
| ready | Boolean | `false` |
| consensus | String | `'loading'` |
| established | Boolean | `false` |
| headHash | [Nimiq.Hash](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/consensus/base/primitive/Hash.js~Hash.html) | `null` |
| head | [Nimiq.Block](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/consensus/base/block/Block.js~Block.html) | `null` |
| height | Number | `0` |
| networkStatistics | [Nimiq.Client.NetworkStatistics](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/api/NetworkClient.js~NetworkStatistics.html) | `Object` |
| peerCount | Number | `0` |
| accounts | Array<{address: [Nimiq.Address](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/consensus/base/account/Address.js~Address.html), ...}> | `[]` |
| accountsRefreshing | Boolean | `false` |
| newTransaction | [Nimiq.Client.TransactionDetails](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/api/TransactionDetails.js~TransactionDetails.html) | `null` |
| transactions | Array<[Nimiq.Client.TransactionDetails](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/api/TransactionDetails.js~TransactionDetails.html)> | `[]` |
| transactionsRefreshing | Boolean | `false` |

## Writable Stores

The `accounts` and `transactions` stores expose methods to write to them and trigger actions.

### accounts.add()

You can add one or more accounts to the `accounts` store by passing the following types to the `accounts.add()` method:

* [`Nimiq.Address`](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/consensus/base/account/Address.js~Address.html)
* `string` (userfriendly, hex or base64 address representation)
* `Object<{address: Nimiq.Address | string}>`
* Array of these types

If you add objects, they can include whatever properties you like, as long as they have an `address` property
which can be interpreted as a Nimiq address. All other properties on the object are preserved and added to the store.
You can use this for example to store an account `label` or other meta data in the `accounts` store.

>The `accounts` store automatically populates and updates accounts' `balance` and `type` fields from the blockchain,
>while consensus is established (as well as other relevant fields for [vesting contracts](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/consensus/base/account/VestingContract.js~VestingContract.html) and [HTLCs](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/consensus/base/account/HashedTimeLockedContract.js~HashedTimeLockedContract.html)).

### accounts.remove()

Accounts may be removed from the `accounts` store with the `accounts.remove()` method.
The method takes the same arguments as the `add` method above.

### accounts.refresh()

The `accounts.refresh()` method can be used to manually trigger a blockchain sync of some or all stored `accounts`.
When passed any of the arguments accepted by the `add` method, only those accounts are refreshed.
If no argument is passed, all stored accounts are refreshed.

### transactions.add()

You can add single or an array of known [`Nimiq.Client.TransactionDetails`](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/api/TransactionDetails.js~TransactionDetails.html) to the `transactions` store with
the `transactions.add()` method.
These transaction details are then used when fetching the transaction history for an account, and prevent
a great amount of data from being downloaded again.

>When subscribing to the `transactions` store, the transaction history for all stored and newly added accounts
>is automatically fetched while consensus is established.

### transactions.refresh()

The `transactions.refresh()` method can be used to manually trigger fetching the transaction history of some
or all stored `accounts`. When passed any of the arguments accepted by the `accounts.add` method, only the histories
of those accounts are refreshed. If no argument is passed, the history of all stored accounts is refreshed.

### transactions.setSort()

The `transactions.setSort()` method allows you to overwrite the library's transaction-sorting algorithm.
(By default, transactions are sorted 'newest first'.)

Pass your custom sort function to the `setSort()` method.
Your sort function receives two arguments, both `Nimiq.Client.TransactionDetails`, and is required to return a number:
`< 0` if the first argument should be sorted first, `> 0` if the second argument should be sorted first, `0` if both arguments should be sorted equally.

```js
// Sort transactions by their hash value, ascending
function customSort(a, b) {
    const aValue = parseInt(a.transactionHash.toHex().substring(0, 10), 16)
    const bValue = parseInt(b.transactionHash.toHex().substring(0, 10), 16)
    return a - b
}

transactions.setSort(customSort)
```

## Client

This library exposes a [Nimiq Client](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/api/Client.js~Client.html) as the `client` export.
For configuration options, see [Config](#config).
The `client` export is `undefined` until:

- the `ready` store turned `true` or
- the `start()` function resolves

There are two ways to make sure the `client` is defined when you use it:

1. Only enable `client`-triggering elements when the `ready` store turned `true`:

```html
<script>
    import { ready, client } from 'nimiq-svelte-stores'

    function sendTransaction() {
        const tx = ...
        client.sendTransaction(tx)
    }
</script>

<button disabled={!$ready} on:click={sendTransaction}>Send Transaction</button>
```

2. Await the exported `start()` function:

```js
import { start, client } from 'nimiq-svelte-stores'

async function sendTransaction() {
    const tx = ...;
    await start() // or: const client = await start()
    client.sendTransaction(tx)
}
```

The Nimiq Client API is documented here: https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/api/Client.js~Client.html

Tutorials for sending transactions are here: https://nimiq.github.io/tutorials/basics-3-transactions

## Config

The Nimiq Client that is created internally in this library can be configured during first start-up.
The `start()` method therefore takes a callback as its first argument.
This callback receives a [`Nimiq.Client.ConfigurationBuilder`](https://doc.esdoc.org/github.com/nimiq/core-js/class/src/main/generic/api/Configuration.js~ConfigurationBuilder.html) instance as an argument, which can be manipulated inside the callback.

*Note:* The client is only created once, so the config callback is only effective in the _first call_ to the `start()` function.

```js
start((config) => {
    // Create a volatile consensus (not storing peer information across page reloads)
    config.volatile(true)

    // Require a local mempool (creates a light consensus)
    client.feature(Nimiq.Client.Feature.MEMPOOL)
})
```
