# Rails Elastic Beanstalk Starter App

This starter application and attached guide is perfect for launching a Ruby on Rails application
into elastic beanstalk and configuring it utilize an RDS PostgreSQL instance, S3 for secure
token/password storage, and configuring top to bottom SSL.

This example has been created using ActiveRecord as a session store, but could trivially be
modified to use Redis or another solution.

## Install and Configure Application

#### 1. Clone the repository

`git clone https://github.com/xrnm/rails-elastic-beanstalk.git`

#### 2. Update `.elasticbeanstalk/config.yml`
Update `.elasticbeanstalk/config.yml` with your application, region, cli profile, and other data
Here is an example of a working configuration. Obviously your file will contain different information 

```
branch-defaults:
  master:
    environment: rails-eb-dev
    group_suffix: null
global:
  application_name: rails-beanstalk
  branch: null
  default_ec2_keyname: rails-elastic-beanstalk
  default_platform: Ruby 2.3 (Puma)
  default_region: us-west-2
  include_git_submodules: true
  instance_profile: null
  platform_name: null
  platform_version: null
  profile: eb-cli
  repository: null
  sc: git
  workspace_type: Application

```

#### 3. Update `.ebextensions/01_files.config`
This configuration file executes during deployment and pulls important files from S3 so
sensitive data does not have to be stored in the repository. The 5 files that are stored
in S3 for this starter application are:

1. SSL Key
2. SSL Certificate
3. application.yml - May contain API keypairs or other sensitive data.
4. database.yml - Will contain access credentials to RDS
5. secrets.yml - Contains a secret used by Rails for Cookie verification

You will need to configure your bucket name, region, and the role your elasticbeanstalk ec2 instances
use. This role is `aws-elasticbeanstalk-ec2-role` by default.

#### 4. AWS Configurations
In order to deploy you will need to setup a few things in AWS

1. Create an RDS server and a database.yml file with the connection information. 
2. Create an S3 bucket and upload the 5 files referenced above. Be sure the bucket name
and file names match the changes you made to `.ebextensions/01_files.config`
3. Permit EC2 instances to read the new bucket. In order to do this configure the IAM
role associated with your elastic beanstalk EC2 instances (`aws-elasticbeanstalk-ec2-role` by default)
with the following security policy.
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "s3:Get*",
            "Resource": "arn:aws:s3:::<bucket>/*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "s3:List*",
            "Resource": "arn:aws:s3:::<bucket>/*"
        }
    ]
}
```
Finally, ensure your RDS instance is configured to accept connections from the instances

#### 4. Deploy

Once all of that is completed, from the application root directory commit all of your changes
and execute the command `eb create <environment>` where <environment> is of the same name
as in step 2.

Once the deployment completes do

`eb open`

#### 5. Success


## MIT License

Copyright (c) 2018 Justin Edwards

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

