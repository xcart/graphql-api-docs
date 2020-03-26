## X-Cart GraphQL API documentation

Demo endpoint - https://mobile.x-cart.com/admin.php?target=graphql_api

## How to use

You can either go straight to https://xcart.github.io/graphql-api-docs/ or use any GraphiQL-client, e.g. ChromeiQL (https://chrome.google.com/webstore/detail/chromeiql/fkkiamalmpiidkljmicmjfbieiclmeij)

## How to regenerate

You must have Ruby and bundler gem. Run the following cli commands to generate documentation:

```
bundle install
bundle exec rake
```

Replace the `schema.graphql` file to change schema.