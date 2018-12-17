## X-Cart GraphQL API documentation

Endpoint - https://mobile.x-cart.com/vl44LF6dBvo2zSDi8-UnON8dD5oIUA8m.php?target=graphql_api

## How to use

You can either go straight to https://xcart.github.io/graphql-api-docs/ or use any GraphiQL-client, e.g. ChromeiQL (https://chrome.google.com/webstore/detail/chromeiql/fkkiamalmpiidkljmicmjfbieiclmeij)

## How to update

You must have @2fd/graphdoc installed globally:

```
npm install -g @2fd/graphdoc
```

After that, simply run this:

```
graphdoc -e https://mobile.x-cart.com/vl44LF6dBvo2zSDi8-UnON8dD5oIUA8m.php?target=graphql_api -o ./docs
```
