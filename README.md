# Openshift quickstart: Django

The steps in this document assume that you have access to an OpenShift deployment that you can deploy applications on.

## Deploying to OpenShift

To follow the next steps, you need to be logged in to an OpenShift cluster and have an OpenShift project where you can work on.

The template `django.json` contains just a minimal set of components to get your Django application into OpenShift.

Copy/Download the okd warrior template: `django.json` from the root folder of the project

A) Deploy using Openshift Web Console:

Login to openshift web-console.

Create a new namespace in the Openshift Web Console
Browse or Paste the okd template in the created namespace

Adjust the parameter values to suit your configuration. Most times you can just accept the default values, however you will probably want to set the `GIT_REPOSITORY` parameter to point to your fork and the `DATABASE_*` parameters to match your database configuration.


In the web console, the overview tab shows you a service, by default called "django-example", that encapsulates all pods running your Django application. You can access your application by browsing to the service's IP address and port.  You can determine these by running

    oc get svc


B) Deploy using command line of okd cluster:

oc login to openshift command line.

create a namespace using 
`oc new-project <your-app-name>`

Deploy the template using following command in the cli
`oc new-app -f <template_name.json`


The aplication is configured to access in node port:30199 , so it can accessed with the <route>:<nodeport>
Ex: http://django-example-hw-dj.apps.openshift.com:30199

## Logs

By default your Django application is served with gunicorn and configured to output its access log to stderr.
You can look at the combined stdout and stderr of a given pod with this command:

    oc get pods         # list all pods in your project
    oc logs <pod-name>

This can be useful to observe the correct functioning of your application.


## Special environment variables

### APP_CONFIG

You can fine tune the gunicorn configuration through the environment variable `APP_CONFIG` that, when set, should point to a config file as documented [here](http://docs.gunicorn.org/en/latest/settings.html).

### DJANGO_SECRET_KEY

When using one of the templates provided in this repository, this environment variable has its value automatically generated. For security purposes, make sure to set this to a random string as documented [here](https://docs.djangoproject.com/en/1.8/ref/settings/#std:setting-SECRET_KEY).


## One-off command execution

At times you might want to manually execute some command in the context of a running application in OpenShift.
You can drop into a Python shell for debugging, create a new user for the Django Admin interface, or perform any other task.

You can do all that by using regular CLI commands from OpenShift.
To make it a little more convenient, you can use the script `openshift/scripts/run-in-container.sh` that wraps some calls to `oc`.
In the future, the `oc` CLI tool might incorporate changes
that make this script obsolete.

Here is how you would run a command in a pod specified by label:

1. Inspect the output of the command below to find the name of a pod that matches a given label:

        oc get pods -l <your-label-selector>

2. Open a shell in the pod of your choice. Because of how the images produced
  with CentOS and RHEL work currently, we need to wrap commands with `bash` to
  enable any Software Collections that may be used (done automatically inside
  every bash shell).

        oc exec -p <pod-name> -it -- bash

3. Finally, execute any command that you need and exit the shell.
