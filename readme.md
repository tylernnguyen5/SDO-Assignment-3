**Student Name: Thach Ngoc Nguyen**

**Student Number: s3651311**

# ACME App

---

## HELM Chart



---

## Deploy application into a non-production environment


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

Download and apply the manifest file for FluentD via this link

---

## CLEAN UP INSTRUCTION

