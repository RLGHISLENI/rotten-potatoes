# Desafio 02 e 03 - Kubernetes e Pipeline CI/CD

## Pré requisitos

- Linux
- Terminal
- VSCode
- Docker
- Git
- ASDF
- kubectl
- [kubectx](https://github.com/ahmetb/kubectx)
- k3d

## Instalação de pré requisitos e inicialização do Docker

Primeiro certifique-se de ter os [pré requisitos](https://github.com/RLGHISLENI/rotten-potatoes) instalados e configurados em sua máquina.

## Como baixar o projeto em sua máquina e realizar build na imagem docker

Abra o terminal e clone este repositório público utilizando o git

```zsh
git clone https://github.com/RLGHISLENI/rotten-potatoes.git
```

Navegue até a pasta fonte do projeto

```zsh
cd rotten-potatoes/src
```

Abra o VSCode para poder editar os arquivos Dockerfile e .dockerignore

```zsh
code .
```

Realize as alterações necessárias no projeto e nos arquivos docker e salve suas alterações.

Execute o build para atualizar a imagem docker.

> ATENÇÃO: Incremente a versão denominada :**v[n]**, onde **[n]** é o número da versão atual mais 1.
> Por exemplo _**:v2**_

```zsh
docker image build -t rlghisleni/rotten-potatoes:v[n] .
```

Atualize a versão _**:latest**_ da imagem

```zsh
docker tag rlghisleni/rotten-potatoes:v[n] rlghisleni/rotten-potatoes:latest
```

Liste as imagens para visualizar a nova versão da imagem criada

```zsh
docker images
```

## Como baixar e executar a imagem docker do projeto em sua máquina

Abra o terminal de comandos e execute o comando abaixo:

```zsh
docker run -d -p 8080:8080 --name=rotten-potatoes rlghisleni/rotten-potatoes:latest
```

Liste os containers em execução e verifique se _**rlghisleni/rotten-potatoes**_ consta na relação.

```zsh
docker container ls
```

Abra o navegador de sua preferência na url _**http://localhost:8080/**_ e o projeto será apresentado.

> Mantenha este container **PARADO** pois ele será utilizado através do cluster kubernetes **mapeando a mesma porta 8080**.

## Como executar ou parar o container do projeto em sua máquina

Para **iniciar a executar do container** em sua máquina, utilize o comando abaixo

```zsh
docker container start rotten-potatoes
```

Para **para a execução do container** em sua máquina, utilize o comando abaixo

```zsh
docker container stop rotten-potatoes
```

## Como atualizar a imagem no Registry DockerHub

Realize o login na conta do DockerHub, informando o usuário e senha para se autenticar a plataforma.

```zsh
docker login
```

Envie a nova versão da imagem alterada para o servidor Registry

```zsh
docker push rlghisleni/rotten-potatoes:v[n]
```

Envie também a imagem **latest** atualizada

```zsh
docker push rlghisleni/rotten-potatoes:latest
```

## Imagem do projeto no Docker Hub

- [Repositório no Docker Hub contendo a imagem do Projeto](https://hub.docker.com/r/rlghisleni/rotten-potatoes)

## Como criar o cluster kubernetes

Abra o terminal e utilize o k3d para criar o cluster kubernetes como descrito abaixo

```zsh
k3d cluster create meucluster --agents 1 --servers 1 -p "8080:30000@loadbalancer"
```
**Importante:**

A string **_`meucluster`_** representa o nome do seu cluster kubernetes, portando você pode utilizar o nome que melhor se adequar a sua realidade.

Os parâmentros **_`agents`_** e **_`servers`_** podem ser dimencionados conforme sua necessidade e poder computacional.

O parâmetro **_`-p`_** realiza o **bind** da porta `8080` para a porta `30000` definida na instância do **service** como porta externa de entrada ao node kubernetes.

A instrução **_`@loadbalancer`_** define que será utilizado o balanceamento de carga a partir da execução do container de proxy criado pelo **k3d**.

## Como criar o deployment do kubernetes

Navegue até a pasta `k8s` contida no projeto e a partir do terminal execute o comando abaixo para realizar a criação dos pods, replicasets, service e deployment da aplicação

```zsh
kubectl apply -f deployment.yaml
```
Pronto, sua aplicação está no ar utilizando todo o poder dos containers docker e sendo orquestrada pelo kubernetes.

Abra o navegador de sua preferência na url _**http://localhost:8080/**_ e o projeto será apresentado.

## Comandos úteis

```zsh
# Exibe cada objeto em execução
kubectl get nodes
kubectl get pods
kubectl get service
kubectl get replicaset
kubectl get deployment

# Exibe todos objeto em execução
kubectl get all

# Exibe os pods de sistema em execução
kubectl get pods --namespace kube-system

# Exibe os pods filtrando pela label infomada
kubectl get pods -l app=web

# Exibe informações sobre o cluster
kubectl cluster-info

# Exibe todos os recursos disponíveis e suas versões
kubectl api-resources

# Exibe uma descrição completa sobre o pod
kubectl describe pod/meupod

# Realiza a criação dos pods através do manifesto informado (melhor utilizar o apply)
kubectl create -f pod.yaml

# Realiza a execução do manifesto informado
kubectl apply -f pod.yaml
kubectl apply -f replicaset.yaml
kubectl apply -f deployment.yaml

# Realiza a exclusão completa dos manifestos informados
kubectl delete -f pod.yaml
kubectl delete -f replicaset.yaml
kubectl delete -f deployment.yaml

# Realiza a exclusão de cada objeto definidi
kubectl delete pod meupod
kubectl delete replicaset meureplicaset
kubectl delete deployment meudeployment

# Executa o pod permitindo acesso via browser através da porta 8080
kubectl port-forward pod/meupod 8080:80

# Altera a quantidade de replicaset por linha de comando
kubectl scale replicaset meureplicaset --replicas 10

# Altera a imagem definida no deployment através da linha de comando
kubectl set image deployment meudeployment web=kubedevio/web-page:green
```

## **Explicações Importantes sobre o uso de Docker e Kubernets**

### **Para que serve o Kubernetes e porque ele não substitui o Docker**

Docker é uma **plataforma de conteinerização**, capaz de construir, distribuir e rodar contêineres, já o Kubernetes é **uma ferramenta auxiliar de orquestração de containers** que em conjunto com o Docker nos provê uma série de benefícios como escalabilidade, resiliência e balanceamento de carga evitando sobrecargas além de possuir uma estratégia de atualização que mantem sua aplicação no ar enquanto o processo está rolando.

### **Como funciona a arquitetura do Kubernetes e quais os elementos usados pra fazer o deploy de forma correta no Kubernetes**

O kubernetes é formado por um **cluster** (conjunto de recursos ou máquinas) onde cada uma delas vai realizar o papel de **_"Control Plane"_** ou **_"Node"_**.

O **Node é quem executa os containers** das aplicações, ele quem executa toda a carga enquanto o Control Plane gerencia os Nodes e orquestra todo o Cluster.

Para garantir alta disponibilidade no cluster **é recomendado que tenhamos mais de um Control Plane no cluster** pois caso um deles tenha algum problema o outro poderá assumir seu papel.

Para realizar o deploy de nossa aplicação precisamos de uma **imagem Docker** contendo a nossa aplicação completa e rodando sem bugs e disponível em algum servidor registry como DockerHub por exemplo.

O próximo passa é criarmos nosso **arquivo de manifesto** e definirmos a criação dos** pods** que irão **executar o container**, feito isso podemos configurar e implementar nossa **replicaset** que será responsável por **manter todos os pods sempre ativos e em execução**.

Cada vez que um pod apresenta algum problema a **replicaset** sobe outra instância do pod para **manter a quantidades de réplicas em execução**.

Para garantir que nosso ambiente não sofra com atualizações em produção, criamos e configuramos nosso **deployment** que irá garantir que uma replicaset seja **executada em paralelo** mantendo a replica atual rodando até a conclusão da atualização, permitindo assim a **alta disponibilidade** do serviço.


## **Explicações Importantes sobre o uso de CI e CD**

### **Como o uso de pipelines CI e CD pode ajudar a ganhar mais velocidade e eficiência**

O **CI/CD** faz parte do processo de **pipeline** utilizado pelas equipes de desenvolvimento para **automatizar** a construção e **execução dos testes** de forma rápida e eficiente, permitindo agilizar a entrega de qualquer artefato em diversos ambientes de forma **paralela ou sequencial**.

Este processo aumenta a **confiabilidade, segurança e consistência** das entregas da equipe, evitando que algo deixe de ser realizado ou esquecido por acidente.

Atualmente estas práticas são fundamentais para construção de qualquer tipo de aplicação e garantem uma **maior organização** entre as **etapas de desenvolvimento**.

### **O que é o GitHub Actions e quais seus elementos básicos**

GitHub Actions é uma **plataforma de integração contínua** e **entrega contínua** (CI/CD) que **permite automatizar** a sua compilação, testar e pipeline de implantação. 

É possível **criar fluxos de trabalho** completamente personalizado que criam e testam cada pull request no seu repositório, ou implantar pull requests mesclados em produção.

Os principais elementos utilizados no GitHub Actions são: Workflow, Events, Jobs, Steps, Actions e Runners. 

## Referências

- [Meu Registry Server - DockerHub](https://hub.docker.com/u/rlghisleni)
- [Exemplos básicos de pod, replicaset e deployment](https://github.com/RLGHISLENI/iniciativa-kubernetes-manifest)
- [Desafio 01 - Docker](https://github.com/RLGHISLENI/conversao-temperatura)
