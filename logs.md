## Observability configuration on microservice project
    - Setup the .npmrc with your authToken for gitlab registry
      ```
        ...
        //gitlab.com/api/v4/packages/npm/:_authToken=<gitlab_generated_auth_token>
        @pagofx:registry=https://gitlab.com/api/v4/packages/npm/
        ...
      ```
    - Import security library on package.json, with the last version published ```npm install @pagoxf/security```
      ```
        ...
        "@pagofx/observability": "1.0.0-beta.0.0.0",
        ...
      ```
    - Create the src/common/logger.ts with something like this
      ```
        import {
          debugWithLogger, errorWithLogger, infoWithLogger, traceWithLogger, warnWithLogger, PagoNxtLogger,  Environment, Platform
        } from '@pagofx/observability';
        import { env } from '@pagofx/utils';
        import winston from 'winston';
        /**
        * Obtains deployment Environment for PagoNxtLogger
        *
        * @returns Environment
        */
        export const getEnvironment = (): Environment => {
          const environment = env('DEPLOYMENT_ENVIRONMENT');
          const deploymentEnvToLoggerEnv: Record<string, Environment> = {
            local: 'LOCAL',
            dev: 'CERT',
            feature: 'FEATURE',
            pro: 'PRO',
            pre: 'UAT',
          };
          return deploymentEnvToLoggerEnv[environment];
        };
        export const consoleLogTransporter = new winston.transports.Console();
        export const logger = new PagoNxtLogger(
          env('SERVICE_NAME'),
          (env('PLATFORM_NAME',{isOptional: true}) ?? <PLATFORM_NAME>) as Platform,
          getEnvironment(),
          consoleLogTransporter,
        );
        export const info = infoWithLogger(logger);
        export const warning = warnWithLogger(logger);
        export const error = errorWithLogger(logger);
        export const debug = debugWithLogger(logger);
        export const trace = traceWithLogger(logger);
      ```
    - Call the logger on every place you need on your microservice, for example:
      ```
        ...
        import { info } from './../common/logger';
        ...
          info(
            'onetrade-webhooks.service',
            { message: `Listening on port ${PORT}` },
            'ACTIVITY'
          );
        ...
      ```
      ```
        ...
        import { error } from '../common/logger';
        ...
          error(
            <Descriptive_string_to_log>,
            <log_information>,
            <log_type>
          );
        ...
      ```

  * Could get more information [here](https://pagonxtconsumer.atlassian.net/wiki/spaces/PAGOFX/pages/526712835/Observability-log+Library)