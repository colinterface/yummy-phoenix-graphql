# Yummy Phoenix GraphQL

[![Build Status](https://gitlab.com/matthieusegret/yummy-phoenix-graphql/badges/master/build.svg)](https://gitlab.com/matthieusegret/yummy-phoenix-graphql/commits/master)

This application can be used as **starter kit** if you want to get started building an app with **Phoenix**, **React**, and **GraphQL**. This is a simple cooking recipe sharing application using ordinary features which can be found in most web applications.

## Technologies

### Frontend

- [TypeScript](https://github.com/Microsoft/TypeScript) - A superset of JavaScript that compiles to clean JavaScript output.
- [React](https://facebook.github.io/react) - A JavaScript library for building user interfaces. It introduces many great concepts, such as, Virtual DOM, Data flow, etc.
- [Webpack 4](https://github.com/webpack/webpack) - A bundler for javascript and friends. Packs many modules into a few bundled assets.
- [Bulma](https://bulma.io) - Bulma is a modern CSS framework based on Flexbox
- [Apollo 2](http://dev.apollodata.com) - A flexible, fully-featured GraphQL client for every platform.
- [React Final Form](https://github.com/erikras/react-final-form) - High performance subscription-based form state management for React.

### Backend

- Elixir 1.6
- Phoenix 1.3
- Erlang 20.3
- [Absinthe](https://github.com/absinthe-graphql/absinthe) - The [GraphQL](http://graphql.org) toolkit for Elixir.
- [Graphiql](https://github.com/graphql/graphiql) - Graphiql is an in-browser IDE for exploring GraphQL.
- [Arc](https://github.com/stavro/arc) - Flexible file upload and attachment library for Elixir.
- [ExAws](https://github.com/CargoSense/ex_aws) - A flexible, easy to use set of clients AWS APIs for Elixir. Use to store attachments in Amazon S3.
- PostgreSQL for database.

## Features

- CRUD (create / read / update / delete) on recipes
- Creating comments on recipe page
- Pagination on recipes listing
- Searching on recipes
- Authentication with token
- Creating user account
- Update user profile and changing password
- Confirm user account with a code sends by email
- Display comments and recipes in real-time
- Application ready for production

## GraphQL Using

- Queries et mutations
- FetchMore for pagination
- Using `apollo-cache-inmemory`
- Apollo Link (dedup, onError, auth)
- Subscription
- [Managing local state](https://github.com/apollographql/apollo-link-state) with Apollo Link
- Optimistic UI
- [Static GraphQL queries](https://dev-blog.apollodata.com/5-benefits-of-static-graphql-queries-b7fa90b0b69a)
- Validation management and integration with Final Form
- Authentication and authorizations
- Protect queries and mutations on GraphQL API
- Batching of SQL queries backend side

## Prerequisites

- Erlang 20 ([Installing Erlang](https://github.com/asdf-vm/asdf)
- Elixir 1.6 ([Installing Elixir](https://elixir-lang.org/install.html))
- Phoenix 1.3 ([Installing Phoenix](https://hexdocs.pm/phoenix/installation.html))
- Node 8.11 ([Installing Node](https://nodejs.org/en/download/package-manager))
- ImageMagick 7
- PostgreSQL
- _Ruby only for integration tests with FakeS3_

## Getting Started

- Checkout the yummy git tree from Github

          $ git clone https://github.com/MatthieuSegret/yummy-phoenix-graphql.git
          $ cd yummy-phoenix-graphql
          yummy-phoenix-graphql$

- Install dependencies :

          yummy-phoenix-graphql$ mix deps.get

- Create and migrate your database :

          yummy-phoenix-graphql$ mix ecto.create && mix ecto.migrate

- Load sample records:

          yummy-phoenix-graphql$ mix run priv/repo/seeds.exs

- Start Phoenix endpoint

          yummy-phoenix-graphql$ mix phx.server

- Run npm to install javascript package in other terminal:

          yummy-phoenix-graphql$ cd client
          yummy-phoenix-graphql/client$ npm install

- Start client in development mode. You should be able to go to `http://localhost:3000` :

            yummy-phoenix-graphql/client$ npm start

## Testing

Integration tests with [Wallaby](https://github.com/keathley/wallaby) and ChromeDriver. Instructions:

1.  Install **[ChromeDriver](http://chromedriver.chromium.org)**
2.  Install **FakeS3** with `gem install fakes3`. Fake S3 simulate Amazon S3. It minimize runtime dependencies and be more of a development tool to test S3 calls.
3.  Run **FakeS3** in a new terminal window with `fakes3 -r $HOME/.s3bucket -p 4567`
4.  Build frontend with `API_HOST=localhost:4000 npm run build`
5.  Start frontend test server `npm run test.server`
6.  Run tests with `mix test`

## Production with Kubernetes

This example explains how to deploy the application on **[Google Kubernetes Engine](https://cloud.google.com/kubernetes-engine)**.

### Preparing environment

1.  Create account on **[Google Cloud](https://cloud.google.com)** and create the project `yummy-phoenix-graphql`
2.  Install **[gcloud](https://cloud.google.com/sdk)**
3.  Initialize gcloud with `gcloud init`
4.  Install kubectl with gcloud : `gcloud components install kubectl`
5.  Create a static IP address named yummy-phoenix-graphql-ip `gcloud compute addresses create yummy-phoenix-graphql-ip --global`
6.  Install **jq** to parse json in command-line : https://stedolan.github.io/jq
7.  Install **only** the Helm client (helm) : https://docs.helm.sh/using_helm/#installing-helm
8.  Set execute permissions on scripts `chmod +x kubernetes/script/*`

### Building and pushing docker images to Google Container Registry

1.  Install **[Docker](https://docs.docker.com/install)**
2.  Read **[Quickstart for Google Container Registry](https://cloud.google.com/container-registry/docs/quickstart)**
3.  Login to docker registry `gcloud auth configure-docker`
4.  Build frontend image : `docker build -t gcr.io/yummy-phoenix-graphql/frontend:latest -f dockerfiles/frontend.dockerfile .`
5.  Build api backend image : `docker build -t gcr.io/yummy-phoenix-graphql/api:latest -f dockerfiles/api.dockerfile .`
6.  Push images to registry : `docker push gcr.io/yummy-phoenix-graphql/frontend:latest && docker push gcr.io/yummy-phoenix-graphql/api:latest`

### Creating Kubernetes cluster

1.  Create account on **[Sendgrid](https://sendgrid.com)** to send emails
2.  Create account on **[AWS S3](https://aws.amazon.com/fr/s3)** to upload images
3.  Set environement variables :

```
export NAMESPACE=yummy-staging
export PUBLIC_HOST=$(gcloud compute addresses describe yummy-phoenix-graphql-ip --global --format="value(address)")

export SENDGRID_API_KEY=<your-sendgrid-api-key>
export S3_KEY=<your-s3-key>
export S3_SECRET=<your-s3-secret>
export S3_BUCKET=<your-s3-bucket>
```

4.  Create cluster with `./kubernetes/script/create-cluster.sh`
5.  Install yummy helm chart :

```
helm install --name yummy --namespace=$NAMESPACE \
  --set publicHost=$PUBLIC_HOST \
  --set api.sendgridApiKey=$SENDGRID_API_KEY \
  --set api.s3Key=$S3_KEY \
  --set api.s3Secret=$S3_SECRET \
  --set api.s3Bucket=$S3_BUCKET \
  ./kubernetes/yummy
```

5.  Create and populate the Database `./kubernetes/script/create-database.sh`

### Upgrading yummy chart

You can edit or append to the existing customized values and templates, then apply the changes with :

```
helm upgrade yummy ./kubernetes/yummy
```

### Clean up

To avoid incurring charges to your Google Cloud Platform account for the resources used :

1.  Delete cluster with `./kubernetes/script/destroy-cluster.sh`
2.  if you don't want to recreate this cluster in the future, delete the static ip : `gcloud compute addresses delete yummy-phoenix-graphql-ip --global`
3.  Delete any orphaned resources : `gcloud compute backend-services delete --global -q <BACKEND_SERVICES>`

## Next step

- [ ] Migrate from S3 to Google CDN
- [ ] Connect Erlang nodes between them on Kubernetes cluster with **[Peerage](https://github.com/mrluc/peerage)**
- [ ] Migrate frontend from TypeScript to ReasonML

## Screens

#### Listing recipes

<img alt="Listing recipes" src="http://documents.matthieusegret.com/listing-recipes.png" width="500">

#### Editing recipe

<img alt="Editing recipe" src="http://documents.matthieusegret.com/editing-recipe.png" width="500">

#### Recipe page

<img alt="Recipe page" src="http://documents.matthieusegret.com/recipe-page.png" width="500">

#### Account confirmation

<img alt="Account confirmation" src="http://documents.matthieusegret.com/confirmation.png" width="500">

## License

MIT © [Matthieu Segret](http://matthieusegret.com)
