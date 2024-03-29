<p align="center">
    <img src="https://user-images.githubusercontent.com/73451363/187207442-bae7ea26-7eac-4cab-8806-42779629c653.png" alt="Tailslide logo" width="400">
</p>

# Node.js SDK

---

This package is a server-side SDK for applications written in Node.js for the Tailslide feature flag framework.

Visit the https://github.com/tailslide-io repository or see Tailslide’s [case study](https://tailslide-io.github.io) page for more information.

## Installation

---

Install the Tailslide npm package with `npm install tailslide`

## Basic Usage

---

### Instantiating and Initializing FlagManager

The `FlagManager`class is the entry point of this SDK. It is responsible for retrieving all the flag rulesets for a given app with its `appId` and creating new `Toggler` instances to handle toggling of feature flags within that app. To instantiate a `FlagManager` object, a user must provide a configuration object:

```javascript
const FlagManager = require('tailslide');

const config = {
  natsServer: 'nats://localhost:4222',
  natsStream: 'flags_ruleset',
  appId: 1,
  userContext: '375d39e6-9c3f-4f58-80bd-e5960b710295',
  sdkKey: 'myToken',
  redisHost: 'http://localhost',
  redisPort: 6379,
};

const manager = new FlagManager(config);
await manager.initialize();
```

- `natsServer` is the NATS JetStream server `address:port`
- `natsStream` is the NATS JetStream’s stream name that stores all the apps and their flag rulesets
- `appId` is the ID number of the app the user wants to retrieve its flag ruleset from
- `userContext` is the UUID string that identifies the current user
- `sdkKey` is the SDK key for the Tailslide, it is used as a password for NATS JetStream token authentication
- `redisHost` is the address to the Redis database
- `redisPort` is the port number that the Redis database runs on

After instantiating a `FlagManager`, invoke the `initialize` method. This method connects the `FlagManager` instance to both NATS JetStream and Redis Timeseries, and asynchronously retrieves the latest and any new flag ruleset data.

---

### Using Feature Flag with Toggler

Once the `FlagManager` is initialized, it can create a `Toggler`, with the `newToggler` method, for each feature flag that the developer wants to wrap the new and old features in. A `Toggler`’s `isFlagActive` method checks whether the flag with its `flagName` is active or not based on the flag ruleset. A `Toggler`’s `isFlagActive` method returns a boolean value, which can be used to evaluate whether a new feature should be used or not.

```javascript
const flagConfig = {
  flagName: 'App 1 Flag 1',
};

const flagToggler = manager.newToggler(flagConfig);

if (flagToggler.isFlagActive()) {
  // call new feature here
} else {
  // call old feature here
}
```

---

### Emitting Success or Failture

To use a `Toggler` instance to record successful or failed operations, call its `emitSuccess` or `emitFailure` methods:

```javascript
if (successCondition) {
  await flagToggler.emitSuccess();
} else {
  await flagToggler.emitFailure();
}
```

## Documentation

---

### FlagManager

The `FlagManager` class is the entry point of the SDK. A new `FlagManager` object will need to be created for each app.

#### FlagManager Constructor

**Parameters:**

- An object with the following keys
  - `natsServer` is the NATS JetStream server `address:port`
  - `natsStream` is the NATS JetStream’s stream name that stores all the apps and their flag rulesets
  - `appId` a number representing the application the microservice belongs to
  - `sdkKey` a string generated via the Tower front-end for NATS JetStream authentication
  - `userContext` a string representing the user’s UUID
  - `redisHost` a string that represents the url of the Redis server
  - `redisPort` a number that represents the port number of the Redis server

---

#### Instance Methods

##### `flagmanager.initialize()`

Asynchronously initialize `flagmanager` connections to NATS JetStream and Redis database

**Parameters:**

- `null`

**Return Value:**

- `null`

---

##### `FlagManager.prototype.setUserContext(newUserContext)`

Set the current user's context for the `flagmanager`

**Parameters:**

- `newUserContext`: A UUID string that represents the current active user

**Return Value:**

- `null`

---

##### `FlagManager.prototype.getUserContext()`

Returns the current user context

**Parameters:**

- `null`

**Return Value:**

- The UUID string that represents the current active user

---

##### `FlagManager.prototype.newToggler(options)`

Creates a new toggler to check for a feature flag's status from the current app's flag ruleset by the flag's name.

**Parameters:**

- `options`: An object with key of `flagName` and a string value representing the name of the feature flag for the new toggler to check whether the new feature is enabled

**Return Value:**

- A `Toggler` object

---

##### `FlagManager.prototype.disconnect()`

Asynchronously disconnects the `FlagManager` instance from NATS JetStream and Redis database

**Parameters:**

- `null`

**Return Value:**

- `null`

---

### Toggler

The Toggler class provides methods that determine whether or not new feature code is run and handles success/failure emissions. Each toggler handles one feature flag, and is created by `FlagManager.prototype.newToggler()`.

---

#### Instance Methods

##### `Toggler.prototype.isFlagActive()`


Checks for flag status, whitelisted users, and rollout percentage in that order to determine whether the new feature is enabled.

- If the flag's active status is false, the function returns `false`
- If current user's UUID is in the whitelist of users, the function returns `true`
- If current user's UUID hashes to a value within user rollout percentage, the function returns `true`
- If current user's UUID hashes to a value outside user rollout percentage, the function returns `false`

**Parameters:**

- `null`

**Return Value**

- `true` or `false` depending on whether the feature flag is active

---


##### `Toggler.prototype.emitSuccess()`


Records a successful operation to the Redis Timeseries database, with key `flagId:success` and value of current timestamp

**Parameters:**

- `null`

**Return Value**

- `null`

---


##### `Toggler.prototype.emitFailure()`


Records a failure operation to the Redis Timeseries database, with key `flagId:success` and value of current timestamp

**Parameters:**

- `null`

**Return Value**

- `null`
