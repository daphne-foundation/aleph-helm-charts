# Aleph in KIND


# Creating the local cluster
Create cluster

```
make create-infra
```
This creates a 7 node cluster named `alephlocal` using [`kind`](https://kind.sigs.k8s.io/). The configuration
of this cluster can be changed by editing [`kind-config.yml`](kind-config.yml)


# Creating namespaces

Create namespaces

```
kubectl create -f namespace.yaml
```
This creates 2 namespaces named `dev` and `staging` in the k8s cluster.

# Setting up kubectl contexts

Setup kubectl contexts

```
kubectl config set-context dev --namespace=dev --cluster=kind-alephlocal --user=kind-alephlocal
kubectl config set-context staging --namespace=staging --cluster=kind-alephlocal --user=kind-alephlocal
```
This creates 2 kubectl contexts named `dev` and `staging` which correspond to the 2 namespaces we created earlier.
We can operate within one particular namespace by activating the corresponding context.

To use one particular context:

```
kubectl config use-context staging
```
This activates the `staging` context and all our operations will affect the `staging` namespace unless we specify another namespace explicitly.

# Setting up backend services

Set up services like redis, postgres, es
```
make create-services ENV=staging
```
This creates Redis, Postgresql and Elasticsearch services using helm. The config for each of these services can be tweaked by changing values in `charts/values/*.yaml` files.

# Creating secrets

Some configurations like Aleph's secret key, authorized database url etc should be kept secrets. These files are
stored in `secrets/` directory. **The contents of this directory should be encrypted using [`git-crypt`](https://github.com/AGWA/git-crypt).**

Create secrets etc
```
make create-secrets ENV=staging
make create-service-accounts ENV=staging
```

If using Google services like storage, vision api, please save the service-account.json file to `secrets/$(ENV)/service-accounts/service-account-aleph.json`.

If using AWS services, please save the access key and secret key to `secrets/$(ENV)/aleph/AWS_ACCESS_KEY_ID` and `secrets/$(ENV)/aleph/AWS_SECRET_ACCESS_KEY_ID`.

# Install Aleph and included microservices

Install Aleph and related services
```
helm install aleph ./charts/aleph -f ./charts/values/staging.yaml
```
Configuration values for Aleph can be changed in `charts/values/$(ENV).yaml`

Checkout the [`README`](charts/aleph/README.md) for available options.

# Install nginx-ingress

Install ingress controller
```
make install-ingress
```

Install ingresses
```
kubectl apply -f ingress.dev.yaml
kubectl apply -f ingress.staging.yaml
```

You'll have to edit the hostnames and create appropriate DNS entries for the hostnames (in `/etc/hosts` in case of localhost).

