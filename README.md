# DISCLAIMER

> This is a work in progress. The library is usable, however it is still evolving and may have some breaking changes in the future. These will most likely be minor, in addition to extending functionality.

In the future this library will be a wrapper around the new implementation of MAM [https://github.com/iotaledger/entangled/tree/develop/mam](https://github.com/iotaledger/entangled/tree/develop/mam)

# MAM Client JS Library

It is possible to publish transactions to the Tangle that contain only messages, with no value. This introduces many possibilities for data integrity and communication, but comes with the caveat that message-only signatures are not checked. What we introduce is a method of symmetric-key encrypted, signed data that takes advantage of merkle-tree winternitz signatures for extended public key usability, that can be found trivially by those who know to look for it.

This is wrapper library for the WASM/ASM.js output of the [IOTA Bindings repository](https://github.com/iotaledger/iota-bindings). For a more in depth look at how Masked Authenticated Messaging works please check out the [Overview](../master/docs/overview.md)

## Getting Started

Add the package to your project with:

```shell
npm install @iota/mam

or

yarn add @iota/mam
```

After adding the package it will provide access to the functions described below. To import the module simple use one of the following methods, depending on which version of JavaScript you are using.

```javascript
const Mam = require('@iota/mam');
Mam.init(...);

or

import * as Mam from '@iota/mam';
Mam.init(...);
```

or in the browser using

```html
<script src="./lib/mam.web.min.js"></script>
<script>
    Mam.init(...)
</script>
```

For a simple user experience you are advised to call the `init()` function to enable to tracking of state in your channels. When calling `init()` you should pass in the `provider` which is the address of an IRI node. This will provide access to some extra functionality including attaching, fetching and subscribing.

## Examples

* [Publishing in Public mode](../master/example/publishAndFetchPublic.js)
* [Publishing in Public mode with Custom Tag](../master/example/publishAndFetchPublicCustomTag.js)
* [Publishing in Restricted mode](../master/example/publishAndFetchRestricted.js)

* [Publishing in Public mode from Browser](../master/example/publishAndFetchPublic.html)

## API

### `init`

This initialises the state. This will return a state object that tracks the progress of your stream and streams you are following

#### Input

```javascript
Mam.init(settings, seed, security)
```

1. **settings**: `Object` or `String` Configuration object or network provider URL.
    Configuration object:
    1. **provider**: `String` Network provider URL.
    2. **attachToTangle** `Function` function to override default `attachToTangle` to use another Node to do the PoW or use a PoW service.
2. **seed**: `String` Optional tryte-encoded seed. *Null value generates a random seed*
3. **security**: `Integer` Optional security of the keys used. *Null value defaults to `2`*

#### Return

1. **Object** - Initialised state object to be used in future actions

------

### `changeMode`

This takes the state object and changes the default stream mode from `public` to the specified mode and `sidekey`. There are only three possible modes: `public`, `private`, & `restricted`. If you fail to pass one of these modes it will default to `public`. This will return a state object that tracks the progress of your stream and streams you are following

#### Input

```javascript
Mam.changeMode(state, mode, sidekey)
```

1. **state**: `Object` Initialised IOTA library with a provider set.
2. **mode**: `String` Intended channel mode. Can be only: `public`, `private` or `restricted`
3. **sideKey**: `String` Tryte-encoded encryption key, `81 trytes` long. *Required for restricted mode*

#### Return

1. **Object** - Initialised state object to be used in future actions

------

### `getRoot`

This method will return the root for the supplied mam state.

#### Input

```javascript
Mam.getRoot(state)
```

1. **state**: `Object` Initialised IOTA library with a provider set.

#### Return

1. **string** - The root calculated from the provided state.

------

### `create`

Creates a MAM message payload from a state object, tryte-encoded message and an optional side key. Returns an updated state and the payload for sending.

#### Input

```javascript
Mam.create(state, message)
```

1. **state**: `Object` Initialised IOTA library with a provider set.
2. **message**: `String` Tryte-encoded payload to be encrypted. Tryte-encoded payload can be generated by calling `asciiToTrytes` from the `@iota/converter` and passing a stringified JSON object

#### Return

1. **state**: `Object` Updated state object to be used with future actions.
2. **payload**: `String` Tryte-encoded payload.
3. **root**: `String` Tryte-encoded root of the payload.
4. **address**: `String` Tryte-encoded address used as an location to attach the payload.

------

### `decode`

Enables a user to decode a payload

#### Input

```javascript
Mam.decode(payload, sideKey, root)
```

1. **payload**: `String` Tryte-encoded payload.
2. **sideKey**: `String` Tryte-encoded encryption key. *Null value falls back to default key*
3. **root**: `String` Tryte-encoded string used as the address to attach the payload.

#### Return

1. **state**: `Object` Updated state object to be used with future actions.
2. **payload**: `String` Tryte-encoded payload.
3. **root**: `String` Tryte-encoded root used as an address to attach the payload.

------

### `subscribe`

This method will add a subscription to your state object using the provided channel details.

#### Input

```javascript
Mam.subscribe(state, channelRoot, channelKey)
```

1. **state**: `Object` Initialised IOTA library with a provider set.
2. **channelRoot**: `String` The root of the channel to subscribe to.
3. **channelKey**: `String` Optional, The key of the channel to subscribe to.

#### Return

1. **Object** - Updated state object to be used with future actions.

------

### `listen`

Listen to a channel for new messages.

#### Input

```javascript
Mam.listen(channel, callback)
```

1. **channel**: `Object` The channel object to listen to.
2. **callback**: `String` Callback called when new messages arrive.

#### Return

Nothing

------

### `attach` - async

Attaches a payload to the Tangle.

#### Input

```javascript
await Mam.attach(payload, address, depth, minWeightMagnitude, tag)
```

1. **payload**: `String` Tryte-encoded payload to be attached to the Tangle.
2. **root**: `String` Tryte-encoded string returned from the `Mam.create()` function.
3. **depth**: `number` Optional depth at which Random Walk starts. A value of 3 is typically used by wallets, meaning that RW starts 3 milestones back. *Null value will set depth to 3*
4. **minWeightMagnitude**: `number` Optional minimum number of trailing zeros in transaction hash. This is used by `attachToTangle` function to search for a valid nonce. Currently is 14 on mainnet & spamnnet and 9 on most other devnets. *Null value will set minWeightMagnitude to 9*
5. **tag**: `String` Optional tag of 0-27 trytes. *Null value will set tag to empty string*

#### Return

1. `Array` Transaction objects that have been attached to the network.

------

### `fetch` - async

Fetches the stream sequentially from a known `root` and optional `sidekey`. This call can be used in two ways: **Without a callback** will cause the function to read the entire stream before returning. **With a callback** the application will return data through the callback and finally the `nextroot` when finished.

#### Input

```javascript
await Mam.fetch(root, mode, sidekey, callback, limit)
```

1. **root**: `String` Tryte-encoded string used as the entry point to a stream. *NOT the address!*
2. **mode**: `String` Stream mode. Can one of `public`, `private` or `restricted` *Null value falls back to public*
3. **sideKey**: `String` Tryte-encoded encryption key. *Null value falls back to default key*
4. **callback**: `Function` Optional callback. *Null value will cause the function to push payload into the messages array.*
5. **limit**: `Number` Optional limits the number of items returned, defaults to all.

#### Return

1. **nextRoot**: `String` Tryte-encoded string pointing to the next root.
2. **messages**: `Array` Array of Tryte-encoded messages from the stream.

------

### `fetchSingle` - async

Fetches a single message from a known `root` and optional `sidekey`.

#### Input

```javascript
await Mam.fetchSingle(root, mode, sidekey)
```

1. **root**: `String` Tryte-encoded string used as the entry point to a stream. *NOT the address!*
2. **mode**: `String` Stream mode. Can one of `public`, `private` or `restricted` *Null value falls back to public*
3. **sideKey**: `String` Tryte-encoded encryption key. *Null value falls back to default key*

#### Return

1. **nextRoot**: `String` Tryte-encoded string pointing to the next root.
2. **payload**: `String` Tryte-encoded messages from the stream.

## Building the library

Compiled libs are included in the repository.
Compiling the Rust bindings can require some complex environmental setup to get to work, so if you are unfamiliar just stick to the compiled files.

This repo provides wrappers for both Browser and Node environments. The build script discriminates between a WASM.js and ASM.js build methods and returns files that are includable in your project.

### CommonJS Module for NodeJS/Browser with Module Loader

The below commands will build a file called `mam.client.js` in the `lib/` directory. You can then include the pacakge in your code using `require/import`.

```shell
// Install dependencies
yarn
// Build a development version lib/mam.client.js
yarn build-node-dev

// Build a production/minified version lib/mam.client.min.js
yarn build-node-prod
```

### Browser Script Include

The below commands will build a file called `mam.web.js` in the `lib/` directory. You can then include the package in your code using `<script src="">`

```shell
// Install dependencies
yarn
// Build a development version lib/mam.web.js
yarn build-web-dev
// Build a production/minified version lib/mam.web.min.js
yarn build-web-prod
```

### Build All 

To build all the libraries just run:

```shell
yarn dist
```
