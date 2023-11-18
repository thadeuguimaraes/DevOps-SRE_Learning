## O que são volumes?

Para simplificar o seu entendimento nesse momento, volumes nada mais são do que um diretório dentro do Pod que pode ser utilizado para armazenar dados. Eles podem ser utilizados para armazenar dados que precisam ser persistidos, como por exemplo, dados de um banco de dados, ou dados de um sistema de arquivos distribuído.

Quando estamos falando sobre volumes no Kubernetes, precisamos entender que temos basicamente dois tipos de volumes, os `ephemeral volumes` e os `persistent volumes`.

Os ephemeral volumes, que inclusive já vimos durante o treinamento o `emptyDir`, são volumes que são criados e destruídos junto com o Pod. Ele é um volume também, porém com uma diferença, ele não é persistente. Caso ocorra algum problema com o Pod e ele seja removido, o `emptyDir` também será removido.

Agora quando estamos falando sobre volumes do tipo `persistent volumes`, estamos falando sobre volumes que são criados e não são destruídos junto com o Pod, eles são persistidos, são volumes que seus dados são mantidos mesmo que o Pod seja removido.

Esse tipo de volume é super importante para aplicações que precisam armazenar dados que precisam ser mantidos mesmo que o Pod seja removido, como por exemplo, um banco de dados.

## StorageClass

Uma StorageClass no Kubernetes é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Essas classes de armazenamento podem ser usadas para provisionar dinamicamente PersistentVolumes (PVs) de acordo com os requisitos dos PersistentVolumeClaims (PVCs) criados pelos usuários.

A StorageClass é útil para gerenciar e organizar diferentes tipos de armazenamento, como armazenamento em disco rápido e caro ou armazenamento em disco mais lento e barato. Além disso, a StorageClass pode ser usada para definir diferentes políticas de retenção, provisionamento e outras características de armazenamento específicas.

Os administradores do cluster podem criar e gerenciar várias StorageClasses para permitir que os usuários finais escolham a classe de armazenamento adequada para suas necessidades.

Cada StorageClass é definida com um provisionador, que é responsável por criar PersistentVolumes dinamicamente conforme necessário. Os provisionadores podem ser internos (fornecidos pelo próprio Kubernetes) ou externos (fornecidos por provedores de armazenamento específicos).

Inclusive os provisionadores podem ser diferentes para cada provedor de nuvem ou onde o Kubernetes está sendo executado. Vou listar alguns provisionadores que são usados e seus respectivos provedores:

- `kubernetes.io/aws-ebs`: AWS Elastic Block Store (EBS)
- `kubernetes.io/azure-disk`: Azure Disk
- `kubernetes.io/gce-pd`: Google Compute Engine (GCE) Persistent Disk
- `kubernetes.io/cinder:` OpenStack Cinder
- `kubernetes.io/vsphere-volume`: vSphere
- `kubernetes.io/no-provisioner`: Volumes locais
- `kubernetes.io/host-path`: Volumes locais

E se você estiver usando o Kubernetes em um ambiente local, como o Minikube, o provisionador padrão é o `kubernetes.io/host-path`, que cria volumes PersistentVolume no diretório do host. Já no Kind, o provisionador padrão é o `rancher.io/local-path`, que cria volumes PersistentVolume no diretório do host.

Para ver a lista completa de provisionadores, consulte a documentação do Kubernetes no link https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner.

Para você ver os Storage Classes disponíveis no seu cluster, basta executar o seguinte comando:

```bash
kubectl get storageclass
```

| NAME      | PROVISIONER            | RECLAIMPOLICY | VOLUMEBINDINGMODE    | ALLOWVOLUMEEXPANSION | AGE |
| --------- | ---------------------- | ------------- | -------------------- | -------------------- | --- |
| standard  | rancher.io/local-path` | Delete        | WaitForFirstConsumer | false                | 21m |
| (default) |                        |               |                      |                      |     |

Como você pode ver, no Kind, o provisionador padrão é o rancher.io/local-path, que cria volumes PersistentVolume no diretório do host.

Já no EKS, o provisionador padrão é o kubernetes.io/aws-ebs, que cria volumes PersistentVolume no EBS da AWS.

| NAME      | PROVISIONER           | RECLAIMPOLICY | VOLUMEBINDINGMODE    | ALLOWVOLUMEEXPANSION | AGE  |
| --------- | --------------------- | ------------- | -------------------- | -------------------- | ---- |
| gp2       | kubernetes.io/aws-ebs | Delete        | WaitForFirstConsumer | false                | 6h5m |
| (default) |                       |               |                      |                      |      |

Vamos ver os detalhes do nosso Storage Class padrão:

```bash
kubectl describe storageclass standard
```

Name: standard
IsDefaultClass: Yes
Annotations: kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner: rancher.io/local-path
Parameters: <none>
AllowVolumeExpansion: <unset>
MountOptions: <none>
ReclaimPolicy: Delete
VolumeBindingMode: WaitForFirstConsumer
Events: <none>

Uma coisa que podemos ver é que o nosso Storage Class está com a opção IsDefaultClass como Yes, o que significa que ele é o Storage Class padrão do nosso cluster, com isso todos os Persistent Volume Claims que não tiverem um Storage Class definido, irão utilizar esse Storage Class como padrão.

Vamos criar um novo Storage Class para o nosso cluster Kubernetes no kind, com o nome "local-storage", e vamos definir o provisionador como "kubernetes.io/host-path", que cria volumes PersistentVolume no diretório do host.

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
name: giropops
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer

```bash
kubectl apply -f storageclass.yaml
```

`storageclass.storage.k8s.io/giropops created`

Pronto! Agora nós temos um novo S`torage Class` criado no nosso cluster Kubernetes no kind, com o nome "giropops", e com o provisionador "kubernetes.io/no-provisioner", que cria volumes PersistentVolume no diretório do host.

Para saber mais detalhes sobre o `Storage Class` que criamos, execute o seguinte comando:

```bash
kubectl describe storageclass giropops
```

Name: giropops
IsDefaultClass: No
Annotations: kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"giropops"},"provisioner":"kubernetes.io/no-provisioner","reclaimPolicy":"Retain","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner: kubernetes.io/no-provisioner
Parameters: <none>
AllowVolumeExpansion: <unset>
MountOptions: <none>
ReclaimPolicy: Retain
VolumeBindingMode: WaitForFirstConsumer
Events: <none>

Lembrando que criamos esse Storage Class com o provisionador "kubernetes.io/no-provisioner", mas você pode criar um Storage Class com qualquer provisionador que você quiser, como o "kubernetes.io/aws-ebs", que cria volumes PersistentVolume no EBS da AWS.

## PV - Persistent Volume

O PV é um objeto que representa um recurso de armazenamento físico em um cluster Kubernetes. Ele pode ser um disco rígido em um nó do cluster, um dispositivo de armazenamento em rede (NAS) ou mesmo um serviço de armazenamento em nuvem, como o AWS EBS ou Google Cloud Persistent Disk.

O PV é utilizado para fornecer armazenamento durável, ou seja, os dados armazenados no PV permanecem disponíveis mesmo quando o container é reiniciado ou movido para outro nó.

No Kubernetes, você pode usar várias soluções de armazenamento como Persistent Volumes (PVs). Essas soluções podem ser divididas em dois tipos: armazenamento local e armazenamento em rede. Vou te dar exemplos de algumas opções populares de cada tipo:

### Armazenamento local:

- `HostPath:`
  É uma maneira simples de usar um diretório do nó do cluster como armazenamento. É útil principalmente para testes e desenvolvimento, pois não é apropriado para ambientes de produção, já que os dados armazenados só estão disponíveis no nó específico.

### Armazenamento em rede:

- `NFS (Network File System):` É um sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede. É uma opção comum para armazenamento compartilhado em um cluster Kubernetes.
- `iSCSI (Internet Small Computer System Interface):`É um protocolo que permite a conexão de dispositivos de armazenamento de blocos, como SAN (Storage Area Network), por meio de redes IP. Pode ser usado como um PV no Kubernetes.

- `Ceph RBD (RADOS Block Device):`É uma solução de armazenamento distribuído e altamente escalável que oferece suporte ao armazenamento em bloco, objeto e arquivo. Com o RBD, você pode criar volumes de blocos virtualizados que podem ser montados como PVs no Kubernetes.
- `GlusterFS:`É um sistema de arquivos distribuído e escalável que permite criar volumes de armazenamento compartilhado em vários nós do cluster. Pode ser usado como um PV no Kubernetes.
- `Serviços de armazenamento em nuvem:`Fornecedores de nuvem como AWS, Google Cloud e Microsoft Azure oferecem soluções de armazenamento que podem ser integradas ao Kubernetes. Exemplos incluem AWS Elastic Block Store (EBS), Google Cloud Persistent Disk e Azure Disk Storage.

Agora que já sabemos o que é um PV, vamos entender como nós podemos utilizar o `kubectl` para gerenciar os PVs.

Primeira coisa, vamos listar os PVs que temos no nosso cluster:

```bash
kubectl get pv -A
```

```bash
No resources found
```

Com o comando acima estamos listando todos os PVs que temos no nosso cluster, e como podemos ver, não temos nenhum PV criado, ainda. :)

Vamos resolver isso, bora criar um PV?

Para isso, vamos criar um arquivo chamado `pv.yaml:`

apiVersion: v1 # Versão da API do Kubernetes
kind: PersistentVolume # Tipo de objeto que estamos criando, no caso um PersistentVolume
metadata: # Informações sobre o objeto
name: meu-pv # Nome do nosso PV
labels:
storage: local
spec: # Especificações do nosso PV
capacity: # Capacidade do PV
storage: 1Gi # 1 Gigabyte de armazenamento
accessModes: # Modos de acesso ao PV - ReadWriteOnce # Modo de acesso ReadWriteOnce, ou seja, o PV pode ser montado como leitura e escrita por um único nó
persistentVolumeReclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
hostPath: # Tipo de armazenamento que vamos utilizar, no caso um hostPath
path: "/mnt/data" # Caminho do hostPath, do nosso nó, onde o PV será criado
storageClassName: standard # Nome da classe de armazenamento que será utilizada

Antes de criar o PV, eu preciso falar um pouquinho mais sobre o arquivo que criamos, principalmente sobre o que temos de diferente em relação aos outros arquivos que criamos até agora.

- `kind: PersistentVolume`: Aqui estamos definindo o tipo de objeto que estamos criando, no caso um `PersistentVolume`.

Outro ponto importante de mencionar é a seção spec, que é onde definimos as especificações do nosso PV.

- `spec.capacity.storage`: Aqui estamos definindo a capacidade do nosso PV, no caso 1 Gigabyte de armazenamento.
- `spec.accessModes`: Aqui estamos definindo os modos de acesso ao PV, no caso o modo ReadWriteOnce, que significa que o PV pode ser montado como leitura e escrita por um único nó. Aqui nós temos mais alguns modos de acesso:
  - `ReadOnlyMany`: O PV pode ser montado como somente leitura por vários nós.
  - `ReadWriteMany`: O PV pode ser montado como leitura e escrita por vários nós.
- `spec.persistentVolumeReclaimPolicy`: Aqui estamos definindo a política de reivindicação do PV, no caso a política Retain, que significa que o PV não será excluído quando o PVC for excluído. Aqui nós temos mais algumas políticas: - `Recycle`: O PV será excluído quando o PVC for excluído, mas antes disso ele será limpo, ou seja, todos os dados serão apagados. - `Delete`: O PV será excluído quando o PVC for excluído.
  Outra seção importante é a seção `hostPath`, que é onde definimos o tipo de armazenamento que vamos utilizar, no caso um `hostPath`. Vou detalhar abaixo os tipos de armazenamento que podemos utilizar:

- `hostPath`: É uma maneira simples de usar um diretório do nó do cluster como armazenamento. É útil principalmente para testes e desenvolvimento, pois não é apropriado para ambientes de produção, já que os dados armazenados só estão disponíveis no nó específico. Ele é ideal em cenários de testes com somente um node.
- `nfs`: É um sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede. É uma opção comum para armazenamento compartilhado em um cluster Kubernetes.
- `iscsi`: É um protocolo que permite a conexão de dispositivos de armazenamento de blocos, como SAN (Storage Area Network), por meio de redes IP.
- `csi`: Que significa Container Storage Interface, é um recurso que permite a integração de soluções de armazenamento de terceiros com o Kubernetes. O CSI permite que os provedores de armazenamento implementem seus próprios plugins de armazenamento e os integrem ao Kubernetes. É graças ao CSI que podemos utilizar soluções de armazenamento de terceiros, como o AWS EBS, Google Cloud Persistent Disk e Azure Disk Storage.
- `cephfs`: É um sistema de arquivos distribuído e escalável que permite criar volumes de armazenamento compartilhado em vários nós do cluster.
- `local`: É um tipo de armazenamento que permite a criação de volumes locais, onde você pode especificar o caminho do diretório onde os dados serão armazenados. É útil principalmente para testes e desenvolvimento, já que não é apropriado para ambientes de produção, já que os dados armazenados só estão disponíveis no node específico. A diferença entre o hostPath e o local é que o local é um recurso nativo do Kubernetes, enquanto o hostPath é um recurso do Kubernetes que utiliza o recurso nativo do Docker e não é recomendado quando estamos com mais de um node no cluster.
- `fc`: É um protocolo que permite a conexão de dispositivos de armazenamento de blocos utilizando redes de fibra óptica. É uma opção comum para armazenamento compartilhado em um cluster Kubernetes.
  Eu listei somente os tipos de armazenamento mais comuns, mas você pode encontrar mais informações sobre os tipos de armazenamento no Kubernetes Docs.

E por último, temos a seção storageClassName, que é onde definimos o nome da classe de armazenamento que iremos adicionar o PV.

Conforme vamos avançando no treinamento, vamos conhecendo mais detalhes sobre cada tipo de armazenamento.

Pronto, tudo está pronto para criarmos o PV.

```bash
kubectl apply -f pv.yaml
```

```bash
persistentvolume/meu-pv created
```

Vamos listar o nosso PV para ver se ele foi criado corretamente.

```bash
kubectl get pv
```

| NAME   | CAPACITY | ACCESS MODES | RECLAIM POLICY | STATUS    | CLAIM | STORAGECLASS | REASON | AGE |
| ------ | -------- | ------------ | -------------- | --------- | ----- | ------------ | ------ | --- |
| meu-pv | 1Gi      | RWO          | Retain         | Available |       | standard     |        | 10s |

PV criado com sucesso.

Podemos ver que o nosso PV está com o status Available, o que significa que ele está disponível para ser utilizado por um PVC.

Vamos ver os detalhes do nosso PV.

```bash
kubectl describe pv meu-pv
```

Name: meu-pv
Labels: storage=local
Annotations: <none>
Finalizers: [kubernetes.io/pv-protection]
StorageClass: standard
Status: Available
Claim:  
Reclaim Policy: Retain
Access Modes: RWO
VolumeMode: Filesystem
Capacity: 1Gi
Node Affinity: <none>
Message:  
Source:
Type: HostPath (bare host directory volume)
Path: /mnt/data
HostPathType:  
Events: <none>

Dessa forma estamos criando o PV utilizando o provisionador hostPath, que é um provisionador para ser utilizado em testes e desenvolvimento, já que os dados armazenados só estão disponíveis no node específico, por isso bora para mais um exemplo, mas agora utilizando o provisionador nfs, que é um sistema de arquivos de rede que permite compartilhar arquivos entre várias máquinas na rede.

## PVC - Persistent Volume Claim

O PVC é uma solicitação de armazenamento feita pelos usuários ou aplicativos no cluster Kubernetes. Ele permite que os usuários solicitem um volume específico, com base em tamanho, tipo de armazenamento e outras características. O PVC age como uma "assinatura" que reivindica um PV para ser usado por um contêiner. O Kubernetes tenta associar automaticamente um PVC a um PV compatível, garantindo que o armazenamento seja alocado corretamente.

Através do PVC, as pessoas podem abstrair os detalhes de cada tipo de armazenamento, permitindo maior flexibilidade e portabilidade entre diferentes ambientes e provedores de infraestrutura. Ele também permite que os usuários solicitem volumes com diferentes características, como tamanho, tipo de armazenamento e modo de acesso.

Todo PVC é associado a um `Storage Class` ou a um `Persistent Volume`. O Storage Class é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Já o `Persistent Volume` é um recurso que representa um volume de armazenamento disponível para ser usado pelo cluster.

Primeira coisa que vamos fazer é criar o diretório que será compartilhado entre os nodes do cluster. Lembrando que para esse exemplo, estou utilizando uma máquina Linux para criar o compartilhamento NFS, mas você pode utilizar qualquer outro sistema operacional, desde que ele tenha suporte ao NFS.

mkdir /mnt/nfs

Precisamos instalar os pacotes nfs-kernel-server e nfs-common para que o servidor NFS e o cliente NFS sejam instalados.

```bash
sudo apt-get install nfs-kernel-server nfs-common
```

Vamos editar o arquivo `/etc/exports`, que é o arquivo de configuração do NFS, e adicionar o diretório que será compartilhado entre os nodes do cluster.

```bash
sudo vim /etc/exports
```

```bash
/mnt/nfs *(rw,sync,no_root_squash,no_subtree_check)
```

## Onde:

- /mnt/nfs: é o diretório que você deseja compartilhar.
- _:: permite que qualquer host acesse o diretório compartilhado. Para maior segurança, você pode substituir _ por um intervalo de IPs ou por IPs específicos dos clientes que terão acesso ao diretório compartilhado. Por exemplo, 192.168.1.0/24 permitiria que todos os hosts na sub-rede 192.168.1.0/24 acessassem o diretório compartilhado.
- rw: concede permissões de leitura e gravação aos clientes.
- sync: garante que as solicitações de gravação sejam confirmadas somente quando as alterações tiverem sido realmente gravadas no disco.

- no_root_squash: permite que o usuário root em um cliente NFS acesse os arquivos como root. Caso contrário, o acesso seria limitado a um usuário não privilegiado.

- no_subtree_check: desativa a verificação de subárvore, o que pode melhorar a confiabilidade em alguns casos. A verificação de subárvore normalmente verifica se um arquivo faz parte do diretório exportado.

Agora vamos falar para o NFS que o diretório /mnt/nfs está disponível para ser compartilhado.

```bash
sudo exports -arv
```

Maravilha! Vamos agora verificar se o NFS está funcionando corretamente.

```bash
shomount -e
```

Export list for localhost:
/mnt/nfs \*

Já era! O nosso NFS está funcionando corretamente. \o/

Agora que já temos o nosso NFS funcionando, vamos criar o nosso StorageClass para o provisionador `nfs`.

Para esse exemplo, vamos criar um arquivo chamado `storageclass-nfs.yaml` e adicionar o seguinte conteúdo.

apiVersion: storage.k8s.io/v1 # Versão da API do Kubernetes
kind: StorageClass # Tipo de objeto que estamos criando, no caso um StorageClass
metadata: # Informações sobre o objeto
name: nfs # Nome do nosso StorageClass
provisioner: kubernetes.io/no-provisioner # Provisionador que será utilizado para criar o PV
reclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
volumeBindingMode: WaitForFirstConsumer
parameters: # Parâmetros que serão utilizados pelo provisionador
archiveOnDelete: "false" # Parâmetro que indica se os dados do PV devem ser arquivados quando o PV for excluído

Kubernetes não possui um provisionador `nfs` nativo, então não é possível fazer com que o provisionador `kubernetes.io/no-provisioner` crie um PV utilizando um servidor NFS automaticamente, para que isso seja possível, precisamos utilizar um provisionador nfs externo, mas isso não é o foco nesse momento, então vamos criar o nosso PV manualmente, afinal de contas, já estamos experts em PVs, certo?

Bora lá!

Então já podemos criar o PV e associa-lo ao Storage Class, e para isso vamos criar um novo arquivo chamado `pv-nfs.yaml` e adicionar o seguinte conteúdo.

apiVersion: v1 # Versão da API do Kubernetes
kind: PersistentVolume # Tipo de objeto que estamos criando, no caso um PersistentVolume
metadata: # Informações sobre o objeto
name: meu-pv-nfs # Nome do nosso PV
labels:
storage: nfs # Label que será utilizada para identificar o PV
spec: # Especificações do nosso PV
capacity: # Capacidade do PV
storage: 1Gi # 1 Gigabyte de armazenamento
accessModes: # Modos de acesso ao PV - ReadWriteOnce # Modo de acesso ReadWriteOnce, ou seja, o PV pode ser montado como leitura e escrita por um único nó
persistentVolumeReclaimPolicy: Retain # Política de reivindicação do PV, ou seja, o PV não será excluído quando o PVC for excluído
nfs: # Tipo de armazenamento que vamos utilizar, no caso o NFS
server: IP_DO_SERVIDOR_NFS # Endereço do servidor NFS
path: "/mnt/nfs" # Compartilhamento do servidor NFS
storageClassName: nfs # Nome da classe de armazenamento que será utilizada

## Agora vamos criar o nosso PV.

```bash
kubectl apply -f pv-nfs.yaml
```

```bash
persistentvolume/meu-pv created
```

### PVC - Persistent Volume Claim
