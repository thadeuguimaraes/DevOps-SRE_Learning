## O que é um StatefulSet?

Os `StatefulSets` são uma funcionalidade do Kubernetes que gerencia o deployment e o scaling de um conjunto de Pods, fornecendo garantias sobre a ordem de deployment e a singularidade desses Pods.

Diferente dos Deployments e Replicasets que são considerados stateless (sem estado), os `StatefulSets` são utilizados quando você precisa de mais garantias sobre o deployment e scaling. Eles garantem que os nomes e endereços dos Pods sejam consistentes e estáveis ao longo do tempo.

## Quando usar StatefulSets?

Os `StatefulSets` são úteis para aplicações que necessitam de um ou mais dos seguintes:

- Identidade de rede estável e única.
- Armazenamento persistente estável.
- Ordem de deployment e scaling garantida.
- Ordem de rolling updates e rollbacks garantida.
- Algumas aplicações que se encaixam nesses requisitos são bancos de dados, sistemas de filas e quaisquer aplicativos que necessitam de persistência de dados ou identidade de rede estável.

## E como ele funciona?

Os StatefulSets funcionam criando uma série de Pods replicados. Cada réplica é uma instância da mesma aplicação que é criada a partir do mesmo spec, mas pode ser diferenciada por seu índice e hostname.

Ao contrário dos Deployments e Replicasets, onde as réplicas são intercambiáveis, cada Pod em um StatefulSet tem um índice persistente e um hostname que se vinculam a sua identidade.

Por exemplo, se um StatefulSet tiver um nome giropops e um spec com três réplicas, ele criará três Pods: giropops-0, giropops-1, giropops-2. A ordem dos índices é garantida. O Pod giropops-1 não será iniciado até que o Pod giropops-0 esteja disponível e pronto.

A mesma garantia de ordem é aplicada ao scaling e aos updates.

## O StatefulSet e os volumes persistentes

Um aspecto chave dos `StatefulSets` é a integração com Volumes Persistentes. Quando um Pod é recriado, ele se reconecta ao mesmo Volume Persistente, garantindo a persistência dos dados entre as recriações dos Pods.

Por padrão, o Kubernetes cria um PersistentVolume para cada Pod em um StatefulSet, que é então vinculado a esse Pod para a vida útil do StatefulSet.

Isso é útil para aplicações que precisam de um armazenamento persistente e estável, como bancos de dados.

## O StatefulSet e o Headless Service

Para entender a relação entre o StatefulSet e o` Headless Service`, é preciso primeiro entender o que é um` Headless Service`.

No Kubernetes, um serviço é uma abstração que define um conjunto lógico de Pods e uma maneira de acessá-los. Normalmente, um serviço tem um IP e encaminha o tráfego para os Pods. No entanto, um` Headless Service` é um tipo especial de serviço que não tem um IP próprio. Em vez disso, ele retorna diretamente os IPs dos Pods que estão associados a ele.

Agora, o que isso tem a ver com os `StatefulSets`?

Os `StatefulSets` e os` Headless Service`s geralmente trabalham juntos no gerenciamento de aplicações stateful. O` Headless Service` é responsável por permitir a comunicação de rede entre os Pods em um `StatefulSet`, enquanto o ` gerencia o deployment e o scaling desses Pods.

Aqui está como eles funcionam juntos:

Quando um StatefulSet é criado, ele geralmente é associado a um` Headless Service`. Ele é usado para controlar o domínio DNS dos Pods criados pelo StatefulSet. Cada Pod obtém um nome de host DNS que segue o formato: <pod-name>.<service-name>.<namespace>.svc.cluster.local. Isso permite que cada Pod seja alcançado individualmente.

Por exemplo, se você tiver um StatefulSet chamado giropops com três réplicas e um` Headless Service` chamado nginx, os Pods criados serão giropops-0, giropops-1, giropops-2 e eles terão os seguintes endereços de host DNS: giropops-0.nginx.default.svc.cluster.local, giropops-1.nginx.default.svc.cluster.local, giropops-2.nginx.default.svc.cluster.local.

Essa combinação de `StatefulSets` com` Headless Service`s permite que aplicações stateful, como bancos de dados, tenham uma identidade de rede estável e previsível, facilitando a comunicação entre diferentes instâncias da mesma aplicação.

Bem, agora que você já entendeu como isso funciona, eu acho que já podemos dar os primeiros passos com o `StatefulSet`.

## Criando um Statefulset

Para a criação de um `StatefulSet` precisamos de um arquivo de configuração que descreva o `StatefulSet` que queremos criar. Não é possível criar um `StatefulSet` sem um manifesto yaml, diferente do que acontece com o Pod e o `Deployment`.

Para o nosso exemplo, vamos utilizar o Nginx como aplicação que será gerenciada pelo `StatefulSet`, e vamos fazer com que cada Pod tenha um volume persistente associado a ele, e com isso, vamos ter uma página web diferente para cada Pod.

Para isso, vamos criar o arquivo `nginx-statefulset.yaml` com o seguinte conteúdo:

```ỳaml
apiVersion: apps/v1
kind: StatefulSet # Tipo do recurso que estamos criando, no caso, um StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # Como estamos utilizando StatefulSet, precisamos criar um template de volume para cada Pod, entã ao invés de criarmos um volume diretamente, criamos um template que será utilizado para criar um volume para cada Pod
  - metadata:
      name: www # Nome do volume, assim teremos o volume www-0, www-1 e www-2
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

Agora, vamos criar o `StatefulSet` com o comando:

```bash
kubectl apply -f nginx-statefulset.yaml
```

Para verificar se o `StatefulSet` foi criado, podemos utilizar o comando:

```bash
kubectl get statefulset
```

Caso queira ver com mais detalhes, podemos utilizar o comando:

```bash
kubectl describe statefulset nginx
```

Para verificar se os Pods foram criados, podemos utilizar o comando:

```bash
kubectl get pods
```

## O que é um StatefulSet?

Os `StatefulSets` são uma funcionalidade do Kubernetes que gerencia o deployment e o scaling de um conjunto de Pods, fornecendo garantias sobre a ordem de deployment e a singularidade desses Pods.

Diferente dos Deployments e Replicasets que são considerados stateless (sem estado), os `StatefulSets` são utilizados quando você precisa de mais garantias sobre o deployment e scaling. Eles garantem que os nomes e endereços dos Pods sejam consistentes e estáveis ao longo do tempo.

## Quando usar StatefulSets?

Os `StatefulSets` são úteis para aplicações que necessitam de um ou mais dos seguintes:

- Identidade de rede estável e única.
- Armazenamento persistente estável.
- Ordem de deployment e scaling garantida.
- Ordem de rolling updates e rollbacks garantida.
- Algumas aplicações que se encaixam nesses requisitos são bancos de dados, sistemas de filas e quaisquer aplicativos que necessitam de persistência de dados ou identidade de rede estável.

## E como ele funciona?

Os StatefulSets funcionam criando uma série de Pods replicados. Cada réplica é uma instância da mesma aplicação que é criada a partir do mesmo spec, mas pode ser diferenciada por seu índice e hostname.

Ao contrário dos Deployments e Replicasets, onde as réplicas são intercambiáveis, cada Pod em um StatefulSet tem um índice persistente e um hostname que se vinculam a sua identidade.

Por exemplo, se um StatefulSet tiver um nome giropops e um spec com três réplicas, ele criará três Pods: giropops-0, giropops-1, giropops-2. A ordem dos índices é garantida. O Pod giropops-1 não será iniciado até que o Pod giropops-0 esteja disponível e pronto.

A mesma garantia de ordem é aplicada ao scaling e aos updates.

## O StatefulSet e os volumes persistentes

Um aspecto chave dos `StatefulSets` é a integração com Volumes Persistentes. Quando um Pod é recriado, ele se reconecta ao mesmo Volume Persistente, garantindo a persistência dos dados entre as recriações dos Pods.

Por padrão, o Kubernetes cria um PersistentVolume para cada Pod em um StatefulSet, que é então vinculado a esse Pod para a vida útil do StatefulSet.

Isso é útil para aplicações que precisam de um armazenamento persistente e estável, como bancos de dados.

## O StatefulSet e o Headless Service

Para entender a relação entre o StatefulSet e o` Headless Service`, é preciso primeiro entender o que é um` Headless Service`.

No Kubernetes, um serviço é uma abstração que define um conjunto lógico de Pods e uma maneira de acessá-los. Normalmente, um serviço tem um IP e encaminha o tráfego para os Pods. No entanto, um` Headless Service` é um tipo especial de serviço que não tem um IP próprio. Em vez disso, ele retorna diretamente os IPs dos Pods que estão associados a ele.

Agora, o que isso tem a ver com os `StatefulSets`?

Os `StatefulSets` e os` Headless Service`s geralmente trabalham juntos no gerenciamento de aplicações stateful. O` Headless Service` é responsável por permitir a comunicação de rede entre os Pods em um `StatefulSet`, enquanto o ` gerencia o deployment e o scaling desses Pods.

Aqui está como eles funcionam juntos:

Quando um StatefulSet é criado, ele geralmente é associado a um` Headless Service`. Ele é usado para controlar o domínio DNS dos Pods criados pelo StatefulSet. Cada Pod obtém um nome de host DNS que segue o formato: <pod-name>.<service-name>.<namespace>.svc.cluster.local. Isso permite que cada Pod seja alcançado individualmente.

Por exemplo, se você tiver um StatefulSet chamado giropops com três réplicas e um` Headless Service` chamado nginx, os Pods criados serão giropops-0, giropops-1, giropops-2 e eles terão os seguintes endereços de host DNS: giropops-0.nginx.default.svc.cluster.local, giropops-1.nginx.default.svc.cluster.local, giropops-2.nginx.default.svc.cluster.local.

Essa combinação de `StatefulSets` com` Headless Service`s permite que aplicações stateful, como bancos de dados, tenham uma identidade de rede estável e previsível, facilitando a comunicação entre diferentes instâncias da mesma aplicação.

Bem, agora que você já entendeu como isso funciona, eu acho que já podemos dar os primeiros passos com o `StatefulSet`.

## Criando um Statefulset

Para a criação de um `StatefulSet` precisamos de um arquivo de configuração que descreva o `StatefulSet` que queremos criar. Não é possível criar um `StatefulSet` sem um manifesto yaml, diferente do que acontece com o Pod e o `Deployment`.

Para o nosso exemplo, vamos utilizar o Nginx como aplicação que será gerenciada pelo `StatefulSet`, e vamos fazer com que cada Pod tenha um volume persistente associado a ele, e com isso, vamos ter uma página web diferente para cada Pod.

Para isso, vamos criar o arquivo `nginx-statefulset.yaml` com o seguinte conteúdo:

```ỳaml
apiVersion: apps/v1
kind: StatefulSet # Tipo do recurso que estamos criando, no caso, um StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # Como estamos utilizando StatefulSet, precisamos criar um template de volume para cada Pod, entã ao invés de criarmos um volume diretamente, criamos um template que será utilizado para criar um volume para cada Pod
  - metadata:
      name: www # Nome do volume, assim teremos o volume www-0, www-1 e www-2
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

Agora, vamos criar o `StatefulSet` com o comando:

```bash
kubectl apply -f nginx-statefulset.yaml
```

Para verificar se o `StatefulSet` foi criado, podemos utilizar o comando:

```bash
kubectl get statefulset
```

Caso queira ver com mais detalhes, podemos utilizar o comando:

```bash
kubectl describe statefulset nginx
```

Para verificar se os Pods foram criados, podemos utilizar o comando:

```bash
kubectl get pods
```

O nosso StatefulSet está criado, mas ainda temos que criar o `Headless Service` para que possamos acessar os Pods individualmente, e para isso, vamos criar o arquivo nginx-headless-service.yaml com o seguinte conteúdo:

```yaml
apiVersion: v1
kind: Service
metadata:
name: nginx
labels:
app: nginx
spec:
ports:
  - port: 80
    name: web
    clusterIP: None # Como estamos criando um Headless Service, não queremos que ele tenha um IP, então definimos o clusterIP como None
    selector:
    app: nginx
```

Pronto, arquivo criado, hora de criar o `Headless Service` com o comando:

```bash
kubectl apply -f nginx-headless-service.yaml
```

Para verificar se o Headless Service foi criado, podemos utilizar o comando:

```bash
kubectl get service
```

Caso queira ver com mais detalhes, podemos utilizar o comando:

```bash
kubectl describe service nginx
```

Agora que já temos o `StatefulSet` e o `Headless Service` criados, podemos acessar cada Pod individualmente, para isso, vamos utilizar o comando:

kubectl run -it --rm debug --image=busybox --restart=Never -- sh

Agora vamos utilizar o comando nslookup para verificar o endereço IP de cada Pod, para isso, vamos utilizar o comando:

```bash
nslookup nginx-0.nginx.default.svc.cluster.local
```

Agora vamos acessar o Pod utilizando o endereço IP, para isso, vamos utilizar o comando:

```bash
wget -O- http://<endereço-ip-do-pod>
```

Precisamos mudar a página web de cada Pod para que possamos verificar se estamos acessando o Pod correto, para isso, vamos utilizar o comando:

```bash
echo "Pod 0" > /usr/share/nginx/html/index.html
```

Agora vamos acessar o Pod novamente, para isso, vamos utilizar o comando:

```bash
wget -O- http://<endereço-ip-do-pod>
```

A saída do comando deve ser:

```bash
Connecting to <endereço-ip-do-pod>:80 (<endereço-ip-do-pod>:80)
Pod 0
```

Caso queira, você pode fazer o mesmo para os outros Pods, basta mudar o número do Pod no comando `nslookup` e no comando echo.
