# aula 02
stateless - sem estado. deployment e replicaset

satatefulset - com estado. necessário garantias para aplicação. ligação com persistent volumes. quando se cria um statefulset é criado um pv para cada pod. sempre que o pod morre, o novo pod utiliza o pv que já existe. muito utilizado com banco de dados, monitoração, mensageria... utilizado quando se quer endereçamento de rede estável, orderm de roolout e rollback, armazenamento...

headless service - muito utilizado com statefulset. omite o endereçamento ip, utiliza estratégia com cluster ip

# aula 06
services - maneira de fazer com que os pods sejam expostos para outros serviços ou para usuários fora do cluster. service utiliza labels para identificar qual pod redirecionar a requisição.

endpoints - criado após criação dos services. ip que leva serviço até o pod que está em execução

cluster ip - ganha ip local e vale somente dentro do cluster

node port - grampea porta do node e redireciona para o pod. utiliza NAT

load balancer - interessante quando se utiliza kubernetes gerenciado. criado automaticamente quando utilizado em cloud provider. metallb é utilizado em cluster onpremises

external name - não expõe pods. criaod serviço para serviço externo. app se conecta com banco de dados externo.

# aula 07

kubectl expose - pode utilizar para pod, deployment, replicaset, service e replicationcontroller

kubectl expose  deployment nome-deployment - por padrão é criado service node port

kubectl get svc - lista serviços no cluster

kubectl expose deployment nome-deployment --type NodePort

kubectl get endpoints
