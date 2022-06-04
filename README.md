[serverless-appsync-plugin](https://github.com/sid88in/serverless-appsync-plugin/) dropped [support for several APIs](https://github.com/sid88in/serverless-appsync-plugin/blob/alpha/doc/upgrading-from-v1.md#single-api-only) in the same stack starting from v2.
This repository shows an example of how you can still deploy several APIs, even if they have shared schemas and resources by leveraging [Serverless Framework Compose](https://www.serverless.com/framework/docs/guides/compose). This is the new reommended way to achieve this.

A common use case is to be able to build a public and private APIs that share some parts of the schema, dataSources and resolvers.

# Shared resources

When working on a project with several APIs, they may share some resources. For example, they might need to access the same DynamoDB table. Thanks to [Serverless Framework Compose](https://www.serverless.com/framework/docs/guides/compose), you can deploy those in a first "common" stack and then pass down the references as input parameters to the underlying services.

example:

```yaml
services:
  common:
    path: common

  private:
      path: private
      dependsOn: common
      params:
        usersTableName: ${common.usersTableName}

    public:
      path: public
      dependsOn: common
      params:
        usersTableName: ${common.usersTableName}
```

For more information, refer to the [official documentation](https://www.serverless.com/framework/docs/guides/compose)

# Shared and extended types

serverless-appsync-plugin supports defining your schema in several files.
This allows you to place common parts of the schema in a file that will be shared by several APIs.

```yaml
schema:
  - ../assets/schemas/*.graphql
  - schema.graphql
```

All the schema files will be merged together

You might also want to extend some types. For example, the admin API might need to expose a `isAdmin` field on the `User` type that you want to keep secret (unexposed) in the public API. You can do so thanks to [schema extension](https://spec.graphql.org/October2021/#sec-Schema-Extension). In your private schema, extend the `User` type:

```graphql
extend type User {
  isAdmin: Boolean!
}
```

for more details and examples, check the [common](assets/schemas//schema.graphql), [public](public/schema.graphql) and [private](private/schema.graphql) schema files.

# Shared data sources and resolvers

When several APIs share some Queries and Mutations, you don't want to have to duplicate the `dataSources` and `resolvers` definitions, or your mapping templates.
To avoid that, you can place them in [separate files](https://www.serverless.com/framework/docs/providers/aws/guide/variables#multiple-configuration-files) and reference them in your API definitions.

```yaml
dataSources:
  # common dataSources
  - ${file(../assets/templates/dataSources.yml)}

resolvers:
  # common resolvers
  - ${file(../assets/templates/usersResolvers.yml)}

  # public only resolvers
  - Query.getMe: users
    Mutation.updateMe: users
```

See [private/serverless.yml](private/serverless.yml) and [public/serverless.yml](public/serverless.yml)
