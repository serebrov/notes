Elastic Beanstalk - deploy from different machines / by different users (or how to get rid of absolute paths in configs)
============================================

By default Elastic Beanstalk console tool (eb) adds config files to .gitignore.
If there are manual changes to EB configs it can be complex to manually sync these changes
between different machines / different users.
Of cause it is possible to add config files to git repository but there are also several
parameters in the main config which are absolute paths to local files.
This way it makes configs not useful for other users (except for the case when different
users have exactly the same files layout).
Here is how this problem can be fixed:

1. Add eb tools under git control
1. Add eb configs under git control
1. Patch eb tools to accept relative file paths
1. Change paths in configs to relative

Below are details for each step.

Add eb tools under git control
--------------------------------------------
This is necessary because we want all users to have the same tools version.
Also we are going to make some changes to eb sources to make it understand relative paths.

Just download [eb tools](http://aws.amazon.com/code/6752709412171743) and extract somewhere inside your project.
Or copy existing version into the project.

Add a wrapper script to run the original eb tool:

    #!/bin/sh

    # Run the eb tool from the root project directory as
    # console/eb {parameters}
    # Use linux python2.7 version

    SCRIPT_PATH=`dirname $0`

    export PATH=$PATH:$SCRIPT_PATH/AWS-ElasticBeanstalk-CLI-2.5.1/eb/linux/python2.7
    eb "$@"

Script should be on the same lever as the extracted directory.
For example I have:

    project_root/
    | .ebextensions/
    | .elasticbeanstalk/
    | .git/
    | application/
    | console/
    | | AWS-ElasticBeanstalk-CLI-2.5.1/
    | | eb*
    | | ...
    | ...
    | .gitignore

So I launch the 'eb' as 'console/eb parameters' from the project root directory.

Add eb configs under git control
--------------------------------------------
Add the EB configs under git control:

    $ git add -f .elasticbeanstalk

Patch eb tools to accept relative file paths
--------------------------------------------
For the 2.5.1 version I had to make two changes in the config file parser class.

The file path is .../AWS-ElasticBeanstalk-CLI-2.5.1/eb/linux/python2.7/lib/utility/configfile_parser.py.

And the changes are:

    [Lines 41 - 47]
    def read(self, pathfilename):
        #seb: expand path to allow using homedir and relative paths
        pathfilename = os.path.realpath(os.path.expanduser(pathfilename))
        print 'Load sectioned config file: ' + pathfilename

        with codecs.open(pathfilename, 'r', encoding=ServiceDefault.CHAR_CODEC) as input_file:
            _RawConfigParser.readfp(self, input_file)

    [Lines 74 - 80]
    def read(self, pathfilename):
        #seb: expand path to allow using homedir and relative paths
        pathfilename = os.path.realpath(os.path.expanduser(pathfilename))
        print 'Load config file: ' + pathfilename

        with codecs.open(pathfilename, 'r', encoding=ServiceDefault.CHAR_CODEC) as input_file:
            config_pairs = input_file.read()

Now you can use relative to project root paths in your configs.

Change paths in configs to relative
--------------------------------------------
I had three absolute paths in my ./elasticbeanstalk/config:

    [global]
    ..
    AwsCredentialFile=/home/username/.elasticbeanstalk/aws_credential_file
    ..

    [branches]
    staging=staging
    production=production

    [branch:staging]
    OptionSettingFile=/path/to/project/.elasticbeanstalk/optionsettings.staging
    ...

    [branch:production]
    OptionSettingFile=/path/to/project/.elasticbeanstalk/optionsettings.production
    ..

Path to aws credential file can be easily fixed by just removing it - eb tools will use
config from your home dir by default (~/.elacticbeanstalk/aws_credential_file).

Other two paths (for branch configs) can now be changed to relative:

    [global]
    # Default credential file path (recognized by both eb tool and git aws.push tool)
    # AwsCredentialFile=~/.elasticbeanstalk/aws_credential_file
    ...

    [branches]
    staging=staging
    production=production

    [branch:staging]
    OptionSettingFile=./.elasticbeanstalk/optionsettings.staging
    ...

    [branch:production]
    OptionSettingFile=./.elasticbeanstalk/optionsettings.production
    ...

Use it
--------------------------------------------
Make sure you are using the 'eb' wrapper script from the root project directory.
For example, for 'eb status':

    $ cd project/root/dir
    $ ./path/to/eb status
    Load sectioned config file: project/root/dir/.elasticbeanstalk/config
    Load config file: /home/user/.elasticbeanstalk/aws_credential_file
    URL	: staging-jp48gay9nx.elasticbeanstalk.com
    Status	: Ready
    Health	: Green

