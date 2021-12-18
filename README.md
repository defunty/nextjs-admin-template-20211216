# progress
doing...

# about
This is nextjs-admin-template for myself.

- TypeScript
- Nothing css (You can style freely.)

# provisioning

## Hasura

- Change TZ in docker-compose.yaml.

- Up docker-compose
```
docker-compose up -d
```
- To http://localhost:8080/console
- Connect database 
- Database URL is PG_DATABASE_URL in docker-compose.yaml

- Create users tables.
```
users table
  id (type integer auto-increment)
  name (type text)
  created_at (type timestamp and default now())
  last_seen (type timestamp and nullable)

  primary key is id
```

```
todos table
  id (type integer;auto-increment)
  title (type text)
  is_completed (type boolean and default false)
  is_public (type boolean and default false)
  created_at (type timestamp and default now())
  user_id (type integer)

  primary key is id

  foreign key
    Reference Table is users
    From: user_id
    To: id
```

- Create todos relationships
```
Create Realationships
  todos . user_id  → users . id
  Name: user
```

## Auth0
- Create Auth0 application and do something.
https://manage.auth0.com/

- Add Custom Rule
```hasura-jwt-claim
function (user, context, callback) {
  const namespace = "https://hasura.io/jwt/claims";
  context.accessToken[namespace] = 
    { 
      'x-hasura-default-role': 'user',
      // do some custom logic to decide allowed roles
      'x-hasura-allowed-roles': ['user'],
      'x-hasura-user-id': user.user_id
    };
  callback(null, user, context);
}
```

```insert-user
function (user, context, callback) {
  const userId = user.user_id;
  const nickname = user.nickname;

  const admin_secret = "xxxx";
  const url = "xxxx";

  request.post({
      headers: {'content-type' : 'application/json', 'x-hasura-admin-secret': admin_secret},
      url:   url,
      body:    `{\"query\":\"mutation($userId: String!, $nickname: String) {\\n          insert_users(\\n            objects: [{ id: $userId, name: $nickname }]\\n            on_conflict: {\\n              constraint: users_pkey\\n              update_columns: [last_seen, name]\\n            }\\n          ) {\\n            affected_rows\\n          }\\n        }\",\"variables\":{\"userId\":\"${userId}\",\"nickname\":\"${nickname}\"}}`
  }, function(error, response, body){
       console.log(body);
       callback(null, user, context);
  });
}
```

Caution: Change "admin_secret" in insert-user rule

```sync-user
function (user, context, callback) {
  const userId = user.user_id;
  const nickname = user.nickname;
  
  const admin_secret = "xxxx";
  const url = "xxxx";
  const query = `mutation($userId: String!, $nickname: String) {
    insert_users(objects: [{
      id: $userId, name: $nickname, last_seen: "now()"
    }], on_conflict: {constraint: users_pkey, update_columns: [last_seen, name]}
    ) {
      affected_rows
    }
  }`;

  const variables = { "userId": userId, "nickname": nickname };

  request.post({
      url: url,
      headers: {'content-type' : 'application/json', 'x-hasura-admin-secret': admin_secret},
      body: JSON.stringify({
        query: query,
        variables: variables
      })
  }, function(error, response, body){
       console.log(body);
       callback(null, user, context);
  });
}
```

Caution: Change "admin_secret" in insert-user rule


- Generate JWT Config
https://hasura.io/jwt-config/


- Set JWT config as environment variables
```
cp ./docker/docker-compose.yaml ./
```

```docker-compose.yaml
HASURA_GRAPHQL_JWT_SECRET: '{"type": ... "}'
```

### Authentication debug (if you need)
- Install extension (Auth0 Authentication API Debugger)

- Configuration
  - Select "Application"
  - Memo "Callback URL"

- Add "(Authentication API Debugger) Callback URL" to "Allowed callback URL" in "Application Properties"
```
ex:
http://localhost:3000/callback,
https://~~~/auth0-authentication-api-debugger
```

- Back to Authentication API Debugger configuration
  - select OAuth2 / OIDC
  - Click OAUTH2 / OIDC LOGIN
  - If success, you can get access_token.








# todo
- [] State
  - [] SWR or useState

- [] GraphQl
  - [] Apollo Client
  - [] Hasura Engine
  - [] Auth0

- [] CRUD(basical)
  - [] React Hook Form

// if can ...
// - [] Test
//   - [] Jest
//   - [] Storybook
//   - [] Cyplus


# Scrap

[hasura.io - Fullstack GraphQL Tutorials](https://hasura.io/learn/)
[docker.com - grahql-engine image](https://hub.docker.com/r/hasura/graphql-engine)
[auth0.com - Test Applications Locally](https://auth0.com/docs/configure/applications/work-with-auth0-locally)
[auth0.com - Authentication API Debugger Extension document](https://auth0.com/docs/extensions/authentication-api-debugger-extension)
[github.com - auth0-authentication-api-debugger-extension](https://github.com/auth0-extensions/auth0-authentication-api-debugger-extension)

# my question
- Hasuraでtextからintegerにコンバートしたい場合どうする？（データを保存してそれらをゴニョゴニョして新しいカラムに入れる方法も知っておきたい）
- Postgresでschema情報をsql形式exportするコマンド知りたい（それをHasuraからsqlでimportできるようにしたい）
