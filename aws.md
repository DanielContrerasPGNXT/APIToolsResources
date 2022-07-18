[Setup User](https://gitlab.com/pagofx/emoney/infrastructure/-/blob/develop/docs/setup_user.md)

## Connect to AWS environment

### Only first time
  Install [aws-cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

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