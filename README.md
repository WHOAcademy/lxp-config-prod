# 🦄 LXP Config Repo 🔥

🧰 This repo is an Argo App definition which references [other helm charts](https://github.com/redhat-cop/charts.git). It should not exclusively run Helm Templates but be a more generic Argo App which could reference Kustomize or Operators etc. 

It will contain the configuration that ArgoCD will consume to manage all LXP applications

## What's in the box? 👨

- `values-test` is the argocd app of apps definition for the test environment. This is updated using Jenkins when code is pushed to `master` branch.
- `values-staging` is the argocd app of apps definition for the staging environment. This is updated using Jenkins on successful completion of the e2e tests.

## How do I run it? 🏃‍♀️

### Prereq 
0. OpenShift 4.3 or greater (cluster admin user required) - https://try.openshift.com
1. Install helm v3 (cli) or greater - https://helm.sh/docs/intro/quickstart
2. Install Argo CD (cli) 1.4.2+ or greater - https://argoproj.github.io/argo-cd/getting_started/#2-download-argo-cd-cli

### Create the ArgoCD Project and Apps 🤠

* To login with argocd from CLI using sso:
```bash
argocd login $(oc get route argocd-server --template='{{ .spec.host }}' -n labs-ci-cd):443 --sso --insecure
```
else if no sso:
```bash
argocd login --grpc-web $(oc get routes argocd-server -o jsonpath='{.spec.host}' -n labs-ci-cd) --insecure
```

* Create the ArgoCD Project for your `test` environment apps

```bash
argocd proj create test \
    --description "Test environment" \
    -d https://kubernetes.default.svc,labs-test \
    -d https://kubernetes.default.svc,labs-ci-cd \
    -s https://github.com/redhat-cop/helm-charts.git \
    -s https://github.com/WHOAcademy/lxp-config.git \
    -s http://nexus-labs-ci-cd.apps.who.lxp.academy.who.int/repository/helm-charts/
```

* Create the Argo app `lxp-test` for this project:
```bash
argocd app create lxp-test \
    --project test \
    --revision master \
    --sync-policy automated \
    --dest-namespace labs-ci-cd \
    --dest-server https://kubernetes.default.svc \
    --repo https://github.com/WHOAcademy/lxp-config.git \
    --path "lxp-deployment" --values "values-test.yaml"
```

* Create the ArgoCD Project for your `staging` environment apps

```bash
argocd proj create staging \
    --description "Staging environment" \
    -d https://kubernetes.default.svc,labs-staging \
    -d https://kubernetes.default.svc,labs-ci-cd \
    -s https://github.com/redhat-cop/helm-charts.git \
    -s https://github.com/WHOAcademy/lxp-config.git \
    -s http://nexus-labs-ci-cd.apps.who.lxp.academy.who.int/repository/helm-charts/
```

* Create the Argo app `lxp-staging`:
```bash
argocd app create lxp-staging \
    --project staging \
    --revision master \
    --sync-policy automated \
    --dest-namespace labs-ci-cd \
    --dest-server https://kubernetes.default.svc \
    --repo https://github.com/WHOAcademy/lxp-config.git \
    --path "lxp-deployment" --values "values-staging.yaml"
```
