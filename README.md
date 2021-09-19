# Capstone

[![CircleCI](https://circleci.com/gh/zypher9/capstone/tree/main.svg?style=svg)](https://circleci.com/gh/zypher9/capstone/tree/main)

## Project Overview

It is Capstone Udacity project to demonstrate implemnting docker image and kubernetes using Circleci pipeline. In this project Circleci orbs has been used
  - circleci/kubernetes@0.12.0
  - circleci/aws-eks@1.1.0
---

## Setup the Environment

* Create a virtualenv and activate it
   ```
   python3 -m venv ~/capstone
   source ~/.capstone/bin/activate
   ```
* Run `make install` to install the necessary dependencies
  ```
  pip install --upgrade pip &&\
	pip install -r requirements.txt &&\
	sudo wget -O /bin/hadolint https://github.com/hadolint/hadolint/releases/download/v1.16.3/hadolint-Linux-x86_64 &&\
	sudo chmod +x /bin/hadolint
  ```

## Lint the App `make lint`
  ```
  hadolint Dockerfile --ignore DL3013
	pylint --disable=R,C,W1203,W1202 app.py
  ```

## Creating the infrastructure using orbs circleci/aws-eks@1.0.3
  ```
  aws-eks/create-cluster:
        cluster-name: capstone
  ```

## Creating the deployment steps with aws-eks/python3 executor
  ```
  - kubernetes/install
  - aws-eks/update-kubeconfig-with-authenticator:
      cluster-name: << parameters.cluster-name >>
      install-kubectl: true
  - kubernetes/create-or-update-resource:
      get-rollout-status: true
      resource-file-path: deploy.yml
      resource-name: deployment/capstone
  ```
  
## Update the container image using `aws-eks/update-container-image`
  ```
  aws-eks/update-container-image:
    cluster-name: capstone
    container-image-updates: capstone=zypher9/capstone
    post-steps:
        - kubernetes/delete-resource:
            resource-names: capstone
            resource-types: deployment
            wait: true
    record: true
    requires:
        - create-deployment
    resource-name: deployment/capstone
  ```

## Testing the cluster steps
  ```
  - kubernetes/install
  - aws-eks/update-kubeconfig-with-authenticator:
      cluster-name: << parameters.cluster-name >>
  - run:
      name: Test cluster
      command: |
        kubectl get nodes
        kubectl get deployment
  ```
  
 ## References
 - https://circleci.com/developer/orbs/orb/circleci/kubernetes
 - https://circleci.com/developer/orbs/orb/circleci/aws-eks
