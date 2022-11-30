## Continuous cronjob container to upload, replicate and delete test data from the Rucio Data Lake

### Docker container
The Github action is set up in a way to push the Dockerfile container to the [projectescape/eosc-crons Dockerhub](https://hub.docker.com/repository/docker/projectescape/eosc-crons) every time there is a commit on the main branch. 
The container will contain the Rucio config file and all the VOMS specifications to authenticate through the [IAM ESCAPE](https://github.com/indigo-iam/escape-docs) instance. 

### Assigning role 


### Cronjobs
Cronjob.yaml file is applied to the K8s cluster to start a periodic job. The cronjob will run in the container and will execute all the scripts in this repo, namely production of noise and DB cleanup. 

### Secrets
These are the secrets needed to upload and replicate files via a rucio client accessible by the whole escape-cern-ops@cern.ch e-group. The client can be found on the IAM Dashboard under the name 'ESCAPE Rucio Service Account'
(ewp2c01). 

```
kubectl create secret generic escape-sso-client-key --from-file=client-sso.key
kubectl create secret generic escape-sso-client-crt --from-file=client-sso.crt
kubectl apply -f escape-sso-client-account.yaml
```

where the escape-sso-client-account.yaml secret looks like:

```
apiVersion: v1
data:
  escape-sso-client-password: base64_password
  escape-sso-client-username: base64_uname
metadata:
  name: escape-sso-client-account
  namespace: crons
kind: Secret
type: Opaque
```
