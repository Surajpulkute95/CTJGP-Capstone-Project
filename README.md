# Capstone-Project

## Terraform Task:

### Problem Statement: 
Launch a Ubuntu EC2 instance (t2.micro) to be used as your terraform workstation. From that WS, using Terraform, launch an EC2 
instance (instance type: t2.micro, OS: Red Hat Linux) to be used as an ansible 
workstation for the ansible task. Please make sure that you create a key (using ssh-keygen) and use it while launching the EC2 so that we can SSH into the 
ansible WS once it is created.

#### Hints:
In your terraform WS, install terraform using following commands
```
sudo apt update
```
```
sudo apt install wget unzip -y
```
```
wget https://releases.hashicorp.com/terraform/1.0.11/terraform_1.0.11_linux_amd64.zip
```
```
unzip terraform_1.0.11_linux_amd64.zip
```
```
sudo mv terraform /usr/local/bin
```

install the aws cli
```
sudo apt-get install python3-pip -y
```
```
sudo pip3 install awscli
```

Use aws configure and give your credentials
```
aws configure
```

Create a directory and inside that directory create your terraform files to create an instance and ssh into it.

Refer to the below

* [Create Key-pair and Launch EC2 Instance](https://github.com/Mehar-Nafis/TerraformLabs/blob/main/AWS-Key%20Pair%20Generation.md)

Create key pair using manually, if not using through Terraform
```
ssh-keygen -f mykey
```



## Ansible Tasks:

### Problem Statement: 

Once you have created a new instance using Terraform (as part of Terraform task), ssh into that instance and install Ansible in it. After that, you have to install httpd webserver in the managed node. You do not have separate managed nodes. So use your ansible workstation itself as the managed node by adding the below line in your host inventory file:
localhost ansible_connection = local

#### Hint
Install ansible using the following commands
```
sudo yum check-update
```
```
sudo yum install python3 python3-pip wget -y
```
```
sudo pip3 install awscli boto boto3 ansible
```
```
ansible --version
```

Use aws configure and pass your credentials
```
aws configure
```

Create a inventory in the location /etc/ansible/hosts
localhost ansible_connection=local


## Docker & Kubernetes Task:

### Problem Statement: 
Build a docker image to use the python api and push it to the DockerHub. 
Create a pod and nodeport service with that Docker image.

#### Hint: 
A KOPS cluster would be provided to you. You can use the worker nodes to 
write DockerFile and build image. Use the DockerFile provided to you if needed to 
create the docker image

requirements.txt

Flask==1.0.1
requests==2.8.1

Dockerfile

FROM ubuntu:18.04
LABEL maintainer="Admin CloudThat"
RUN apt-get update -y && \
    apt-get install -y python-pip python-dev
COPY ./requirements.txt /app/requirements.txt
WORKDIR /app
RUN pip install -r requirements.txt
COPY ./code /app
CMD [ "python", "./app.py" ]

app.py

#!/usr/bin/env python3.8
# -*- coding: utf-8 -*-
import os

from flask import Flask, jsonify, request

app = Flask(__name__)

notes = [
    {
        "author": "hightower",
        "title": "Kubernetes up and Running"
    },
    {
        "author": "navathe",
        "title": "Database Fundamentals"
    },
    {
        "author": "ritchie",
        "title": "Let us C"
    }
]

@app.route("/")
def hello():
    return "Welcome to my bookstore!"

@app.route("/v1/books/")
def list_all_books():
    list = []
    for item in notes:
       list.append({'book':item['title']})
    return jsonify(list)

@app.route("/v1/books/<string:author>")
def get_by_author(author):
    for item in notes:
	    if item['author'] == author:
	       data = item
    return jsonify(data)
    if not item:
        return jsonify({'error': 'Author does not exist'}), 404

@app.route("/v1/books/", methods=["POST"])
def add_book():
    author = request.json.get('author')
    book = request.json.get('title')
    if not author or not book:
        return jsonify({'error': 'Please provide Author and Title'}), 400
    else:
        data = request.get_json()
        notes.append(data)
        return jsonify({'message': 'Added book successfully','author':author,'book': book}),200

if __name__ == '__main__':
    app.run(threaded=True, host='0.0.0.0', port=5000)
