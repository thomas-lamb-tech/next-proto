# 🔥 Next.js Starter using GraphQL, MobX 🔥
*Production-ready* `Next.js` boilerplate, `GraphQL` and `MobX` based. [Erion Classroom](https://classroom.erion.io) uses this boilerplate in production. Authentication status can be stored in cookie with `Secure`,`HttpOnly` options.

## Translations
- [한국어](/translations/ko.md) 🇰🇷

## Pre-Installed
The following libraries / Framework are pre-installed.

### React.js and core framework
- [Next.js](https://nextjs.org)
- [TypeScript](https://www.typescriptlang.org)
- [Babel](https://babeljs.io)
- [Express.js](https://expressjs.com)

### GraphQL
- [Apollo Client](https://github.com/apollographql/apollo-client)
- [React Apollo](https://github.com/apollographql/react-apollo)
- [React Apollo Hooks](https://github.com/trojanowski/react-apollo-hooks)
- [GraphQL Code Generator](https://graphql-code-generator.com)

### State Management
- [MobX 5](https://github.com/mobxjs/mobx)

### Styling
- [styled-components](https://www.styled-components.com)

### Asset Management
- [next-optimized-images](https://github.com/cyrilwanner/next-optimized-images)

### Deployment
- [Serverless Framework](https://serverless.com)
- [AWS Lambda](https://aws.amazon.com/lambda)
- [Dotenv](https://github.com/motdotla/dotenv)

# 1. Getting Started
```bash
# Clone the repository
$ git clone https://github.com/tonyfromundefined/next-starter
$ cd next-starter

# Flush git project
$ rm -rf .git

# Initialize your own git project
$ git init

# Install dependencies
$ yarn

# Start development server
$ yarn dev
```

# 2. Folder Structure
### 1. `/api`
You can create a sub-API based on *Express.js*. You can dynamically `Set-Cookie` by `ajax` call.
> Recommendation👏: Let the URL be a `* .json` to distinguish it from the REST API that returns JSON.

### 2. `/apollo`
*Apollo Client* settings. injects `accessToken` in the Authorization Header in the GraphQL request.
- Implementation about refreshing the `accessToken` is required

### 3. `/generated`
Codes automatically generated by *GraphQL Code Generator* is saved.

### 4. `/pages`
File-based page routing. All the aliases are handled through `export {default}`, but all implementations are done inside `/services`.
```typescript
export { default } from '~/services/home/pages/index'
```
> I've separated the code with service module for future scalability. Separate common elements used in page implementations such as `/queries`, `/helpers`, `/components` by service name [https://softwareengineering.stackexchange.com/questions/338597/folder-by-type-or-folder-by-feature](https://softwareengineering.stackexchange.com/questions/338597/folder-by-type-or-folder-by-feature)

### 5. `/serverless`
Entry JavaScript files (CommonJS) for a serverless deployment.
> Recommendation👏: Separate entry files into service units.

### 6. `/services`
divide big application into several abstract services and store in a folder. (`/home`, `/auth`, ...)
> Recommendation👏: Isolate components, and also business logic in service unit folder
- `~/services/{service}/components/**.tsx`
- `~/services/{service}/queries/**.graphql`
- `~/services/{service}/helpers/**.ts`
- `~/services/{service}/assets/**.png`
- `~/services/{service}/types/**.ts`
- ...

### 7. `/store`
One store that is globally used. Because most of the cache processing is handled by the *Apollo Client*, the Store is only used for global status management, such as maintaining a authentication (storing `jsonwebtoken`).

### 8. `/styled`
Implementation of themes, global css styles

### 9. `/types`
Type declarations. (`.d.ts`)
> Recommendation👏: The type used in each service unit should be stored in `~/services/{service}/types` in each service.

# 3. Development
```bash
# Start development server
$ yarn dev
```

## Use MobX store in Components
with Hooks API

##### `~/services/home/pages/index.tsx`
```tsx
import { useStore } from '~/store'

export default function PageIndex() {
  const store = useStore()

  /* ... */
}
```

## GraphQL Query, Mutation with GraphQL Code Generator and `react-apollo-hooks` in Components
- Edit GraphQL Endpoint (`NEXT_APP_GRAPHQL_ENDPOINT`) in `.env.development`, `.env.production`
  ##### `.env.development`
  ```
  NEXT_APP_STAGE = "development"
  NEXT_APP_GRAPHQL_ENDPOINT = "https://graphql-pokemon.now.sh/"
  NEXT_APP_VERSION = "0.0.1"
  ```

- create a `.graphql` file in service unit folder (`~/services/{service}/queries/**.graphql`)
  ##### `~/services/home/queries/getPikachu.graphql`
  ```graphql
  query getPikachu {
    pokemon(name: "Pikachu") {
      id
      number
      name
    }
  }
  ```

- read all `.graphql` files in project and generate HOCs, hooks, components
  ```bash
  $ yarn generate
  ```

- Import the created hook, and utilize
  ##### `~/services/home/components/Pikachu.tsx`
  ```tsx
  import { useGetPikachuQuery } from '~/generated/graphql'

  export default function Pikachu() {
    const { data, loading, error } = useGetPikachuQuery()

    /* ... */
  }
  ```

## Styled Components with Theme Injections
- Inject variables that used globally
  ##### `./styled/themes/base.ts`
  ```ts
  import openColor from 'open-color'

  const theme = {
    ...openColor,
  }
  ```

- Use the variables in `styled` statements with `props.theme`.
  ```tsx
  const Title = styled.h3`
    font-size: 1.5rem;
    background-color: ${(props) => props.theme.yellow[4]};
    margin: .5rem 0 1rem;
    border-radius: .25rem;
    padding: .25rem;
  `
  ```

## Inject Environment variables with MobX store hydration
To deploy multiple stage in a single build, it doesn't use `dotenv-webpack`. (Include environment variable in webpack bundle) Instead of including variables in webpack, it inject server's environment variables in MobX state. So, server and client can use variables in server and client both.
- To use environment variables in client and server both, append `NEXT_APP` in variable name
  ##### `.env.**`
  ```
  NEXT_APP_VERSION = "1.0.0"
  ```
- To use environment variables in server only by security reason, do not append `NEXT_APP` in variable name
  ##### `.env.**`
  ```
  SECRET_KEY = "dc35abc5-80e1-5725-8a7b-7a6ce1a21c24"
  ```

# 4. Build
```bash
# Build server and client bundles
$ yarn build
```

# 5-a. Deployment on Server (EC2, Elastic Beanstalk, Docker, ...)
```bash
# Run server with bundled assets in 80 port
$ yarn start
```

# 5-b. Deployment on Serverless (AWS Lambda + API Gateway + S3)
## Pre-requisites
- 🔑 **IAM Account** for *Serverless framework* (Requires pre-configuration using `aws configure`)

  ```bash
  $ aws configure
  ```

## Configuration
Edit `serverless.yml`

```yaml
service: next-starter  # 1. Edit service name

plugins:
  - serverless-s3-sync
  - serverless-apigw-binary
  - serverless-dotenv-plugin

package:
  individually: true
  excludeDevDependencies: false

provider:
  name: aws
  runtime: nodejs10.x
  stage: ${opt:stage, 'dev'}
  region: us-east-1  # 2. Edit AWS region name

custom:
  #######################################
  # Unique ID included in resource names.
  # Replace it with a random value for every first distribution.
  # https://www.random.org/strings/?num=1&len=6&digits=on&loweralpha=on&unique=on&format=html&rnd=new
  stackId: '0uelbz'  # 3. Update Random Stack ID
  #######################################

  buckets:
    ASSETS_BUCKET_NAME: ${self:service}-${self:custom.stackId}-${self:provider.stage}-assets
  s3Sync:
    - bucketName: ${self:custom.buckets.ASSETS_BUCKET_NAME}
      localDir: dist
  apigwBinary:
    types:
      - '*/*'

functions:
  main:
    name: ${self:service}-${self:custom.stackId}-${self:provider.stage}-main
    handler: dist/serverless/bundles/main.handler
    memorySize: 2048
    timeout: 10
    environment:
      NODE_ENV: production
    package:
      include:
        - dist/serverless/bundles/main.js
      exclude:
        - '**'
    events:
      - http:
          path: /
          method: any
      - http:
          path: /{proxy+}
          method: any
      - http:
          path: /_next/{proxy+}
          method: any
          integration: http-proxy
          request:
            uri: https://${self:custom.buckets.ASSETS_BUCKET_NAME}.s3.${self:provider.region}.amazonaws.com/{proxy}
            parameters:
              paths:
                proxy: true

  # 4. If you implement more than 1 entry, add entries.
  # hello:
  #   name: ${self:service}-${self:custom.stackId}-${self:provider.stage}-hello
  #   handler: dist/serverless/bundles/hello.handler
  #   memorySize: 2048
  #   timeout: 10
  #   environment:
  #     NODE_ENV: production
  #   package:
  #     include:
  #       - dist/serverless/bundles/hello.js
  #     exclude:
  #       - '**'
  #   events:
  #     - http:
  #         path: /
  #         method: any
  #     - http:
  #         path: /{proxy+}
  #         method: any
```

## Run deployment by serverless framework
```bash
# development stage
$ yarn deploy:dev

# staging stage
$ yarn deploy:stage

# production stage
$ yarn deploy:prod
```

# 6. To-do
If you have a feature request, please create a new issue. And also, pull requests are always welcome🙏
