## X-Cart GraphQL API documentation

Demo endpoint - https://mobile.x-cart.com/admin.php?target=graphql_api

## How to use

You can either go straight to https://xcart.github.io/graphql-api-docs/ or use any GraphiQL-client, e.g. ChromeiQL (https://chrome.google.com/webstore/detail/chromeiql/fkkiamalmpiidkljmicmjfbieiclmeij)

## How to publish changes

Replace the `schema.graphql` file to change GraphQL Api schema definition file (you can create it by using [get-graphql-schema](https://www.npmjs.com/package/get-graphql-schema) tool.

You must have Ruby and bundler gem in order to render documentation website. Run the following cli commands to generate documentation:

```
bundle install
bundle exec rake
```

After that, simply commit and push them to the repo.
