[![Build Status](https://travis-ci.org/MoveOnOrg/Spoke.svg?branch=main)](https://travis-ci.org/MoveOnOrg/Spoke)

# Spoke

Spoke is an open source text-distribution tool for organizations to mobilize supporters and members into action. Spoke allows you to upload phone numbers, customize scripts and assign volunteers to communicate with supporters while allowing organizations to manage the process.

Spoke was created by Saikat Chakrabarti and Sheena Pakanati, and is now maintained by MoveOn.org.

The latest version is [2.0.0](https://github.com/MoveOnOrg/Spoke/tree/v2.0) (see [release notes](https://github.com/MoveOnOrg/Spoke/blob/main/docs/RELEASE_NOTES.md#v20)) which we recommend for production use, while our `main` branch is where features still in development and testing will be available.

## Deploy to Heroku

<a href="https://heroku.com/deploy">
  <img src="https://www.herokucdn.com/deploy/button.svg" alt="Deploy">
</a>

Follow up instructions located [here](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO_HEROKU_DEPLOY.md).

Please let us know if you deployed by filling out this form [here](https://act.moveon.org/survey/tech/)

## Getting started

### Downloading

1. Install either sqlite (or another [knex](http://knexjs.org/#Installation-client)-supported database)
2. Install the Node version listed under `engines` in `package.json`. [NVM](https://github.com/creationix/nvm) is one way to do this.
3. `yarn install`
5. `cp .env.example .env`
6. If you want to use Postgres:
    - In `.env` set `DB_TYPE=pg`. (Otherwise, you will use sqlite.)
    - Set `DB_PORT=5432`, which is the default port for Postgres.
    - Create the spokedev database:  `psql -c "create database spokedev;"`
7. Create an [Auth0](https://auth0.com) account. In your Auth0 account, go to [Applications](https://manage.auth0.com/#/applications/), click on `Default App` and then grab your Client ID, Client Secret, and your Auth0 domain (should look like xxx.auth0.com). Add those inside your `.env` file (AUTH0_CLIENT_ID, AUTH0_CLIENT_SECRET, AUTH0_DOMAIN respectively).
8. Run `yarn dev` to create and populate the tables.
9. In your Auth0 app settings, add `http://localhost:3000/login-callback` , `http://localhost:3000` and `http://localhost:3000/logout-callback` to "Allowed Callback URLs", "Allowed Web Origins" and  "Allowed Logout URLs" respectively. (If you get an error when logging in later about "OIDC", go to Advanced Settings section, and then OAuth, and turn off 'OIDC Conformant')
10. Add a new [rule](https://manage.auth0.com/#/rules/create) in Auth0:
```javascript
function (user, context, callback) {
context.idToken["https://spoke/user_metadata"] = user.user_metadata;
callback(null, user, context);
}
```
11. Update the Auth0 [Universal Landing page](https://manage.auth0.com/#/login_page), click on the `Customize Login Page` toggle, and copy and paste following code in the drop down into the `Default Templates` space:

    <details>
    <summary>Code to paste into Auth0</summary>

    ```html
    <!DOCTYPE html>
    <html>
    <head>
      <meta charset="utf-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
      <title>Sign In with Auth0</title>
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    </head>
    <body>
      <!--[if IE 8]>
      <script src="//cdnjs.cloudflare.com/ajax/libs/ie8/0.2.5/ie8.js"></script>
      <![endif]-->

      <!--[if lte IE 9]>
      <script src="https://cdn.auth0.com/js/base64.js"></script>
      <script src="https://cdn.auth0.com/js/es5-shim.min.js"></script>
      <![endif]-->
      <script src="https://cdn.auth0.com/js/lock/11.11/lock.min.js"></script>
      <script>
        // Decode utf8 characters properly
        var config = JSON.parse(decodeURIComponent(escape(window.atob('@@config@@'))));
        config.extraParams = config.extraParams || {};
        var connection = config.connection;
        var prompt = config.prompt;
        var languageDictionary;
        var language;

        if (config.dict && config.dict.signin && config.dict.signin.title) {
          languageDictionary = { title: config.dict.signin.title };
        } else if (typeof config.dict === 'string') {
          language = config.dict;
        }
        var loginHint = config.extraParams.login_hint;

        // Available Lock configuration options: https://auth0.com/docs/libraries/lock/v11/configuration
        var lock = new Auth0Lock(config.clientID, config.auth0Domain, {
          auth: {
            redirectUrl: config.callbackURL,
            responseType: (config.internalOptions || {}).response_type ||
              (config.callbackOnLocationHash ? 'token' : 'code'),
            params: config.internalOptions
          },
          // Additional configuration needed for custom domains: https://auth0.com/docs/custom-domains/additional-configuration
          // configurationBaseUrl: config.clientConfigurationBaseUrl,
          // overrides: {
          //   __tenant: config.auth0Tenant,
          //   __token_issuer: 'YOUR_CUSTOM_DOMAIN'
          // },
          assetsUrl:  config.assetsUrl,
          allowedConnections: ['Username-Password-Authentication'],
          rememberLastLogin: !prompt,
          language: language,
          languageDictionary: {
            title: 'Spoke',
            signUpTerms: 'I agree to the <a href="YOUR_LINK HERE" target="_new">terms of service and privacy policy</a>.'
          },
          mustAcceptTerms: true,
          theme: {
            logo:            '',
            primaryColor:    'rgb(83, 180, 119)'
          },
          additionalSignUpFields: [{
            name: 'given_name',
            icon: 'https://upload.wikimedia.org/wikipedia/commons/c/ca/1x1.png',
            placeholder: 'First Name'
          }, {
            name: 'family_name',
            placeholder: 'Last Name',
            icon: 'https://upload.wikimedia.org/wikipedia/commons/c/ca/1x1.png'
          }, {
            name: 'cell',
            placeholder: 'Cell Phone',
            icon: 'https://upload.wikimedia.org/wikipedia/commons/c/ca/1x1.png',
            validator: (cell) => ({
              valid: cell.length >= 10,
              hint: 'Must be a valid phone number'
            })
          }],
          prefill: loginHint ? { email: loginHint, username: loginHint } : null,
          closable: false,
          defaultADUsernameFromEmailPrefix: false,
          // Uncomment if you want small buttons for social providers
          // socialButtonStyle: 'small'
        });
        lock.show();
      </script>
    </body>
    </html>
    ```
    yarn install
    ```
5. Create a real environment file:
    ```
    cp .env.example .env
    ```
  - This creates a copy of `.env.example`, but renames it `.env` so the system will use it. *Make sure you use this new file.*

### Filling out your `.env` file

    </details>
12. If the application is still running from step 8, kill the process and re-run `yarn dev` to restart the app. Wait until you see both "Node app is running ..." and "webpack: Compiled successfully." before attempting to connect. (make sure environment variable `JOBS_SAME_PROCESS=1`)
13. Go to `http://localhost:3000` to load the app.
14. As long as you leave `SUPPRESS_SELF_INVITE=` blank and unset in your `.env` you should be able to invite yourself from the homepage.
    - If you DO set that variable, then spoke will be invite-only and you'll need to generate an invite. Run:
      ```
      echo "INSERT INTO invite (hash,is_valid) VALUES ('abc', 1);" |sqlite3 mydb.sqlite
      # Note: When doing this with PostgreSQL, you would replace the `1` with `true`
      ```
    - Then use the generated key to visit an invite link, e.g.: http://localhost:3000/invite/abc. This should redirect you to the login screen. Use the "Sign Up" option to create your account.
4. You should then be prompted to create an organization. Create it.
5. See the [Admin and Texter demos](https://opensource.moveon.org/spoke-p2p#block-yui_3_17_2_25_1509554076500_36334) to learn about how Spoke works.
6. See [Getting Started with Development](#more-documentation) below.

### SMS

For development, you can set `DEFAULT_SERVICE=fakeservice` to skip using an SMS provider (Twilio or Nexmo) and insert the message directly into the database.

To simulate receiving a reply from a contact you can use the Send Replies utility: `http://localhost:3000/admin/1/campaigns/1/send-replies`, updating the app and campaign IDs as necessary.  You can also include "autorespond" in the script message text, and an automatic reply will be generated (just for `fakeservice`!)

**Twilio**

Twilio provides [test credentials](https://www.twilio.com/docs/iam/test-credentials) that will not charge your account as described in their documentation. To setup Twilio follow our [Twilio setup guide](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO_INTEGRATE_TWILIO.md).


## Getting started with Docker

1. `cp .env.example .env`
2. Follow Steps 7, 9, & 10 above to set up your [Auth0](https://auth0.com) account.
3. Build and run Spoke with `docker-compose up --build`
    - You can stop docker compose at any time with `CTRL+C`, and data will persist next time you run `docker-compose up`.
4. Go to [localhost:3000](http://localhost:3000) to load the app.
5. Follow Step 13 above.
    - But if you need to generate an invite, run:
      ```bash
      docker-compose exec postgres psql -U spoke -d spokedev -c "INSERT INTO invite (hash,is_valid) VALUES ('<your-hash>', true);"
      ```
    - Then use the generated key to visit an invite link, e.g.: `http://localhost:3000/invite/<your-hash>`. This should redirect you to the login screen. Use the "Sign Up" option to create your account.
6. You should then be prompted to create an organization. Create it.
7. When done testing, clean up resources with `docker-compose down`, or `docker-compose down -v` to **_completely destroy_** your Postgres database & Redis datastore volumes.

## Running Tests

See https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO-run_tests.md

1. `cp .env.example .env` and see the "Filling out your `.env` file" section above for some possible tweaks
2. Build and run Spoke with `docker-compose up --build`
    - You can stop docker compose at any time with `CTRL+C`, and data will persist next time you run `docker-compose up`.
3. Go to [localhost:3000](http://localhost:3000) to load the app.
    - But if you need to generate an invite, run:
      ```bash
      docker-compose exec postgres psql -U spoke -d spokedev -c "INSERT INTO invite (hash,is_valid) VALUES ('<your-hash>', true);"
      ```
    - Then use the generated key to visit an invite link, e.g.: `http://localhost:3000/invite/<your-hash>`. This should redirect you to the login screen. Use the "Sign Up" option to create your account.
5. You should then be prompted to create an organization. Create it.
6. When done testing, clean up resources with `docker-compose down`, or `docker-compose down -v` to **_completely destroy_** your Postgres database & Redis datastore volumes.


## More Documentation

* Getting Started with Development:
  * Welcome! Start with [CONTRIBUTING.md](./CONTRIBUTING.md) for community participation and engagement details.
  * [Development Guidelines and Tips](https://github.com/MoveOnOrg/Spoke/blob/main/docs/EXPLANATION-development-guidelines.md)
  * [Running Tests](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO-run_tests.md)
* More Development documentation
  * [A request example](https://github.com/MoveOnOrg/Spoke/blob/main/docs/EXPLANATION-request-example.md) pointing to different code points that all connect to it.
  * [GraphQL Debugging](https://github.com/MoveOnOrg/Spoke/blob/main/docs/graphql-debug.md)
  * [Environment Variable Reference](https://github.com/MoveOnOrg/Spoke/blob/main/docs/REFERENCE-environment_variables.md)
  * [QA Guide](https://github.com/MoveOnOrg/Spoke/blob/main/docs/QA_GUIDE.md)

* Deploying
  * [Deploying with Heroku](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO_HEROKU_DEPLOY.md) (and see Heroku deploy button above)
  * [Deploying on AWS Lambda](https://github.com/MoveOnOrg/Spoke/blob/main/docs/DEPLOYING_AWS_LAMBDA.md)
  * We recommend using [Auth0 for authentication](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO-configure-auth0.md) in deployed environments (Heroku docs have their own instructions)
  * [How to setup Twilio](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO_INTEGRATE_TWILIO.md)
  * [Configuring Email](https://github.com/MoveOnOrg/Spoke/blob/main/docs/EMAIL_CONFIGURATION.md)
  * [Configuring Data Exports](https://github.com/MoveOnOrg/Spoke/blob/main/docs/DATA_EXPORTING.md) works
  * [Using Redis for Caching](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO_CONNECT_WITH_REDIS.md) to improve server performance
  * Configuration for [Enforcing Texting Hours](https://github.com/MoveOnOrg/Spoke/blob/main/docs/TEXTING-HOURS-ENFORCEMENT.md)

* Integrations
  * [ActionKit](https://github.com/MoveOnOrg/Spoke/blob/main/docs/HOWTO_INTEGRATE_WITH_ACTIONKIT.md)

* Administration
  * Description of the different [Roles and Their Permissions](https://github.com/MoveOnOrg/Spoke/blob/main/docs/ROLES_DESCRIPTION.md)
  * Some DB queries for [Texter Activity](https://github.com/MoveOnOrg/Spoke/blob/main/docs/TEXTER_ACTIVITY_QUERIES.md)


## Deploying Minimally

There are several ways to deploy documented below. This is the 'most minimal' approach:

1. Run `OUTPUT_DIR=./build yarn run prod-build-server`
   This will generate something you can deploy to production in ./build and run nodejs server/server/index.js
2. Run `yarn run prod-build-client`
3. Make a copy of `deploy/spoke-pm2.config.js.template`, e.g. `spoke-pm2.config.js`, add missing environment variables, and run it with [pm2](https://www.npmjs.com/package/pm2), e.g. `pm2 start spoke-pm2.config.js --env production`
4. [Install PostgreSQL](https://wiki.postgresql.org/wiki/Detailed_installation_guides)
5. Start PostgreSQL (e.g. `sudo /etc/init.d/postgresql start`), connect (e.g. `sudo -u postgres psql`), create a user and database (e.g. `create user spoke password 'spoke'; create database spoke owner spoke;`), disconnect (e.g. `\q`) and add credentials to `DB_` variables in spoke-pm2.config.js

## Big Thanks
Cross-browser Testing Platform and Open Source <3 Provided by [Sauce Labs](https://saucelabs.com).

# License
Spoke is licensed under the MIT license.

