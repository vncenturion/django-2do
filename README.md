# django-2do
atividade prática sobre kubernetes

![tela da aplicação](tela.png)

## pré-requisitos

## instalando kubectl e minikube

## projeto

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
