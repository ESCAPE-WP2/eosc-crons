## Continuous cronjob container to upload, replicate and delete test data from the Rucio Data Lake

### Docker container
The Github action (specified in the ```.github/workflows```)is set up in a way to push the Dockerfile container to the [projectescape/eosc-crons Dockerhub](https://hub.docker.com/repository/docker/projectescape/eosc-crons) every time there is a commit on the ```main``` branch. 
The container will contain the Rucio config file (connecting to the user associated with the ESCAPE service account, accessible by all the ESCAPE admins) and all the VOMS specifications to authenticate through the [IAM ESCAPE](https://github.com/indigo-iam/escape-docs) instance. 

### Assigning role 
The file ```assign-gatekeeper.yaml``` is needed in order to provide an environemnt in which the conjobs can run. 

### Cronjobs
Cronjob.yaml file is applied to the K8s cluster to start a periodic job. The cronjob will run in the container and will execute all the scripts in this repo, namely production of noise and DB cleanup. 

### Secrets
These are the secrets needed to upload and replicate files via a rucio client accessible by the whole escape-cern-ops@cern.ch e-group. The client can be found on the IAM Dashboard under the name 'ESCAPE Rucio Service Account'
(ewp2c01). The certificate and the key can be found in the CERN Box directory shared across all of the ESCAPE CERN gitops admins (```CERN-EOSC/ESCAPE-cluster/```). 

```
kubectl create secret generic escape-sso-client-key --from-file=client-sso.key
kubectl create secret generic escape-sso-client-crt --from-file=client-sso.crt
kubectl apply -f escape-sso-client-account.yaml
```

where the ```escape-sso-client-account.yaml``` secret looks like:

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

### FTS certificates delegation on aiadm

In order for transfers to occurr, certain daemons pods need to contain the proxy file (which expires every 12 hours) generated from an x509 certificate. This is because the daemon will delegate the FTS transfers to the entity certified with the certificate. Once more, the certificate used is the ESCAPE service account associated to the identity accessible by anyone in the CERN ESCAPE group (the certificate is the same one referenced above that can be found in the cernbox folder) that can be seen in the ```escape-sso-client-account``` secret. 
A cronjob needs to be started on aiadm (as it is more reliable for all the rest of CERN CA certificates), following the [acron](https://acrondocs.web.cern.ch/) documentation. The job uses the script ```update_proxies.sh``` and can be sarted with: 

```
$ acron jobs create -s '0 */8 * * *' -t aiadm.cern.ch -c '/bin/bash <full_afs/cern.ch/user/_path_to_update_proxies.sh>' -d 'Update robot client proxies'
```

This will generate the voms-proxy every 8 hours and inject it in to the pod of each daemon and create the rucio secrtes, one in the ```rucio``` namespace and the other in the ```crons``` namespace.

### Monitoring dashboards
The upload, transfer and deletion between RSEs (apart from the ones in the list ```/cric-info-tools/disabled_rses.txt```, which are may at the moment due to the ESCAPE project coming to an end) can be seen in the [Grafana dashboard](https://monit-grafana.cern.ch/d/4rmQfGYMz/rucio-events?orgId=51) displaying Rucio events. 
