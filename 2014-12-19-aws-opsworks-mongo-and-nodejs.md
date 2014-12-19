Amazon OpsWorks - node.js app with MongoDB setup
============================================

[Amazon OpsWorks](http://docs.aws.amazon.com/opsworks/latest/userguide/welcome.html) provides a way to manage AWS resources
using [Chef recipes](http://docs.chef.io/recipes.html).

Here I describe a simple setup of the single-instance node.js app with single-node MongoDB server.
It is similar to the php application + mysql setup described
in the OpsWorks [Getting Started guide](http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted_intro.html).

The OpsWorks setup includes:
* [Stack](http://docs.aws.amazon.com/opsworks/latest/userguide/workingstacks.html) - a container for the deployment process we will setup
* Two [Layers](http://docs.aws.amazon.com/opsworks/latest/userguide/workinglayers.html) - node.js app and MongoDB
* Two [Instances](http://docs.aws.amazon.com/opsworks/latest/userguide/workinginstances.html) - one EC2 instance for node app and another for mongo
* One [Application](http://docs.aws.amazon.com/opsworks/latest/userguide/workingapps.html) - this is the code we will deploy

MongoDb setup is based on [this blog post](http://blogs.aws.amazon.com/application-management/post/Tx1RB65XDMNVLUA/Deploying-MongoDB-with-OpsWorks).
We will use [MongoDB Chef cookbook](https://github.com/edelight/chef-mongodb) which allow us to deploy different MongoDB setups - single instance,
replica set, sharding, replica sets with sharding.
It also possible to manage mongo users and setup [MongoDB Monitoring System (MMS)](https://mms.mongodb.com/) agent.

Also the detailed step-by-step guide on working with OpsWorks UI is [here](http://docs.aws.amazon.com/opsworks/latest/userguide/gettingstarted_intro.html), so
I will describe only important steps and problems I had.
Also the difference from the standard setup is that I actually launched two node apps on the node.js server instance - the main application and
the test application (both are in the same repository).

* Create a new stack, set a name and check other default parameters
 * It is good to create and set the 'Default SSH key', so you will be able to ssh to instances later to debug problems with the setup
* Add Node.js App Server layer - select 'Node.js App Server' as a type
* Add MongoDB layer - select 'Custom' as a type

## Node.js layer setup.

As I mentioned above I have two node apps which I wanted to launch on the same server - main app and test app.
The main app works on the port 80 and the test app on the 3300.
Repository structure is this:

```bash
├── app
├── public
├── package.json
├── server.js
├── ...
└── test-app
    ├── bin/www - this is the main app entry point
    ├── app.js
    ├── ...
```

The [standard node.js layer](http://docs.aws.amazon.com/opsworks/latest/userguide/workinglayers-node.html) uses chef recipes from [the opsworks cookbooks repository](https://github.com/aws/opsworks-cookbooks/tree/release-chef-11.10/opsworks_nodejs) and has some limitations:
* The main file must be named server.js and reside in the deployed application's root directory.
* Express apps must include a package.json file in the application's root directory.
* The application must listen on port 80 (for HTTP requests) or port 443 (for HTTPS requests).

In my case the main app already had the required layout (it is based on [mean.js](http://meanjs.org/)).
I only had to setup port 80 for the production environment.

And the test app required a simple custom chef recipe and some additional layer settings.
To use custom recipes it is necessary to create a separate [cookbook repository](http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-installingcustom-repo.html).
It can be, for example, public or private git repository on github or bitbucket.
In my case I used private bitbucket repository with following structure:

```bash
├── Berksfile
├── mongodb-singlenode
│   ├── metadata.rb
│   └── recipes
│       └── default.rb
├── nodeapp
│   ├── metadata.rb
│   ├── recipes
│   │   └── default.rb
│   └── templates
│       └── default
│           └── app_test.monitrc.erb
└── README.md
```

Here I have custom recipe for mongodb setup (described below) and for the nodeapp.
The nodeapp/metadata.rb just contains a cookbook metadata, see [example](https://github.com/aws/opsworks-cookbooks/blob/release-chef-11.10/opsworks_nodejs/metadata.rb).
The recipes/default.rb is this:

```ruby
deploy = node[:deploy]['my_app']

script "setup_test_app" do
  interpreter "bash"
  user "root"
  cwd "#{deploy[:deploy_to]}/current/app-test"
  code <<-EOH
    npm install -d
  EOH
end

template "#{node.default[:monit][:conf_dir]}/node_web_app-app-test.monitrc" do
  source 'app_test.monitrc.erb'
  owner 'root'
  group 'root'
  mode '0644'
  variables(
    :deploy => deploy,
    :deploy_to => "#{deploy[:deploy_to]}/current/app-test",
    :test_port => 3300,
    :test_env => 'production',
    :application_name => 'app-test',
    :monitored_script => "#{deploy[:deploy_to]}/current/app-test/bin/www"
  )
  notifies :restart, "service[monit]", :immediately
end
```

This recipes invokes `npm install -d` inside the test app folder and then creates a monit service config.
So our test app will be watched by monit and restarted in the case of failure.
This is the same setup the standard OpsWorks node.js layer uses for the main application.

The `templates/default/app_test.monitrc.erb` contains a config file template:

```erb
check host node_web_app_<%= @application_name %> with address 127.0.0.1
  start program = "/bin/sh -c 'cd <%= @deploy_to %> ; source <%= @deploy_to %>/../shared/app.env ; /usr/bin/env PORT=<%= @test_port %> NODE_ENV=<%= @test_env %> NODE_PATH=<%= @deploy_to %>/node_modules:<%= @deploy_to %> /usr/local/bin/node <%= @monitored_script %>'"
  stop program = "/usr/bin/pkill -f 'node <%= @monitored_script %>'"
  <% if @deploy[:ssl_support] -%>
  if failed port <%= @test_port %> type TCPSSL protocol HTTP
  <% else -%>
  if failed port <%= @test_port %> protocol HTTP
  <% end -%>
    request /
    with timeout 10 seconds
    then restart
```

This way we setup monit to run and monitor the test app.
App is started with command like `PORT=3300 NODE_ENV=production path/to/deployment/current/app-test/bin/www`.

Now change the layer settings:
* Custom recipe
 * For the Node.js App Server select 'Recipes', click 'Edit'
 * Set the custom recipes repository URL and add 'nodeapp::default' recipe to the 'Deploy' step.
  * This way we will launch our custom recipe each time the app is deployed
* Network - optional step, here you may want to enable Elastic IP, so the app server will always have the same IP
* Security - here you need to add a custom security group to the layer to open port 3300 for our test app
 * Open the EC2 service, Security Groups, create new security group and allow port 3300, [see also docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html)
 * Go back to the node app layer settings, Security tab and add a custom security group you created

## MongoDB layer setup

The setup is based on [this blog post](http://blogs.aws.amazon.com/application-management/post/Tx1RB65XDMNVLUA/Deploying-MongoDB-with-OpsWorks).
It uses the custom chef recipe from our cookbook repository:

```bash
├── Berksfile
├── mongodb-singlenode
│   ├── metadata.rb
│   └── recipes
│       └── default.rb
├── nodeapp
│   ├── ...
```

Here we have Berksfile which describes dependencies:

```ruby
source 'https://supermarket.getchef.com'

cookbook 'mongodb'
```

The `mongodb-singlenode/metadata.rb` with dependency info:

```ruby
name        "mongodb-singlenode"
description 'MongoDB single node Berkshelf based install'
maintainer  "Company"
license     "Apache 2.0"
version     "1.0.0"

depends 'mongodb'
```

And a custom recipe `mongodb-singlenode/recipes/default.rb`:

```ruby
node.normal[:mongodb][:config][:bind_ip] = "127.0.0.1,#{node[:opsworks][:instance][:private_ip]}"

include_recipe "mongodb::default"
```

The important part here is the first string where we app instance private ip to the 'bind_ip' parameter.
This way the MongoDB instance will be available to our node.js app server.

The way I set the `bind_ip` parameter is different from the [recommended](https://github.com/edelight/chef-mongodb#single-mongodb-instance)
because I had an [issue with recommended setup](https://github.com/edelight/chef-mongodb#single-mongodb-instance).
Also it is necessary to use Amazon Linux instance for mongo because there is also a problem (solvable, but I didn't tested the solution) with ubuntu setup.

Now open the mongo layer in the OpsWorks UI, 'Recipes' settings and add our custom `mongodb-singlenode::default` recipe to the 'Setup' step.

## Finalize the setup

Now add one instance to each layer. You can do this from both `Layers` and `Instances` page in the OpsWorks UI.
I use Ubuntu 14.04 for node.js app layer and Amazon Linux 2014.09 for mongo layer.
Put some meaningful name to the mongo instance `Hostname` parameter - this data goes to `/etc/hosts` on each instance.

For example, if the mongo host is 'my-app-db' then the node.js app can connect to the database using `mongodb://my-app-db/dbname` connection string.

Add an application (from `Apps` page). Essential parameters are:
* Type - Node.js
* Data source type - None (we use custom mongo setup)
* Set repository parameters
* Add an environment variable: NODE_ENV - production

Now go to 'Deployments' and deploy an app.
If everything is done right it will setup mongodb and node.js and deploy application code.
In the case of failure the OpsWorks will display a link to the log file.

## Additional information and related links

* MongoDB documentation [has a section about setup on Amazon EC2](http://docs.mongodb.org/ecosystem/platforms/amazon-ec2/).
* Already mentioned [blog post](http://blogs.aws.amazon.com/application-management/post/Tx1RB65XDMNVLUA/Deploying-MongoDB-with-OpsWorks) about mongo setup with OpsWorks and another [blog post about replicaset setup](http://netinlet.com/blog/2014/01/18/setting-up-a-mongodb-replicaset-with-aws-opsworks/).
* OpsWorks [cookbooks repository](https://github.com/aws/opsworks-cookbooks/tree/release-chef-11.4)
* [Stackoverflow: setting up mongodb via AWS opsworks](http://stackoverflow.com/questions/18637735/setting-up-mongodb-via-aws-opsworks)
* [opsworks-mongodb-example repository](https://github.com/ujuettner/opsworks-mongodb-example)
* [MongoDB on AWS (RDS-Style)](https://github.com/9apps/mongodb)
* [Fault Tolerant MongoDB on EC2](http://softwarebyjosh.com/2012/03/11/elastic-ips-and-mongodb-replica-sets.html)
* [NoSQL Database in the Cloud: MongoDB on AWS](https://media.amazonwebservices.com/AWS_NoSQL_MongoDB.pdf)
* [High Performance MongoDB Clusters with Amazon EBS Provisioned IOPS](http://www.slideshare.net/AmazonWebServices/ebs-mongo-dbwebinarfinal-nn)
* [Building a Mongo Replica Set with Chef and Vagrant article](https://medium.com/@polkaspots/building-a-mongo-replica-set-with-chef-and-vagrant-24dc2f59dc90)
* [Linode: Creating a MongoDB Replication Set on Ubuntu 12.04 (Precise)](https://www.linode.com/docs/databases/mongodb/creating-a-mongodb-replication-set-on-ubuntu-12-04-precise)
* [Hosting meteor with MongoDb on Webfaction](http://racingtadpole.com/blog/meteor-mongodb-webfaction/)

OpsWorks, Chef and Ruby:
* [OpsWorks attribute reference](http://docs.aws.amazon.com/opsworks/latest/userguide/attributes.html)
* [OpsWorks resource reference](https://docs.chef.io/resource.html)
* [Cookbook repository structure](http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-installingcustom-repo.html)
* [How to use shell scripts](http://docs.aws.amazon.com/opsworks/latest/userguide/workingcookbook-extend-scripts.html)
* [Stackoverflow: How I can change a file with chef?](http://stackoverflow.com/questions/14848110/how-i-can-change-a-file-with-chef)
* [Just Enough Ruby for Chef](https://docs.chef.io/ruby.html)
* [About the Recipe DSL](https://docs.chef.io/dsl_recipe.html)
* [Stackoverflow: what ruby features are used in chef recipes?](http://stackoverflow.com/questions/20569521/what-ruby-features-are-used-in-chef-recipes)
* [Stackoverflow: Ruby Code Blocks and Chef](http://stackoverflow.com/questions/19719968/ruby-code-blocks-and-chef/19726723#19726723)

Hosted MongoDB:
[compose.io (as I understand former MongoHQ)](https://www.compose.io/mongodb/), [MongoDirector](http://mongodirector.com/), [mongolab](https://mongolab.com/), [ObjectRocket](http://objectrocket.com/) and [dotCloud](http://docs.dotcloud.com/services/mongodb/).
