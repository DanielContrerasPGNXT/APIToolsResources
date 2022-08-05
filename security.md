## JWT security validation on microservice project
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
      "@pagofx/security": "1.0.0-pfxb-361.0.2.0",
      ...
    ```
  - Add ca-pemstore inside the envFrom tag at files ```infrastructure/*/manifests/deployment.yml```
    ```
      ...
            - configMapRef:
                name: ca-pemstore
      ...
    ```
  - Add these environment variables at files ```infrastructure/*/manifests/*-config-map.properties```
    ```
      ...
        APP_NAME=<microservice_name>
        APP_FULL_NAME=<microservice_name>
        SERVICE_NAME=<microservice_name>

        STS_ES_USERNAME=user
        STS_ES_PASSWORD=pass
        STS_BE_USERNAME=user
        STS_BE_PASSWORD=pass
        STS_GB_USERNAME=user
        STS_GB_PASSWORD=pass

        # For dev and feature enviroments
        STS_BASE_URL=https://sts-estructural-seguridad-dev.apps.ocp02.gts.dev.weu1.azure.paas.cloudcenter.corp:443
        SECURITY_PKM_GET_PUBLIC_KEY_URL=https://pkm-estructural-seguridad-dev.apps.ocp02.gts.dev.weu1.azure.paas.cloudcenter.corp

        # For pre enviroment
        STS_BASE_URL=https://sos-estructural-seguridad-pre.apps.ocp02.gts.pre.weu2.azure.paas.cloudcenter.corp
        SECURITY_PKM_GET_PUBLIC_KEY_URL=https://pkm-estructural-seguridad-pre.apps.ocp02.gts.pre.weu2.azure.paas.cloudcenter.corp

        # For pro environment
        STS_BASE_URL=https://sts-estructural-seguridad-pro.apps.ocp02.gts.pro.weu2.azure.paas.cloudcenter.corp
        SECURITY_PKM_GET_PUBLIC_KEY_URL=https://pkm-estructural-seguridad-pro.apps.ocp02.gts.pro.weu2.azure.paas.cloudcenter.corp

        NODE_TLS_REJECT_UNAUTHORIZED=0

        MAX_CACHED_PUBLIC_KEYS_AMOUNT=2
        REMOVE_CACHED_PUBLIC_KEYS_AMOUNT=2
      ...
    ```
  - Call the middleware before the router on the main app file, src/app/index.ts by default
    ```
      ...
      import { authorizationMiddleware } from '@pagofx/security';
      ...
      app.use(authorizationMiddleware());
      ...
    ```