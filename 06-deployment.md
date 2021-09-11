
# Deployment to GCP

## Create a service account for deployment

- Go to [GCP Console](https://console.cloud.google.com/home/dashboard), search for  "Service accounts" from the top search box. or go to: "IAM & Admins" > "Service accounts" from the top-left menu and create a new service account called "deployment". For "Service account permissions" select "Cloud Storage" > "Storage Bucket Reader". Then click done.
- This will create a service account
- On the right "Actions" column click the vertical ... and select "Create key". A prompt for Create private key for "bucket-reader" will appear select "JSON" and click create. This will download a Private key json file to your computer. Copy this json file into the **secrets** folder.
- Rename the json key file to `deployment.json`
- Follow the same process Create another service account called `gcp-service`

- Go to "IAM & Admins" and give the following roles:
- For `deployment`:
    - Compute Admin
    - Compute OS Login
    - Container Registry Service Agent
    - Kubernetes Engine Admin
    - Service Account User
    - Storage Admin
- For `gcp-service`:
    - Storage Object Viewer

## API's to enable in GCP for project
* Compute Engine API
* Service Usage API
* Cloud Resource Manager API
* Google Container Registry API

## Start Docker Container (Ansible, Docker, Kubernetes)
-  `cd deployment`
- Run `sh docker-shell.sh` or `docker-shell.bat` for windows


## SSH Setup
### Configuring OS Login for service account
```
gcloud compute project-info add-metadata --project ai5-project --metadata enable-oslogin=TRUE
```

### Create SSH key for service account
```
cd /secrets
ssh-keygen -f ssh-key-deployment
cd /app
```

### Providing public SSH keys to instances
```
gcloud compute os-login ssh-keys add --key-file=/secrets/ssh-key-deployment.pub
```
From the output of the above command keep note of the username. Here is a snippet of the output 
```
- accountId: ai5-project
    gid: '4241727572'
    homeDirectory: /home/sa_105148290446446408899
    name: users/deployment@ai5-project.iam.gserviceaccount.com/projects/ai5-project
    operatingSystemType: LINUX
    primary: true
    uid: '4241727572'
    username: sa_105148290446446408899
```
The username is `sa_105148290446446408899`


## Deployment Setup
* Add ansible user details in inventory.yml file
* GCP project details in inventory.yml file
* GCP Compute instance details in inventory.yml file

## Deployment
#### Create Compute Instance (VM) Server in GCP
```
ansible-playbook deploy-create-instance.yml -i inventory.yml --extra-vars cluster_state=present
```
Once the command runs successfully get the IP address of the compute instance from GCP Console and update the appserver>hosts in inventory.yml file

#### Provision Dev Server in GCP
```
ansible-playbook deploy-provision-instance.yml -i inventory.yml
```

#### Build and Push Docker Containers to GCR
```
ansible-playbook deploy-docker-images.yml -i inventory.yml
```

#### Setup Docker Containers in the  Compute Instance
```
ansible-playbook deploy-setup-containers.yml -i inventory.yml
```


You can SSH into the server from the GCP console and see status of containers
```
sudo docker container ls
sudo docker container logs api-service -f
```



#### Configure Nginx file for Web Server
* Create nginx.conf file for defaults routes in web server

#### Setup Webserver on the Compute Instance
```
ansible-playbook deploy-setup-webserver.yml -i inventory.yml
```
Once the command runs go to `http://<External IP>/` 

#### Delete the Compute Instance / IP / Persistent disk
```
ansible-playbook deploy-create-instance.yml -i inventory.yml --extra-vars cluster_state=absent
```
 