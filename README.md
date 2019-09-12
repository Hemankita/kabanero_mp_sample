# Kabanero Sample - MicroProfile

## Appsody sample on Codewind

This project is created using Codewind in the Eclipse IDE. For details, refer [Getting started: Codewind for Eclipse](https://www.eclipse.org/codewind/mdteclipsegettingstarted.html).

You can access the codewind explorer as follows.

```
Window > Show View > Other...
```

<p align="center">
    <img src="images/show_view.png">
</p>

It opens a dialog box as follows.

<p align="center">
    <img src="images/codewind.png">
</p>

Choose `Codewind Explorer` and it will show up in Eclpise.

<p align="center">
    <img src="images/codewind_explorer.png">
</p>

To create a new project, Right click on the `Projects` > Choose `Create New Project`. You will see something like below.

<p align="center">
    <img src="images/createproject.png">
</p>

Name the project, choose the template you want and click Finish. For this sample, we named it `kabanero_mp_sample` and choose `Appsody Eclipse MicroProfile Template`. Once, the project is successfully created, you will see something like below.

<p align="center">
    <img src="images/explorer_new_project.png">
</p>

By refreshing the contents of `Project Explorer`, the project view is also available as follows.

<p align="center">
    <img src="images/project_view.png">
</p>

Various options are available as follows.

<p align="center">
    <img src="images/explorer_new_project.png">
</p>

You can open the application in the IDE by using `Open Application` option. Also, the project reloads automatically when you make changes. You can view them by refreshing the browser. You need not re-start the server every time.

## Kabanero Foundation Setup on Minishift

Using Minishift as our environment.

- Start the minishift as follows.

```
minishift config set cpus 6

minishift config set memory 16GB

minishift start --vm-driver=hyperkit
```

You will see something like below once started.

```
Login to server ...
Creating initial project "myproject" ...
Server Information ...
OpenShift server started.

The server is accessible via web console at:
    https://192.168.64.29:8443/console

You are logged in as:
    User:     developer
    Password: <any value>

To login as administrator:
    oc login -u system:admin
```

- Set it to use `oc` cli.

```
eval $(minishift oc-env)
```

To set up `Kabanero Foundation`, follow the below steps.

- Clone the github repo as follows.

```
git clone https://github.com/kabanero-io/kabanero-foundation.git
```

- Go to the `scripts`.

```
cd kabanero-foundation/scripts
```

- To install the foundation setup, you need to login as `admin`

```
oc login -u system:admin
```

If successfully logged in, you will see something like below.

```
$ oc login -u system:admin
Logged into "https://192.168.64.29:8443" as "system:admin" using existing credentials.

You have access to the following projects and can switch between them with 'oc project <projectname>':

    default
    kube-dns
    kube-proxy
    kube-public
    kube-system
  * myproject
    openshift
    openshift-apiserver
    openshift-controller-manager
    openshift-core-operators
    openshift-infra
    openshift-node
    openshift-service-cert-signer
    openshift-web-console

Using project "myproject".
```
- Run the below command to install kabanero foundation setup.

```
openshift_master_default_subdomain=<minishift_ip>.nip.io ./install-kabanero-foundation.sh
```

In the above case, it will be `openshift_master_default_subdomain=192.168.64.29.nip.io ./install-kabanero-foundation.sh`

- Once it is successfully installed, you can run `oc get projects` to see what it installed.

```
$ oc get projects
NAME                            DISPLAY NAME   STATUS
istio-system                                   Active
kabanero                                       Active
knative-eventing                               Active
knative-serving                                Active
knative-sources                                Active
```

The above are all installed as part of Kabanero foundation setup.

## Tekton

You can access the tekton pipeline at `tekton-dashboard-kabanero.<minishift_ip>.nip.io`.

In the above case it will be `tekton-dashboard-kabanero.192.168.64.29.nip.io`

<p align="center">
    <img src="images/tekton_home.png">
</p>

- To generate the deploy file, run the below command.

```
appsody deploy --generate-only
```

- Now, setup your git locally with the content of the application

```
git init
git add .
git commit -m "initial commit"
```

- Create a github repository and push the code to the remote repository

```
git remote add origin $GIT_URL
git push -u origin master
```

### Manual Pipeline Setup

- Get the openshift registry info as follows.

```
minishift openshift registry
```

You will see something like below.

```
$ minishift openshift registry
172.30.1.1:5000
```

You will need this later.

- To generate a manual pipeline, run the below command.

```
DOCKER_IMAGE=<registry_host:port>/kabanero/java-microprofile APP_REPO=<github_repo_url> ./appsody-tekton-example-manual-run.sh
```

For instance, for the above application, it will be as follows.

```
DOCKER_IMAGE=172.30.1.1:5000/kabanero/java-microprofile APP_REPO=https://github.com/Hemankita/kabanero_mp_sample ./appsody-tekton-example-manual-run.sh
```

Once it is successfully completely, at the end you will see somethinng like below.

```
+ oc -n kabanero apply -f -
pipelineresource.tekton.dev/docker-image created
pipelineresource.tekton.dev/git-source created
+ oc -n kabanero delete pipelinerun manual-pipeline-run
pipelinerun.tekton.dev "manual-pipeline-run" deleted
+ cat
+ oc -n kabanero apply -f -
pipelinerun.tekton.dev/manual-pipeline-run created
```

- Now go to the tekton dashboard at `tekton-dashboard-kabanero.<minishift_ip>.nip.io`.

<p align="center">
    <img src="images/tekton_home.png">
</p>

- Click on the project pipeline `java-microprofile-build-deploy-pipeline`.

<p align="center">
    <img src="images/tekton_manual_pipeline.png">
</p>

- Once the tasks are completed, you can access the application.
