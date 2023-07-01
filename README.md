# Otomi installation on minikube - macOS

You need

* kubectl
* helm
* docker
* minikube

```bash
minikube config set memory 8g
minikube config set cpus 8
minikube start --driver=hyperkit --kubernetes-version=v1.24.15 --cni calico
minikube addons enable metallb

MINIKUBE_IP=$(minikube ip)
expect << _EOF_
spawn minikube addons configure metallb
expect "Enter Load Balancer Start IP:" { send "${MINIKUBE_IP}\\r" }
expect "Enter Load Balancer End IP:" { send "${MINIKUBE_IP}\\r" }
expect eof
_EOF_

helm repo add otomi https://otomi.io/otomi-core
helm repo update

helm install -f values.yaml --set otomi.adminPassword="<an otomi password>" otomi otomi/otomi
```

## Monitor installation

```bash
kubectl get job otomi -w
# or watch
watch helm list -Aa
```

## Once installation done

```bash
kubectl logs jobs/otomi -n default -f
```

and you see

```text
####################################################################
#  To start using Otomi, go to https://otomi.<your minikube ip>.nip.io and sign in to the web console
#  with username "otomi-admin" and password "<your otomi password>".
#  Then activate Drone. For more information see: https://otomi.io/docs/installation//activation
####################################################################
```

1. register on https://portal.otomi.cloud/
    1. click on register cluster
    2. copy license key, and continue
2. open UI, sign in with `otomi-admin` and your password.
3. add the license and activate.
4. once logged, left menu, download CA
5. import the CA with `sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain <the downloaded CA></the>`
