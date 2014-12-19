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
We will use [MongoDB Chef cookbook](https://github.com/edelight/chef-mongodb) which allows to deploy different MongoDB setups - single instance,
replica set, sharding, replica sets with sharding.
It also allows to manage mongo users and setup [MongoDB Monitoring System (MMS)](https://mms.mongodb.com/) agent.

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

Here I have custom recipe for mongodb setup (descibed below) and for the nodeapp.
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
App is started with command like `PORT=3300 NODE_ENV=production path/to/deployment/current/app-test/bin/www'.

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

Here we have Berksfile which descibes dependencies:

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
