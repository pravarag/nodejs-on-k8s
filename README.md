# nodejs-on-k8s

This project bootstraps a nodejs based *Hello World* application
inside a Kubernetes cluster

```shell
.
├── Dockerfile
├── Dockerfile~
├── README.md
├── k8s-manifests
│   ├── nodeApp-deploy.yml
│   ├── nodeApp-hpa.yml
│   ├── nodeApp-priorityclass.yml
│   └── nodeApp-service.yml
├── nodejs-chart
│   ├── Chart.yaml
│   ├── charts
│   ├── templates
│   │   ├── NOTES.txt
│   │   ├── _helpers.tpl
│   │   ├── deployment.yaml
│   │   ├── deployment.yaml~
│   │   ├── hpa.yaml
│   │   ├── priority_class.yaml
│   │   ├── service.yaml
│   │   └── tests
│   │       └── test-connection.yaml
│   └── values.yaml
├── package.json
└── server.js

```
## Deployment using kubectl
Below are kubectl commands to get started with the application deployment:
```shell
$ kubectl apply -f k8s-manifests
```

Above command will create the below resources once applied:
1. A deployment with 10 replica pods
2. A service of type Loadbalancer which will create an EC2 classic loadbalancer
3. An HPA with configuration to run a minimum of 7 and a max of 50 pods
4. A priority class with value 1200000 for deployment pods

The HPA is configured to scale at average 50% of cpu and 60% of memory
load.

I've created a custom docker image for demo purpose hosted at
**dockerhub: pravarag/nodejs-test** and at **AWS ECR** as well.

Below are the steps to create an image locally and push to Amazon ECR:

1. After building an image locally from the Dockerfile, we need to tag our image with AWS ECR registry, repository and optional image tag combination

```shell
$ docker tag image_id <aws_account_id>.dkr.ecr.us-east-1.amazonaws/nodejs-test
```
(in this case for region us-east-1 but can be any region as well)

2. Now we need to push our image to remote ECR registry but for that we need to authenticate our Docker client to a registry(in this case ECR) so that we can use push or pull
commands for our images. And the easisest way to authenticate is by using an Authorization token and below is the command:

```shell
$ aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.us-east-1.amazonaws.com
```

where we need to replace aws_account_id with our account_id value. once we see Login as succedded, we can push our image using the below command:
```shell
$ docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/nodejs-test.
```
But we need to make sure, the repository "nodejs-test" already exists for that particular region
in AWS ECR.


## Deployment using Helm

All the K8s manifests are packages as a helm chart in this repo under
**nodejs-chart** . To deploy this helm chart with default values.yaml
follow below commands:

```shell
$ helm install <Name> <chart_name>
```

Above command will create required manifests inside the Kubernetes
cluster as per the values defined in **values.yaml** file. We can
modify any value inside values.yaml and then upgrade the chart to
inhibit newer values.