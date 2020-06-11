**Student Name: Thach Ngoc Nguyen**

**Student Number: s3651311**

# ACME App

---

## Dependencies

### Deploy AWS infrastructure

First thing we need to do is deploy our infrastructure, which is located in the `environment` directory. In order to do that, open the project directory in an terminal shell and run the commands below:

```
cd environment
make up
make kube-up
```

*Rememeber to update your AWS credentials*

---

### Update the endpoints

After we deploy our infrastructure, the output from the terminal would look similar to this screenshot below:

**Add a screenshot when finish deploying**

A few steps need to be done:

- In the `.circleci/config.yml`, update the `rmit-kops-state-xxxxxx` value in the `setup-cd` command

- In the `.circleci/config.yml`, update the `ECR` value in the `package` job

- In the `infra/Makefile`, update the `--backend-config` with our `DynamoDB` and `S3 Bucket` endpoints

- In the `infra/terraform.tfvars`, update the `vpc_id` and `subnet_ids`

---

### Before starting CircleCI

We need to check that we got the namespaces needed for the CI, which are `test`, `prod` and `amazon-cloudwatch`.

Check by running the command below in your terminal:

```
kubectl get namespaces
```

If the namespaces mentioned were not created, run the commands below in your terminal to create them:

```
kubectl create namespace test 
kubectl create namespace prod
kubectl create namespace amazon-cloudwatch
```


*If everything is checked out, the app is ready for CircleCI deployment*

---

## HELM Chart

### Source files

We have a directory called `helm`. Inside we have a HELM chart called `acme`.

- `Chart.yaml` - containing the description of the chart

- `values.yaml` - this file is for storing the default values for a chart, which can be overriden by the user via `helm upgrade` or `helm install`

- `templates/deployment.yml` - A manifest for creating a Kubernetes deployment

- `templates/service.yml` - A manifest for creating a service endpoint for your deployment

---

### Variables

- As you can see inside the `templates/deployment.yml`, there are some values that are written between `{{ .Values.xxx }}`.

- These values can be either passed in via the `values.yaml` file or from the `helm upgrade` and `helm install` with the `--set` option (you can evaluate an example in `.circleci/config.yml`, in `deploy-test` job)

- In the `.circleci/config.yml` file, the Image in the AWS ECR is stored at `artifacts/image.txt`

- In the `.circleci/config.yml` file, the database endpoint in the AWS ECR is stored at `artifacts/dbendpoint.txt`

---

### Deployment Manifest

---

### Service Manifest


---

## Deploy application into a non-production environment

In the `.circleci/config.yml` file, this job has been declared as `deploy-test`

---

Firstly, it will deploy the RDS instance so that the application can connect to later

*Uncomment the `make down` and re-arrange the workflow if you just want to tear down the RDS instance*


```
- run: 
    name: Deploy infra
    command: |
    cd artifacts/infra

    # Init remote backend
    make init

    # Deploy RDS instance
    make up 

    terraform output endpoint > ../dbendpoint.txt

    # Destroy
    # make down
```

---

Before install the pod(s), we will remove the old pod(s) from the previous builds to prevent potential confliction

```
- run:
    name: Uninstall old pod(s)
    command: |
    helm delete acme -n test
```

---

Then, the application will be deploy without the database connection and upgrade the database connection in the later step

```
- run:
    name: Deploy app without database
    command: |
    helm upgrade acme artifacts/acme-0.1.0.tgz -i -n test --set image=$(cat artifacts/image.txt)


- run:
    name: Upgrade app with database
    command: |
    helm upgrade acme artifacts/acme-0.1.0.tgz -i -n test --wait --set image=$(cat artifacts/image.txt),dbhost=$(cat artifacts/dbendpoint.txt)
```

*Notes for the options in the `helm upgrade` command:*

- *-i : is to make the build install the chart if it isn’t already installed* 

- *-n test : is telling helm to use the test namespace* 

- *--set : is telling helm to pass in values at `{{ .Values.xxx }}`* 

- *--wait : is telling helm to wait until all actions are completed before continuing*

---

After that, the updating database script will be executed from the HELM deployment.

*This step may invoke error when executed while trying to terminate **the old pods** and set **the new pod** running*

**Solution:** *rerun the job from CircleCI*

```
- run:
    name: Run database update script
    command: |
    # Run update script
    kubectl exec deployment/acme -n test --pod-running-timeout=5m -- ./node_modules/.bin/sequelize db:migrate
```

*Notes for the options in the `kubectl exec` command:*

- *-n test : indicating which namespace our deployment is in (test namespace)* 

- *--pod-running-timeout : the length of time to wait until at least one pod is running*

---

## End-to-end test to run against the non-production environment


---

## Deploy application into a production environment


---

## Integrate logging to the solution

First we create a namespace named `amazon-cloudwatch` with `kubectl` using this command:

```
kubectl create namespace amazon-cloudwatch
```

Then we create a configuration map for the FluentD service

```
kubectl create configmap cluster-info \ 
--from-literal=cluster.name=rmit.k8s.local \
--from-literal=logs.region=us-east-1 -n amazon-cloudwatch
```

Download the manifest file for FluentD via this [link] (https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluentd/fluentd.yaml) and apply with this command from your terminal:

```
kubectl apply -f fluentd.yaml
```



---

## CLEAN UP INSTRUCTION

Simply change into `environment` directory and run these commands:

```
make kube-down
make down
```