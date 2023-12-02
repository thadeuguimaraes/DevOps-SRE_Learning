## O que iremos ver hoje ?

Hoje é dia de falar sobre duas coisas super importantes no mundo do Kubernetes, hoje nós vamos falar sobre `Secrets` e `ConfigMaps`.

Sim,essas duas peças fundamentais para que você possa ter a sua aplicação rodando no Kubernetes da melhor forma possível, pois elas são responsáveis por armazenar as informações sensíveis da sua aplicação, como por exemplo, senhas, tokens, chaves de acesso, configurações, etc.

Depois do dia de hoje você vai entender como funciona o armazenamento de informações sensíveis no Kubernetes e como você pode fazer para que a sua aplicação possa consumir essas informações da melhor forma possível.

Bora lá?

## O que são Secrets?

Os Secrets fornecem uma maneira segura e flexível de gerenciar informações sensíveis, como senhas, tokens OAuth, chaves SSH e outros dados que você não quer expor nas configurações de seus aplicaçãos.

Um Secret é um objeto que contém um pequeno volume de informações sensíveis, como uma senha, um token ou uma chave. Essas informações podem ser consumidas por Pods ou usadas pelo sistema para realizar ações em nome de um Pod.

### Como os Secrets funcionam

Os Secrets são armazenados no Etcd, o armazenamento de dados distribuído do Kubernetes. Por padrão, eles são armazenados sem criptografia, embora o Etcd suporte criptografia para proteger os dados armazenados nele. Além disso, o acesso aos Secrets é restrito por meio de Role-Based Access Control (RBAC), o que permite controlar quais usuários e Pods podem acessar quais Secrets.

Os Secrets podem ser montados em Pods como arquivos em volumes ou podem ser usados para preencher variáveis de ambiente para um container dentro de um Pod. Quando um Secret é atualizado, o Kubernetes não atualiza automaticamente os volumes montados ou as variáveis de ambiente que se referem a ele.
