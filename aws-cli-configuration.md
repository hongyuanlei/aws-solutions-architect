
#### How AWS CLI looks for credentials and configuration settings?

The AWS CLI looks for credentials and configuration settings in the following order:

- Command Line Options – region, output format and profile can be specified as command options to override default settings.

```
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json
```
To update any of your settings, simply run aws configure again and enter new values as appropriate.

```
# can specify a particular profile name:
$ aws configure --profile profilename
```

- Environment Variables – `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.

```
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
export AWS_SESSION_TOKEN="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```

- The AWS credentials file – located at `~/.aws/credentials`. This file can contain multiple named profiles in addition to a default profile.

```
#~/.aws/credentials
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

[user2]
aws_access_key_id=AKIAI44QH8DHBEXAMPLE
aws_secret_access_key=je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
```

- The CLI configuration file – typically located at `~/.aws/config`. This file can contain a default profile, named profiles, and CLI specific configuration parameters for each.

```
#~/.aws/config
[default]
region=us-west-2
output=json

[profile user2]
region=us-east-1
output=text
```

- Instance profile credentials – these credentials can be used on EC2 instances with an assigned instance role, and are delivered through the Amazon EC2 metadata service.
