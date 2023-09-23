# Descomplicando Kubernetes-2023

## Day-1 - Entendendo o que são containers e o Kuberenets

- `O que é um container ?`
- `O que é um container engine ?`
- `o que é OCI ?`
- `O que é o Kubernetes ?`
- `O que são os workers e o control plane do Kubernetes ?`
- `Quais os componentes do control plane do Kubernetes`
- `Quais os componentes dos workes do kubernetes ?`
- `Quais as portas TCP e UDP dos componentes do kubernetes`
- `Introdução a pods, replica sets, deployments e service`
- `Entendo e instalando o Kubectl`
- `Criando o nosso primeiro cluster com o Kind`
- `Primeiros passos no Kubernetes com o kubectl`
- `Conhecendo o YAML e o kubectl com dry-run`

## Day-2 - Descomplicando os pods e Limites de Recursos

- `O que é um Pod?`
- `Os sencacionais kubectl get pods e o kubectl describe pods`
- `Conhecendo o kubectl attach e o kubectl exec`
- `Criando o nosso primeiro pod multicontainer utilizando um manifesto`
- `Limitando o consumo de recursos de CPU e Memória`
- `Configurando o nosso primeiro volume EmptyDir`

## Day-3 Descomplicando Deployments e estratégias de rollout

- `O que é um Deployment`
- ``
- ``
- ``
- ``

## Livro Descomplicando o Kubernetes Gratuito

https://livro.descomplicandokubernetes.com.br/?utm_medium=social&utm_source=linktree&utm_campaign=livro+descomplicando+o+kubernetes+gratuito

```bash
git add .
```

```bash
git commit -m ""
```

```bash
git push origin [branch_principal]
```

```bash
kubectl get pod
```

```bash
kubectl get nodes
```

```bash
 kubectl get pod -o wide
```

```bash
kubectl get pods matrix -o yaml
```

```bash
kubectl exec -ti matrix -- bash
```

```bash
kubectl attach matrix -c matrix -ti
```

```bash
kubectl get pods -A
```

```bash
kubectl get pods --all-namespaces
```

```bash
kubectl get pods -n kube-system
```

```bash
 kubectl exec matrix -- cat /usr/share/nginx/html/index.html
```

```bash
kubectl run matrix-1 --image alpine --dry-run=client -o yaml
```

```bash
kubectl logs matrix-1 -f
```

```bash
kubectl logs matrix-1 -c busybox
```

```bash

```

```bash

```

```bash

```

```bash

```
