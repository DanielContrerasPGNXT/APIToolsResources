## Database configuration on Infrastructure project
  * [Infrastructure project](https://gitlab.com/pagofx/emoney/infrastructure):
    - Terraform values -> main/dev/terraform.tfvars
      ```
      <microservice_name> = {
          sql_db_enable = true
          sql_db_create = true
          prefix        = "<microservice_name>"
          schema        = ["<microservice_db_schema_name>"]
          users         = []
          hostname      = "ms"
          zone          = "openapi"
          uris          = ["/<microservice_name>"]
        }
      ```
    - DB Passwords -> main/dev/secrets_passwords.enc.json
      ```
        "db_<microservice_name>_user_password": "<generated_encrypted_password>",
        "db_<microservice_name>_read_user_password": "<generated_encrypted_password>",
        "db_<microservice_name>_editor_user_password": "<generated_encrypted_password>",
      ```

## Database configuration on microservice project
  * gitlab-ci.yml:
    ```
      variables:
        POD_NAME: '<microservice_name>'
        POD_TYPE: 'deployment'
        DB_PREFIX: '<microservice_name>'
        DB_DATABASE: '<microservice_name>-db'
        RUN_FEAT: 'true'
        REGSECRET: 'regsecret-github-microservices'
        SHARED_TOKEN_REGISTRY: $SHARED_TOKEN_REGISTRY
        URI: '<microservice_name>'

      include:
        - template: Security/SAST.gitlab-ci.yml
        - project: pagofx/ci/backend-pipeline
          ref: develop-aws
          file:
            # Shared definitions
            - gitlab-ci/.gitlab-ci-shared.yml
            # Release
            - gitlab-ci/release-github.yml
            # Stage dbcreation
            - gitlab-ci/db-snapshot-downloader.yml
            - gitlab-ci/db-snapshot-downloader-test.yml
            # Stage migrate
            - gitlab-ci/db.yml
            - gitlab-ci/db-test.yml
            - gitlab-ci/db-pg-diff.yml
            # Stage test-lint-build
            # - gitlab-ci/pre-merge.yml
            # Stage build-deploy
            - gitlab-ci/db-snapshot-uploader.yml
            # Stage build deploy
            - gitlab-ci/build-deploy.yml
            # Cleanup
            - gitlab-ci/cleanup.yml
            # SAST
            - gitlab-ci/sast.yml
            # Config maps
            - gitlab-ci/config-maps.yml
      # Stage sonar
      # - local: gitlab-ci/sonarcloud.yml
      #  - local: gitlab-ci/pre-merge.yml
    ```
  * Setup the .npmrc with your authToken for gitlab registry
    ```
      ...
      //gitlab.com/api/v4/packages/npm/:_authToken=<gitlab_generated_auth_token>
      @pagofx:registry=https://gitlab.com/api/v4/packages/npm/
      ...
    ```
  * package.json:
    ```
      "scripts": {
        ...
        "db:local:create": "./scripts/create-local-db $npm_package_config_container_name",
        "db:local:start": "docker start $npm_package_config_container_name",
        "db:local:stop": "docker stop $npm_package_config_container_name",
        "db:local:up": "db-migrate up --migrations-dir './db/migrations' --config './db/database.json' --env 'dev'",
        "db:local:down": "db-migrate down --migrations-dir './db/migrations' --config './db/database.json' --env 'dev'",
        "db:migration:create": "db-migrate create $migration_name --sql-file --migrations-dir './db/migrations' --config './db/database.json' --env 'dev'"
      }
      ...
        "dependencies": {
      ...
      "@pagofx/db": "latest",
      ...
      "db-migrate": "^0.11.12",
      "db-migrate-pg": "^1.2.2",
      ...
      }
    ```
  * infrastructure/**/*-config-map.properties:
    ```
      PGDATABASE=<microservice_name>-db
      DEPLOYMENT_ENVIRONMENT=<envionmnt (dev | pre | pro)>
      EXPOSE_API_DOCS=true
      APP_SHORT_NAME=<microservice_short_name>
    ```
  * infrastructure/**/deployment.yml:
    ```
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: <APP_NAME>
        labels:
          branch: <BRANCH_NAME>
          repo: <REPO>
      spec:
        selector:
          matchLabels:
            app: <APP_NAME>
        replicas: 1
        template:
          metadata:
            labels:
              env: <ENV>
              app: <APP_NAME>
          spec:
            imagePullSecrets:
              - name: <REGSECRET>
            containers:
              - name: <APP_NAME>
                image: <IMAGE>
                ports:
                  - containerPort: 3000
                    name: http-alt
                envFrom:
                  - configMapRef:
                      name: db-config-map
                  - configMapRef:
                      name: common-config-map
                  - configMapRef:
                      name: onetrade-beneficiary-mgmt-config-map
                env:
                  - name: PGUSER
                    valueFrom:
                      secretKeyRef:
                        name: <DB_DATABASE>
                        key: user
                  - name: PGPASSWORD
                    valueFrom:
                      secretKeyRef:
                        name: <DB_DATABASE>
                        key: password
                  - name: MICROSERVICE_NAME
                    value: <REPO>
    ```
  * src/app/index.ts:
    ```
      ...
      import { defaultClient } from '@pagofx/db';
      ...
    ```
  * db/**:
    - Copy the hole folder to the microservice folder
    - Remove migrations folder content
    - run ```npm install```
    - run ```npm run migration```
    - Change db/downloader-entrypoint (with '_' instead of '-' in the microservice_name)):
      ```
      ...
      CREATE USER <microservice_name>_read_user;
      CREATE USER <microservice_name>_editor_user;
      ...
      ```
  * Change (if exists) chart/Chart.yaml:
    ```
    ...
    name: <microservice_name>
    ...
    ```
  * Change (if exists) chart/values.yaml:
    ```
    ...
       repository: <registry>/ccoe-alm/<microservice_name>
       pullPolicy: Always
       tag: latest>

     imagePullSecrets: []
       # Uncomment and remove [] to specify custom secret
       # DEV default secret 'github'; INT default secret 'acr'
       # - name: <namePullSecret>>

     service:
       type: ClusterIP
       portName: http
       port: 80
       targetPort: 80>

     ingress:
       enabled: true
       hosts:
         - host: <microservice_name>.monitor.pagonxt.dev.corp
           paths:
             - path: "/"
               pathType: Prefix
       tls:
         - hosts:
             - <microservice_name>.monitor.pagonxt.dev.corp
    ...
    ```
  * src/db/*.ts:
    >
    > Code needs to be changed to launch the specific microservice CRUD on database
    >
    > - ```src/db/db.ts``` is a wrapper to connect to ```@pagofx/db```library and add logger and custom control, for example:
      ```
        import {
          Client, execute as executeMonorepo,
          executeAll as executeAllMonorepo, findAll as findAllMonorepo,
          findOne as findOneMonorepo, transaction as transactionMonorepo
        } from '@pagofx/db/lib/src';
        import { Query } from '@pagofx/db/lib/src/types/query';
        import { QueryResult } from 'pg';
        import { info } from '../common/logger';

        /**
        * Special logger to pass the db log info to the parent thread.
        *
        * Extracted from monorepo-imports and tweaked to work with parentLogger
        *
        * @param _eventType - Type of the event
        * @param event - The event itself
        */
        // eslint-disable-next-line @typescript-eslint/no-explicit-any
        const dbLog = (_eventType: string, event: any = {}): void => {
          info(
            'onetrade-webhooks.db',
            {'event':JSON.stringify(event),'eventType':_eventType},
            'ACTIVITY'
          );
        };

        /**
        * Performs a Query in the DB and returns an array of results
        *
        * @param query - Query to be performed
        * @returns results
        */
        export const findAll = async <T>(query: Query): Promise<T[]> => {
          return await findAllMonorepo(query, dbLog);
        };

        /**
        * Performs a Query in the DB and returns the first result
        *
        * @param query - Query to be performed
        * @returns results
        */
        export const findOne = async <T>(query: Query): Promise<T> => {
          return await findOneMonorepo(query, dbLog);
        };

        /**
        * Executes a query
        *
        * @param query - Query to be executed
        * @returns results
        */
        export const execute = async (query: Query): Promise<QueryResult> => {
          return await executeMonorepo(query, dbLog);
        };

        /**
        * Creates a transaction and executes all the queries inside it
        *
        * @param transactionName - name of the transaction
        * @param queries - queries to be executed
        * @returns results
        */
        export const executeAll = async (
          transactionName: string,
          queries: Query[],
        ): Promise<QueryResult[]> => {
          return await executeAllMonorepo(transactionName, queries, dbLog);
        };

        /**
        * Create a transaction and execute it
        *
        * @param name - Name of the transaction
        * @param fn - function to execute queries during the transaction
        * @returns A promise that fulfills when the transaction ends
        */
        export const transaction = async <T>(
          name: string,
          fn: (client: Client) => Promise<T>,
        ): Promise<T> => {
          return await transactionMonorepo(name, fn, dbLog);
        };
      ```
    > - ```src/queries/*.ts``` has the queries for the database, using the ```@pagofx/db```methods, at least one file for type of query (select, delete, insert, update), for example:
      ```
        import { sql } from '@pagofx/db/lib/src/sql-tag';
        import { Query } from '@pagofx/db/lib/src/types/query';

        export const getRecords = (): Query => sql(
          'get-records',
        )`
          SELECT
            *
          FROM
            database.records
        `;
      ```
    > - ```src/<microservice_name>DB.ts``` has the specific methoods to call the queries against the database, that should be called from the router methods, for example:
    ```
      import { execute, findAll, findOne } from './db';
      import { error } from '../common/logger';
      import { InternalServerError } from '../common/types/response';
      import { getRecords } from './queries';
      ...
        export const getRecordsDB = async (
        ): Promise<Array<Record>> => {
          try {
            const records: Array<Record> = await findAll(getRecords());
            if(!records.length){
              error(
                <Descriptive_string_to_log>,
                <log_information>,
                <log_type>
              );
            }
            return records;
          } catch (err) {
              error(
                <Descriptive_string_to_log>,
                <log_information>,
                <log_type>
              );
              return [];
          }
        };
      ...
    ```