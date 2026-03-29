# aula 02
Deployment é obrigatório para alta disponibiidade
Possível criar réplicas com deployment
Quando deployment é criado, automaticamente é criado replicaset
Replicaset garante a quantidade de réplicas de um deployment
Deployment garante a saúde dos pods
não subir aplicação utilizando pod. pod é para teste, não para produção
deployment garante que os pods estejam espalhados pelo cluster
deployment garante estratégias de atualização, para aplicação manter disponibilidade

# aula 03
utilizar .yaml na criação de deployment para persistir as configurações que estão sendo executadas no cluster, guardar informações no github e ter pipelines
selector é muito importante - deployment sabe quais pods tem que controlar
matchLabels - informa ao kubernetes quais pods vão ser gerenciados

kubectl apply -f nome-deployment.yaml - cria ou atualiza deployments
kubectl get deployments - lista deployments
kubectl get replicaset

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector: # deployment sabe quais pods tem que controlar
    matchLabels: # informa ao kubernetes quais pods vão ser gerenciados
      app: nginx-deployment # labels do pod que vai ser gerenciado
  strategy: {}
  template: # template para criação dos containers qu fazem parte do deployment
    metadata:
      labels:
        app: nginx-deployment
    spec: # especificação dos containers
      containers:
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: "256Mi"
          requests:
            cpu: "0.3"
            memory: "128Mi"
```

# aula 04
kubectl describe deployment nome-deployment
sempre que há alteração no deploymento, é criado um novo replicaset
kubectl delete -f nome-deployment.yaml
kubectl create deployment nome-deployment --image nginx --port 80 --replicas 3 --dry-run=client -o yaml > nome-deployment.yaml
kubectl get deployment nome-deployment -o yaml

# aula 05
kubectl create namespace none-namespace
kubectl delete namespace nome-namespace

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources:
          limits:
            cpu: "1.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"

```

# aula 06
estratégias de update
rollingupdate - padrão, caso não tenha informado strategy. possível indicar características de deploy, informando quantidade de pods a serem criados e removidos durante o deploy. Faz com que a aplicação não tenha indisponibilidade

MaxSurge - quantidade de pods que podem ultrapassar limite de réplicas informadas. possível utilizar %
MaxUnavailable - quantidade de pods indisponíveis durante o update. possível utilizar %

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.17.0
        name: nginx
        resources:
          limits:
            cpu: "1.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"
```

kubectl rollout status deployment -n nome-namespace nome-deployment - mostra status do rollout

rollout - faz atualização
rollback - desfaz atualização

estratégia de deploy recriate - destroy todos os pods. aplicação fica fora do ar. utilizar quando há grandes updates na aplicação, onde versões diferentes não funcionam bem

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
  namespace: nginx
spec:
  replicas: 10
  selector:
    matchLabels:
      app: nginx-deployment
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.15.0
        name: nginx
        resources:
          limits:
            cpu: "1.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"
```

# aula 07
kubectl rollout undo deployment nome-deployment - faz rollback da aplicação, funciona com daemonset, deployment e statefulset

kubectl rollout history deployment -n nome-namespace nome-deployment

kubectl rollout history deployment -n nome-namespace nome-deployment --revision numero-revision

interessante utilizar annotations para identificar causa no history

kubectl rollout history deployment -n nome-namespace nome-deployment --to-revision numero revision

kubectl rollout pause deployment -n nome-namespace nome-deployment -  não deixa que tenha mais atualizações

kubectl rollout resume deployment -n nome-namespace nome-deployment - volta a ter atualizações

ideial deixar pods críticos com pause para não ter problema de atualizações

kubectl rollout restart deployment -n nome-namespace nome-deployment - mata todos o deployment e cria novamente

kubectl scale deployment -n nome-namespace --replicas numero-replicas nome-deployment
