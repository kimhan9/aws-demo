# IAM Database Authentication for MySQL
Users can connect to an Amazon RDS DB instance or cluster using IAM user or role credentials and an authentication token.

## Activate IAM DB authentication
Enable `IAM database authentication` by using the Amazon RDS console.

## Create a database user account that uses an AWS authentication token
Connect to the DB instance or cluster endpoint by with master credential. Run command to create user:
```sh
CREATE USER {dbusername} IDENTIFIED WITH AWSAuthenticationPlugin as 'RDS';
```
By default, the database user is created with no privileges. To require a user account to connect using SSL and other privileges, run command:
```sh
ALTER USER {dbusername} REQUIRE SSL;
GRANT SELECT, INSERT, UPDATE, DELETE, ALTER ON *.* TO '{dbusername}'@'%';
```

### Add an IAM policy
Enter a policy that allows the rds-db:connect action to the required user.
```sh
{
    "Version": "2012-10-17",
    "Statement": [
       {
          "Effect": "Allow",
          "Action": [
              "rds-db:connect"
          ],
          "Resource": [
              "arn:aws:rds-db:ap-southeast-1:111111111111:dbuser:cluster-XXXXXXXXXXXX/*"
          ]
       }
    ]
}
```

## Connect to the RDS DB using IAM role credentials
### Download SSL Certificates
Download the AWS RDS Certificate pem file.
```sh
wget https://truststore.pki.rds.amazonaws.com/ap-southeast-1/ap-southeast-1-bundle.pem
```

### Generate authentication token and connect to DB
You have to generate authentication token first to use to connect to DB. The Authentication tokens have a lifespan of 15 minutes.
```sh
RDSHOST="myrds.ap-southeast-1.rds.amazonaws.com"
DBUSERNAME={dbusername}
TOKEN="$(aws rds generate-db-auth-token --hostname $RDSHOST --port 3306 --region ap-southeast-1 --username $DBUSERNAME)"
mysql --host=$RDSHOST --port=3306 --ssl-ca=/fullpathtopem/ap-southeast-1-bundle.pem --ssl-mode=VERIFY_CA --enable-cleartext-plugin --user=$DBUSERNAME --password=$TOKEN

```

### Note
1. It has tested with mysql cli successfully. For othe mysql client, please read their documentation.
1. The username do not have to be same as AWS account.
1. User still can login using traditional username/password. However another new user need create for login using IAM authentication.
1. User has to login using RDS given endpoint. Alternative DNS name like do not work.
1. The Authentication tokens have a lifespan of 15 minutes.
1. In general, consider using IAM database authentication when your applications create fewer than 200 connections per second, and you don't want to manage usernames and passwords directly in your application code.


##### References
[1] - [AWS Docs - IAM database authentication for MariaDB, MySQL, and PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAMDBAuth.Connecting.html)

[2] - [AWS re:Post - Amazon RDS with IAM credentials](https://aws.amazon.com/premiumsupport/knowledge-center/users-connect-rds-iam/)

[3] - [miztiik - IAM Database Authentication for MySQL](https://github.com/miztiik/AWS-Demos/tree/master/How-To/setup-iam-access-to-rds/)
