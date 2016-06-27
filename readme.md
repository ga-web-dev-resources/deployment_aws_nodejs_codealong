---
title: Deploying Node.js apps to Amazon Web Services
duration: "1:25"
creator:
    name: Mike Dang
    city: Austin
---

# ![](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png) Deploying Node.js apps to Amazon Web Services

### LEARNING OBJECTIVES
*After this lesson, you will be able to:*
- Explain how to deploy Node.js apps to AWS
- Deploy your app using Elastic Beanstalk
- Set any AWS environment variables

### COMPONENTS


This tutorial is for deploying Node.js applications to [Amazon Web Services (AWS)](https://aws.amazon.com/) using [Elastic Beanstalk](https://aws.amazon.com/). Elastic Beanstalk is similar to Heroku in that it's a [Platform as a Service (PaaS)](https://en.wikipedia.org/wiki/Platform_as_a_service) that enables us to deploy code with little effort and manages the complexities of provisioning resources, auto scaling, monitoring and failover of infrastructure for us.

### Requirements

#### AWS Account

1. Go to the [AWS Homepage](https://aws.amazon.com/)
2. Click the button **Create an AWS Account**
3. Either sign in or follow steps to create a new account

> With new accounts, AWS offers 12 months of [free limited usage](https://aws.amazon.com/free/). A credit card is required - be sure you understand the [pricing structure](http://aws.amazon.com/pricing/) of the various services you want to test out. Be careful not to go over the free usage tier as you'll be responsible for any costs incurred beyond that.

#### AWS Access Key ID and Secret Access Key

Think of the access key ID and secret access key as username/passwords for a user to access AWS resources on your behalf. The account you created is considered the root account, it has full access to everything from billing to the ability to provision new servers for your website.

> It's best practice to create new users with limited privileges for different roles, however in the interest of time for our lesson we'll be using the root account for our sample app. This is not what you would want to do in a production application though. You can learn more about [creating IAM users with limited privileges](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html) in the official documentation.

1. Go to the [Security Credentials](https://console.aws.amazon.com/iam/home?#security_credential) page
2. Expand the tab labeled **Access Keys (Access Key ID and Secret Access Key)**
3. Click the button **Create New Access Key**
4. Click the link **Show Access Key**
5. Either download or copy and paste the **Access Key ID** and **Secret Access Key** into a temporary text file, we'll need them in a little bit

#### EB Command Line Interface (CLI)

The [EB CLI](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3.html) will allow us to create, configure, and deploy an application to AWS all from the command line.

##### Mac OSX Installation

Run the following command in the terminal:

```
$ brew install awsebcli
```

Verify the tool installed successfully:

```
$ eb --version
```

##### Linux Installation

See the instructions [Install Python, pip, and the EB CLI on Linux](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html) for details.

### Setting up a new app for deployment

The following steps are a summary of the [official EB CLI docs](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-configuration.html).

First, create a new project folder or navigate to an existing Node.js project.

```
$ mkdir aws_test && cd aws_test
```

If this is a new project, initialize a new Express application with EJS using the generator.

```
$ express -e
$ npm install
```

The default Express application port is 3000, however, this value needs to be 80 for the application to function properly on AWS. We'll set the ``` process.env.PORT ``` environment variable locally so that we can still use 3000 when we develop, but on AWS it'll default to port 80.

```
$ subl ~/.bash_profile
```

Add the following line at the end of your bash profile so that it'll be found in our Express app:

```
export PORT="3000"
```

Now save the file, and either restart the terminal window or source the profile to avoid closing the window.

```
$ source ~/.bash_profile
```

Now we need to edit our Express application to use port 80 by default if no environment variable exists. Replace '3000' with '80' in your ``` bin/www ``` file.

```js
// /bin/www
var port = normalizePort(process.env.PORT || '80');
```

Another "gotcha" with AWS is that our main file can't be called **app.js** because AWS will automatically search for a file with this name and attempt to load it before running ``` npm start ```. We need to run ``` npm start ``` because the file that needs to run first and foremost is ``` bin/www ```. The following is a workaround for that problem.

Rename the file **app.js** to **main.js**.

```
$ git mv app.js main.js
```

Now we need to change ``` bin/www ``` to reflect the name change from **app** to **main**:

```js
// bin/www
var app = require('../main');
```

Connect this project directory to an AWS environment. For additional details on the different options you'll encounter when initializing the project, see [Initializing an EB CLI Project](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-configuration.html).

```
$ eb init
```

Select a region. For now, let's just use the default.

```
Select a default region
1) us-east-1 : US East (N. Virginia)
2) us-west-1 : US West (N. California)
3) us-west-2 : US West (Oregon)
4) eu-west-1 : EU (Ireland)
5) eu-central-1 : EU (Frankfurt)
6) ap-southeast-1 : Asia Pacific (Singapore)
7) ap-southeast-2 : Asia Pacific (Sydney)
8) ap-northeast-1 : Asia Pacific (Tokyo)
9) sa-east-1 : South America (Sao Paulo)
(default is 3): 3
```

Use the **aws-access-id** and **aws-secret-key** we obtained from the requirements section above.

```
You have not yet set up your credentials or your credentials are incorrect
You must provide your credentials.
(aws-access-id): AKIAJOUAASEXAMPLE
(aws-secret-key): 5ZRIrtTM4ciIAvd4EXAMPLEDtm+PiPSzpoK
```

Select a name for your application. It'll use your folder name as the default but you can change it if you'd like.

```
Enter Application Name
(default is "aws_test"): aws_test
Application aws_test has been created.
```

It will guess that we're setting up a Node.js application. Type **y** to confirm.

```
It appears you are using Node.js. Is this correct?
(y/n): y
```

For now go ahead and set up SSH access to our instances as well. Type **y** to confirm.

```
Do you want to set up SSH for your instances?
(y/n): y
```

For keypair name, just hit Enter to use the default.

```
Type a keypair name.
(Default is aws-eb):
```

For the passphrase, just hit Enter to leave it blank for now.

```
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase):
```

Let's set up an environment. In a more real world scenario, we might have an environment for development, testing, and production. However, remember each instance costs money so to keep within the free usage tier let's only have one environment.

```
$ eb create
```

Use the default environment name or choose a new one.

```
Enter Environment Name
(default is aws-test-dev):
```

Use a unique DNS cache name if prompted (e.g. aws-test-dev). This name has to be unique across all AWS users and will show up as the subdomain for the default elasticbeanstalk.com URL.

```
Enter DNS CNAME prefix
```

Hit Enter to create a default service role.

```
2.0+ Platforms require a service role. We will attempt to create one for you. You can specify your own role using the --service-role option.
Type "view" to see the policy, or just press ENTER to continue:
```

### Set any AWS environment variables

If you're using a cloud database for Mongo or Postgres, you'll need to set the connection string in order for the application to work correctly. In this example, we're using an environment variable called ``` process.env.DB_CONN_SAMPLE_APP ```.

```js
// main.js
mongoose.connect(process.env.DB_CONN_SAMPLE_APP);
```

You might have already set it locally for development, but remember that this environment variable is only on your local machine. Let's add it to the AWS instance so our app can connect to the database using the connection string.

1. Go to the [Elastic Beanstalk Dashboard](https://us-west-2.console.aws.amazon.com/elasticbeanstalk/home) in the AWS Console
2. Click on the name of the Application you created
3. On the left hand side, click **Configuration**
4. Click the gear icon for the box **Software Configuration**
5. Scroll to the section titled **Environment Properties**
6. Add the key (e.g. DB_CONN_SAMPLE_APP) and the value for the connection string in the **Property Value** box
7. Click the button **Apply**

## Deploying application changes

The deployment process should feel familiar, as we've done something very similar with Heroku. Remember, you still have to commit your files in order for them to actually be included in the push to AWS.

```
$ eb deploy
```
