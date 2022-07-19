# Install and config the operators

## Install the GitOps operators

Subscribe to the operators:

```shell
oc apply -f operators/openshift-gitops-operator.yaml
oc apply -f operators/openshift-pipeline-operator.yaml
```

Verify that the default GitOps instance is up-and-running:

```shell
oc get pods -n openshift-gitops
```

## Configure Red Hat Quay

### Install OpenShift Data Foundations

Red Hat Quay uses some Red Hat Data Foundation APIs. To install the ODF operator, follow the instructions [here](https://access.redhat.com/documentation/en-us/red_hat_openshift_data_foundation/4.10).

Also create a default `StorageSystem`.

### Install the Quay operator

Deploy the initial config params:

```shell
oc create secret generic -n openshift-operators --from-file config.yaml=./quay/config.yaml quay-init-config-bundle
```

Subscribe to the Quay operator:

```shell
oc apply -f operators/openshift-quay-operator.yaml
```

Deploy the Quay instance:

```shell
oc create -n openshift-operators -f quay/quay-registry.yaml
```

#### Create the admin user

Get the Quay API endpoint:

```shell
oc get quayregistry -n openshift-operators quay-registry -o jsonpath="{.status.registryEndpoint}" -w
```

```shell
export QUAY_ENDPOINT=https://quay-registry-.... 

curl -X POST -k  "$QUAY_ENDPOINT/api/v1/user/initialize" \
    --header 'Content-Type: application/json' \
    --data '{ "username": "quayadmin", "password":"quaypass123", "email": "quayadmin@example.com", "access_token": true}'

```

#### Configure the Quay Bridge

Log into Quay as the admin user (quayadmin@quaypass123).

Create a `new organization` (gitopshq).

Create a new `application` within the organization (quay-bridge). 

Create an `Access Token` for the application. Give it full rights to the organization.

Create a secret with the above access token:

```shell
oc create secret -n openshift-operators generic quay-integration \
    --from-literal=token=<access_token>
```

Update `quay/quay-integration.yaml` with the actual Quay instance endpoint.

Deploy the Quay Bridge instance:

```shell
oc apply -n openshift-operators -f quay/quay-integration.yaml
```

See [https://github.com/quay/quay-bridge-operator](https://github.com/quay/quay-bridge-operator) for more details on the operator.


### Deploy the Quay Container Security Operator

```shell
oc apply -f operators/openshift-quay-security-operator.yaml
```

See [https://github.com/quay/container-security-operator](https://github.com/quay/container-security-operator) for more details on the operator.


## Usefull commands

```shell
# list installed operators
oc get csv

# list available operators
oc get packagemanifests -n openshift-marketplace

```