# Run Tekton on Minikube

Tekton needs a kubernetes environment to run on, it is possible to run Tekton on minikube on your computer. You just need to setup a few layers:

## Install docker

https://docs.docker.com/install/

## Install kubectl

https://kubernetes.io/docs/tasks/tools/install-kubectl/

## Install minikube

Follow the instructions on this page:

https://kubernetes.io/docs/tasks/tools/install-minikube/#confirm-installation

Or here

https://minikube.sigs.k8s.io/docs/start/

### Upgrade minikube

If you haven't upgraded minikubd in a while have a look at the upgrading instructions

https://minikube.sigs.k8s.io/docs/start/macos/#upgrading-minikube

## Start minikube

Check you have minikube installed:

    $ minikube version

Start minikube

    $ minikube start

Check your kubectl config

    $ kubectl config current-context

You should see:

    minikube

If you are not currently in that context you can switch to it using:

    $ kubectl config use-context minikube

You can get a list of all your available contexts using:

    $ kubectl config get-contexts

## Tekton

### Install Tekton Pipelines

https://github.com/tektoncd/pipeline

As mentioned in the repo, Run:

    $ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml

You can check the installation:

    $ kubectl get pods --namespace tekton-pipelines --watch

### Install Tekton Dashboard

https://github.com/tektoncd/dashboard

As mentioned in the repo, Run:

    $ kubectl apply --filename https://github.com/tektoncd/dashboard/releases/download/v0.6.0/tekton-dashboard-release.yaml

You can check the installation:

    $ kubectl get pods --namespace tekton-pipelines

You can use port forward to view the dashboard:

    $ kubectl get pods -n tekton-pipelines

    Find the pod tekton-dashboard-xxxxxxx-xxx

    $ kubectl port-forward tekton-dashboard-xxxxxxx-xxx -n tekton-pipelines -p 9097:9097

Should see response:

    Forwarding from 127.0.0.1:9097 -> 9097
    Forwarding from [::1]:9097 -> 9097

Open in browser:

http://localhost:9097


## Test Tekton

I have a repo that I setup a couplde examples, I just test some of my test from there on minikube:

    $ git clone git@github.com:CariZa/tekton-examples.git

    $ cd tekton-examples

    $ kubectl apply -f ./example-1-github-read/

You should see:

```
pipeline.tekton.dev/pipeline-test created
pipelineresource.tekton.dev/github-repo created
task.tekton.dev/read-task created
```

You can check the dashboard, you should have a pipeline, a task and a resource setup.

You haven't run anything yet, so there won't be any pipelines runs or task runs.

To trigger the pipeline to run use this command:

```
cat <<EOF | kubectl create -f -
apiVersion: tekton.dev/v1alpha1
kind: PipelineRun
metadata:
  name: pipelinerun-test-$(date +%s)
spec:
  pipelineRef:
    name: pipeline-test
  resources:
    - name: github-repo
      resourceRef:
        name: github-repo
EOF
```

Note:

PipelineRuns are uniquely named. TaskRuns as well. This snippet above uses a timestamps as prt of the naming convention.

It runs "kubectl create -f ..." so this creates the pipelinerun from the yaml provided in the snippet.

View the pipelinerun:

    $ kubectl get pipelinerun

Vew the taskrun created by the pipelinerun:

    $ kubectl get taskrun

View the pod created by the taskrun:

    $ kubectl get pod 

Then view logs using the pod name:

    $ kubectl logs pipelinerun-test-xxxxxx-pipeline-read-task-xxx-pod-xxxx --all-containers

Replace "pipelinerun-test-xxxxxx-pipeline-read-task-xxx-pod-xxxx" with your pod's name.

The logs should end with:

```
# ulmaceae

:shrug:
```

This was from the repo provided in the PipelineResource: "https://github.com/CariZa/ulmaceae"

And the Task step was to cat the README file:

```
  steps:
    - name: catreadme
      image: ubuntu
      command:
        - cat
      args:
        - "github-repo/README.md"
```

You can also take a look at the dashboard again, and click on the pipeline, the pipelinerun, then the task lgos will show and you will see in the dashboard logs:

```
# ulmaceae

:shrug:

Step completed
```



