# Middlewares

Just like for the other request options, **you can provide middlewares at the global level in your API options, at the service's definition level, or in the `options` parameter of the `fetch` method.**

You must provide an **array of promises**, like so : `(serviceDefinition: IAPIService, paths: IMiddlewarePaths, options: IFetchOptions) => any;`, please [take a look at the types](#types) to know more. You don't necessarily need to write asynchronous code in them, but they all must be promises.

Anything you will resolve in those promises will be merged into your request's options !

Here's a barebone example :

```javascript
const API_OPTIONS = {
    // ... all your api options
    middlewares: [exampleMiddleware],
};

async function exampleMiddleware (serviceDefinition, serviceOptions) {
  // This will be printed everytime you call a service
  console.log('You just fired a request for the path ' + serviceDefinition.path);
}
```

You can even make API calls in your middlewares. For instance, you might want to make sure the user is logged in into your API, or you might want to refresh its authentication token once in a while. Like so :

```javascript
const API_OPTIONS = {
    // ... all your api options
    middlewares: [authMiddleware]
}

async function authMiddleware (serviceDefinition, serviceOptions) {
    if (authToken && !tokenExpired) {
        // Our token is up-to-date, add it to the headers of our request
        return { headers: { 'X-Auth-Token': authToken } };
    }
    // Token is missing or outdated, let's fetch a new one
    try {
        // Assuming our login service's method is already set to 'POST'
        const authData = await api.fetch(
            'login',
            // the 'fetcthOptions' key allows us to use any of react-native's fetch method options
            // here, the body of our post request
            { fetchOptions: { body: 'username=user&password=password' } } 
        );
        // Store our new authentication token and add it to the headers of our request
        authToken = authData.authToken;
        tokenExpired = false;
        return { headers: { 'X-Auth-Token': authData.authToken } };
    } catch (err) {
        throw new Error(`Couldn't auth to API, ${err}`);
    }
}
```

## Response middleware


As of `2.3.0`, you can now configure a `responseMiddleware`  at the global level in your API options, at the service's definition level, or in the `options` parameter of the `fetch` method. This allows you to alter the data you're getting from your API without having to touch the result of `api.fetch` afterwards.

Just provide a function that takes the response of your API call as the first parameter, alter it the way you like, and don't forget to return it.

> ℹ️   Keep in mind that **using the `rawData` option will disable this middleware** since it's meant to circumvent any kind of parsing before returning your data.

> **⚠️  This middleware is only triggered when an actual network request is fired**. Your altered response will then be cached as usual, and this middleware will only be fired again when the cache for this request expires, or if you've disabled caching. Do not use this option to do some kind of network logging, you should use the regular `middlewares` option instead.

Here's an example from the demo, where we add to the data the current date :

```javascript
const API_SERVICES = {
    myService: { path: 'myService', responseMiddleware: (res) => ({ ...res, timestamp: Date.now() }) },
};
```

> ℹ️   You definitely can write asynchronous code in there. This could be useful if you need to dispatch a request depending on the response of the first one. For instance, you might want to refresh an authentication token if you find out in your response that the previous one just expired.