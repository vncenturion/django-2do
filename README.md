# django-2do: atividade prática sobre kubernetes

Atividade desenvolvida na disciplina de virtualização do curso superior de tecnologia em redes de computadores do IFPB (2023).

A prática foi realizada em máquina virtual ubuntu 22.04 LTS, 4Gb ram, 2 CPU, 100GB SSD, sobre VMWARE-WORKSTATION, v. 17PRO em hospedeiro WINDOWS 11 home.

## Instruções

Faça o deployment em sistema de orquestração de conteineres local (minikube) da aplicação "django-todo", no repositório https://github.com/diegoep/django-todo.

Essa é uma aplicação Python/Django que roda um banco de dados Sqlite embarcado na própria aplicação.

![tela da aplicação](tela.png)

Na atividade é solicitado ao aluno que:

 1. Faça a construção da imagem docker da aplicação (lembre de rodar "eval $(minikube docker-env)" para usar o docker do ambiente do minikube;
    
 1. Crie os manifestos Kubernetes para implantar a aplicação django-todo e externalizar o acesso: Deployment, service, ingress;
    
 1. Pesquise e indique como seria possível manter estado em múltiplas execuções, visto que a aplicação armazena os dados em um sqlite rodando no próprio container.
    


## pré-requisitos

 1. Git;
 2. Docker;
 3. Virtualbox;
 4. Habilitar virtualização aninhada (quando utilizando máquina virtual)

## instalando kubectl e minikube

## projeto

  1. Clone o repositório
     ```console
        git clone https://github.com/diegoep/django-todo
        cd django-todo
     ```

  1. Ative o ambiente do minikube
     ```console
        eval $(minikube docker-env)
     ```

  1. Construa a imagem Docker
     ```console
        docker build -t django-todo:latest .
     ```

## elaboração dos manifestos

  1. deployment.yaml
     ```yaml
        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: django-todo-deployment
        spec:
          replicas: 1
          selector:
            matchLabels:
              app: django-todo
          template:
            metadata:
              labels:
                app: django-todo
            spec:
              containers:
              - name: django-todo
                image: django-todo:1.0  # criar com latest nao funciona inicialmente
                ports:
                - containerPort: 8000
     ```
  1. service.yaml
     ```yaml
        apiVersion: v1
        kind: Service
        metadata:
          name: django-todo-service
        spec:
          selector:
            app: django-todo
          ports:
            - protocol: TCP
              port: 80
              targetPort: 8000
          type: NodePort  # teste para expor porta externamente
     ```
     
  1. ingress.yaml
     ```yaml
        apiVersion: networking.k8s.io/v1
        kind: Ingress
        metadata:
          name: django-todo-ingress
        spec:
          rules:
          - host: django-todo.local
            http:
              paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: django-todo-service
                    port:
                      number: 80
     ```
     

## deployment

Iniciar deployment:
```console
   kubectl apply -f deployment.yaml
   kubectl apply -f service.yaml
   kubectl apply -f ingress.yaml
```

Interromper deployment:
```console
   kubectl delete deployment django-todo-deployment
   kubectl delete service django-todo-service
   kubectl delete ingress django-todo-ingress
```

## VOLUMES PERSISTENTES em Kubernetes

O gerenciamento de armazenamento é uma questão bem diferente do gerenciamento de instâncias computacionais. O subsistema PersistentVolume provê uma API para usuários e administradores que mostra de forma detalhada de como o armazenamento é provido e como ele é consumido. Para isso, nós introduzimos duas novas APIs: PersistentVolume e PersistentVolumeClaim.

Um PersistentVolume (PV) é uma parte do armazenamento dentro do cluster que tenha sido provisionada por um administrador, ou dinamicamente utilizando Classes de Armazenamento. Isso é um recurso dentro do cluster da mesma forma que um nó também é. PVs são plugins de volume da mesma forma que Volumes, porém eles têm um ciclo de vida independente de qualquer Pod que utilize um PV. Essa API tem por objetivo mostrar os detalhes da implementação do armazenamento, seja ele NFS, iSCSI, ou um armazenamento específico de um provedor de cloud pública.

Uma PersistentVolumeClaim (PVC) é uma requisição para armazenamento por um usuário. É similar a um Pod. Pods utilizam recursos do nó e PVCs utilizam recursos do PV. Pods podem solicitar níveis específicos de recursos (CPU e Memória). Claims podem solicitar tamanho e modos de acesso específicos (exemplo: montagem como ReadWriteOnce, ReadOnlyMany ou ReadWriteMany, veja Modos de Acesso).

Enquanto as PVC - PersistentVolumeClaims permitem que um usuário utilize recursos de armazenamento de forma limitada, é comum que usuários precisem de PersistentVolumes com diversas propriedades, como desempenho, para problemas diversos. Os administradores de cluster precisam estar aptos a oferecer uma variedade de PersistentVolumes que difiram em tamanho e modo de acesso, sem expor os usuários a detalhes de como esses volumes são implementados. Para necessidades como essas, temos o recurso de StorageClass.

### Provisionamento

Existem duas formas de provisionar um PV: estaticamente ou dinamicamente.

Estático: O administrador do cluster cria uma determinada quantidade de PVs. Eles possuem todos os detalhes do armazenamento os quais estão atrelados, que neste caso fica disponível para utilização por um usuário dentro do cluster. Eles estão presentes na API do Kubernetes e disponíveis para utilização.

Dinâmico: Quando nenhum dos PVs estáticos, que foram criados anteriormente pelo administrador, satisfazem os critérios de uma PersistentVolumeClaim enviado por um usuário, o cluster pode tentar realizar um provisionamento dinâmico para atender a essa PVC. Esse provisionamento é baseado em StorageClasses: a PVC deve solicitar uma classe de armazenamento e o administrador deve ter previamente criado e configurado essa classe para que o provisionamento dinâmico possa ocorrer. Requisições que solicitam a classe "" efetivamente desabilitam o provisionamento dinâmico para elas mesmas.

Para habilitar o provisionamento de armazenamento dinâmico baseado em classe de armazenamento, o administrador do cluster precisa habilitar o controle de admissão DefaultStorageClass no servidor da API. Isso pode ser feito, por exemplo, garantindo que DefaultStorageClass esteja entre aspas simples, ordenado por uma lista de valores para a flag --enable-admission-plugins, componente do servidor da API. Para mais informações sobre os comandos das flags do servidor da API, consulte a documentação kube-apiserver.

### Utilização

Pods utilizam requisições como volumes. O cluster inspeciona a requisição para encontrar o volume atrelado a ela e monta esse volume para um Pod. Para volumes que suportam múltiplos modos de acesso, o usuário especifica qual o modo desejado quando utiliza essas requisições.

Uma vez que o usuário tem a requisição atrelada a um PV, ele pertence ao usuário pelo tempo que ele precisar. Usuários agendam Pods e acessam seus PVs requisitados através da seção persistentVolumeClaim no bloco volumes do Pod. Para mais detalhes sobre isso, veja Requisições como Volumes.
