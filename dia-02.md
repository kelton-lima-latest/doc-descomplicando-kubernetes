O que é um POD? Menor unidade do kubernetes

Um POD pode conter mais de um container.  
Agrupamento de containers que compartilham mesma rede, namespace(kernel).  
Normal ter um POD com mais de um container? Sim, é normal quando se fala de sidecar.  
Dois containers que funcionam no mesmo POD vão ser executados no mesmo nó.  
Cada container dentro de um POD tem uma função específica.  
A comunicação entre os containers no mesmo POD vai ser local. Os dois containers vão ter apenas um endereço de rede.  
A comunicação entre dois PODs vai ser via IP.  

alias k=kubectl  

k get pods - lista pods do namespace default  
k get pods -A - lista todos os pods do cluster  
k get pods -n kube-system  

Name - Nome do POD  
Ready - Quantidade em execução/Quantidade desejada  
Status -  
Restarts -  
Age - Tempo de execução  

| NAME                                   | READY | STATUS  | RESTARTS | AGE   |
|----------------------------------------|-------|---------|----------|-------|
| cilium-envoy-7q78z                     | 1/1   | Running | 0        | 117s  |
| cilium-envoy-f5hqg                     | 1/1   | Running | 0        | 4m39s |
| cilium-operator-54d6dc4c4f-8f762       | 1/1   | Running | 0        | 4m39s |
| cilium-rjvnl                           | 1/1   | Running | 0        | 4m39s |
| cilium-zq78l                           | 1/1   | Running | 0        | 117s  |
| coredns-66bc5c9577-4mvww               | 1/1   | Running | 0        | 5m30s |
| coredns-66bc5c9577-ptbgz               | 1/1   | Running | 0        | 5m30s |
| etcd-control-plane                     | 1/1   | Running | 0        | 5m37s |
| kube-apiserver-control-plane           | 1/1   | Running | 0        | 5m37s |
| kube-controller-manager-control-plane  | 1/1   | Running | 0        | 5m37s |
| kube-proxy-jkxpq                       | 1/1   | Running | 0        | 5m30s |
| kube-proxy-tnlbr                       | 1/1   | Running | 0        | 117s  |
| kube-scheduler-control-plane           | 1/1   | Running | 0        | 5m37s |

kubectl describe pods nome-pod - Descreve detalhes do POD  
Detalhes mais importantes do kubectl describe
Name  
Namespace  
Node  
Labels  
Status  
Containers  
Events  

kubectl get pods nginx -o yaml - Lista na tela definição de criação do POD no formato yaml  
kubectl get pods -o wide - Retorna mais detalhes sobre o POD, ip e node.  
kubectl get pods -L nome-label - Retorna labels 

kubectl --help - Descrição de comandos e exemplos
