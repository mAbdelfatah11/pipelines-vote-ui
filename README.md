# Helm charts for Distributing Application

Think about Helm charts as a package manager for Kubernetes applications, like 
RPM or .deb files for Linux. Once a Helm chart is created and hosted on a repository, everyone can install, update, and delete your chart from a running Kubernetes 
installation

Helm has its own directory structure for storing necessary files. Helm allows you to 
create a basic template structure with everything you need (and even more). The 
following command creates a Helm chart structure for a new chart called foo:

```
$ helm create foo
```

But I think it’s better not to create a chart from a template, but to start from scratch. 
To do so, enter the following commands to create a basic file system structure for your 
new Helm chart:

```
$ mkdir helm-chart
$ mkdir helm-chart/templates
$ touch helm-chart/Chart.yaml
$ touch helm-chart/values.yaml
```

**OR:** create all the required structure automatically

```
$ helm create vote-ui .
```

Done. This creates the directory structure for your chart. You now have to put some 
basic data into Chart.yaml, using the following example as a guide to plugging in your 
own values:

```
apiVersion: v2
name: vote-ui
description: A helm chart for the basic vote app
type: application
version: 0.0.3          # new chart release version
appVersion: “1.0.0”     # deployed app version
sources:
- https://github.com/mAbdelfatah11/pipelines-vote-ui
maintainers:
- name: mahmoud abdelfattah
  email: mahmoud.abdelfatah@x.com

```

copy the following files from 
the kustomize directory into the helm-chart/templates folder:

```
$ cp kustomize-ext/base/*.yaml helm-chart/templates/
```

## Package, Install, Upgrade, and Roll Back Your Helm Chart

Now that you have a very simple Helm chart, you can package it:

```
$ helm package helm-chart
Successfully packaged chart and saved it to: vote-ui-0.0.1.tgz
```

The following command installs the Helm chart into a newly created OpenShift project 
called book-helm1:

```
$ oc new-project book-helm1
$ helm install vote-ui vote-ui-0.0.1.tgz
NAME: vote-ui
LAST DEPLOYED: Mon Oct 25 17:10:49 2021
NAMESPACE: book-helm1
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

If you now go to the OpenShift console, you should see the Helm release.

You can get the same overview via the command line. The first command that follows 
shows a list of all installed Helm charts in your namespace. The second command provides the update history of a given Helm chart:

```
$ helm list
$ helm history vote-ui

```

If you create a newer version of the chart, upgrade the release in your Kubernetes 
namespace by entering:

```
$ helm upgrade vote-ui vote-ui-0.0.5.tgz
Release “vote-ui” has been upgraded. Happy Helming!
NAME: vote-ui
LAST DEPLOYED: Tue Oct 26 19:15:07 2021
NAMESPACE: book-helm1
STATUS: deployed
REVISION: 7
TEST SUITE: None
```

The rollback argument helps you return to a specified revision of the installed chart. 
First, get the history of the installed chart by entering:

```
$ helm history vote-ui
1 Tue Oct 26 19:24:10 2021 superseded vote-ui-0.0.5 v1.0.0-test  Install complete
2 Tue Oct 26 19:24:46 2021 deployed vote-ui-0.0.4 v1.0.0-test    Upgrade complete
```

Then roll back to revision 1 by entering:

```
$ helm rollback vote-ui 1
Rollback was a success! Happy Helming!
```

If you are not satisfied with a given chart, you can easily uninstall it by running:

```
$ helm uninstall vote-ui
release “vote-ui” uninstalled
```

### New Content for the Chart

Another nice feature to use with Helm charts is a NOTES.txt file in the helm-chart/
templates folder. This file will be shown right after installation of the chart. It’s also 
available via the OpenShift UI in Developer → Helm → Release Notes). You can enter 
your release notes there as a Markdown-formatted text file like the following example. 
The nice thing is that you can insert named parameters defined in the values.yaml file:


```

# Release NOTES.txt of vote-ui helm chart
- Chart name: {{ .Chart.Name }}
- Chart description: {{ .Chart.Description }}
- Chart version: {{ .Chart.Version }}
- App version: {{ .Chart.AppVersion }}
## Version history
- 0.0.1 Initial release
- 0.0.2 release with some fixed bugs
- 0.0.3 release with image coming from quay.io and better parameter substitution
- 0.0.4 added NOTES.txt and a configmap
- 0.0.5 added a batch Job for post-install and post-upgrade

```

### Parameters

Yes, of course. Sometimes you need to replace standard settings, as we 
did with the OpenShift Templates or Kustomize. To unpack the use of parameters, let’s 
have a closer look into the values.yaml file:

```
deployment:
    image: quay.io/openshift-pipeline/vote-ui
    tag: latest
    replicas: 2
    includeHealthChecks: false

config:
    - name: VOTING_API_SERVICE_HOST
      value: pipelines-vote-api
    - name: VOTING_API_SERVICE_PORT
      value: "9000"

```

You are just defining your variables in the file. The following YAML configuration file 
illustrates how to insert the variables through curly braces:

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config
data:
  {{- range .Values.config }}
  {{ .name }}: "{{ .value }}"       # make sure that the value is quoted as a string
  {{- end }}

```

## Helm also defines some built-in objects

• .Release can be used after the Helm chart is installed in a Kubernetes environment. The parameter defines variables such as .Name and .Namespace.

• .Capabilities provides information about the Kubernetes cluster where the chart is installed.

• .Chart provides access to the content of the Chart.yaml file. Any data in that file is accessible.


## Debugging Templates

Typically, after you execute helm install, all the generated files are sent directly to 
Kubernetes. A couple of commands help you debug your templates first:

• ``helm lint`` command checks whether your chart follows best practices.

• ``helm install --dry-run --debug`` command renders your files without sending them to Kubernetes.