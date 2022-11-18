# Continuous cronjob container to upload, replicate and delete test data from the Rucio Data Lake

- build the image from the Docker file. The container will contain the Rucio config file and all the VOMS commands to authenticate through the IAM ESCAPE instance
- apply the cronjob.yaml file to the K8s cluster to start a periodic file. The cronjob will run in the container and will execute the scripts container in this repo. 
