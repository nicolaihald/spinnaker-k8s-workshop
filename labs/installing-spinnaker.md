# Installing Spinnnaker

## Install Helm
[OPTIONAL] make sure there’s a k8s cluster?
 
Download and install helm binary
From https://github.com/kubernetes/helm/blob/master/docs/quickstart.md
```shell
$ wget https://storage.googleapis.com/kubernetes-helm/helm-v2.4.2-linux-amd64.tar.gz
$ sudo tar -C /usr/local -xzf helm-v2.4.2-linux-amd64.tar.gz
```

Add Helm to PATH
```shell
$ export PATH=$PATH:/usr/local/linux-amd64
```
 
# Initialize local CLI
NOTE:  Does the directory matter for anything?
```shell
$ helm init
```

## Configure Spinnaker

### Spinnaker Overview

Spinnkaer has a lot of pieces and parts.  Below is a table listing everything.  You don't need to know *any* of this for the workshop, but it's here for completeness.

| Servivces | Port | Description |
| --- | --- | --- |
| Deck	| 9000 | Deck is a static AngularJS-based UI. |
| Clouddriver	| 7002 | Cloud Driver integrates with each cloud provider (AWS, GCP, Azure, etc.). It is responsible for all cloud provider-specific read and write operations. |
| Echo	| 8089 | Echo provides Spinnaker’s notification support, including integrations with Slack, Hipchat, SMS (via Twilio) and Email. |
| Front50	| 8080 | Front50 stores all application, pipeline and notification metadata. |
| Gate	| 8084 | Gate exposes APIs for all external consumers of Spinnaker (including deck). It is the front door to Spinnaker. |
| Igor	| 8088 | Igor facilitates the use of Jenkins in Spinnaker pipelines (a pipeline can be triggered by a Jenkins job or invoke a Jenkins job) |
| Orca	| 8083 | Orca handles pipeline and task orchestration (ie. starting a cloud driver operation and waiting until it completes). |
| Rosco	| 8087 | Rosco is a packer-based bakery. We believe in immutable infrastructure and rosco provides a means to take a Debian or Red Hat package and turn it into an Amazon Machine Image. Don’t worry, it also supports Google Compute Engine and Azure images. |
| Fiat	| 7003 | Fiat is the authorization server for the Spinnaker system.  It exposes a RESTful interface for querying the access permissions for a particular user. |


TODO: Make this a sed operation
```shell
$ nano values.yaml 
```
 
```shell
# Disable minio the default
minio:
  enabled: false
 
# Enable gcs
gcs:
  enabled: true
  project: <my-project-name>
  jsonKey: '<SERVICE_ACCOUNT_JSON>'
 
# Name has to be unique in GCS.
storageBucket: <my-project-name>-spinnaker 
 
# Configure your Docker registries here
accounts:
- name: gcr
  address: https://gcr.io
  username: _json_key
  password: '<SERVICE_ACCOUNT_JSON>'
  email: 1234@5678.com
```
Replace <my-project-name> with your project name and copy accounts.json text into values.yaml.

```shell
$ SERVICE_ACCOUNT_JSON=$(cat account.json) && echo SERVICE_ACCOUNT_JSON
```


## Deploy Spinnaker Chart

Everything in this helm chart will be labeled cd-spinnaker, so you can search for things like: 
```shell
$ kubectl get deployment -l app=cd-spinnaker
```

To delete everything
```shell
$ helm delete cd --purge
```
 
get the latest list of charts
```shell
$ helm repo update
```
 
install spinnaker
```shell
$ helm install stable/spinnaker --name cd -f values.yaml --timeout 1500 
```
 
NOTE: This is going to take a while. 

### Misc / Monitor Progresss / Troubleshoot / Debug Installation
Let user know things are happening.  In another tab.  Errors will happen this is to be expected while the pods sync up.
```shell
$ kubectl get pods -w
```

NOTE: If you want to make a quick change.  Helm can do a blue green deployment via upgrade.
```shell
$ helm upgrade cd charts/stable/spinnaker -f updated-values.yaml
```

Debugging can be done with
```shell
$ kubectl logs <pod name>
```

Common problems
Front50 will fail if the GCS bucket name is not unique.


## Access the spinnaker UI
```shell
$ DECK_POD=$(kubectl get pods -l "component=deck,app=cd-spinnaker" -o jsonpath="{.items[0].metadata.name}")
$ kubectl port-forward $DECK_POD 9000 >>/dev/null &
```
 
Visit the Spinnaker UI by opening your browser to: http://127.0.0.1:9000