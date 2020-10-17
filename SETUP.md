# Setup

## Pre Requesites

- Java > 8
- minikube v1.11.0 - [Refer here for installation](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- helm v3.2.1 - [Refer here for installation](https://helm.sh/docs/intro/install/)

## Setup Setps

### Minikube setup

- Setup minikube

  ```bash
  minikube start --memory=8g --cpus=4 --disk-size=35g --embed-certs=true

  kubectl config set-credentials minikube --embed-certs=true --client-certificate=/Users/rmkanda/.minikube/profiles/minikube/client.crt --client-key=/Users/rmkanda/.minikube/profiles/minikube/client.key
  ```

### Minio setup

- Setup

  ```
  kubectl create ns spinnaker

  helm install minio --namespace spinnaker --set accessKey="minioaccesskey" --set secretKey="miniosecretkey" --set persistence.enabled=false stable/minio
  ```

- Verify minio is up and running

  ```
  export POD_NAME=$(kubectl get pods --namespace spinnaker -l "release=minio" -o jsonpath="{.items[0].metadata.name}")

  kubectl port-forward $POD_NAME 9001:9000 --namespace spinnaker
  ```

  Visit - http://localhost:9001/minio/

### HAL setup

- Installation
  ```bash
  curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/macos/InstallHalyard.sh
  sudo bash InstallHalyard.sh
  ```
- Check hal is up and running

  ```bash
  hal -v
  ```

- Set Spinnaker version

  ```bash
  hal config version edit --version 1.20.5
  ```

  Note: you can use `hal version list` to list avaialble versions

- Link Kubernetes

  ```
  hal config stats disable
  hal config provider kubernetes account add my-k8s-account --context minikube --docker-registries docker-hub --kubeconfig-file /Users/rmkanda/.kube/config --namespaces default
  hal config provider kubernetes enable
  hal config deploy edit --type=distributed --account-name my-k8s-account
  ```

- Link GitHub

  ```
  hal config artifact github enable
  hal config artifact github account add my-github-artifact-account --token
  ```

- Link Jenkins

  ```
  hal config ci jenkins enable
  echo $JENKINS_USER_TOKEN | hal config ci jenkins master add my-jenkins-master --address http://jenkins:8080/ --username admin --password
  ```

- Link Docker Hub

  ```
  hal config provider docker-registry enable
  hal config provider docker-registry account edit my-docker-registry --address index.docker.io  --username rmkanda --password
  ```

- Disable Versioning, since minio don't support versioning

  ```
  mkdir ~/.hal/default/profiles
  echo "spinnaker.s3.versioning: false" > ~/.hal/default/profiles/front50-local.yml
  ```

- Set the storage type to minio/s3

  ```
  hal config storage s3 edit --endpoint http://minio:9000 --access-key-id "minioaccesskey" --secret-access-key "miniosecretkey"
  hal config storage s3 edit --path-style-access true
  hal config storage edit --type s3
  ```

- Deploy Spinnaker

  ```
  hal deploy apply
  ```

- Connect to Spinnaker

  ```
  hal deploy connect
  ```

  OR use port-forwarding

  ```
  alias spin_gate='kubectl port-forward -n spinnaker $(kubectl get pods -n spinnaker -o=custom-columns=NAME:.metadata.name | grep gate) 8084:8084'

  alias spin_deck='kubectl port-forward -n spinnaker $(kubectl get pods -n spinnaker -o=custom-columns=NAME:.metadata.name | grep deck) 9001:9000'

  alias spinnaker='spin_gate &; spin_deck &'
  ```

- Visit http://localhost:9000/
