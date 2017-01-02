# ansible-role-jenkins

## Overview
This Ansible role provides an automated way to install Jenkins on Linux and
manage its configuration so it is possible to easily rebuild it automatically.
This role will install the Jenkins package, then use the APIs to install the
required plugins. It will also create the jenkins user accounts. Finally it
creates a special job, the jenkins-dsl job which will create or update all other
jenkins jobs.

At this stage this role supports the basic installation. More work will be
required to manage things such as the configuration of the slaves and additional
plugins. This role has been tested with Jenkins 2.32.

## Jenkins DSL
The jenkins dsl plugin allows you to write groovy code which will generate Jobs
in jenkins programatically. This ansible role creates a special job which will
create the other jobs from the groovy code using the plugin when it is executed:
https://wiki.jenkins-ci.org/display/JENKINS/Job+DSL+Plugin
If you want this role to prepare this Jenkins-DSL job then you must keep all
your groovy files in the same repository as your ansible files but the files
can be in a directory outside of this role so they can be managed separately.
The files should all be stored in the same directory and have a ".groovy.j2"
extension. Then you must provide the path to this directory using the following
parameter: jenkins_dsl_source_files. You can use a path relative to this role
using a value such as '{{ role_path }}/../../my-role/templates/my-dsl-files'
This ansible role will first use the ansible template module to copy these files
to the disk on the jenkins server so you can use jinja templates to parameterize
these groovy files using ansible variables. You can see how the dummy parameter
named "jenkins_dsl_dummy_variable" is used in the template for the dummy job.
Once the DSL files have been created from the templates on the jenkins server
the Jenkins-DSL job will point to this directory on the local disk. With this
approach you can have all your ansible and jenkins configured stored in a single
git repository. Also you can use ansible parameters in the groovy code which
creates your jenkins jobs. Hence everything can be centralized in a single place
and you can avoid lot of duplicate variables between your ansible code and your
jenkins job configuration.

## How it works
First this role add support for a yum or apt repository which provides packages
for the LTS version of Jenkins. Then it installs the package. It creates XML
configurations for each jenkins user account listed in the parameters. One of
these account must be a special account to be used by Ansible so it can access
the Jenkins APIs via HTTP as this is required to install plugins automatically
and to create the initial DSL job.

## How to use it
   * Install this role using ansible-galaxy
     https://galaxy.ansible.com/fdupoux/jenkins/
   * Make sure all parameters listed in the defaults/main.yml file of this role
     are defined properly. You can override definitions for these parameters
     using the ansible group_vars or host_vars if necessary so it matches your
     requirements.
   * If you want to use the jenkins-dsl plugin to manage the configuration of
     your jenkins jobs then you must store the groovy code in a git repository
     and you must pass the details to this role.
   * Apply this role in one of your playbooks

## Testing
This role has been tested on CentOS 7 and it effort has been made to make it
work on Debian based distributions. But this role has not been tested on
Debian or Ubuntu at this stage.
