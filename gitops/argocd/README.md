# Argo CD Bootstrap for OCP Lab

This folder bootstraps Argo CD and a root `Application` for this repository.

## Structure

- `install/` - Argo CD installation manifests (namespace + upstream install + OpenShift route)
- `apps/` - Argo CD `Application` resources

## Prerequisites

- Logged in to target OpenShift cluster with `oc`
- Access to this repository branch: `ocplab1-spoke`

## 1) Install Argo CD

```bash
oc apply -k https://github.com/ooadeniyi71/ocplab/gitops/argocd/install?ref=ocplab1-spoke
```

## 2) Apply root app

```bash
oc apply -k https://github.com/ooadeniyi71/ocplab/gitops/argocd/apps?ref=ocplab1-spoke
```

## 3) Verify

```bash
oc get ns argocd
oc get pods -n argocd
oc get application -n argocd
oc get route -n argocd
```

## 4) Access Argo CD UI

```bash
oc get route argocd-server -n argocd -o jsonpath='{.spec.host}{"\n"}'
```

Open: `https://<route-host>`

## 5) Initial admin password

```bash
oc -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 -d; echo
```

Username: `admin`

## Notes

- Root app is defined in `apps/ocplab-root-app.yaml`
- It targets:
  - `repoURL`: `https://github.com/ooadeniyi71/ocplab.git`
  - `targetRevision`: `ocplab1-spoke`
  - `path`: `gitops`
