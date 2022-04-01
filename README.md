# Desafio 02 - Kubernetes

## Pré requisitos

- Linux
- Terminal
- VSCode
- Docker
- Git
- ASDF
- kubectl
- k3d

## Instalação de pré requisitos

Para este desafio utilizaremos o **k3d** para criação de nosso cluster kubernetes e o **kubectl** para criação de pods, replicasets, services e deployments.

Primiero vamos instalar o **asdf** para podermos controlar todas as versões de pacotes instalados em nossa distribuição linux, permitindo a coexistência de mais de uma versão da mesma ferramenta em nosso sistema operacional.

### **Instalando o asdf**

Abra o terminal na pasta raiz do seu usuário e clone o repositório do asdf

```zsh
git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.9.0
```

Abra e edite o arquivo `.bashrc` como permissão de administrador utilizando o editor `nano`

```zsh
sudo nano ~/.bashrc
```

Adicione a instrução abaixo na última linha do arquivo, salve e feche

```zsh
. $HOME/.asdf/asdf.sh
```

Execute o comando abaixo para validar a instalação do asdf

```zsh
asdf info
```

Caso não funcione, reinicie seu terminal e tente novamente.

### **Instalando o k3d e kubectl**

Após a instalação do asdf, abra o terminal e execute a lista de comandos abaixo:

**k3d:**
```zsh
asdf plugin-add k3d && asdf install k3d latest && asdf global k3d latest
```

**kubectl:**
```zsh
asdf plugin-add kubectl && asdf install kubectl latest && asdf global kubectl latest
```

Estes comandos combinados irão realizar a **adição do plugin** k3d/kubectl ao controlador asdf, **intalação** no sistema operacional e definição do escopo de utilizaçõa da **última versão como Global**

Para validar a ação do asdf, utilize o comando descrito abaixo

```zsh
asdf list
```

ou

```zsh
asdf current
``` 
Deverá retornar a lista de ferramentas instaladas e suas versões.

## Iniciar o Docker na estação

Verifique o status do docker para saber se ele já está rodando na máquina.

```zsh
sudo service docker status
```

Caso não esteja rodando, inicie o docker conforme abaixo:

```zsh
sudo service docker start
```

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

## Como criar o cluster kubernetes

Abra o terminal e utilize o k3d para criar o cluster kubernetes como descrito abaixo

```zsh
k3d cluster create meucluster --agents 1 --servers 1 -p "8080:30000@loadbalancer"
```
**Importante:**

A string **_`meucluster`_** representa o nome do seu cluster kubernetes, portando você pode utilizar o nome que melhor se adequar a sua realidade.

Os parâmentros **`_agents_`** e **`_servers_`** podem ser dimencionados conforme sua necessidade e poder computacional.

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