# dataloader-sequelize

Batching, caching and simplification of Sequelize with facebook/dataloader

# How it works

dataloader-sequelize is designed to provide per-request caching/batching for sequelize lookups, most likely in a graphql environment

# API

## `createContext(sequelize, object options)`
* Should be called after all models and associations are defined
* `sequelize` a sequelize instance
* `options.max=500` the maximum number of simultaneous dataloaders to store in memory. The loaders are stored in an LRU cache

# Usage
```js
import {createContext, EXPECTED_OPTIONS_KEY} from 'dataloader-sequelize';

/* Per request */
const context = createContext(sequelize); // must not be called before all models and associations are defined
// or
const context = createContext(sequelize, { cache: false }); // disable the use of LRU and Dataloader cache for stateful subscriptions
await User.findById(2, {[EXPECTED_OPTIONS_KEY]: context});
await User.findById(2, {[EXPECTED_OPTIONS_KEY]: context}); // Cached or batched, depending on timing
```

## Priming

Commonly you might have some sort of custom findAll requests that isn't going through the dataloader. To reuse the results from a call such as this in later findById calls you need to prime the cache:

```js
import {createContext, EXPECTED_OPTIONS_KEY} from 'dataloader-sequelize';
const context = createContext(sequelize);

const results = await User.findAll({where: {/* super complicated */}});
context.prime(results);

await User.findById(2, {[EXPECTED_OPTIONS_KEY]: context}); // Cached, if was in results and not cache: false
```
