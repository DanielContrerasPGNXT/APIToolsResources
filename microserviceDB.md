# Add DB to microservice API

## Login with AWS Web console.

 - Login using https://myapplications.microsoft.com/. You have to click the `Amazon_Web_Services_SDA` icon. In case you have several roles available, you will have to choose one of them to continue:
<br/><br/>
 ![iam_roles](./images/iam_roles.png "IAM Roles")\
 <br/><br/>
- Once you select the role and login into the web console, you'll still need to do a `Switch Role`. To do this, click into the `Switch Role` button from your user menu and fill in the required values:
<br/><br/>
 ![switch_role](./images/switch_role.png "Switch Role")\
 <br/><br/>
 ![switch_role2](./images/switch_role2.png "Switch Role")\
<br/><br/>
<br/><br/>

Note: Remember that you have to select this zone in the upper menu: Europe(Ireland) ---- eu-west-1

## Login with AWS CLI.

- In order to login using AWS CLI, the following needs to be done the first time:
  ```
  npm install -g aws-azure-login
  ```
  ```
  export AWS_PROFILE=sso (this profile shouldn't exist, or it will be overwritten)
  ```
  ```
  aws-azure-login --configure
  ````

  You'll need these values for the above command:
  ```
  Azure Tenant ID: 35595a02-4d6d-44ac-99e1-f9ab4cd872db
  Azure App ID URI: https://signin.aws.amazon.com/saml
  ```

  Create a new profile in the `~/.aws/config` file like this (replace with proper values):
  ```
  [profile sso-backend-dev]
  source_profile=sso
  region=eu-west-1
  role_arn=arn:aws:iam::145476053377:role/Backend-dev
  role_session_name=x404872@gruposantander.com
  ```

  From now on, you can login with the following command using your LDAP username/password:
  ```
  aws-azure-login --mode=gui
  ```
  Please note that it will ask you to establish a `session duration`. It must be set to 1 hour or the login process will fail.

  Once logged, export the new `AWS_PROFILE` that you've created in a previous step:
  ```
  export AWS_PROFILE=sso-backend-dev
  ```
  If the token expires, you can renew it this way (replace the `sso` value with your profile's name):
  ```
  aws-azure-login --profile=sso --no-prompt
  ```

## Connect to AWS environment

### Only first time
  First ...
  ```
  aws eks --region eu-west-1 update-kubeconfig --name potd1aireksemoneypmts001
  ```
  ... second
  ```
  ./scripts/aws-ssm-ec2-proxy-command.sh
  ```

### Every time yo need to connect to AWS
To connect:
  ```
  ./scripts/wfh_aws
  ```

### Every time yo need to test your App
To port forwarding your app and your database
  ```
  ./scripts/connect_app <db_boolean> <pod_name> [<branch_name>]
  ```
  After that you can connecto to your DB with ```localhost<microserviceName>-db:5433``` and to your app with ```http://localhost:8080/<microserviceName>```

## Database configuration on Infrastructure project
  * Proyecto de infrastructure:
    - Valores de Terraform -> main/dev/terraform.tfvars
      > todo
    - Passwords de BBDD -> main/dev/secrets_passwords.enc.json
      > todo

## Database configuration on microservice project
  * gitlab-ci.yml:
    - todo
      > todo
  * package.json:
    - todo
      > todo
  * infrastructure/**/*-config-map.properties:
    - todo
      > todo
  * infrastructure/**/deployment.yml:
    - todo
      > todo
  * src/app/index.ts:
    - todo
      > todo
  * db/**:
    - todo
      > todo