# django-2do: atividade prática sobre kubernetes

Atividade desenvolvida na disciplina de virtualização do curso superior de tecnologia em redes de computadores do IFPB (2023).

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
