the reason for the crd folder with the hardcoded CRD:
> prometheuses.monitoring.coreos.com
is resource versioning is used through last-applied annotation,
which hits huge size once applied on this kind of CRD - 8,000 lines of specfications..
so i added it under crds/ with an argocd annoation:
```
argocd.argoproj.io/sync-options: Replace=true
```
