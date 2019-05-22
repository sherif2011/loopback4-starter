# loopback4-starter

This is a LoopBack 4 starter application to get you upto speed with initial setup of a very standard REST API application with LoopBack 4 as framework. It provides you with all the core requisite features for most of the applications over and above framework features, viz.,

1. Users
2. Authentication
3. Authorization
4. Environment configs
5. Audit logs
6. DB upgrade, migration and versioning

In addition to that, it also provides you with some of the good to have items

1. Multi-tenant architecture system
2. Soft deletes

## Table of Contents

- [How to use](#how-to-use)
  - [Clone the repository](#clone-the-repository)
  - [Install dependencies](#install-dependencies)
  - [Update app name](#update-app-name)
  - [Setup database](#setup-database)
  - [Setup redis](#setup-redis)
  - [Configure environment](#configure-environment)
  - [Run DB migrations](#run-db-migrations)
  - [Start server](#start-server)
- [Architecture overview](#architecture-overview)
  - [Database Model](#database-model)
    - [Key Notes](#key-notes)
    - [Multi-tenant system](#multi-tenant-system)
  - [Authentication](#authentication)
  - [Authorization](#authorization)
  - [Soft Delete](#soft-delete)
  - [Audit Logs](#audit-logs)
- [Feedback](#feedback)
- [Contributing](#contributing)
- [Code of conduct](#code-of-conduct)
- [License](#license)

### How to use

Please follow below guidelines to use this starter app to create a fresh LB4 application.

_Do remember that this is just one of the way to use it. You can always build your own LB4 application using LB4 CLI as mentioned [here](https://loopback.io/doc/en/lb4/Command-line-interface.html). Then, you can just cherry-pick what you need from here. Clone this repo in separate directory, then copy-paste. :)_

#### Clone the repository

```sh
git clone https://github.com/sourcefuse/loopback4-starter.git
```

#### Install dependencies

```sh
npm i
```

#### Update app name

Say the application you are developing is named 'To Do List'. Change as below.

- Rename the directory to to-do-list.
- package.json and package-lock.json - Replace loopback4-starter with to-do-list in entire file. Update description.
- public/index.html - Update title tag and h1 tag inside body as per your app name.
- src/application.ts - Rename class to ToDoListApplication and all its references. If you are using VSCode, select the class name, press F2 and then rename it. This will update all its references as well.
- src/index.ts - Replace all refences of Loopback4StarterApplication with ToDoListApplication.
- src/\_\_tests\_\_/\*\* - Replace all refences of Loopback4StarterApplication with ToDoListApplication.
- Update README.md with details of your application
- Update CONTRIBUTING.md with details as per your application.

#### Setup database

Database supported in this starter project is postgresql. But you can use any other SQL database. You will need to replace _loopback-connector-postgresql_ in dependencies with your required connector. If you are using postgresql, follow as below.
Setup a [postgresql](https://www.postgresql.org/) instance. Create a database named **todolist**.
DB Schema will be setup automatically via our aplication later.

We are using [DbSchema](https://www.dbschema.com/) in this starter project to design our ERD for database. You can replace lbstarter.dbs with your own schema file.

#### Setup redis

This starter project uses redis as key value db for keeping revoked tokens (logout).
Setup a redis instance. Follow the quick start guide [here](https://redis.io/topics/quickstart).

#### Configure environment

You need to configure your environment variables now. Copy [.env.example](https://github.com/sourcefuse/loopback4-starter/blob/master/.env.example) and rename as .env. Now provide values for the keys mentioned there. These are going to be database credentials ([created here](#setup-database)) and redis credentials ([created here](#setup-redis)). You don't need to provide all the keys though. For some of these (not needed to be secure keys), we have already specified default values in [.env.defaults](https://github.com/sourcefuse/loopback4-starter/blob/master/.env.defaults). You can remove them. You will require to fill at least these ones.

```json
DB_HOST=localhost
DB_PORT=5432
DB_USER=postgres
DB_PASSWORD=********
DB_DATABASE=todolist
DB_SCHEMA=todolist
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_URL=
REDIS_PASSWORD=
REDIS_DATABASE=0
```

#### Run DB migrations

In this project, we are using [db-migrate](https://db-migrate.readthedocs.io/en/latest/) for db migrations, upgrades and versioning.

LoopBack4 also provides [auto migration](https://loopback.io/doc/en/lb4/Database-migrations.html) from models to DB but considering that its not guaranteed to be safe as mentioned [here](https://loopback.io/doc/en/lb4/Database-migrations.html#overview), we prefer to keep DB stuff outside of the ORM layer. This also helps in abstracting our models in codebase from DB tables. If you are good with LB4 way, please ignore below.

Run the below command to setup the database schema and tables with initial seed data.

```sh
npm run db:migrate
```

The above will setup the database as per the diagram [here](#database-model). It will also add a default first super admin user. Details for that is as follows

```json
username=super_admin
password=test123!@#
```

#### Start server

Start the node server as below.

```sh
npm start
```

In a browser, visit http://127.0.0.1:3000/ping

### Architecture overview

#### Database Model

![db-schema](https://github.com/sourcefuse/loopback4-starter/blob/master/db-schema.png)

##### Key Notes

- Database used is Postgresql.
- Database schema is based on multi-tenant architecture overall.
- Each table have default columns - id (Primary Key), created_by, modified_by, created_on, modified_on, deleted.

##### Multi-tenant system

- Tenants are the organisational entities using the application.
- Tenant types - Customer, Application (Umbrella Tenant for super admin work).
- Every user will need be associated to a tenant in order to access the application.
- User tenant has a m:n relationship, user_tenants table is the link table for the same.
- Every user will have a role associated for every tenant it belongs to, role_id in user_tenants table.
- Every role has associated permissions.
- Each user may have some extra permissions (allowed or denied) per tenant over and above its role, user_tenant_permissions table takes care of it.

For detailed description of database tables and columns, please open lbstarter-schema.html from root folder in your browser.

#### Authentication

This starter project uses [loopback4-authentication](https://github.com/sourcefuse/loopback4-authentication) package for authentication purposes.

_If you only want to use authentication, not the whole starter codebase, you can use the package directly in you application. Refer [loopback4-authentication](https://github.com/sourcefuse/loopback4-authentication) documentation for more details._

There are two different strategies of authentication used here.

- **Oauth2 client password + Passport local** - This is a 2-step auth process

  1. Client need to send client credentials (client id and client public key) and user credentials (username and password) to '/auth/login' API. After successful verification, it will return a jwt code containing client details, signed using client's private key (secret) stored in DB. Expiration time of code will be based on value in DB against that auth client.
  2. Next, client will need to send this code along with client id and username to '/auth/token' API. After successful verification, it will return access token (jwt containing user details) and refresh token. Expiration time of token will be based on value in DB against that auth client.

  _The front end application will be mostly using this strategy._

- **Oauth2 resource owner password** - This is a single step process where in client need to send client credentials (client id and client public key) and user credentials (username and password) to '/auth/login-token' API. In this case, in addition to verifying client credentials and user password, system will also check if the user has permission to access APIs via this auth client. Only those auth clients which have associated userids can use this API for authentication.

  _Any 3rd party application accessing your APIs will be using this strategy, like from AWS Lambda function._

Once access token is obtained, it is needed to be passed into every API as Authorization bearer token header, which will be validated for access to API.

#### Authorization

This starter project uses [loopback4-authorization](https://github.com/sourcefuse/loopback4-authorization) package for authorization purposes.

_If you only want to use authorization, not the whole starter codebase, you can use the package directly in you application. Refer [loopback4-authorization](https://github.com/sourcefuse/loopback4-authorization) documentation for more details._

Permissions are part of the access token generated. Authorization implementation is following the [Role based permissions with user level override strategy here](https://github.com/sourcefuse/loopback4-authorization#loopback4-authorization).

#### Soft Delete

This starter project uses [loopback4-soft-delete](https://github.com/sourcefuse/loopback4-soft-delete) package for authorization purposes.

#### Audit Logs

Audit logs are updated using DB triggers in this project. Refer to the lbstarter-schema.html for the details.

### Feedback

If you've noticed a bug or have a question or have a feature request, [search the issue tracker](https://github.com/sourcefuse/loopback4-starter/issues) to see if someone else in the community has already created a ticket.
If not, go ahead and [make one](https://github.com/sourcefuse/loopback4-starter/issues/new/choose)!
All feature requests are welcome. Implementation time may vary. Feel free to contribute the same, if you can.
If you think this extension is useful, please [star](https://help.github.com/en/articles/about-stars) it. Appreciation really helps in keeping this project alive.

### Contributing

Please read [CONTRIBUTING.md](https://github.com/sourcefuse/loopback4-starter/blob/master/.github/CONTRIBUTING.md) for details on the process for submitting pull requests to us.

### Code of conduct

Code of conduct guidelines [here](https://github.com/sourcefuse/loopback4-starter/blob/master/.github/CODE_OF_CONDUCT.md).

### License

[MIT](https://github.com/sourcefuse/loopback4-starter/blob/master/LICENSE)

<br><br>

[![LoopBack](<https://github.com/strongloop/loopback-next/raw/master/docs/site/imgs/branding/Powered-by-LoopBack-Badge-(blue)-@2x.png>)](http://loopback.io/)
