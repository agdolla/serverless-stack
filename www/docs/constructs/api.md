---
id: Api
title: "Api"
description: "Docs for the sst.Api construct in the @serverless-stack/resources package"
---

The `Api` construct is a higher level CDK construct that makes it easy to create an API. It provides a simple way to define the routes in your API. And allows you to configure the specific Lambda functions if necessary. It also allows you to configure custom domains. See the [examples](#examples) for more details.

Unlike the lower level [`Function`](function.md) construct, the `Api` construct doesn't directly extend a CDK construct, it wraps around a couple of them.

## Initializer

```ts
new Api(scope: Construct, id: string, props: ApiProps)
```

_Parameters_

- scope [`Construct`](https://docs.aws.amazon.com/cdk/api/latest/docs/constructs.Construct.html)
- id `string`
- props [`ApiProps`](#apiprops)

## Examples

The `Api` construct is designed to make it easy to get started it with, while allowing for a way to fully configure it as well. Let's look at how, through a couple of examples.

### Using the minimal config

```js
new Api(this, "Api", {
  routes: {
    "GET    /notes": "src/list.main",
    "POST   /notes": "src/create.main",
    "GET    /notes/{id}": "src/get.main",
    "PUT    /notes/{id}": "src/update.main",
    "DELETE /notes/{id}": "src/delete.main",
  },
});
```

Note that, the route key can have extra spaces in between, they are just ignored.

### Specifying function props for all the routes

You can extend the minimal config, to set some function props and have them apply to all the routes.

```js
new Api(this, "Api", {
  defaultFunctionProps: {
    srcPath: "src/",
    environment: { tableName: table.tableName },
  },
  defaultAuthorizationType: "AWS_IAM",
  routes: {
    "GET  /notes": "list.main",
    "POST /notes": "create.main",
  },
});
```

### Using the full config

Finally, if you wanted to configure each Lambda function separately, you can pass in the [`ApiRouteProps`](#apirouteprops).

```js
new Api(this, "Api", {
  routes: {
    "GET /notes": {
      authorizationType: "AWS_IAM",
      function: {
        srcPath: "src/",
        handler: "list.main",
        environment: { tableName: table.tableName },
      },
    },
  },
});
```

Note that, you can set the `defaultFunctionProps` while using the `function` per route. The `function` will just override the `defaultFunctionProps`.

```js
new Api(this, "Api", {
  defaultFunctionProps: {
    srcPath: "src/",
  },
  routes: {
    "GET /notes": {
      function: {
        handler: "list.main",
        srcPath: "services/functions/",
      },
    },
    "POST /notes": "create.main",
  },
});
```

So in the above example, the `GET /notes` function doesn't use the `srcPath` that is set in the `defaultFunctionProps`. It'll instead use the one that is defined in the function definition (`services/functions/`).

### Configuring a custom domain

```js {2}
new Api(this, "Api", {
  customDomain: "api.domain.com",
  routes: {
    "GET /notes": "src/list.main",
  },
});
```

### Configuring a wildcard custom domain

```js {2}
new Api(this, "Api", {
  customDomain: "*.domain.com",
  routes: {
    "GET /notes": "src/list.main",
  },
});
```

### Configuring custom domain with all the options

```js {2-6}
new Api(this, "Api", {
  customDomain: {
    domainName: "api.domain.com",
    hostedZone: "domain.com",
    path: "v1",
  },
  routes: {
    "GET /notes": "src/list.main",
  },
});
```

### Getting the function for a route

```js {11}
const api = new Api(this, "Api", {
  routes: {
    "GET    /notes": "src/list.main",
    "POST   /notes": "src/create.main",
    "GET    /notes/{id}": "src/get.main",
    "PUT    /notes/{id}": "src/update.main",
    "DELETE /notes/{id}": "src/delete.main",
  },
});

const listFunction = api.getFunction("GET /notes");
```

### Giving the entire API some permissions

Allow the entire API to access S3.

```js {11}
const api = new Api(this, "Api", {
  routes: {
    "GET    /notes": "src/list.main",
    "POST   /notes": "src/create.main",
    "GET    /notes/{id}": "src/get.main",
    "PUT    /notes/{id}": "src/update.main",
    "DELETE /notes/{id}": "src/delete.main",
  },
});

api.attachPermissions(["s3"]);
```

### Giving a specific route some permissions

Allow one of the routes to access S3.

```js {11}
const api = new Api(this, "Api", {
  routes: {
    "GET    /notes": "src/list.main",
    "POST   /notes": "src/create.main",
    "GET    /notes/{id}": "src/get.main",
    "PUT    /notes/{id}": "src/update.main",
    "DELETE /notes/{id}": "src/delete.main",
  },
});

api.attachPermissionsToRoute("GET /notes", ["s3"]);
```

## Properties

An instance of `Api` contains the following properties.

### httpApi

_Type_: [`cdk.aws-apigatewayv2.HttpApi`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-apigatewayv2.HttpApi.html)

The internally created CDK `HttpApi` instance.

### accessLogGroup?

_Type_: [`cdk.aws-logs.LogGroup`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-logs.LogGroup.html)

If access logs are enabled, this is the internally created CDK `LogGroup` instance.

## Methods

An instance of `Api` contains the following methods.

### getFunction

```ts
getFunction(routeKey: string): Function
```

_Parameters_

- **routeKey** `string`

_Returns_

- [`Function`](function.md)

Get the instance of the internally created [`Function`](function.md), for a given route key. Where the `routeKey` is the key used to define a route. For example, `GET /notes`.

### attachPermissions

```ts
attachPermissions(permissions: Permissions)
```

_Parameters_

- **permissions** [`Permissions`](../util/permissions.md#permissions)

Attaches the given list of [permissions](../util/permissions.md#permissions) to all the routes. This allows the functions to access other AWS resources.

Internally calls [`Function.attachPermissions`](function.md#attachpermissions).

### attachPermissionsToRoute

```ts
attachPermissionsToRoute(routeKey: string, permissions: Permissions)
```

_Parameters_

- **routeKey** `string`

- **permissions** [`Permissions`](../util/permissions.md#permissions)

Attaches the given list of [permissions](../util/permissions.md#permissions) to a specific route. This allows that function to access other AWS resources.

Internally calls [`Function.attachPermissions`](function.md#attachpermissions).

## ApiProps

### routes

_Type_ : `{ [key: string]: FunctionDefinition | ApiRouteProps }`

The routes for this API. Takes an associative array, with the key being the route as a string and the value is either a [`FunctionDefinition`](function.md#functiondefinition).

```js
{
  "GET /notes": "src/list.main",
  "GET /notes/{id}": "src/get.main",
}
```

Or the [ApiRouteProps](#apirouteprops).

```js
{
  "GET /notes": {
    authorizationType: "AWS_IAM",
    function: {
      handler: "src/list.main",
      environment: {
        TABLE_NAME: "notesTable",
      },
    }
  },
}
```

### cors?

_Type_ : `boolean`, _defaults to_ `true`

CORS support for all the endpoints in the API.

### accessLog?

_Type_ : `boolean`, _defaults to_ `true`

CloudWatch access logs for the API.

### customDomain?

_Type_ : `string | ApiCustomDomainProps`

The customDomain for this API. Takes either the domain as a string.

```js
"api.domain.com";

```

Or the [ApiCustomDomainProps](#apicustomdomainprops).

```js
{
  domainName: "api.domain.com",
  hostedZone: "domain.com",
  path: "v1",
}
```

### httpApi?

_Type_ : [`cdk.aws-apigatewayv2.HttpApi`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_aws-apigatewayv2.HttpApi.html), _defaults to_ `undefined`

Optionally, pass in an instance of the CDK `HttpApi`. This will override the default settings this construct uses to create the CDK `HttpApi` internally.

### defaultFunctionProps?

_Type_ : [`FunctionProps`](function.md#functionprops), _defaults to_ `{}`

The default function props to be applied to all the Lambda functions in the API. If the `function` is specified for a route, these default values are overridden.

### defaultAuthorizationType?

_Type_ : `string`, _defaults to_ `true`

The authorization type for all the endpoints in the API. Currently, supports `NONE` or `AWS_IAM`.

## ApiRouteProps

### function?

_Type_ : [`FunctionDefinition`](function.md#functiondefinition)

The function definition used to create the function for this route.

### authorizationType?

_Type_ : `string`, _defaults to_ `NONE`

The authorization type for the specific route. Curently, supports `NONE` or `AWS_IAM`.

## ApiCustomDomainProps

### domainName

_Type_ : `string`

The domain to be assigned to the API endpoint. For ex, `api.domain.com`.

### hostedZone?

_Type_ : `string`, _defaults to the base domain_

The name of the hosted zone in Route 53 that contains the domain. By default, SST will look for a hosted zone by stripping out the first part of the `domainName` that's passed in. So, if your `domainName` is `api.domain.com`. SST will default the `hostedZone` to `domain.com`.

Set this option if SST cannot find the hosted zone in Route 53.

### path?

_Type_ : `string`, _defaults to_ `undefined`

The base mapping for the custom domain. For example, by setting the `domainName` to `api.domain.com` and `path` to `v1`, the custom domain URL for the API will become `https://api.domain.com/v1`. If the `path` is not set, the custom domain URL will be `https://api.domain.com`.

:::caution
You cannot change the path once it has been set.
:::

Note, if the `path` was not defined initially, it cannot be defined later. If the `path` was initially defined, it cannot be later changed to _undefined_. Instead, you'd need to remove the `customDomain` option from the construct, deploy it. And then set it to the new path value.
