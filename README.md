# Build-docker-image-and-push-to-hub-using-automation

Here we will build a docker image with different versions as we need and push it to the docker hub using Ansible. This task is performed in the local system, so we use "localhost" as the host. We will also build a docker container from this pushed image to the docker hub and will run a template in this docker container.

## Prerequisites

- Need one Amazon Linux instance.
- A domain name to point to the public IP the of the container consists server
- A Docker hub account to push and pull image


## Install ansible on an instance

First install the python on the machine.
```bash
sudo amazon-linux-extras install python3.8 -y
```
then install the ansible using pip3.8

```bash
pip3.8 install ansible
```
to check the ansible version

```bash
ansible --version
```
![image](https://user-images.githubusercontent.com/100775027/166636421-e8f09bde-b510-4961-b779-f1d836fa56ad.png)

## Configure Hosts file (Inventory file)

The orginal Hosts file for Ansible is "/etc/ansible/hosts", but am not using this file as my Hosts file, instead am creating new file with name "inventory". 

```bash
# vim inventory

[linux]
 <your-server-private-ip> ansible_user="ec2-user" ansible_port=22 ansible_ssh_private_key_file="<your-private-key-file-name>"
 

 ```
 The headings in brackets are group names [linux], which are used in classifying hosts and deciding what hosts you are controlling at what times and for what purpose. 
 >  Make sure that you add the "your-server-private-ip" and "your-private-key-file-name" of worker instances on the manager server. Change the "your-private-key-file-name" permission to "400" !
  
 For checking SSH connection to worker server/instance:
  
  ```bash
  ansible -i inventory linux -m ping
  ```
  ![image](https://user-images.githubusercontent.com/100775027/166636594-f379bfd0-9b8a-4d26-b54f-fdecd320ba69.png)


## Sample template

Here I create a directory for our project and we perform the below steps in this directory.

```bash
mkdir project
```

 I'm using HTML template for EC2 instance, you can use this link for getting [sample HTML templates](https://www.tooplate.com/). 
To get sample HTML template


```bash
wget https://www.tooplate.com/zip-templates/2121_wave_cafe.zip ; unzip 2121_wave_cafe.zip ; mv 2121_wave_cafe website; rm -rf 2121_wave_cafe.zip
```
## Create Dockerfile

First we need to create a Dockerfile for building the docker image :
```bash
vim Dockerfile
```

Add:

```bash
FROM httpd:latest
COPY ./website/ /usr/local/apache2/htdocs/
CMD ["httpd-foreground"]
```

## Create playbook

We need to write configuration file for ansible for performing docker image creation and image push to docker hub:

```bash
vim docker
```

then add,

```bash
---

- name: "Docker image pull using ansible"
  hosts: localhost
  become: yes
  vars:
    packages:
      - docker
      - python-pip
    users:
      - "ec2-user"
    image_name: myproject-docker-image
    version: v
    docker_user: <your-DockerHub-username>
    docker_password: <your-DokcerHub-password>
  tasks:
    
    - name: "installin pip and docker"
      yum:
        name: "{{packages}}"
        state: present

    - name: "installing docker-py"
      pip:
        name: docker-py
        state: present

    - name: " add the {{users}} to docker grp"
      user:
        name: "{{item}}"
        groups: docker
        append: yes
      with_items:
        - "{{users}}"

    - name: "Restarting and enabling docker service"
      service:
        name: docker
        state: restarted
        enabled: true

    - name: "login to DockerHub"
      docker_login:
        username: "{{docker_user}}"
        password: "{{docker_password}}"
        email: "{{docker_email}}"

    - name: "Docker image build and push"
      docker_image:
        build:
          path: /home/ec2-user/project/ 
        name: "{{docker_user}}/{{image_name}}"
        tag: "{{item}}"
        source: build
        push: yes
      with_items:
        - "{{version}}"
        - latest

    - name: "launch the conatiner from the image"
      docker_container:
        name: mycontainer
        state: started
        image: "{{docker_user}}/{{image_name}}:latest"
        pull: true
        ports:
          - "80:80"
```
docker-py: python library for the Docker Remote API. It does everything the docker command does, but from within Python – run containers, manage them, pull/push images, etc.


Here we used variables declared by "vars", The variables are:

- docker_user & docker_password: Need to substitute your docker hub credentials
- version: Need to specify your desired version number or tag
- image_name: Need to put user docker hub username/(any variable)
- packages: Specified the necessary packgaes needed for our project
- User: specify your system user or desired user

 
 **Check syntax**
 
 Once the playbook is created, we need to check the syntax and execute the playbook:
 
 ```bash
 ansible-playbook -i inventory docker-container --syntax-check
 ```
 ![image](https://user-images.githubusercontent.com/100775027/166636654-f9480cdc-1ee3-4ee3-8383-1f758f539cf6.png)

**Execute the playbook**
 
 ```bash
 ansible-playbook -i inventory docker-container
 ```
 Now our setup is complete and you can point your server public IP to the domain name.


 
## Conclusion
This is how an docker image is build and pushed to docker hub using ansible Please contact me when you encounter any difficulty error while using this terrform code. Thank you and have a great day!

 ### ⚙️ Connect with Me
<p align="center">
<a href="https://www.linkedin.com/in/radin-lawrence-8b3270102/"><img src="https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white"/></a>
<a href="mailto:radin.lawrence@gmail.com"><img src="https://img.shields.io/badge/Gmail-D14836?style=for-the-badge&logo=gmail&logoColor=white"/></a>
