# oidc-provider API documentation

oidc-provider allows to be extended and configured in various ways to fit a variety of use cases. You
will have to configure your instance with how to find your user accounts, where to store and retrieve
persisted data from and where your end-user interactions happen. The [example](/example) application
is a good starting point to get an idea of what you should provide.

## Sponsor

[<img width="65" height="65" align="left" src="https://avatars.githubusercontent.com/u/2824157?s=75&v=4" alt="auth0-logo">][sponsor-auth0] If you want to quickly add OpenID Connect authentication to Node.js apps, feel free to check out Auth0's Node.js SDK and free plan at [auth0.com/overview][sponsor-auth0].<br><br>

## Support

[<img src="https://c5.patreon.com/external/logo/become_a_patron_button@2x.png" width="160" align="right">][support-patreon]
If you or your business use oidc-provider, please consider becoming a [Patron][support-patreon] so I can continue maintaining it and adding new features carefree. You may also donate one-time via [PayPal][support-paypal].
[<img src="https://cdn.jsdelivr.net/gh/gregoiresgt/payment-icons@183140a5ff8f39b5a19d59ebeb2c77f03c3a24d3/Assets/Payment/PayPal/Paypal@2x.png" width="100" align="right">][support-paypal]

<br>

---

**Table of Contents**

- [Basic configuration example](#basic-configuration-example)
- [Default configuration values](#default-configuration-values)
- [Accounts](#accounts)
- [Clients](#clients)
- [Certificates](#certificates)
- [Configuring available claims](#configuring-available-claims)
- [Configuring available scopes](#configuring-available-scopes)
- [Persistence](#persistence)
- [Interaction](#interaction)
- [Custom Grant Types](#custom-grant-types)
- [Extending Authorization with Custom Parameters](#extending-authorization-with-custom-parameters)
- [Extending Discovery with Custom Properties](#extending-discovery-with-custom-properties)
- [Configuring Routes](#configuring-routes)
- [Fine-tuning supported algorithms](#fine-tuning-supported-algorithms)
- [HTTP Request Library / Proxy settings](#http-request-library--proxy-settings)
- [Changing HTTP Request Defaults](#changing-http-request-defaults)
- [Authentication Context Class Reference](#authentication-context-class-reference)
- [Registering module middlewares (helmet, ip-filters, rate-limiters, etc)](#registering-module-middlewares-helmet-ip-filters-rate-limiters-etc)
- [Pre- and post-middlewares](#pre--and-post-middlewares)
- [Mounting oidc-provider](#mounting-oidc-provider)
  - [to an express application](#to-an-express-application)
  - [to a koa application](#to-a-koa-application)
- [Trusting TLS offloading proxies](#trusting-tls-offloading-proxies)
- [Aggregated and Distributed claims](#aggregated-and-distributed-claims)
- [Configuration options](#configuration-options)
  - [features](#features)
  - [features.backchannelLogout](#featuresbackchannellogout)
  - [features.certificateBoundAccessTokens](#featurescertificateboundaccesstokens)
  - [features.claimsParameter](#featuresclaimsparameter)
  - [features.clientCredentials](#featuresclientcredentials)
  - [features.devInteractions](#featuresdevinteractions)
  - [features.deviceFlow](#featuresdeviceflow)
  - [features.discovery](#featuresdiscovery)
  - [features.encryption](#featuresencryption)
  - [features.frontchannelLogout](#featuresfrontchannellogout)
  - [features.introspection](#featuresintrospection)
  - [features.jwtIntrospection](#featuresjwtintrospection)
  - [features.jwtResponseModes](#featuresjwtresponsemodes)
  - [features.registration](#featuresregistration)
  - [features.registrationManagement](#featuresregistrationmanagement)
  - [features.request](#featuresrequest)
  - [features.requestUri](#featuresrequesturi)
  - [features.resourceIndicators](#featuresresourceindicators)
  - [features.revocation](#featuresrevocation)
  - [features.sessionManagement](#featuressessionmanagement)
  - [features.webMessageResponseMode](#featureswebmessageresponsemode)
  - [acrValues](#acrvalues)
  - [audiences](#audiences)
  - [claims](#claims)
  - [clockTolerance](#clocktolerance)
  - [conformIdTokenClaims](#conformidtokenclaims)
  - [cookies](#cookies)
  - [cookies.keys](#cookieskeys)
  - [cookies.long](#cookieslong)
  - [cookies.names](#cookiesnames)
  - [cookies.short](#cookiesshort)
  - [discovery](#discovery)
  - [dynamicScopes](#dynamicscopes)
  - [expiresWithSession](#expireswithsession)
  - [extraClientMetadata](#extraclientmetadata)
  - [extraParams](#extraparams)
  - [findById](#findbyid)
  - [formats](#formats)
  - [interactionUrl](#interactionurl)
  - [introspectionEndpointAuthMethods](#introspectionendpointauthmethods)
  - [issueRefreshToken](#issuerefreshtoken)
  - [logoutSource](#logoutsource)
  - [pairwiseIdentifier](#pairwiseidentifier)
  - [postLogoutRedirectUri](#postlogoutredirecturi)
  - [pkceMethods](#pkcemethods)
  - [rotateRefreshToken](#rotaterefreshtoken)
  - [renderError](#rendererror)
  - [responseTypes](#responsetypes)
  - [revocationEndpointAuthMethods](#revocationendpointauthmethods)
  - [routes](#routes)
  - [scopes](#scopes)
  - [subjectTypes](#subjecttypes)
  - [tokenEndpointAuthMethods](#tokenendpointauthmethods)
  - [ttl](#ttl)
  - [uniqueness](#uniqueness)
  - [whitelistedJWA](#whitelistedjwa)



## Basic configuration example

```js
const Provider = require('oidc-provider');
const configuration = {
  // ... see the available options in Configuration options section
  features: {
    discovery: true,
    registration: { initialAccessToken: true },
  },
  format: { default: 'opaque' },
  // ...
};
const clients = [{
  client_id: 'foo',
  client_secret: 'bar',
  redirect_uris: ['http://lvh.me:8080/cb'],
  // + other client properties
}];

const oidc = new Provider('http://localhost:3000', configuration);

let server;
(async () => {
  await oidc.initialize({ clients });
  // express/nodejs style application callback (req, res, next) for use with express apps, see /examples/express.js
  oidc.callback

  // koa application for use with koa apps, see /examples/koa.js
  oidc.app

  // or just expose a server standalone, see /examples/standalone.js
  server = oidc.listen(3000, () => {
    console.log('oidc-provider listening on port 3000, check http://localhost:3000/.well-known/openid-configuration');
  });
})().catch((err) => {
  if (server && server.listening) server.close();
  console.error(err);
  process.exitCode = 1;
});
```


## Default configuration values
Default values are available for all configuration options. Available in [code][defaults] as well as
in this [document](#configuration-options).


## Accounts

oidc-provider needs to be able to find an account and once found the account needs to have an
`accountId` property as well as `claims()` function returning an object with claims that correspond
to the claims your issuer supports. Tell oidc-provider how to find your account by an ID.
`#claims()` can also return a Promise later resolved / rejected.

```js
const oidc = new Provider('http://localhost:3000', {
  async findById(ctx, id) {
    return {
      accountId: id,
      async claims(use, scope) { return { sub: id }; },
    };
  }
});
```


## Clients
Clients can be passed to your provider instance during the `initialize` call or left to be loaded
via your provided Adapter. oidc-provider will use the adapter's `find` method when a non-cached
client_id is encountered. If you only wish to support clients that are initialized and no dynamic
registration then make it so that your adapter resolves client find calls with a falsy value. (e.g.
`return Promise.resolve()`).  

Available [Client Metadata][client-metadata] is validated as defined by the specifications. This list
is extended by other adjacent-specification related properties such as introspection and revocation
endpoint authentication, Session Management, Front and Back-Channel Logout, etc.

**via Provider interface**  
To add pre-established clients use the `initialize` method on a oidc-provider instance. This accepts
a clients array with metadata objects and rejects when the client metadata would be invalid.

```js
const provider = new Provider('http://localhost:3000');
const clients = [
  {
    token_endpoint_auth_method: 'none',
    client_id: 'mywebsite',
    grant_types: ['implicit'],
    response_types: ['id_token'],
    redirect_uris: ['https://client.example.com/cb'],
  },
  {
    // ...
  },
];

provider.initialize({ clients }).then(fulfillmentHandler, rejectionHandler);
```

**via Adapter**  
Storing client metadata in your storage is recommended for distributed deployments. Also when you
want to provide a client configuration GUI or plan on changing this data often. Clients get loaded
*! and validated !* when they are first needed, any metadata validation error encountered during
this first load will be thrown and handled like any other context specific errors.

Note: Make sure your adapter returns an object with the correct property value types as if they were
submitted via dynamic registration.


## Certificates
See [Certificates](/docs/keystores.md).


## Configuring available claims
The `claims` configuration parameter can be used to define which claims fall under what scope
as well as to expose additional claims that are available to RPs via the `claims` authorization
parameter. The configuration value uses the following scheme:

```js
new Provider('http://localhost:3000', {
  claims: {
    [scope name]: ['claim name', 'claim name'],
    // or
    [scope name]: {
      [claim name]: null,
    },
    // or (for standalone claims) - only requestable via claims parameter
    //   (when features.claimsParameter is true)
    [standalone claim name]: null
  }
});
```

To follow the [Core-defined scope-to-claim mapping][core-account-claims] use:

```js
new Provider('http://localhost:3000', {
  claims: {
    address: ['address'],
    email: ['email', 'email_verified'],
    phone: ['phone_number', 'phone_number_verified'],
    profile: ['birthdate', 'family_name', 'gender', 'given_name', 'locale', 'middle_name', 'name',
      'nickname', 'picture', 'preferred_username', 'profile', 'updated_at', 'website', 'zoneinfo'],
  },
});
```

## Configuring available scopes
Use the `scopes` configuration parameter to configure the scope values that you wish to support.
This list is extended by all scope names detected in the claims parameter as well.

Use the `dynamicScopes` configuration parameter to configure dynamic scope values.

## Persistence
The provided example and any new instance of oidc-provider will use the basic in-memory adapter for
storing issued tokens, codes, user sessions and dynamically registered clients. This is fine as
long as you develop, configure and generally just play around since every time you restart your
process all information will be lost. As soon as you cannot live with this limitation you will be
required to provide your own custom adapter constructor for oidc-provider to
use. This constructor will be called for every model accessed the first time it
is needed. A static `connect` method is called if present during the `provider.initialize()` call.

```js
const MyAdapter = require('./my_adapter');
const provider = new Provider('http://localhost:3000');
provider.initialize({ adapter: MyAdapter });
```

The API oidc-provider expects is documented [here](/example/my_adapter.js). For reference see the
[memory adapter](/lib/adapters/memory_adapter.js) and [redis](/example/adapters/redis.js) or
[mongodb](/example/adapters/mongodb.js) adapters.


## Interaction
Since oidc-provider only comes with feature-less views and interaction handlers it is up to you to fill
those in, here is how oidc-provider allows you to do so:

When oidc-provider cannot fulfill the authorization request for any of the possible reasons (missing
user session, requested ACR not fulfilled, prompt requested, ...) it will resolve an `interactionUrl`
(configurable) and redirect the User-Agent to that url. Before doing so it will save a short-lived
session and dump its identifier into a cookie scoped to the resolved interaction path.

This session contains:

- details of the interaction that is required
- all authorization request parameters
- current session account ID should there be one
- the uid of the authorization request
- the url to redirect the user to once interaction is finished

oidc-provider expects that you resolve all future interactions in one go and only then redirect the
User-Agent back with the results

Once the required interactions are finished you are expected to redirect back to the authorization
endpoint, affixed by the uid of the original request and the interaction results stored in the
interaction session object.

The Provider instance comes with helpers that aid with getting interaction details as well as
packing the results. See them used in the [step-by-step](https://github.com/panva/node-oidc-provider-example)
or [in-repo](/example/index.js) examples.


**`#provider.interactionDetails(req)`**
```js
// with express
expressApp.get('/interaction/:uid', async (req, res) => {
  const details = await provider.interactionDetails(req);
  // ...
});

// with koa
router.get('/interaction/:uid', async (ctx, next) => {
  const details = await provider.interactionDetails(ctx.req);
  // ...
});
```

**`#provider.interactionFinished(req, res, result)`**
```js
// with express
expressApp.post('/interaction/:uid/login', async (req, res) => {
  return provider.interactionFinished(req, res, result); // result object below
});

// with koa
router.post('/interaction/:uid', async (ctx, next) => {
  return provider.interactionFinished(ctx.req, ctx.res, result); // result object below
});

// result should be an object with some or all the following properties
{
  // authentication/login prompt got resolved, omit if no authentication happened, i.e. the user
  // cancelled
  login: {
    account: '7ff1d19a-d3fd-4863-978e-8cce75fa880c', // logged-in account id
    acr: string, // acr value for the authentication
    remember: boolean, // true if provider should use a persistent cookie rather than a session one, defaults to true
    ts: number, // unix timestamp of the authentication, defaults to now()
  },

  // consent was given by the user to the client for this session
  consent: {
    rejectedScopes: [], // array of strings, scope names the end-user has not granted
    rejectedClaims: [], // array of strings, claim names the end-user has not granted
  },

  // meta is a free object you may store alongside an authorization. It can be useful
  // during the interaction check to verify information on the ongoing session.
  meta: {
    // object structure up-to-you
  },

  ['custom prompt name resolved']: {},
}

// optionally, interactions can be primaturely exited with a an error by providing a result
// object as follow:
{
  // an error field used as error code indicating a failure during the interaction
  error: 'access_denied',

  // an optional description for this error
  error_description: 'Insufficient permissions: scope out of reach for this Account',
}
```

**`#provider.interactionResult`**
Unlike `#provider.interactionFinished` authorization request resume uri is returned instead of
immediate http redirect. It should be used when custom response handling is needed e.g. making AJAX
login where redirect information is expected to be available in the response.

```js
// with express
expressApp.post('/interaction/:uid/login', async (req, res) => {
  const redirectTo = await provider.interactionResult(req, res, result);

  res.send({ redirectTo });
});

// with koa
router.post('/interaction/:uid', async (ctx, next) => {
  const redirectTo = await provider.interactionResult(ctx.req, ctx.res, result);

  ctx.body = { redirectTo };
});
```

**`#provider.setProviderSession`**
Sometimes interactions need to be interrupted before finishing and need to be picked up later,
or a session just needs to be established from outside the regular authorization request.
`#provider.setProviderSession` will take care of setting the proper cookies and storing the
updated/created session object.

Signature:
```js
async setProviderSession(req, res, {
  account, // account id string
  ts = epochTime(), // [optional] login timestamp, defaults to current timestamp
  remember = true, // [optional] set the session as persistent, defaults to true
  clients = [], // [optional] array of client id strings to pre-authorize in the updated session
  meta: { // [optional] object with keys being client_ids present in clients with their respective meta
    [client_id]: {},
  }
} = {})
```

```js
// with express
expressApp.post('/interaction/:uid/login', async (req, res) => {
  await provider.setProviderSession(req, res, { account: 'accountId' });
  // ...
});

// with koa
router.post('/interaction/:uid/login', async (ctx, next) => {
  await provider.setProviderSession(ctx.req, ctx.res, { account: 'accountId' });
  // ...
});
```


## Custom Grant Types
oidc-provider comes with the basic grants implemented, but you can register your own grant types,
for example to implement a [password grant type][password-grant] or
[OAuth 2.0 Token Exchange][token-exchange]. You can check the standard grant factories
[here](/lib/actions/grants).

```js
const parameters = ['username', 'password'];

// For OAuth 2.0 Token Exchange you can specify allowedDuplicateParameters as ['audience', 'resource']
const allowedDuplicateParameters = [];

provider.registerGrantType('password', function passwordGrantTypeFactory(providerInstance) {
  return async function passwordGrantType(ctx, next) {
    let account;
    if ((account = await Account.authenticate(ctx.oidc.params.username, ctx.oidc.params.password))) {
      const AccessToken = providerInstance.AccessToken;
      const at = new AccessToken({
        gty: 'password',
        accountId: account.id,
        client: ctx.oidc.client,
      });

      const accessToken = await at.save();

      ctx.body = {
        access_token: accessToken,
        expires_in: at.expiration,
        token_type: 'Bearer',
      };
    } else {
      ctx.body = {
        error: 'invalid_grant',
        error_description: 'invalid credentials provided',
      };
      ctx.status = 400;
    }

    await next();
  };
}, parameters, allowedDuplicateParameters);
```


## Extending Authorization with Custom Parameters
You can extend the whitelisted parameters of authorization endpoint beyond the defaults. These will
be available in `ctx.oidc.params` as well as passed to the interaction session
object for you to read.
```js
const oidc = new Provider('http://localhost:3000', {
  extraParams: ['utm_campaign', 'utm_medium', 'utm_source', 'utm_term'],
});
```


## Extending Discovery with Custom Properties
You can extend the returned discovery properties beyond the defaults
```js
const oidc = new Provider('http://localhost:3000', {
  discovery: {
    service_documentation: 'http://server.example.com/connect/service_documentation.html',
    ui_locales_supported: ['en-US', 'en-GB', 'en-CA', 'fr-FR', 'fr-CA'],
    version: '3.1',
  }
});
```


## Configuring Routes
You can change the default routes by providing a routes object to the oidc-provider constructor.
See the specific routes in [default configuration][defaults].

```js
const oidc = new Provider('http://localhost:3000', {
  routes: {
    authorization: '/authz',
    certificates: '/jwks.json',
  }
});
```


## Fine-tuning supported algorithms
The supported JWA algorithms are configured with the [whitelistedJWA](#whitelistedjwa) configuration
property.

- whitelisted symmetric algorithms are available all the time
- whitelisted asymmetric algorithms are available when the provider is initialized with keystore
  including keys that support those JWAs


## HTTP Request Library / Proxy settings
By default oidc-provider uses the [got][got-library] module. Because of its lightweight nature of
the provider will not use environment-defined http(s) proxies. In order to have them used you'll
need to require and tell oidc-provider to use [request][request-library] instead.

```sh
# add request to your application package bundle
npm install request@^2.0.0 --save
```

```js
// tell oidc-provider to use request instead of got
Provider.useRequest();
```


## Changing HTTP Request Defaults
On four occasions the OIDC Provider needs to venture out to the world wide webs to fetch or post
to external resources, those are

- fetching an authorization request by request_uri reference
- fetching and refreshing client's referenced asymmetric keys (jwks_uri client metadata)
- validating pairwise client's relation to a sector (sector_identifier_uri client metadata)
- posting to client's backchannel_logout_uri

oidc-provider uses these default options for http requests
```js
const DEFAULT_HTTP_OPTIONS = {
  followRedirect: false,
  headers: { 'User-Agent': `${pkg.name}/${pkg.version} (${this.issuer})` },
  retry: 0,
  timeout: 1500,
};
```

Setting `defaultHttpOptions` on `Provider` instance merges your passed options with these defaults,
for example you can add your own headers, change the user-agent used or change the timeout setting
```js
provider.defaultHttpOptions = { timeout: 2500, headers: { 'X-Your-Header': '<whatever>' } };
```

Confirm your httpOptions by
```js
console.log('httpOptions %j', provider.defaultHttpOptions);
```


## Authentication Context Class Reference
Supply an array of string values to acrValues configuration option to set `acr_values_supported`.
Passing an empty array disables the acr claim and removes `acr_values_supported` from discovery.


## Registering module middlewares (helmet, ip-filters, rate-limiters, etc)
When using `provider.app` or `provider.callback` as a mounted application in your own koa or express
stack just follow the respective module's documentation. However, when using the `provider.app` Koa
instance directly to register i.e. koa-helmet you must push the middleware in
front of oidc-provider in the middleware stack.

```js
const helmet = require('koa-helmet');

// Correct, pushes koa-helmet at the end of the middleware stack but BEFORE oidc-provider.
provider.use(helmet());

// Incorrect, pushes koa-helmet at the end of the middleware stack AFTER oidc-provider, not being
// executed when errors are encountered or during actions that do not "await next()".
provider.app.use(helmet());
```


## Pre- and post-middlewares
You can push custom middleware to be executed before and after oidc-provider.

```js
provider.use(async (ctx, next) => {
  /** pre-processing
   * you may target a specific action here by matching `ctx.path`
   */
  console.log('middleware pre', ctx.method, ctx.path);

  await next();
  /** post-processing
   * since internal route matching was already executed you may target a specific action here
   * checking `ctx.oidc.route`, the unique route names used are
   *
   * `authorization`
   * `certificates`
   * `client_delete`
   * `client_update`
   * `code_verification`
   * `device_authorization`
   * `device_resume`
   * `end_session`
   * `introspection`
   * `registration`
   * `resume`
   * `revocation`
   * `token`
   * `userinfo`
   * `webfinger`
   * `check_session`
   * `check_session_origin`
   * `client`
   * `discovery`
   *
   * ctx.method === 'OPTIONS' is then useful for filtering out CORS Pre-flights
   */
   console.log('middleware post', ctx.method, ctx.oidc.route);
});
```

## Mounting oidc-provider
The following snippets show how a provider instance can be mounted to existing applications with a
path prefix.

### to an express application
```js
// assumes express ^4.0.0
const prefix = '/oidc';
expressApp.use(prefix, oidc.callback);
```

### to a koa application
```js
// assumes koa ^2.0.0
const mount = require('koa-mount');
const prefix = '/oidc';
koaApp.use(mount(prefix, oidc.app));
```

## Trusting TLS offloading proxies

Having a TLS offloading proxy in front of Node.js running oidc-provider is
the norm. To let your downstream application know of the original protocol and
ip you have to tell your app to trust `x-forwarded-proto` and `x-forwarded-for`
headers commonly set by those proxies (as with any express/koa application).
This is needed for the provider responses to be correct (e.g. to have the right
https URL endpoints and keeping the right (secure) protocol).

Depending on your setup you should do the following in your downstream
application code

| setup | example |
|---|---|
| standalone oidc-provider | `provider.proxy = true; ` |
| oidc-provider mounted to a koa app | `yourKoaApp.proxy = true` |
| oidc-provider mounted to an express app | `provider.proxy = true; ` |

See http://koajs.com/#settings and the [example](/example/index.js).

It is also necessary that the web server doing the offloading also passes
those headers to the downstream application. Here is a common configuration
for Nginx (assuming that the downstream application is listening on
127.0.0.1:8009). Your configuration may vary, please consult your web server
documentation for details.

```
location / {
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;

  proxy_pass http://127.0.0.1:8009;
  proxy_redirect off;
}
```


## Aggregated and Distributed claims
Returning aggregated and distributed claims is as easy as having your Account#claims method return
the two necessary members `_claim_sources` and `_claim_names` with the
[expected][aggregated-distributed-claims] properties. oidc-provider will include only the
sources for claims that are part of the request scope, omitting the ones that the RP did not request
and leaving out the entire `_claim_sources` and `_claim_sources` if they bear no requested claims.

Note: to make sure the RPs can expect these claims you should configure your discovery to return
the respective claim types via the `claim_types_supported` property.
```js
const oidc = new Provider('http://localhost:3000', {
  discovery: {
    claim_types_supported: ['normal', 'aggregated', 'distributed']
  }
});
```


## Configuration options

<!-- DO NOT EDIT, COMMIT OR STAGE CHANGES BELOW THIS LINE -->
<!-- START CONF OPTIONS -->
### features

Enable/disable features. Some features are still either based on draft or experimental RFCs. Enabling those will produce a warning in your console and you must be aware that breaking changes may occur between draft implementations and that those will be published as minor versions of oidc-provider. See the example below on how to acknowledge the specification is a draft (this will remove the warning log) and ensure the provider instance will fail to initialize if a new version of oidc-provider bundles newer version of the RFC with breaking changes in it.   
  

<details>
  <summary>(Click to expand) Acknowledging a draft / experimental feature
</summary>
  <br>

```js
new Provider('http://localhost:3000', {
  features: {
    webMessageResponseMode: {
      enabled: true,
    },
  },
});
// The above code produces this NOTICE
// NOTICE: The following draft features are enabled and their implemented version not acknowledged
// NOTICE:   - OAuth 2.0 Web Message Response Mode - draft 00 (This is an Individual draft. URL: https://tools.ietf.org/html/draft-sakimura-oauth-wmrm-00)
// NOTICE: Breaking changes between draft version updates may occur and these will be published as MINOR semver oidc-provider updates.
// NOTICE: You may disable this notice and these potentially breaking updates by acknowledging the current draft version. See https://github.com/panva/node-oidc-provider/tree/master/docs#features
new Provider('http://localhost:3000', {
  features: {
    webMessageResponseMode: {
      enabled: true,
      ack: 0, // < we're acknowledging draft 00 of the RFC
    },
  },
});
// No more NOTICE, at this point if the draft implementation changed to 01 and contained no breaking
// changes, you're good to go, still no NOTICE, your code is safe to run.
// Now lets assume you upgrade oidc-provider version and it bundles draft 02 and it contains breaking
// changes
new Provider('http://localhost:3000', {
  features: {
    webMessageResponseMode: {
      enabled: true,
      ack: 0, // < bundled is 2, but we're still acknowledging 0
    },
  },
});
// Thrown:
// Error: An unacknowledged version of a draft feature is included in this oidc-provider version.
```
</details>

### features.backchannelLogout

[Back-Channel Logout 1.0 - draft 04](https://openid.net/specs/openid-connect-backchannel-1_0-04.html)  

Enables Back-Channel Logout features.   
  


_**default value**_:
```js
{
  enabled: false
}
```

### features.certificateBoundAccessTokens

[draft-ietf-oauth-mtls-12](https://tools.ietf.org/html/draft-ietf-oauth-mtls-12) - OAuth 2.0 Mutual TLS Client Authentication and Certificate Bound Access Tokens  

Enables Certificate Bound Access Tokens. Clients may be registered with `tls_client_certificate_bound_access_tokens` to indicate intention to receive mutual TLS client certificate bound access tokens.   
  


_**default value**_:
```js
{
  enabled: false
}
```
<details>
  <summary>(Click to expand) Setting up the environment for Certificate Bound Access Tokens</summary>
  <br>


To enable Certificate Bound Access Tokens the provider expects your TLS-offloading proxy to handle the client certificate validation, parsing, handling, etc. Once set up you are expected to forward `x-ssl-client-cert` header with variable values set by this proxy. An important aspect is to sanitize the inbound request headers at the proxy. <br/><br/> The most common openssl based proxies are Apache and NGINX, with those you're looking to use <br/><br/> __`SSLVerifyClient` (Apache) / `ssl_verify_client` (NGINX)__ with the appropriate configuration value that matches your setup requirements. <br/><br/> Set the proxy request header with variable set as a result of enabling mutual TLS
  

```nginx
# NGINX
proxy_set_header x-ssl-client-cert $ssl_client_cert;
```
```apache
# Apache
RequestHeader set x-ssl-client-cert  ""
RequestHeader set x-ssl-client-cert "%{SSL_CLIENT_CERT}s"
```
You should also consider hosting the endpoints supporting client authentication, on a separate host name or port in order to prevent unintended impact on the TLS behaviour of your other endpoints, e.g. Discovery or the authorization endpoint and changing the discovery values for them with a post-middleware.
  

```js
provider.use(async (ctx, next) => {
  await next();
  if (ctx.oidc.route === 'discovery' && ctx.method === 'GET' && ctx.status === 200) {
    ctx.body.userinfo_endpoint = '...';
    ctx.body.token_endpoint = '...';
  }
});
```
When doing that be sure to remove the client provided headers of the same name on the non-mutual TLS enabled host name / port in your proxy setup or block the routes for these there completely.  


</details>

### features.claimsParameter

[Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#rfc.section.5.5) - Requesting Claims using the "claims" Request Parameter  

Enables the use and validations of `claims` parameter as described in the specification.   
  


_**default value**_:
```js
{
  enabled: false
}
```

### features.clientCredentials

[RFC6749](https://tools.ietf.org/html/rfc6749#section-1.3.4) - Client Credentials  

Enables `grant_type=client_credentials` to be used on the token endpoint.  


_**default value**_:
```js
{
  enabled: false
}
```

### features.devInteractions

Development-ONLY out of the box interaction views bundled with the library allow you to skip the boring frontend part while experimenting with oidc-provider. Enter any username (will be used as sub claim value) and any password to proceed.   
 Be sure to disable and replace this feature with your actual frontend flows and End-User authentication flows as soon as possible. These views are not meant to ever be seen by actual users.  


_**default value**_:
```js
{
  enabled: true
}
```

### features.deviceFlow

[draft-ietf-oauth-device-flow-15](https://tools.ietf.org/html/draft-ietf-oauth-device-flow-15) - OAuth 2.0 Device Authorization Grant  

Enables Device Authorization Grant  


_**default value**_:
```js
{
  enabled: false,
  charset: 'base-20',
  mask: '****-****',
  deviceInfo: [Function: deviceInfo], // see expanded details below
  userCodeInputSource: [AsyncFunction: userCodeInputSource], // see expanded details below
  userCodeConfirmSource: [AsyncFunction: userCodeConfirmSource], // see expanded details below
  successSource: [AsyncFunction: successSource]
}
```
<details>
  <summary>(Click to expand) features.deviceFlow options details</summary>
  <br>


#### charset

alias for a character set of the generated user codes. Supported values are
 - `base-20` uses BCDFGHJKLMNPQRSTVWXZ
 - `digits` uses 0123456789  


_**default value**_:
```js
'base-20'
```

#### deviceInfo

Helper function used to extract details from the device authorization endpoint request. This is then available during the end-user confirm screen and is supposed to aid the user confirm that the particular authorization initiated by the user from a device in his possession  


_**default value**_:
```js
deviceInfo(ctx) {
  return {
    ip: ctx.ip,
    ua: ctx.get('user-agent'),
  };
}
```

#### mask

a string used as a template for the generated user codes, `*` characters will be replaced by random chars from the charset, `-`(dash) and ` ` (space) characters may be included for readability. See the RFC for details about minimal recommended entropy  


_**default value**_:
```js
'****-****'
```

#### successSource

HTML source rendered when device code feature renders a success page for the User-Agent.  


_**default value**_:
```js
async successSource(ctx) {
  // @param ctx - koa request context
  const {
    clientId, clientName, clientUri, initiateLoginUri, logoUri, policyUri, tosUri,
  } = ctx.oidc.client;
  ctx.body = `<!DOCTYPE html>
ead>
<title>Sign-in Success</title>
<style>/* css and html classes omitted for brevity, see lib/helpers/defaults.js */</style>
head>
ody>
<div>
  <h1>Sign-in Success</h1>
  <p>Your login ${clientName ? `with ${clientName}` : ''} was successful, you can now close this page.</p>
</div>
body>
html>`;
}
```

#### userCodeConfirmSource

HTML source rendered when device code feature renders an a confirmation prompt for ther User-Agent.  


_**default value**_:
```js
async userCodeConfirmSource(ctx, form, client, deviceInfo, userCode) {
  // @param ctx - koa request context
  // @param form - form source (id="op.deviceConfirmForm") to be embedded in the page and
  //   submitted by the End-User.
  // @param deviceInfo - device information from the device_authorization_endpoint call
  // @param userCode - formatted user code by the configured mask
  const {
    clientId, clientName, clientUri, logoUri, policyUri, tosUri,
  } = ctx.oidc.client;
  ctx.body = `<!DOCTYPE html>
ead>
<title>Device Login Confirmation</title>
<style>/* css and html classes omitted for brevity, see lib/helpers/defaults.js */</style>
head>
ody>
<div>
  <h1>Confirm Device</h1>
  <p>
    <strong>${clientName || clientId}</strong>
    <br/><br/>
    The following code should be displayed on your device<br/><br/>
    <code>${userCode}</code>
    <br/><br/>
    <small>If you did not initiate this action, the code does not match or are unaware of such device in your possession please close this window or click abort.</small>
  </p>
  <script>
    function abort() {
      var form = document.getElementById('op.deviceConfirmForm');
      var input = document.createElement('input');
      input.type = 'hidden';
      input.name = 'abort';
      input.value = 'yes';
      form.appendChild(input);
      form.submit();
    }
  </script>
  ${form}
  <button autofocus type="submit" form="op.deviceConfirmForm">Continue</button>
  <div>
    <a onclick="abort(); return false;" href="">[ Abort ]</a>
  </div>
</div>
body>
html>`;
}
```

#### userCodeInputSource

HTML source rendered when device code feature renders an input prompt for the User-Agent.  


_**default value**_:
```js
async userCodeInputSource(ctx, form, out, err) {
  // @param ctx - koa request context
  // @param form - form source (id="op.deviceInputForm") to be embedded in the page and submitted
  //   by the End-User.
  // @param out - if an error is returned the out object contains details that are fit to be
  //   rendered, i.e. does not include internal error messages
  // @param err - error object with an optional userCode property passed when the form is being
  //   re-rendered due to code missing/invalid/expired
  let msg;
  if (err && (err.userCode || err.name === 'NoCodeError')) {
    msg = '<p>The code you entered is incorrect. Try again</p>';
  } else if (err && err.name === 'AbortedError') {
    msg = '<p>The Sign-in request was interrupted</p>';
  } else if (err) {
    msg = '<p>There was an error processing your request</p>';
  } else {
    msg = '<p>Enter the code displayed on your device</p>';
  }
  ctx.body = `<!DOCTYPE html>
ead>
<title>Sign-in</title>
<style>/* css and html classes omitted for brevity, see lib/helpers/defaults.js */</style>
head>
ody>
<div>
  <h1>Sign-in</h1>
  ${msg}
  ${form}
  <button type="submit" form="op.deviceInputForm">Continue</button>
</div>
body>
html>`;
}
```

</details>

### features.discovery

[Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html)  

Exposes `/.well-known/openid-configuration` endpoint with your provider's actual configuration, i.e. Available claims, features and so on.  


_**default value**_:
```js
{
  enabled: true
}
```

### features.encryption

Enables encryption features such as receiving encrypted UserInfo responses, encrypted ID Tokens and allow receiving encrypted Request Objects.  


_**default value**_:
```js
{
  enabled: false
}
```

### features.frontchannelLogout

[Front-Channel Logout 1.0 - draft 02](https://openid.net/specs/openid-connect-frontchannel-1_0-02.html)  

Enables Front-Channel Logout features  


_**default value**_:
```js
{
  enabled: false,
  logoutPendingSource: [AsyncFunction: logoutPendingSource]
}
```
<details>
  <summary>(Click to expand) features.frontchannelLogout options details</summary>
  <br>


#### logoutPendingSource

HTML source rendered when there are pending front-channel logout iframes to be called to trigger RP logouts. It should handle waiting for the frames to be loaded as well as have a timeout mechanism in it.  


_**default value**_:
```js
async logoutPendingSource(ctx, frames, postLogoutRedirectUri, timeout) {
  ctx.body = `<!DOCTYPE html>
ead>
<title>Logout</title>
<style>/* css and html classes omitted for brevity, see lib/helpers/defaults.js */</style>
head>
ody>
${frames.join('')}
<script>
  var loaded = 0;
  function redirect() {
    window.location.replace("${postLogoutRedirectUri}");
  }
  function frameOnLoad() {
    loaded += 1;
    if (loaded === ${frames.length}) redirect();
  }
  Array.prototype.slice.call(document.querySelectorAll('iframe')).forEach(function (element) {
    element.onload = frameOnLoad;
  });
  setTimeout(redirect, ${timeout});
</script>
body>
html>`;
}
```

</details>

### features.introspection

[RFC7662](https://tools.ietf.org/html/rfc7662) - OAuth 2.0 Token Introspection  

Enables Token Introspection features   
  


_**default value**_:
```js
{
  enabled: false
}
```

### features.jwtIntrospection

[draft-ietf-oauth-jwt-introspection-response-02](https://tools.ietf.org/html/draft-ietf-oauth-jwt-introspection-response-02) - JWT Response for OAuth Token Introspection  

Enables JWT responses for Token Introspection features   
  


_**default value**_:
```js
{
  enabled: false
}
```

### features.jwtResponseModes

[openid-financial-api-jarm-wd-02](https://openid.net/specs/openid-financial-api-jarm-wd-02.html) - JWT Secured Authorization Response Mode (JARM)  

Enables JWT Secured Authorization Responses   
  


_**default value**_:
```js
{
  enabled: false
}
```

### features.registration

[Dynamic Client Registration 1.0](https://openid.net/specs/openid-connect-registration-1_0.html)  

Enables Dynamic Client Registration.  


_**default value**_:
```js
{
  enabled: false,
  initialAccessToken: false,
  policies: undefined,
  idFactory: [Function: idFactory], // see expanded details below
  secretFactory: [Function: secretFactory]
}
```
<details>
  <summary>(Click to expand) features.registration options details</summary>
  <br>


#### idFactory

helper generating random client identifiers during dynamic client registration  


_**default value**_:
```js
idFactory() {
  return nanoid();
}
```

#### initialAccessToken

Enables registration_endpoint to check a valid initial access token is provided as a bearer token during the registration call. Supported types are
 - `string` the string value will be checked as a static initial access token
 - `boolean` true/false to enable/disable adapter backed initial access tokens   
  


_**default value**_:
```js
false
```
<details>
  <summary>(Click to expand) To add an adapter backed initial access token and retrive its value
</summary>
  <br>

```js
new (provider.InitialAccessToken)({}).save().then(console.log);
```
</details>

#### policies

define registration and registration management policies applied to client properties. Policies are sync/async functions that are assigned to an Initial Access Token that run before the regular client property validations are run. Multiple policies may be assigned to an Initial Access Token and by default the same policies will transfer over to the Registration Access Token. A policy may throw / reject and it may modify the properties object.   
  


_**default value**_:
```js
undefined
```
<details>
  <summary>(Click to expand) To define registration and registration management policies</summary>
  <br>


To define policy functions configure `features.registration` to be an object like so:
  

```js
{
  enabled: true,
  initialAccessToken: true, // to enable adapter-backed initial access tokens
  policies: {
    'my-policy': function (ctx, properties) {
      // @param ctx - koa request context
      // @param properties - the client properties which are about to be validated
      // example of setting a default
      if (!('client_name' in properties)) {
        properties.client_name = generateRandomClientName();
      }
      // example of forcing a value
      properties.userinfo_signed_response_alg = 'RS256';
      // example of throwing a validation error
      if (someCondition(ctx, properties)) {
        throw new Provider.errors.InvalidClientMetadata('validation error message');
      }
    },
    'my-policy-2': async function (ctx, properties) {},
  },
}
```
An Initial Access Token with those policies being executed (one by one in that order) is created like so
  

```js
new (provider.InitialAccessToken)({ policies: ['my-policy', 'my-policy-2'] }).save().then(console.log);
```
Note: referenced policies must always be present when encountered on a token, an AssertionError will be thrown inside the request context if it is not, resulting in a 500 Server Error. Note: the same policies will be assigned to the Registration Access Token after a successful validation. If you wish to assign different policies to the Registration Access Token
  

```js
// inside your final ran policy
ctx.oidc.entities.RegistrationAccessToken.policies = ['update-policy'];
```
</details>
<details>
  <summary>(Click to expand) Using Initial Access Token policies for software_statement dynamic client registration property</summary>
  <br>


Support modules:
  

```js
const { verify } = require('jsonwebtoken');
const {
  errors: { InvalidSoftwareStatement, UnapprovedSoftwareStatement, InvalidClientMetadata },
} = require('oidc-provider');
```
features.registration configuration:
  

```js
{
 enabled: true,
 initialAccessToken: true, // to enable adapter-backed initial access tokens
 policies: {
   'softwareStatement': async function (ctx, metadata) {
     if (!('software_statement' in metadata)) {
       throw new InvalidClientMetadata('software_statement must be provided');
     }
     const softwareStatementKey = await loadKeyForThisPolicy();
     const statement = metadata.software_statement;
     let payload;
     try {
       payload = verify(value, softwareStatementKey, {
         algorithms: ['RS256'],
         issuer: 'Software Statement Issuer',
       });
       if (!approvedStatement(value, payload)) {
         throw new UnapprovedSoftwareStatement('software_statement not approved for use');
       }
       // cherry pick the software_statement values and assign them
       // Note: regular validations will run!
       const { client_name, client_uri } = payload;
       Object.assign(metadata, { client_name, client_uri });
     } catch (err) {
       throw new InvalidSoftwareStatement('could not verify software_statement');
     }
   },
 },
}
```
An Initial Access Token that requires and validates the given software statement is created like so
  

```js
new (provider.InitialAccessToken)({ policies: ['softwareStatement'] }).save().then(console.log);
```
</details>

#### secretFactory

helper generating random client secrets during dynamic client registration  


_**default value**_:
```js
secretFactory() {
  return base64url(crypto.randomBytes(64)); // 512 base64url random bits
}
```

</details>

### features.registrationManagement

[OAuth 2.0 Dynamic Client Registration Management Protocol](https://tools.ietf.org/html/rfc7592)  

Enables Update and Delete features described in the RFC  


_**default value**_:
```js
{
  enabled: false,
  rotateRegistrationAccessToken: false
}
```
<details>
  <summary>(Click to expand) features.registrationManagement options details</summary>
  <br>


#### rotateRegistrationAccessToken

Enables registration access token rotation. The provider will discard the current Registration Access Token with a successful update and issue a new one, returning it to the client with the Registration Update Response. Supported values are
 - `false` registration access tokens are not rotated
 - `true` registration access tokens are rotated when used
 - function returning true/false, true when rotation should occur, false when it shouldn't  


_**default value**_:
```js
false
```
<details>
  <summary>(Click to expand) function use
</summary>
  <br>

```js
{
  features: {
    registrationManagement: {
      enabled: true,
      async rotateRegistrationAccessToken(ctx) {
        // return tokenRecentlyRotated(ctx.oidc.entities.RegistrationAccessToken);
        // or
        // return customClientBasedPolicy(ctx.oidc.entities.Client);
      }
    }
  }
}
```
</details>

</details>

### features.request

[Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#rfc.section.6.1) - Passing a Request Object by Value  

Enables the use and validations of `request` parameter  


_**default value**_:
```js
{
  enabled: false
}
```

### features.requestUri

[Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#rfc.section.6.2) - Passing a Request Object by Reference  

Enables the use and validations of `request_uri` parameter  


_**default value**_:
```js
{
  enabled: true,
  requireUriRegistration: true
}
```
<details>
  <summary>(Click to expand) features.requestUri options details</summary>
  <br>


#### requireUriRegistration

makes request_uri pre-registration mandatory/optional  


_**default value**_:
```js
true
```

</details>

### features.resourceIndicators

[draft-ietf-oauth-resource-indicators-02](https://tools.ietf.org/html/draft-ietf-oauth-resource-indicators-02) - Resource Indicators for OAuth 2.0  

Enables the use of `resource` parameter for the authorization and token endpoints. In order for the feature to be any useful you must also use the `audiences` helper function to validate the resource(s) and transform it to jwt's token audience.   
  


_**default value**_:
```js
{
  enabled: false
}
```
<details>
  <summary>(Click to expand) Example use</summary>
  <br>


This example will
 - throw based on an OP policy when unrecognized or unauthorized resources are requested
 - transform resources to audience and push them down to the audience of access tokens
 - take both, the parameter and previously granted resources into consideration
  

```js
// const { InvalidTarget } = Provider.errors;
// `resourceAllowedForClient` is the custom OP policy
// `transform` is mapping the resource values to actual aud values
{
  // ...
  async function audiences(ctx, sub, token, use) {
    if (use === 'access_token') {
      const { oidc: { route, client, params: { resource: resourceParam } } } = ctx;
      let grantedResource;
      if (route === 'token') {
        const { oidc: { params: { grant_type } } } = ctx;
        switch (grant_type) {
          case 'authorization_code':
            grantedResource = ctx.oidc.entities.AuthorizationCode.resource;
            break;
          case 'refresh_token':
            grantedResource = ctx.oidc.entities.RefreshToken.resource;
            break;
          case 'urn:ietf:params:oauth:grant-type:device_code':
            grantedResource = ctx.oidc.entities.DeviceCode.resource;
            break;
          default:
        }
      }
      const allowed = await resourceAllowedForClient(resourceParam, grantedResource, client);
      if (!allowed) {
        throw new InvalidResource('unauthorized "resource" requested');
      }
      // => array of validated and transformed string audiences or undefined if no audiences
      //    are to be listed
      return transform(resourceParam, grantedResource);
    }
  },
  formats: {
    default: 'opaque',
    AccessToken(ctx, token) {
      if (Array.isArray(token.aud)) {
        return 'jwt';
      }
      return 'opaque';
    }
  },
  // ...
}
```
</details>

### features.revocation

[RFC7009](https://tools.ietf.org/html/rfc7009) - OAuth 2.0 Token Revocation  

Enables Token Revocation   
  


_**default value**_:
```js
{
  enabled: false
}
```

### features.sessionManagement

[Session Management 1.0 - draft 28](https://openid.net/specs/openid-connect-session-1_0-28.html)  

Enables Session Management features.  


_**default value**_:
```js
{
  enabled: false,
  keepHeaders: false
}
```
<details>
  <summary>(Click to expand) features.sessionManagement options details</summary>
  <br>


#### keepHeaders

Enables/Disables removing frame-ancestors from Content-Security-Policy and X-Frame-Options headers.  

_**recommendation**_: Only enable this if you know what you're doing either in a followup middleware or your app server, otherwise you shouldn't have the need to touch this option.  

_**default value**_:
```js
false
```

</details>

### features.webMessageResponseMode

[draft-sakimura-oauth-wmrm-00](https://tools.ietf.org/html/draft-sakimura-oauth-wmrm-00) - OAuth 2.0 Web Message Response Mode  

Enables `web_message` response mode.   
 Note: Although a general advise to use a `helmet` ([express](https://www.npmjs.com/package/helmet), [koa](https://www.npmjs.com/package/koa-helmet)) it is especially advised for your interaction views routes if Web Message Response Mode is available on your deployment.  


_**default value**_:
```js
{
  enabled: false
}
```

### acrValues

Array of strings, the Authentication Context Class References that OP supports.  


_**default value**_:
```js
[]
```

### audiences

Helper used by the OP to push additional audiences to issued Access and ClientCredentials Tokens. The return value should either be falsy to omit adding additional audiences or an array of strings to push.  


_**default value**_:
```js
async audiences(ctx, sub, token, use) {
  // @param ctx   - koa request context
  // @param sub   - account identifier (subject)
  // @param token - the token to which these additional audiences will be passed to
  // @param use   - can be one of "access_token" or "client_credentials"
  //   depending on where the specific audiences are intended to be put in
  return undefined;
}
```

### claims

Array of the Claim Names of the Claims that the OpenID Provider MAY be able to supply values for.  


_**default value**_:
```js
{
  acr: null,
  sid: null,
  auth_time: null,
  iss: null,
  openid: [
    'sub'
  ]
}
```

### clockTolerance

A `Number` value (in seconds) describing the allowed system clock skew for validating client-provided JWTs, e.g. Request objects and otherwise comparing timestamps  

_**recommendation**_: Only set this to a reasonable value when needed to cover server-side client and oidc-provider server clock skew. More than 5 minutes (if needed) is probably a sign something else is wrong  

_**default value**_:
```js
0
```

### conformIdTokenClaims

ID Token only contains End-User claims when the requested `response_type` is `id_token`  

[Core 1.0 - 5.4. Requesting Claims using Scope Values](https://openid.net/specs/openid-connect-core-1_0.html#rfc.section.5.4) defines that claims requested using the `scope` parameter are only returned from the UserInfo Endpoint unless the `response_type` is `id_token`. This is the default oidc-provider behaviour, you can turn this behaviour off and return End-User claims with all ID Tokens by providing this configuration as `false`.   
  


_**default value**_:
```js
true
```

### cookies

Options for the [cookie module](https://github.com/pillarjs/cookies#cookiesset-name--value---options--) used by the OP to keep track of various User-Agent states.  


### cookies.keys

[Keygrip][keygrip-module] Signing keys used for cookie signing to prevent tampering.  

_**recommendation**_: Rotate regularly (by prepending new keys) with a reasonable interval and keep a reasonable history of keys to allow for returning user session cookies to still be valid and re-signed  

_**default value**_:
```js
[]
```

### cookies.long

Options for long-term cookies  

_**recommendation**_: set cookies.keys and cookies.long.signed = true  

_**default value**_:
```js
{
  secure: undefined,
  signed: undefined,
  httpOnly: true,
  maxAge: 1209600000,
  overwrite: true
}
```

### cookies.names

Cookie names used by the OP to store and transfer various states.  


_**default value**_:
```js
{
  session: '_session',
  interaction: '_interaction',
  resume: '_interaction_resume',
  state: '_state'
}
```

### cookies.short

Options for short-term cookies  

_**recommendation**_: set cookies.keys and cookies.short.signed = true  

_**default value**_:
```js
{
  secure: undefined,
  signed: undefined,
  httpOnly: true,
  maxAge: 600000,
  overwrite: true
}
```

### discovery

Pass additional properties to this object to extend the discovery document  


_**default value**_:
```js
{
  claim_types_supported: [
    'normal'
  ],
  claims_locales_supported: undefined,
  display_values_supported: undefined,
  op_policy_uri: undefined,
  op_tos_uri: undefined,
  service_documentation: undefined,
  ui_locales_supported: undefined
}
```

### dynamicScopes

Array of the dynamic scope values that the OP supports. These must be regular expressions that the OP will check string scope values, that aren't in the static list, against.   
  


_**default value**_:
```js
[]
```
<details>
  <summary>(Click to expand) Example: To enable a dynamic scope values like `api:write:{hex id}` and `api:read:{hex id}`</summary>
  <br>


Configure `dynamicScopes` like so:
  

```js
[
  /^api:write:[a-fA-F0-9]{2,}$/,
  /^api:read:[a-fA-F0-9]{2,}$/,
]
```
</details>

### expiresWithSession

Helper used by the OP to decide whether the given authorization code/ device code or implicit returned access token be bound to the user session. This will be applied to  all tokens issued from the authorization / device code in the future. When tokens are session-bound the session will be loaded by its `uid` every time the token is encountered. Session bound tokens will effectively get revoked if the end-user logs out.  


_**default value**_:
```js
async expiresWithSession(ctx, token) {
  return !token.scopes.has('offline_access');
}
```

### extraClientMetadata

Allows for custom client metadata to be defined, validated, manipulated as well as for existing property validations to be extended  


### extraClientMetadata.properties

Array of property names that clients will be allowed to have defined. Property names will have to strictly follow the ones defined here. However, on a Client instance property names will be snakeCased.  


_**default value**_:
```js
[]
```

### extraClientMetadata.validator

validator function that will be executed in order once for every property defined in `extraClientMetadata.properties`, regardless of its value or presence on the client metadata passed in. Must be synchronous, async validators or functions returning Promise will be rejected during runtime. To modify the current client metadata values (for current key or any other) just modify the passed in `metadata` argument.   
  


_**default value**_:
```js
validator(key, value, metadata) {
  // validations for key, value, other related metadata
  // throw new Provider.errors.InvalidClientMetadata() to reject the client metadata (see all
  //   errors on Provider.errors)
  // metadata[key] = value; to assign values
  // return not necessary, metadata is already a reference.
}
```
<details>
  <summary>(Click to expand) Using extraClientMetadata to allow software_statement dynamic client registration property
</summary>
  <br>

```js
const { verify } = require('jsonwebtoken');
const {
  errors: { InvalidSoftwareStatement, UnapprovedSoftwareStatement },
} = require('oidc-provider');
const softwareStatementKey = require('path/to/public/key');
{
  extraClientMetadata: {
    properties: ['software_statement'],
    validator(key, value, metadata) {
      if (key === 'software_statement') {
        if (value === undefined) return;
        // software_statement is not stored, but used to convey client metadata
        delete metadata.software_statement;
        let payload;
        try {
          // extraClientMetadata.validator must be sync :sadface:
          payload = verify(value, softwareStatementKey, {
            algorithms: ['RS256'],
            issuer: 'Software Statement Issuer',
          });
          if (!approvedStatement(value, payload)) {
            throw new UnapprovedSoftwareStatement('software_statement not approved for use');
          }
          // cherry pick the software_statement values and assign them
          // Note: there will be no further validation ran on those values, so make sure
          //   they're conform
          const { client_name, client_uri } = payload;
          Object.assign(metadata, { client_name, client_uri });
        } catch (err) {
          throw new InvalidSoftwareStatement('could not verify software_statement');
        }
      }
    }
  }
}
```
</details>

### extraParams

Pass an iterable object (i.e. Array or Set of strings) to extend the parameters recognised by the authorization and device authorization endpoints. These parameters are then available in `ctx.oidc.params` as well as passed to interaction session details  


_**default value**_:
```js
[]
```

### findById

Helper used by the OP to load an account and retrieve its available claims. The return value should be a Promise and #claims() can return a Promise too  


_**default value**_:
```js
async findById(ctx, sub, token) {
  // @param ctx - koa request context
  // @param sub {string} - account identifier (subject)
  // @param token - is a reference to the token used for which a given account is being loaded,
  //   is undefined in scenarios where claims are returned from authorization endpoint
  return {
    accountId: sub,
    // @param use {string} - can either be "id_token" or "userinfo", depending on
    //   where the specific claims are intended to be put in
    // @param scope {string} - the intended scope, while oidc-provider will mask
    //   claims depending on the scope automatically you might want to skip
    //   loading some claims from external resources or through db projection etc. based on this
    //   detail or not return them in ID Tokens but only UserInfo and so on
    // @param claims {object} - the part of the claims authorization parameter for either
    //   "id_token" or "userinfo" (depends on the "use" param)
    // @param rejected {Array[String]} - claim names that were rejected by the end-user, you might
    //   want to skip loading some claims from external resources or through db projection
    async claims(use, scope, claims, rejected) {
      return { sub };
    },
  };
}
```

### formats

This option allows to configure the token storage and value formats. The different values change how a client-facing token value is generated as well as what properties get sent to the adapter for storage.
 - `opaque` (default) formatted tokens store every property as a root property in your adapter
 - `jwt` formatted tokens are issued as JWTs and stored the same as `opaque` only with additional property `jwt`. The signing algorithm for these tokens uses the client's `id_token_signed_response_alg` value and falls back to `RS256` for tokens with no relation to a client or when the client's alg is `none`
 - the value may also be a function dynamically determining the format (returning either `jwt` or `opaque` depending on the token itself)   
  


_**default value**_:
```js
{
  extraJwtAccessTokenClaims: [AsyncFunction: extraJwtAccessTokenClaims], // see expanded details below
  AccessToken: undefined,
  ClientCredentials: undefined
}
```
<details>
  <summary>(Click to expand) To enable JWT Access Tokens</summary>
  <br>


Configure `formats`:
  

```js
{ AccessToken: 'jwt' }
```
</details>
<details>
  <summary>(Click to expand) To dynamically decide on the format used, e.g. only if it is intended for more audiences</summary>
  <br>


Configure `formats`:
  

```js
{
  AccessToken(ctx, token) {
    if (Array.isArray(token.aud)) {
      return 'jwt';
    }
    return 'opaque';
  }
}
```
</details>

### formats.extraJwtAccessTokenClaims

helper function used by the OP to get additional JWT formatted token claims when it is being created  


_**default value**_:
```js
async extraJwtAccessTokenClaims(ctx, token) {
  return undefined;
}
```
<details>
  <summary>(Click to expand) To push additional claims to a JWT format Access Token
</summary>
  <br>

```js
{
  formats: {
    AccessToken: 'jwt',
    async extraJwtAccessTokenClaims(ctx, token) {
      return {
        preferred_username: 'johnny',
      };
    }
  }
}
```
</details>

### interactionUrl

Helper used by the OP to determine where to redirect User-Agent for necessary interaction, can return both absolute and relative urls  


_**default value**_:
```js
async interactionUrl(ctx, interaction) {
  return `/interaction/${ctx.oidc.uid}`;
}
```

### interactions

structure of Prompts and their checks formed by Prompt and Check class instances. The default you can modify and the classes are available under `Provider.interaction`.   
  


_**default value**_:
```js
[
  Prompt {
    name: 'login',
    requestable: true,
    details: (ctx) => {
      const { oidc } = ctx;

      return {
        ...(oidc.params.max_age === undefined ? { max_age: oidc.params.max_age } : undefined),
        ...(oidc.params.login_hint === undefined ? { login_hint: oidc.params.login_hint } : undefined),
        ...(oidc.params.id_token_hint === undefined ? { id_token_hint: oidc.params.id_token_hint } : undefined),
      };
    },
    checks: [
      Check {
        reason: 'login_prompt',
        description: 'login prompt was not resolved',
        error: 'login_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          if (oidc.prompts.has(name) && oidc.promptPending(name)) {
            return true;
          }

          return false;
        }
      },
      Check {
        reason: 'no_session',
        description: 'End-User authentication is required',
        error: 'login_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          if (oidc.session.accountId()) {
            return false;
          }

          return true;
        }
      },
      Check {
        reason: 'max_age',
        description: 'End-User authentication could not be obtained',
        error: 'login_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          if (oidc.params.max_age === undefined) {
            return false;
          }

          if (!oidc.session.accountId()) {
            return true;
          }

          if (oidc.session.past(oidc.params.max_age)) {
            return true;
          }

          return false;
        }
      },
      Check {
        reason: 'id_token_hint',
        description: 'id_token_hint and authenticated subject do not match',
        error: 'login_required',
        details: () => {},
        check: async (ctx) => {
          const { oidc } = ctx;
          const hint = oidc.params.id_token_hint;
          if (hint === undefined) {
            return false;
          }

          let payload;
          try {
            ({ payload } = await oidc.provider.IdToken.validate(hint, oidc.client));
          } catch (err) {
            throw new errors.InvalidRequest(`could not validate id_token_hint (${err.message})`);
          }

          let sub = oidc.session.accountId();
          if (sub === undefined) {
            return true;
          }

          if (oidc.client.sectorIdentifier) {
            sub = await instance(oidc.provider).configuration('pairwiseIdentifier')(ctx, sub, oidc.client);
          }

          if (payload.sub !== sub) {
            return true;
          }

          return false;
        }
      },
      Check {
        reason: 'claims_id_token_sub_value',
        description: 'requested subject could not be obtained',
        error: 'login_required',
        details: ({ oidc }) => ({ sub: oidc.claims.id_token.sub }),
        check: async (ctx) => {
          const { oidc } = ctx;
          if (!has(oidc.claims, 'id_token.sub.value')) {
            return false;
          }

          let sub = oidc.session.accountId();
          if (sub === undefined) {
            return true;
          }

          if (oidc.client.sectorIdentifier) {
            sub = await instance(oidc.provider).configuration('pairwiseIdentifier')(ctx, sub, oidc.client);
          }

          if (oidc.claims.id_token.sub.value !== sub) {
            return true;
          }

          return false;
        }
      },
      Check {
        reason: 'essential_acrs',
        description: 'none of the requested ACRs could not be obtained',
        error: 'login_required',
        details: ({ oidc }) => ({ acr: oidc.claims.id_token.acr }),
        check: (ctx) => {
          const { oidc } = ctx;
          const request = get(oidc.claims, 'id_token.acr', {});

          if (!request || !request.essential || !request.values) {
            return false;
          }

          if (!Array.isArray(oidc.claims.id_token.acr.values)) {
            throw new errors.InvalidRequest('invalid claims.id_token.acr.values type');
          }

          if (request.values.includes(oidc.acr)) {
            return false;
          }

          return true;
        }
      },
      Check {
        reason: 'essential_acr',
        description: 'requested ACR could not be obtained',
        error: 'login_required',
        details: ({ oidc }) => ({ acr: oidc.claims.id_token.acr }),
        check: (ctx) => {
          const { oidc } = ctx;
          const request = get(oidc.claims, 'id_token.acr', {});

          if (!request || !request.essential || !request.value) {
            return false;
          }

          if (request.value === oidc.acr) {
            return false;
          }

          return true;
        }
      }
    ]
  },
  Prompt {
    name: 'consent',
    requestable: true,
    details: (ctx) => {
      const { oidc } = ctx;

      const acceptedScopes = oidc.session.acceptedScopesFor(oidc.params.client_id);
      const rejectedScopes = oidc.session.rejectedScopesFor(oidc.params.client_id);
      const acceptedClaims = oidc.session.acceptedClaimsFor(oidc.params.client_id);
      const rejectedClaims = oidc.session.rejectedClaimsFor(oidc.params.client_id);

      const details = {
        scopes: {
          new: [...oidc.requestParamScopes]
            .filter(x => !acceptedScopes.has(x) && !rejectedScopes.has(x)),
          accepted: [...acceptedScopes],
          rejected: [...rejectedScopes],
        },
        claims: {
          new: [...oidc.requestParamClaims]
            .filter(x => !acceptedClaims.has(x) && !rejectedClaims.has(x)),
          accepted: [...acceptedClaims],
          rejected: [...rejectedClaims],
        },
      };

      return omitBy(details, val => val === undefined);
    },
    checks: [
      Check {
        reason: 'consent_prompt',
        description: 'consent prompt was not resolved',
        error: 'consent_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          if (oidc.prompts.has(name) && oidc.promptPending(name)) {
            return true;
          }

          return false;
        }
      },
      Check {
        reason: 'client_not_authorized',
        description: 'client not authorized for End-User session yet',
        error: 'interaction_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          if (oidc.session.sidFor(oidc.client.clientId)) {
            return false;
          }

          return true;
        }
      },
      Check {
        reason: 'native_client_prompt',
        description: 'native clients require End-User interaction',
        error: 'interaction_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          if (
            oidc.client.applicationType === 'native'
            && oidc.params.response_type !== 'none'
            && (!oidc.result || !('consent' in oidc.result))
          ) {
            return true;
          }

          return false;
        }
      },
      Check {
        reason: 'scopes_missing',
        description: 'requested scopes not granted by End-User',
        error: 'consent_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          const promptedScopes = oidc.session.promptedScopesFor(oidc.client.clientId);

          for (const scope of oidc.requestParamScopes) { // eslint-disable-line no-restricted-syntax
            if (!promptedScopes.has(scope)) {
              return true;
            }
          }

          return false;
        }
      },
      Check {
        reason: 'claims_missing',
        description: 'requested claims not granted by End-User',
        error: 'consent_required',
        details: () => {},
        check: (ctx) => {
          const { oidc } = ctx;
          const promptedClaims = oidc.session.promptedClaimsFor(oidc.client.clientId);

          for (const claim of oidc.requestParamClaims) { // eslint-disable-line no-restricted-syntax
            if (!promptedClaims.has(claim) && !['sub', 'sid', 'auth_time', 'acr', 'amr', 'iss'].includes(claim)) {
              return true;
            }
          }

          return false;
        }
      }
    ]
  }
]

```
<details>
  <summary>(Click to expand) configuring prompts
</summary>
  <br>

```js
const { interaction: { Prompt, Check, DEFAULT } } = require('oidc-provider');
// DEFAULT.get(name) => returns a Prompt instance by its name
// DEFAULT.remove(name) => removes a Prompt instance by its name
// DEFAULT.add(prompt, index) => adds a Prompt instance to a specific index, default is to last index
// prompt.checks.get(reason) => returns a Check instance by its reason
// prompt.checks.remove(reason) => removes a Check instance by its reason
// prompt.checks.add(check, index) => adds a Check instance to a specific index, default is to last index
```
</details>

### introspectionEndpointAuthMethods

Array of Client Authentication methods supported by this OP's Introspection Endpoint. If no configuration value is provided the same values as for tokenEndpointAuthMethods will be used. Supported values list is the same as for tokenEndpointAuthMethods.  


_**default value**_:
```js
[
  'none',
  'client_secret_basic',
  'client_secret_jwt',
  'client_secret_post',
  'private_key_jwt'
]
```

### issueRefreshToken

Helper used by the OP to decide whether a refresh token will be issued or not   
  


_**default value**_:
```js
async issueRefreshToken(ctx, client, code) {
  return client.grantTypes.includes('refresh_token') && code.scopes.has('offline_access');
}
```
<details>
  <summary>(Click to expand) To always issue a refresh token if a client has the grant whitelisted</summary>
  <br>


Configure `issueRefreshToken` like so, this is how the removed `alwaysIssueRefresh` feature worked
  

```js
async issueRefreshToken(ctx, client, code) {
  return client.grantTypes.includes('refresh_token');
}
example: To only issue a refresh token if a client has the grant whitelisted and it is offline_access
Configure `issueRefreshToken` like so
```js

async issueRefreshToken(ctx, client, code) {
   return client.grantTypes.includes('refresh_token');
 }
  

```
</details>

### logoutSource

HTML source rendered when when session management feature renders a confirmation prompt for the User-Agent.  


_**default value**_:
```js
async logoutSource(ctx, form) {
  // @param ctx - koa request context
  // @param form - form source (id="op.logoutForm") to be embedded in the page and submitted by
  //   the End-User
  ctx.body = `<!DOCTYPE html>
<head>
<title>Logout Request</title>
<style>/* css and html classes omitted for brevity, see lib/helpers/defaults.js */</style>
</head>
<body>
<div>
  <h1>Do you want to sign-out from ${ctx.host}?</h1>
  <script>
    function logout() {
      var form = document.getElementById('op.logoutForm');
      var input = document.createElement('input');
      input.type = 'hidden';
      input.name = 'logout';
      input.value = 'yes';
      form.appendChild(input);
      form.submit();
    }
    function rpLogoutOnly() {
      var form = document.getElementById('op.logoutForm');
      form.submit();
    }
  </script>
  ${form}
  <button onclick="logout()">Yes, sign me out</button>
  <button onclick="rpLogoutOnly()">No, stay signed in</button>
</div>
</body>
</html>`;
}
```

### pairwiseIdentifier

Function used by the OP when resolving pairwise ID Token and Userinfo sub claim values. See [Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#rfc.section.8.1)  

_**recommendation**_: Since this might be called several times in one request with the same arguments consider using memoization or otherwise caching the result based on account and client ids.  

_**default value**_:
```js
async pairwiseIdentifier(ctx, accountId, client) {
  return crypto.createHash('sha256')
    .update(client.sectorIdentifier)
    .update(accountId)
    .update(os.hostname()) // put your own unique salt here, or implement other mechanism
    .digest('hex');
}
```

### pkceMethods

fine-tune the supported code challenge methods. Supported values are
 - `S256`
 - `plain`  


_**default value**_:
```js
[
  'S256'
]
```

### postLogoutRedirectUri

URL to which the OP redirects the User-Agent when no post_logout_redirect_uri or id_token_hint is provided by the RP   
  


_**default value**_:
```js
async postLogoutRedirectUri(ctx) {
  return ctx.origin;
}
```
<details>
  <summary>(Click to expand) validating post_logout_redirect_uri without id_token_hint</summary>
  <br>


Session Management 1.0 [lacks](https://bitbucket.org/openid/connect/issues/1032/rp-initiated-logout-proposal-for-client_id) the client_id parameter, despite use-cases being there it is not part of the specification, its absence means the provider cannot efficiently load a client to validate a post_logout_redirect_uri. Here's how to add support for a custom client_id parameter so that your clients without id_token_hint can use post_logout_redirect_uri. __Note:__ It is still the best recommended approach to send in the id_token_hint if client has access to it.
  

```js
async postLogoutRedirectUri(ctx) {
  let client
  let clientId
  let providedPostLogoutRedirectUri
  if (ctx.method === 'GET') {
    clientId = ctx.query.client_id
    providedPostLogoutRedirectUri = ctx.query.post_logout_redirect_uri
  } else {
    clientId = ctx.oidc.body.client_id
    providedPostLogoutRedirectUri = ctx.oidc.body.post_logout_redirect_uri
  }
  if (typeof clientId !== 'undefined') {
    client = await ctx.oidc.provider.Client.find(clientId);
    if (!client) {
      throw new InvalidClient('provided client_id is invalid');
    }
  }
  if (client && typeof providedPostLogoutRedirectUri !== 'undefined') {
    if (!client.postLogoutRedirectUriAllowed(providedPostLogoutRedirectUri)) {
      throw new InvalidRequest('post_logout_redirect_uri not registered');
    }
    return providedPostLogoutRedirectUri;
  }
  return ctx.origin; // or any other static value to redirect users to afterwards
}
```
</details>

### renderError

Helper used by the OP to present errors to the User-Agent  


_**default value**_:
```js
async renderError(ctx, out, error) {
  ctx.type = 'html';
  ctx.body = `<!DOCTYPE html>
<head>
<title>oops! something went wrong</title>
<style>/* css and html classes omitted for brevity, see lib/helpers/defaults.js */</style>
</head>
<body>
<div>
  <h1>oops! something went wrong</h1>
  ${Object.entries(out).map(([key, value]) => `<pre><strong>${key}</strong>: ${value}</pre>`).join('')}
</div>
</body>
</html>`;
}
```

### responseTypes

Array of response_type values that OP supports   
  


_**default value**_:
```js
[
  'code id_token token',
  'code id_token',
  'code token',
  'code',
  'id_token token',
  'id_token',
  'none'
]
```
<details>
  <summary>(Click to expand) Supported values list</summary>
  <br>


These are values defined in [Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html#Authentication) and [OAuth 2.0 Multiple Response Type Encoding Practices](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html)
  

```js
[
  'code',
  'id_token', 'id_token token',
  'code id_token', 'code token', 'code id_token token',
  'none',
]
```
</details>

### revocationEndpointAuthMethods

Array of Client Authentication methods supported by this OP's Revocation Endpoint. If no configuration value is provided the same values as for tokenEndpointAuthMethods will be used. Supported values list is the same as for tokenEndpointAuthMethods.  


_**default value**_:
```js
[
  'none',
  'client_secret_basic',
  'client_secret_jwt',
  'client_secret_post',
  'private_key_jwt'
]
```

### rotateRefreshToken

Configures if and how the OP rotates refresh tokens after they are used. Supported values are
 - `false` refresh tokens are not rotated and their initial expiration date is final
 - `true` refresh tokens are rotated when used, current token is marked as consumed and new one is issued with new TTL, when a consumed refresh token is encountered an error is returned instead and the whole token chain (grant) is revoked
 - function returning true/false, true when rotation should occur, false when it shouldn't  


_**default value**_:
```js
true
```
<details>
  <summary>(Click to expand) function use
</summary>
  <br>

```js
async function rotateRefreshToken(ctx) {
  // e.g.
  // return refreshTokenCloseToExpiration(ctx.oidc.entities.RefreshToken);
  // or
  // return refreshTokenRecentlyRotated(ctx.oidc.entities.RefreshToken);
  // or
  // return customClientBasedPolicy(ctx.oidc.entities.Client);
}
```
</details>

### routes

Routing values used by the OP. Only provide routes starting with "/"  


_**default value**_:
```js
{
  authorization: '/auth',
  certificates: '/certs',
  check_session: '/session/check',
  device_authorization: '/device/auth',
  end_session: '/session/end',
  introspection: '/token/introspection',
  registration: '/reg',
  revocation: '/token/revocation',
  token: '/token',
  userinfo: '/me',
  code_verification: '/device'
}
```

### scopes

Array of the scope values that the OP supports  


_**default value**_:
```js
[
  'openid',
  'offline_access'
]
```

### subjectTypes

Array of the Subject Identifier types that this OP supports. Valid types are
 - `public`
 - `pairwise`  


_**default value**_:
```js
[
  'public'
]
```

### tokenEndpointAuthMethods

Array of Client Authentication methods supported by this OP's Token Endpoint  


_**default value**_:
```js
[
  'none',
  'client_secret_basic',
  'client_secret_jwt',
  'client_secret_post',
  'private_key_jwt'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'none',
  'client_secret_basic', 'client_secret_post',
  'client_secret_jwt', 'private_key_jwt',
  'tls_client_auth', 'self_signed_tls_client_auth',
]
```
</details>
<details>
  <summary>(Click to expand) Setting up the environment for tls_client_auth and self_signed_tls_client_auth</summary>
  <br>


To enable mutual TLS based authentication methods the provider expects your TLS-offloading proxy to handle the client certificate validation, parsing, handling, etc. Once set up you are expected to forward `x-ssl-client-verify`, `x-ssl-client-s-dn` and `x-ssl-client-cert` headers with variable values set by this proxy. An important aspect is to sanitize the inbound request headers at the proxy. <br/><br/> The most common openssl based proxies are Apache and NGINX, with those you're looking to use <br/><br/> __`SSLVerifyClient` (Apache) / `ssl_verify_client` (NGINX)__ with the appropriate configuration value that matches your setup requirements. <br/><br/> __`SSLCACertificateFile` or `SSLCACertificatePath` (Apache) / `ssl_client_certificate` (NGINX)__ with the values pointing to your accepted CA Certificates. <br/><br/> Set the proxy request headers with variables set as a result of enabling mutual TLS
  

```nginx
# NGINX
proxy_set_header x-ssl-client-cert $ssl_client_cert;
proxy_set_header x-ssl-client-verify $ssl_client_verify;
proxy_set_header x-ssl-client-s-dn $ssl_client_s_dn;
```
```apache
# Apache
RequestHeader set x-ssl-client-cert  ""
RequestHeader set x-ssl-client-cert "%{SSL_CLIENT_CERT}s"
RequestHeader set x-ssl-client-verify  ""
RequestHeader set x-ssl-client-verify "%{SSL_CLIENT_VERIFY}s"
RequestHeader set x-ssl-client-s-dn  ""
RequestHeader set x-ssl-client-s-dn "%{SSL_CLIENT_S_DN}s"
```
You should also consider hosting the endpoints supporting client authentication, on a separate host name or port in order to prevent unintended impact on the TLS behaviour of your other endpoints, e.g. Discovery or the authorization endpoint and changing the discovery values for them with a post-middleware.
  

```js
provider.use(async (ctx, next) => {
  await next();
  if (ctx.oidc.route === 'discovery' && ctx.method === 'GET' && ctx.status === 200) {
    ctx.body.token_endpoint = '...';
    ctx.body.introspection_endpoint = '...';
    ctx.body.revocation_endpoint = '...';
  }
});
```
When doing that be sure to remove the client provided headers of the same name on the non-mutual TLS enabled host name / port in your proxy setup or block the routes for these there completely.  


</details>

### ttl

Expirations (in seconds, or dynamically returned value) for all token types   
  


_**default value**_:
```js
{
  AccessToken: 3600,
  AuthorizationCode: 600,
  ClientCredentials: 600,
  DeviceCode: 600,
  IdToken: 3600,
  RefreshToken: 1209600
}
```
<details>
  <summary>(Click to expand) To resolve a ttl on runtime for each new token</summary>
  <br>


Configure `ttl` for a given token type with a function like so, this must return a value, not a Promise.
  

```js
{
  ttl: {
    AccessToken(ctx, token, client) {
      // return a Number (in seconds) for the given token (first argument), the associated client is
      // passed as a second argument
      // Tip: if the values are entirely client based memoize the results
      return resolveTTLfor(token, client);
    },
  },
}
```
</details>

### uniqueness

Function resolving whether a given value with expiration is presented first time  

_**recommendation**_: configure this option to use a shared store if client_secret_jwt and private_key_jwt are used  

_**default value**_:
```js
async uniqueness(ctx, jti, expiresAt) {
  if (cache.get(jti)) return false;
  cache.set(jti, true, (expiresAt - epochTime()) * 1000);
  return true;
}
```

### whitelistedJWA

Fine-tune the algorithms your provider will support by declaring algorithm values for each respective JWA use  

_**recommendation**_: Only allow JWA algs that are necessary. The current defaults are based on recommendations from the [JWA specification](https://tools.ietf.org/html/rfc7518) + enables RSASSA-PSS based on current guidance in FAPI. "none" JWT algs are disabled by default but available if you need them.  

### whitelistedJWA.authorizationEncryptionAlgValues

JWA algorithms the provider supports to wrap keys for JWT Authorization response encryption   
  


_**default value**_:
```js
[
  'A128KW',
  'A256KW',
  'ECDH-ES',
  'ECDH-ES+A128KW',
  'ECDH-ES+A256KW',
  'RSA-OAEP'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  // asymmetric RSAES based
  'RSA-OAEP', 'RSA1_5',
  // asymmetric ECDH-ES based
  'ECDH-ES', 'ECDH-ES+A128KW', 'ECDH-ES+A192KW', 'ECDH-ES+A256KW',
  // symmetric AES
  'A128KW', 'A192KW', 'A256KW',
  // symmetric AES GCM based
  'A128GCMKW', 'A192GCMKW', 'A256GCMKW',
  // symmetric PBES2 + AES
  'PBES2-HS256+A128KW', 'PBES2-HS384+A192KW', 'PBES2-HS512+A256KW',
]
```
</details>

### whitelistedJWA.authorizationEncryptionEncValues

JWA algorithms the provider supports to encrypt JWT Authorization Responses with   
  


_**default value**_:
```js
[
  'A128CBC-HS256',
  'A128GCM',
  'A256CBC-HS512',
  'A256GCM'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'A128CBC-HS256', 'A128GCM', 'A192CBC-HS384', 'A192GCM', 'A256CBC-HS512', 'A256GCM',
]
```
</details>

### whitelistedJWA.authorizationSigningAlgValues

JWA algorithms the provider supports to sign JWT Authorization Responses with   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>

### whitelistedJWA.idTokenEncryptionAlgValues

JWA algorithms the provider supports to wrap keys for ID Token encryption   
  


_**default value**_:
```js
[
  'A128KW',
  'A256KW',
  'ECDH-ES',
  'ECDH-ES+A128KW',
  'ECDH-ES+A256KW',
  'RSA-OAEP'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  // asymmetric RSAES based
  'RSA-OAEP', 'RSA1_5',
  // asymmetric ECDH-ES based
  'ECDH-ES', 'ECDH-ES+A128KW', 'ECDH-ES+A192KW', 'ECDH-ES+A256KW',
  // symmetric AES
  'A128KW', 'A192KW', 'A256KW',
  // symmetric AES GCM based
  'A128GCMKW', 'A192GCMKW', 'A256GCMKW',
  // symmetric PBES2 + AES
  'PBES2-HS256+A128KW', 'PBES2-HS384+A192KW', 'PBES2-HS512+A256KW',
]
```
</details>

### whitelistedJWA.idTokenEncryptionEncValues

JWA algorithms the provider supports to encrypt ID Tokens with   
  


_**default value**_:
```js
[
  'A128CBC-HS256',
  'A128GCM',
  'A256CBC-HS512',
  'A256GCM'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'A128CBC-HS256', 'A128GCM', 'A192CBC-HS384', 'A192GCM', 'A256CBC-HS512', 'A256GCM',
]
```
</details>

### whitelistedJWA.idTokenSigningAlgValues

JWA algorithms the provider supports to sign ID Tokens with   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'none',
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>

### whitelistedJWA.introspectionEncryptionAlgValues

JWA algorithms the provider supports to wrap keys for JWT Introspection response encryption   
  


_**default value**_:
```js
[
  'A128KW',
  'A256KW',
  'ECDH-ES',
  'ECDH-ES+A128KW',
  'ECDH-ES+A256KW',
  'RSA-OAEP'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  // asymmetric RSAES based
  'RSA-OAEP', 'RSA1_5',
  // asymmetric ECDH-ES based
  'ECDH-ES', 'ECDH-ES+A128KW', 'ECDH-ES+A192KW', 'ECDH-ES+A256KW',
  // symmetric AES
  'A128KW', 'A192KW', 'A256KW',
  // symmetric AES GCM based
  'A128GCMKW', 'A192GCMKW', 'A256GCMKW',
  // symmetric PBES2 + AES
  'PBES2-HS256+A128KW', 'PBES2-HS384+A192KW', 'PBES2-HS512+A256KW',
]
```
</details>

### whitelistedJWA.introspectionEncryptionEncValues

JWA algorithms the provider supports to encrypt JWT Introspection responses with   
  


_**default value**_:
```js
[
  'A128CBC-HS256',
  'A128GCM',
  'A256CBC-HS512',
  'A256GCM'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'A128CBC-HS256', 'A128GCM', 'A192CBC-HS384', 'A192GCM', 'A256CBC-HS512', 'A256GCM',
]
```
</details>

### whitelistedJWA.introspectionEndpointAuthSigningAlgValues

JWA algorithms the provider supports on the introspection endpoint   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>

### whitelistedJWA.introspectionSigningAlgValues

JWA algorithms the provider supports to sign JWT Introspection responses with   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'none',
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>

### whitelistedJWA.requestObjectEncryptionAlgValues

JWA algorithms the provider supports to receive encrypted Request Object keys wrapped with   
  


_**default value**_:
```js
[
  'A128KW',
  'A256KW',
  'ECDH-ES',
  'ECDH-ES+A128KW',
  'ECDH-ES+A256KW',
  'RSA-OAEP'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  // asymmetric RSAES based
  'RSA-OAEP', 'RSA1_5',
  // asymmetric ECDH-ES based
  'ECDH-ES', 'ECDH-ES+A128KW', 'ECDH-ES+A192KW', 'ECDH-ES+A256KW',
  // symmetric AES
  'A128KW', 'A192KW', 'A256KW',
  // symmetric AES GCM based
  'A128GCMKW', 'A192GCMKW', 'A256GCMKW',
  // symmetric PBES2 + AES
  'PBES2-HS256+A128KW', 'PBES2-HS384+A192KW', 'PBES2-HS512+A256KW',
]
```
</details>

### whitelistedJWA.requestObjectEncryptionEncValues

JWA algorithms the provider supports decrypt Request Objects with encryption   
  


_**default value**_:
```js
[
  'A128CBC-HS256',
  'A128GCM',
  'A256CBC-HS512',
  'A256GCM'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'A128CBC-HS256', 'A128GCM', 'A192CBC-HS384', 'A192GCM', 'A256CBC-HS512', 'A256GCM',
]
```
</details>

### whitelistedJWA.requestObjectSigningAlgValues

JWA algorithms the provider supports to receive Request Objects with   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'none',
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>

### whitelistedJWA.revocationEndpointAuthSigningAlgValues

JWA algorithms the provider supports on the revocation endpoint   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>

### whitelistedJWA.tokenEndpointAuthSigningAlgValues

JWA algorithms the provider supports on the token endpoint   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>

### whitelistedJWA.userinfoEncryptionAlgValues

JWA algorithms the provider supports to wrap keys for UserInfo Response encryption   
  


_**default value**_:
```js
[
  'A128KW',
  'A256KW',
  'ECDH-ES',
  'ECDH-ES+A128KW',
  'ECDH-ES+A256KW',
  'RSA-OAEP'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  // asymmetric RSAES based
  'RSA-OAEP', 'RSA1_5',
  // asymmetric ECDH-ES based
  'ECDH-ES', 'ECDH-ES+A128KW', 'ECDH-ES+A192KW', 'ECDH-ES+A256KW',
  // symmetric AES
  'A128KW', 'A192KW', 'A256KW',
  // symmetric AES GCM based
  'A128GCMKW', 'A192GCMKW', 'A256GCMKW',
  // symmetric PBES2 + AES
  'PBES2-HS256+A128KW', 'PBES2-HS384+A192KW', 'PBES2-HS512+A256KW',
]
```
</details>

### whitelistedJWA.userinfoEncryptionEncValues

JWA algorithms the provider supports to encrypt UserInfo responses with   
  


_**default value**_:
```js
[
  'A128CBC-HS256',
  'A128GCM',
  'A256CBC-HS512',
  'A256GCM'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'A128CBC-HS256', 'A128GCM', 'A192CBC-HS384', 'A192GCM', 'A256CBC-HS512', 'A256GCM',
]
```
</details>

### whitelistedJWA.userinfoSigningAlgValues

JWA algorithms the provider supports to sign UserInfo responses with   
  


_**default value**_:
```js
[
  'HS256',
  'RS256',
  'PS256',
  'ES256'
]
```
<details>
  <summary>(Click to expand) Supported values list
</summary>
  <br>

```js
[
  'none',
  'HS256', 'HS384', 'HS512',
  'RS256', 'RS384', 'RS512',
  'PS256', 'PS384', 'PS512',
  'ES256', 'ES384', 'ES512',
]
```
</details>
<!-- END CONF OPTIONS -->

[client-metadata]: https://openid.net/specs/openid-connect-registration-1_0.html#ClientMetadata
[core-account-claims]: https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims
[core-offline-access]: https://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess
[core-jwt-parameters]: https://openid.net/specs/openid-connect-core-1_0.html#JWTRequests
[revocation]: https://tools.ietf.org/html/rfc7009
[session-management]: https://openid.net/specs/openid-connect-session-1_0-28.html
[got-library]: https://github.com/sindresorhus/got
[request-library]: https://github.com/request/request
[password-grant]: https://tools.ietf.org/html/rfc6749#section-4.3
[token-exchange]: https://tools.ietf.org/html/draft-ietf-oauth-token-exchange
[defaults]: /lib/helpers/defaults.js
[cookie-module]: https://github.com/pillarjs/cookies#cookiesset-name--value---options--
[keygrip-module]: https://www.npmjs.com/package/keygrip
[support-patreon]: https://www.patreon.com/panva
[support-paypal]: https://www.paypal.me/panva
[sponsor-auth0]: https://auth0.com/overview?utm_source=GHsponsor&utm_medium=GHsponsor&utm_campaign=oidc-provider&utm_content=auth
