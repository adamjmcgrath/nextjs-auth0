# @auth0/nextjs-auth0 TEST

Auth0 SDK for signing in to your Next.js applications.

[![CircleCI](https://img.shields.io/circleci/build/github/auth0/nextjs-auth0/main?style=flat-square)](https://circleci.com/gh/auth0/nextjs-auth0/tree/main)
[![NPM version](https://img.shields.io/npm/v/@auth0/nextjs-auth0.svg?style=flat-square)](https://npmjs.org/package/@auth0/nextjs-auth0)
[![License](https://img.shields.io/:license-mit-blue.svg?style=flat)](https://opensource.org/licenses/MIT)

## Table of Contents

- [Installation](#installation)
- [Getting Started](#getting-started)
  - [Auth0 Configuration](#auth0-configuration)
  - [Basic Setup](#basic-setup)
- [Documentation](#documentation)
  - [QuickStart](https://auth0.com/docs/quickstart/webapp/nextjs)
  - [API Reference](#api-reference)
  - [v1 Migration Guide](./V1_MIGRATION_GUIDE.md)
  - [Cookies and Security](#cookies-and-security)
  - [Architecture](./ARCHITECTURE.md)
  - [Comparison with auth0-react](#comparison-with-auth0-react)
  - [Testing](#testing)
- [Contributing](#contributing)
- [Vulnerability Reporting](#vulnerability-reporting)
- [What is Auth0?](#what-is-auth0)
- [License](#license)

## Installation

Using [npm](https://npmjs.org):

```sh
npm install @auth0/nextjs-auth0
```

> Note that this package supports the following versions of Node.js: `^10.13.0 || >=12.0.0` and the following versions of Next.js: `>=10`.

## Getting Started

### Auth0 Configuration

Create a **Regular Web Application** in the [Auth0 Dashboard](https://manage.auth0.com/). If you're using an existing application you'll want to verify that the following settings are configured as follows.

_In your application's Advanced Settings (click "Show Advanced Settings"):_
- **Json Web Token Signature Algorithm**: `RS256`
- **OIDC Conformant**: `True`

_Go ahead and configure the following URLs for your application under Application URIs:_

- **Allowed Callback URLs**: http://localhost:3000/api/auth/callback
- **Allowed Logout URLs**: http://localhost:3000/

Take note of the **Client ID**, **Client Secret** and **Domain** of your application because you'll need it in the next step.

### Basic Setup

The library needs the following required configuration keys. These can be configured in a `.env.local` file in the root of your application (See more info about [loading environmental variables in Next.js](https://nextjs.org/docs/basic-features/environment-variables)):

```sh
# A long secret value used to encrypt the session cookie
AUTH0_SECRET='LONG_RANDOM_VALUE'
# The base url of your application
AUTH0_BASE_URL='http://localhost:3000'
# The url of your Auth0 tenant domain
AUTH0_ISSUER_BASE_URL='https://YOUR_AUTH0_DOMAIN.auth0.com'
# Your Auth0 application's Client ID
AUTH0_CLIENT_ID='YOUR_AUTH0_CLIENT_ID'
# Your Auth0 application's Client Secret
AUTH0_CLIENT_SECRET='YOUR_AUTH0_CLIENT_SECRET'
```

For a [full list of configuration options](https://auth0.github.io/nextjs-auth0/modules/config.html) see the docs.

Then, create a [Dynamic API Route handler](https://nextjs.org/docs/api-routes/dynamic-api-routes) at `/pages/api/auth/[...auth0].js`.

```js
import { handleAuth } from '@auth0/nextjs-auth0';

export default handleAuth();
```

This will create the following urls: `/api/auth/login`, `/api/auth/callback`, `/api/auth/logout` and `/api/auth/me`.

Wrap your `pages/_app.js` component in the `UserProvider` component.

```jsx
// pages/_app.js
import React from 'react';
import { UserProvider } from '@auth0/nextjs-auth0';

export default function App({ Component, pageProps }) {
  return (
    <UserProvider>
      <Component {...pageProps} />
    </UserProvider>
  );
}
```

Check whether a user is authenticated by checking that `user` has a value, and log them in or out from the front end by redirecting to the appropriate automatically-generated route.

```jsx
// pages/index.js
import { useUser } from '@auth0/nextjs-auth0';

export default () => {
  const { user, error, isLoading } = useUser();

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>{error.message}</div>;

  if (user) {
    return (
      <div>
        Welcome {user.name}! <a href="/api/auth/logout">Logout</a>
      </div>
    );
  }

  return <a href="/api/auth/login">Login</a>;
};
```

For more extensive examples see [EXAMPLES.md](./EXAMPLES.md).

## Documentation

### API Reference

- [Configuration Options](https://auth0.github.io/nextjs-auth0/modules/config.html)

**Server Side methods**:

- [handleAuth](https://auth0.github.io/nextjs-auth0/modules/handlers_auth.html)
- [handleLogin](https://auth0.github.io/nextjs-auth0/modules/handlers_login.html#handlelogin)
- [handleCallback](https://auth0.github.io/nextjs-auth0/modules/handlers_callback.html)
- [handleLogout](https://auth0.github.io/nextjs-auth0/modules/handlers_logout.html)
- [handleProfile](https://auth0.github.io/nextjs-auth0/modules/handlers_profile.html)
- [withApiAuthRequired](https://auth0.github.io/nextjs-auth0/modules/helpers_with_api_auth_required.html)
- [withPageAuthRequired](https://auth0.github.io/nextjs-auth0/modules/helpers_with_page_auth_required.html#withpageauthrequired)
- [getSession](https://auth0.github.io/nextjs-auth0/modules/session_get_session.html)
- [getAccessToken](https://auth0.github.io/nextjs-auth0/modules/session_get_access_token.html)
- [initAuth0](https://auth0.github.io/nextjs-auth0/modules/instance.html)

**Client Side methods/components**:

- [UserProvider](https://auth0.github.io/nextjs-auth0/modules/frontend_use_user.html#userprovider)
- [useUser](https://auth0.github.io/nextjs-auth0/modules/frontend_use_user.html)
- [withPageAuthRequired](https://auth0.github.io/nextjs-auth0/modules/frontend_with_page_auth_required.html)

Generated [API Docs](https://auth0.github.io/nextjs-auth0/)

### Cookies and Security

All cookies will be set as `HttpOnly, SameSite=Lax` and will be forced to HTTPS (`Secure`) if the application's `AUTH0_BASE_URL` is `https`.

The `HttpOnly` setting will make sure that client-side javascript is unable to access the cookie to reduce the attack surface of XSS attacks while `SameSite=Lax` will help mitigate CSRF attacks. Read more about SameSite [here](https://auth0.com/blog/browser-behavior-changes-what-developers-need-to-know/).

### Comparison with auth0-react

We also provide a SPA React library [auth0-react](https://github.com/auth0/auth0-react), which may also be suitable for your Next.js application.

The SPA security model used by `auth0-react` is different to the Web Application security model used by this SDK. In short, this SDK protects pages and API routes
with a cookie session (See [Cookies and Security](#cookies-and-security)) whereas a SPA library like `auth0-react` will store the user's ID Token and Access Token
directly in the browser and use them to access external APIs directly.

You should be aware of the security implications of both, but if you are:

- Using [Static HTML Export](https://nextjs.org/docs/advanced-features/static-html-export)
- You do not need to access user data during server-side rendering
- You want to get the Access Token and call external API's directly from the frontend rather than using Next.js API Routes as a proxy to call external APIs

Then [auth0-react](https://github.com/auth0/auth0-react) may be more suitable for your needs.

### Testing

By default, the SDK creates and manages a singleton instance to run for the lifetime of the application. When
testing your application you may need to reset this instance, so its state does not leak between tests.
If you're using Jest, we recommend using `jest.resetModules()` after each test. Alternatively, you can look at
[creating your own instance of the SDK](./EXAMPLES.md#create-your-own-instance-of-the-sdk) so it can be recreated between tests.

## Contributing

We appreciate feedback and contribution to this repo! Before you get started, please see the following:

- [Auth0's general contribution guidelines](./CONTRIBUTING.md)
- [Auth0's code of conduct guidelines](./CODE-OF-CONDUCT.md)

Run NPM install first to install the dependencies of this project:

```sh
npm install
```

In order to build a release, you can run the following commands and the output will be stored in the `dist` folder:

```sh
npm run clean
npm run lint
npm run build
```

Additionally, you can also run tests:

```sh
npm run build:test # Build the Next.js test app
npm run test
npm run test:watch
```

## Vulnerability Reporting

Please do not report security vulnerabilities on the public GitHub issue tracker. The [Responsible Disclosure Program](https://auth0.com/responsible-disclosure-policy) details the procedure for disclosing security issues.

## What is Auth0?

Auth0 helps you to easily:

- Implement authentication with multiple identity providers, including social (e.g., Google, Facebook, Microsoft, LinkedIn, GitHub, Twitter, etc), or enterprise (e.g., Windows Azure AD, Google Apps, Active Directory, ADFS, SAML, etc.)
- Log in users with username/password databases, passwordless, or multi-factor authentication
- Link multiple user accounts together
- Generate signed JSON Web Tokens to authorize your API calls and flow the user identity securely
- Access demographics and analytics detailing how, when, and where users are logging in
- Enrich user profiles from other data sources using customizable JavaScript rules

[Why Auth0?](https://auth0.com/why-auth0)

## License

This project is licensed under the MIT license. See the [LICENSE](LICENSE) file for more info.
