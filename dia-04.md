# aula 02
replicaset - principal função é garantir que as réplicas de pods estão no número desejado. criado automaticamente junto da criação do deployment. Não é recomendado criar replicaset na mão
cada vez que o deployment é atualizado, é criado novo replicaset. novo deployment identifica replicaset atual e replicaset anterior. Com isso o rollback para versão anterior fica mais simples, pois o deployment anterior já vai apontar para o replicaset anterior

# aula 03
kubectl get replicaset - lista replicaset em execução

kubectl describe replicaset nome-replicaset - retorna informações do replicaset

pods criados via deployment segue a seguinte nomenclatura - nome-deployment:nomereplicaset:nome-pod

kubectl rollout undo deploy nome-deployment - efetuado rollback para a ultima versão do deploy. retorna também para ultimo replicaset, pois essa informação é guardada no describe do deployment

# aula 04
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx-app
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx
        resources:
          requests:
            cpu: 0.3
            memory: 128Mi
          limits:
            cpu: 0.5
            memory: 256Mi
```

alterações no pod criado a partir de um replicaset sofre alteração após o pod ser deletado. logo é criado pod pelo  replicaset. Criar um deployment é a forma mais segura e efetiva de criar replicaset.

# aula 05
daemonset - isolado, não faz parte da estrtura do deployment e replicaset. Garante que uma rélpicas vai rodar em cada nó. Utilizado para ferramentas de segurança e monitoramento.

# aula 06
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  labels:
    app: node-exporter-daemonset
  name: node-exporter-daemonset
spec:
  selector:
    matchLabels:
      app: node-exporter-daemonset
  template:
    metadata:
      labels:
        app: node-exporter-daemonset
    spec:
      hostNetwork: true # utiliza rede do nó onde está sendo executado o pod
      containers:
      - name: node-exporter-daemonset
        image: prom/node-exporter:latest
        resources:
          requests:
            cpu: 0.3
            memory: 128Mi
          limits:
            cpu: 0.5
            memory: 256Mi
        ports:
        - containerPort: 9100
          hostPort: 9100
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

# aula 07
não é possível criar replicaset e daemonset via comando kubectl create

# aula 08
probes - muito importante para deployments em produção. Modos de verificação para saber se os pods/containers estão funcionando relativamente bem

liveness - verifca se aplicação está respondendo, se continua viva

startup - ao criar o deployment é executado teste para saber se ele foi startado

readness - verifica se pod está ok para receber requisição interna

verificações podem ser feitas via protocolos de rede, via portas e comando exec

# aula 09
liveness probe - garante que o que está rodando dentro do pod está saudável. Maneiras de verificar, http, porta ou comando

probe é especificada para cada container dentro do pod

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
#  namespace: nginx
spec:
  replicas: 3
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
      - image: nginx:1.16.0
        name: nginx
        resources:
          limits:
            cpu: "1.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"
        livenessProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 10 # tempo de espera para iniciar primeira verificação
          periodSeconds: 10 # intervalo de tempo de cada verificação
          timeoutSeconds: 5 # tempo de espera para indicar problema na aplicação
          failureThreshold: 3 # quantidade de vezes que aplicação precisa falhar para ser reconhecido com erro. Após aguardar todas as tentativas, o container é reiniciado
```

# aula 10
readiness probe - faz healthckech e verifica se aplicação está pronta para receber requisição. Se o teste falhar o pod é removido da lista de requisições

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
#  namespace: nginx
spec:
  replicas: 3
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
      - image: nginx:1.16.0
        name: nginx
        resources:
          limits:
            cpu: "1.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10 # tempo de espera para iniciar primeira verificação
          periodSeconds: 10 # intervalo de tempo de cada verificação
          timeoutSeconds: 5 # tempo de espera para indicar problema na aplicação
          failureThreshold: 3 # quantidade de vezes que aplicação precisa falhar para ser reconhecido com erro. Após aguardar todas as tentativas, o container é reiniciado
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10 # tempo de espera para iniciar primeira verificação
          periodSeconds: 10 # intervalo de tempo de cada verificação
          timeoutSeconds: 5 # tempo de espera para indicar problema na aplicação
          failureThreshold: 3 # quantidade de vezes que aplicação precisa falhar para ser reconhecido com erro. Após aguardar todas as tentativas, o container é reiniciado
          successThreshold: 1
```

# aula 11

startup probe - Garante que o container foi iniciado com sucesso. Executado uma única vez, no início de vida do container.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
#  namespace: nginx
spec:
  replicas: 3
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
      - image: nginx:1.16.0
        name: nginx
        resources:
          limits:
            cpu: "1.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"
        livenessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10 # tempo de espera para iniciar primeira verificação
          periodSeconds: 10 # intervalo de tempo de cada verificação
          timeoutSeconds: 5 # tempo de espera para indicar problema na aplicação
          failureThreshold: 3 # quantidade de vezes que aplicação precisa falhar para ser reconhecido com erro. Após aguardar todas as tentativas, o container é reiniciado
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10 # tempo de espera para iniciar primeira verificação
          periodSeconds: 10 # intervalo de tempo de cada verificação
          timeoutSeconds: 5 # tempo de espera para indicar problema na aplicação
          failureThreshold: 3 # quantidade de vezes que aplicação precisa falhar para ser reconhecido com erro. Após aguardar todas as tentativas, o container é reiniciado
          successThreshold: 1
        startupProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 10 # tempo de espera para iniciar primeira verificação
```











