## some argo generic ecosystem with self managed argocd server on docker-desktop
install ingress 
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/cloud/deploy.yaml
```

if using docker desktop windows wsl2 based, ingress controller svc might be stuck on 'pending' waiting for external ip,
reset to lxssmanager through services did the trick. 



create certificate
```
mkcert *.gals.local

kubectl create ns argo
kubectl create secret tls gals-local-tls --key  _wildcard.gals.local-key.pem --cert _wildcard.gals.local.pem
```
install argocd:
(edit values yaml.. mostly ingress & tls secret name)
```
helm install argocd -n argo .\argo\argocd
```
in argocd add manually the root app, it'll include argocd itself:

create application in argo namespace with the galsolom/argo.git repo, point to
branch main with path: 
```
argo/apps
```
and under Helm section, submit
```
values.yaml
```

(important to hit 'Enter' in the UI, otherwise the input will not be submitted.)

linux/gitbash get argocd login
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo

```

powershell get argo-workflows login
```
kubectl get pods -n argo | ?{$_ -like "*workflows-server*"} | %{$_.split()[0]} | %{kubectl exec $_ -n argo  argo auth token} | scb
```


## testing some workflow builds




create docker-registry secret for kaniko:
linux/gitbash
```
echo -n user:pass | base64
```
replace xxx with the encoded credentials
```
{
	"auths": {
		"https://index.docker.io/v1/": {
			"auth": "xxxxxxxxxxxxxxx"
		}
	}
}
```
create the secret
mount it under /kaniko/.docker in the pod spec
```
kubectl create secret generic kaniko-docker-secret --from-file=config.json -n argo
```