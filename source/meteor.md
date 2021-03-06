---
title: Meteor
order: 152
description: Specifics about using Apollo in your Meteor application.
---

The Apollo client and server tools are published on Npm, which makes them available to all JavaScript applications, including those written with [Meteor](https://www.meteor.com/) 1.3 and above. When using Meteor with Apollo, you can use those npm packages directly, or you can use the [`apollo` Atmosphere package](https://github.com/apollostack/meteor-integration/), which simplifies things for you.

To install `apollo`, run these commands:

```text
meteor add apollo
meteor npm install --save apollo-client graphql-server-express express graphql graphql-tools body-parser
```

## Usage

You can see this package in action in the [Apollo Meteor starter kit](https://github.com/apollostack/meteor-starter-kit).

### Client

Connect to the Apollo server with [`meteorClientConfig`](#meteorClientConfig):

```js
import ApolloClient from 'apollo-client';
import { meteorClientConfig } from 'meteor/apollo';

const client = new ApolloClient(meteorClientConfig());
```

### Server

Define your schema and resolvers, and then set up the Apollo server with [`createApolloServer`](#createApolloServer):

```js
import { createApolloServer } from 'meteor/apollo';
import { makeExecutableSchema, addMockFunctionsToSchema } from 'graphql-tools';

import typeDefs from '/imports/api/schema';
import resolvers from '/imports/api/resolvers';

const schema = makeExecutableSchema({
  typeDefs,
  resolvers,
});

createApolloServer({
  schema,
});
```

The [GraphiQL](https://github.com/graphql/graphiql) url by default is [http://localhost:3000/graphiql](http://localhost:3000/graphiql)

Inside your resolvers, if the user is logged in, their id will be `context.userId` and their user doc will be `context.user`:

```js
export const resolvers = {
  Query: {
    user(root, args, context) {
      // Only return the current user, for security
      if (context.userId === args.id) {
        return context.user;
      }
    },
  },
  User: ...
}
```

## API

### meteorClientConfig

`meteorClientConfig(networkInterfaceConfig)`

`networkInterfaceConfig` may contain any of the following fields:
- `path`: path of the GraphQL server. Default: `'/graphql'`.
- `options`: `FetchOptions` passed to [`createNetworkInterface`](http://dev.apollodata.com/core/apollo-client-api.html#createNetworkInterface). Default: `{}`.
- `useMeteorAccounts`: Whether to send the current user's login token to the GraphQL server with each request. Default: `true`.

Returns an [`options` object](http://dev.apollodata.com/core/apollo-client-api.html#apollo-client) for `ApolloClient`:

```
{
  networkInterface
  queryTransformer: addTypenameToSelectionSet
  dataIdFromObject: object.__typename + object._id
}
```

### createApolloServer

`createApolloServer(options, config)`

- `options`: [Apollo Server `options`](http://dev.apollodata.com/tools/apollo-server/setup.html#apolloOptions)
- `config` may contain any of the following fields:
  - `path`: [Path](http://expressjs.com/en/api.html#app.use) of the GraphQL server. Default: `'/graphql'`.
  - `configServer`: Function that is given the express server for further configuration. For example: `{configServer: expressServer => expressServer.use(cors())}`
  - `maxAccountsCacheSizeInMB`: User account ids are cached in memory to reduce the response latency on multiple requests from the same user. Default: `1`.
  - `graphiql`: Whether to enable GraphiQL. Default: `true` in development and `false` in production.
  - `graphiqlPath`: Path for GraphiQL. Default: `/graphiql` (note the i).
  - `graphiqlOptions`: [GraphiQL options](http://dev.apollodata.com/tools/apollo-server/graphiql.html#graphiqlOptions) (optional).



It will use the same port as your Meteor server. Don't put a route or static asset at the same path as the Apollo route or the GraphiQL route if in use (defaults are `/graphql` and `/graphiql` respectively).
