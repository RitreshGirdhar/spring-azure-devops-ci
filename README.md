# SpringBoot-Docker-Ansible


Purpose of this tutorial is to help you in deploying sample microservice/application on any machine via ansible.

Steps we will follow ::
1. Will build our sample application and create local docker image.
2. Will push docker image to central repository(public or private).
3. Will set up folder structure and create docker-compose on dev machine via ansible and we will start docker images on dev machine through docker-compose which we created via ansible.
----

###Step1: Let's build microservice docker image
```$xslt
cd microservice1
mvn install
```

Once build is done. Please check the docker images.
```$xslt
docker image list
```
you will find below images
```$xslt
sample-microservice-ansible/microservice1    latest              1c373c7b60e3        About a minute ago   124MB
```

Now we will have to push this image to docker-repository either public or your private docker repository.

###Step2: Let's to push image to repository
I am pushing my docker image to docker hub. (Do make sure you are successfully login) 
```$xslt
docker tag sample-microservice-ansible/microservice1:latest ritreshgirdhar/sample-microservice-ansible:latest
docker images
REPOSITORY                                   TAG                 IMAGE ID            CREATED             SIZE
sample-microservice-ansible/microservice1    latest              1c373c7b60e3        3 minutes ago       124MB
ritreshgirdhar/sample-microservice-ansible   latest              b0d54cc0d358        3 minutes ag        124MB
```
```$xslt
docker push ritreshgirdhar/sample-microservice-ansible:latest
```

You could see this image https://hub.docker.com/repository/docker/ritreshgirdhar/sample-microservice-ansible 
   
### Step3 - Set up folder structure on dev machine via Ansible.
* Run below command, ansible will create folders on the server defined in deployment-script.yaml hosts machine.

```$xslt
ansible-playbook -i hosts deployment-script.yaml
```

#### Lets understand the hosts file
```$xslt
dev_server ansible_host=18.234.67.26 ansible_user=ec2-user ansible_ssh_private_key_file=./Ritresh.pem
```

* dev_server is the alias name for vm 18.234.67.26. 
* we are using aws rhel-8 for this sample-application.
* ansible_user defined the user name
* ansible_ssh_private_key_file refer to the pem file path

#### Lets understand the default.yml

```$xslt
---
# Microservice Directory Structure
GROUP: "ec2-user" 
OWNER: "ec2-user"

APPLICATION_INSTALL_DIR: "/home/ec2-user/microservices"
PROJECT_NAME: "sample1"
APPLICATION_IMAGE: "ritreshgirdhar/sample-microservice-ansible"
APPLICATION_IMAGE_VERSION: "latest"

PROJECT_ROOT_SUBDIRS:
  - "{{ APPLICATION_INSTALL_DIR }}/{{ PROJECT_NAME }}/logs"
```

* APPLICATION_INSTALL_DIR refer to directory where we will set up our microservice/application
* PROJECT_NAME - refer to directory under APPLICATION_INSTALL_DIR
* APPLICATION_IMAGE - refer to the image which docker-compose will pull
* APPLICATION_IMAGE_VERSION - refer to docker-image version 



#### Output:
```$xslt

PLAY [Deploying Microservices Components on Dev Server] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************************************************************************
ok: [dev_server]

TASK [Checking directory.] ************************************************************************************************************************************************************************************************************
ok: [dev_server]

TASK [echo if directory already existed] **********************************************************************************************************************************************************************************************
ok: [dev_server] => {
    "msg": "the /home/ec2-user/microservices directory is already existed"
}

TASK [Create /home/ec2-user/microservices Directory to deploy/set up application] *****************************************************************************************************************************************************
skipping: [dev_server]

TASK [Create /home/ec2-user/microservices/sample-application directory for application] ************************************************************************************************************************************************
changed: [dev_server]

TASK [Create docker-compose.yml from docker-compose-template.j2 Template] *************************************************************************************************************************************************************
changed: [dev_server]

TASK [Start Application Containers using docker-compose command] **********************************************************************************************************************************************************************
changed: [dev_server]

PLAY RECAP ****************************************************************************************************************************************************************************************************************************
dev_server                 : ok=6    changed=3    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0   
```

#### Directory structure on Dev Machine
```$xslt
[ec2-user@ip-172-31-35-21 ~]$ tree
.
`-- microservices
    `-- sample-application
        `-- docker-compose.yml

2 directories, 1 file
```
Docker images on dev machine
```$xslt
[ec2-user@ip-172-31-35-21 ~]$ docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
ritreshgirdhar/sample-microservice-ansible   latest              511ec0a6b742        19 months ago       148MB
```

Running sample-microservices docker instances 
````$xslt
[ec2-user@ip-172-31-35-21 ~]$ docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
6a09c306b387        ritreshgirdhar/sample-microservice-ansible:latest   "java -Djava.securit…"   2 minutes ago       Up About a minute   0.0.0.0:8080->8080/tcp   sample-microservice-ansible
````


Test deployed application 
```$xslt
curl http://18.234.67.26:8080/v1
ok Hello world
```