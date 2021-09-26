# Node.js and TypeScript Guide: Secure a Function with Auth0

> Note: This guide relies on middleware functionality from an *upcoming release* of the Nitric Node.js SDK. To use this release in your project run `yarn add @nitric/sdk@rc-latest`

In this guide we'll show you how to secure a Nitric Function written in TypeScript (or JavaScript) using Auth0 for authentication.

## Setup Auth0

Building authentication services from scratch is a challenging endeavour which presents a risk if done improperly. Choosing a third-party to provide this functionality is a great option to reduce the risk and minimize the effort required to secure your applications. [Auth0](https://auth0.com) is a great option for this, with comprehensive features and free accounts to get things started. Many of the steps in the guide will apply similarly to other services.

Start by creating a [free Auth0 account](https://auth0.com/signup) if you don't already have one. Once your account is created, the next thing you'll do is create an [Auth0 Tenant](https://auth0.com/docs/getting-started/create-tenant), which is an isolated container for your auth configuration and users.

Once you have an account and tenant, you'll need to create an API in Auth0. An Auth0 API is an entity representing some resource external to Auth0, in this guide we'll use it to represent your application's API (a collection of Nitric Functions). To create a new Auth0 API, navigate to the APIs page in the Auth0 Dashboard, you can find it under `Applications` > `APIs` in the left nav of the dashboard. From the APIs page click the `Create API` button and follow these steps:

1. Add a __Name__ to your API: e.g. `Todo API`
2. Set the API's __Identifier__: e.g. `https://todo-api.example.com`
3. Leave the signing algorithm as `RS256`, this is the best option for security
4. Click the __Create__ button

<img
  src="../../assets/img/auth0-createApi.png"
  height="500"
  alt="Service Anatomy"
/>

> __Identifiers__ are unique strings that identify your APIs. Auth0 recommends using URLs, however, the URLs won't be called and don't need to be public

Now that the API has been created, you'll need to record some details about the API for use in your Nitric application later. The two values you need are __Audience__ and __Domain__, to get these values follow the steps below.

### Get API Audience

1. Click on the __Settings__ tab
2. Find the __Identifier__ field
3. Copy the identifier value and note it down for later

<img
  src="../../assets/img/auth0-api-identifier.png"
  height="500"
  alt="Service Anatomy"
/>

### Get API Domain

1. Click on the __Test__ tab
2. Find the __cURL__ example under __Asking Auth0 for tokens from my application__
3. Copy your Auth0 domain and note it down for later

> Note: The ___domain___  is just part of the `--url` parameter, ignore the `https://` protocol prefix and path suffix. E.g. `tenant.region.auth0.com`

<img
  src="../../assets/img/auth0-api-domain.png"
  height="500"
  alt="Service Anatomy"
/>

## Add Authentication Middleware to Function(s)

You'll now create middleware to run before the handler of your function to protect your handler from unauthenticated or unauthorized requests. When using the `createHandler` function to combine a set of middleware with a handler function, the middleware and handler code will be executed in the order they're specified. When middleware are setup to run *before* a handler, the middleware functions are able to *protect* the handler by intercepting unwanted requests and returning a response before the handler is run.

Start by adding these packages to your project:

- [@nitric/middleware-jwt](https://www.npmjs.com/package/@nitric/middleware-jwt): provides HTTP middleware which validates JWT tokens in requests, stores decoded tokens in the `ctx` object making it easier to work with and validates permissions (scopes) from the JWT to perform RBAC Authorization.
- [jwks-rsa](https://www.npmjs.com/package/jwks-rsa): provides the ability to retrieve RSA secrets from JWKS (JSON Web Key Set) endpoints. In our case, this will retrieve the Auth0 secret used to validate JWT signatures.

```bash
$ yarn add @nitric/middleware-jwt jwks-rsa
```

Since some elements of the JWT validation process will be specific to your project, instead of providing a JWT validation middleware directly, the `@nitric/middleware-jwt` package provides a higher order function, which returns a customized middleware function. Let's start by creating some common code to call the higher order function and return the JWT middleware specific to your project.

```bash
$ touch common/middleware/auth.ts
```

Add the following code to that file:

```typescript
import { jwt, GetSecretOptions } from "@nitric/middleware-jwt";
import * as jwksRsa from "jwks-rsa";
import supportedAlgos from './config';

const nitricJwtSecret = (options) => {
  if (options === null || options === undefined) {
    // TODO: do something else here.
    throw new Error('An options object must be provided when initializing expressJwtSecret');
  }

  const client = jwksRsa(options);
  // const onError = options.handleSigningKeyError || handleSigningKeyError;

  const secretProvider = async ({ctx, header, payload}: GetSecretOptions) => {
    if (!header || !supportedAlgos.includes(header.alg)) {
      return null;
    }

    try {
      const key = await client.getSigningKey(header.kid);
      key.getPublicKey()
      return key.getPublicKey();
    } catch (err) {
      throw err;
    }
  };
  return secretProvider;
};

export interface Auth0Config {
  domain: string;
  audience: string;

}

export const checkJwt = ({domain, audience}: Auth0Config) => jwt({
  secret: nitricJwtSecret({
    cache: true,
    rateLimit: true,
    jwksRequestsPerMinute: 5,
    jwksUri: `https://${domain}/.well-known/jwks.json`
  }),
  // Validate the audience and the issuer.
  verifyOptions: {
    audience,
    issuer: `https://${domain}/`,
  },
  algorithms: ["RS256"]
});
```