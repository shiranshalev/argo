## some argo generic ecosystem with self managed argocd server on docker-desktop

verify kuberentes context and connection
```
kubectl config current-context

# output
# docker-desktop

kubectl get ns

# output
# NAME              STATUS   AGE
# kube-node-lease   Active   1d2h
# kube-public       Active   1d2h
# kube-system       Active   1d2h
# default           Active   1d2h
```

install nginx ingress controller, it'll use localhost on the windows machine and WSL2
```
kubectl apply -f utils\ingress-controller.yaml
```

if using docker desktop windows wsl2 based, ingress controller svc might be stuck on 'pending' waiting for external  ip or vpnkit-controller pod is in error loop
 some problem-causers\solutions:
 powershell as admin
 ```powershell
 restart-service iphlpsvc
 ```
 

create certificate
cmd as admin on windows:
```cmd
choco install mkcert
```
in root dir:
```
mkcert *.gals.local
# creating the tls secret using the cert above
kubectl create ns argo
kubectl create secret tls gals-local-tls --key  _wildcard.gals.local-key.pem --cert _wildcard.gals.local.pem
```
install argocd:
(edit values yaml.. mostly ingress & tls secret name)
```
helm install argocd -n argo .\argo\argocd
```
add all ingress entries under hosts file

if you already have 127.0.0.1 alias, just add more aliases and save (**as admin! notepad, notepad++,etc**), for example:
windows path:
```
C:\Windows\System32\drivers\etc\hosts

127.0.0.1 kubernetes.docker.internal argocd.gals.local argoworkflows.gals.local grafana.gals.local
```
verify argocd login is accessible

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
kubectl get pods -n argo | ?{$_ -like "*workflows-server*"} | %{$_.split()[0]} | %{kubectl exec $_ -n argo -- argo auth token} | scb
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