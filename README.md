# Rails Elastic Beanstalk Starter App

This starter application and attached guide is perfect for launching a Ruby on Rails application
into elastic beanstalk and configuring it utilize an RDS PostgreSQL instance, S3 for secure
token/password storage, and configuring top to bottom SSL.

This example has been created using ActiveRecord as a session store, but could trivially be
modified to use Redis or another solution.

## Install and Configure Application

#### 1. Clone the repository

`git clone https://github.com/xrnm/rails-elastic-beanstalk.git`

#### 2 Initialize The Application

From the app directory run `eb init`

There is a guided setup process

1. Select a region
2. Create an application
3. Select Ruby 2.3 (Puma)
4. Choose to use code commit or not
5. Setup SSH if you would like

This will create a file `.elasticbeanstalk/config.yml`

#### 3. AWS Configurations
In order to deploy you will need to setup a few things in AWS

1. Create or select an RDS instance to use
2. Create an S3 bucket to hold your configuration files.
3. Create an IAM role for your environment. This is an EC2 role which should have 
  the `AWSElasticBeanstalkWebTier`, `AWSElasticBeanstalkMulticontainerDocker`, and `AWSElasticBeanstalkWorkerTier` policies
4. Add a policy to your new IAM role to read the bucket you just created.
You can use the AWS UI or the below JSON policy. If using the policy JSON Be sure to replace `<bucket>` with your S3 bucket's name
5. Ensure your RDS instance is accessible
6. (Optional) Add a DNS rule to give your site a nice domain name

Policy JSON:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "s3:Get*",
            "Resource": [
                "arn:aws:s3:::*/*",
                "arn:aws:s3:::<bucket>"
            ]
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "s3:List*",
            "Resource": [
                "arn:aws:s3:::*/*",
                "arn:aws:s3:::<bucket>"
            ]
        }
    ]
}
```

#### 4. Create and Upload configuration files
1. Create a database.yml file that contains the endpoint and credentials for your new or existing RDS instance
2. Create your application.yml file to hold any environment variables of interest. `rails-elastic-beanstalk` comes with `figaro` installed
3. Create a secrets.yml file. Use may use `rake secret` to generate a new token
4. Find or create your SSL key and certificate for your desired domain
5. Upload all of these files to your new S3 bucket

#### 5. Update `.ebextensions/01_files.config`
This configuration file executes during deployment and pulls important files from S3 so
sensitive data does not have to be stored in the repository. The 5 files that are stored
in S3 for this starter application are:

1. SSL Key
2. SSL Certificate
3. application.yml - May contain API keypairs or other sensitive data.
4. database.yml - Will contain access credentials to RDS
5. secrets.yml - Contains a secret used by Rails for cookie verification

In `.ebextensions/01_files.config` overwrite the placeholders with your bucket name, region, and the role you created in step 3 

#### 6. Deploy

Once all of that is completed, from the application root directory commit all of your changes
and execute the command `eb create <environment>` You can pick any environment name that serves your purposes

Once the deployment completes do

`eb open`

#### 7. Success

## Limitations and Considerations


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

