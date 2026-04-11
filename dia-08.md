# aula 02

secret - serve para guardar informações, tokens, certificados, qualquer informação que queira utilizar/montar dentro do kubernetes. por padrão essas informações não são criptografadas no etcd. informação armazenada por um secret é base64. utilizar rbac para definir quem pode acessar as secrets. proteger o pod, pois a secret pode ser vista dentro do pod.

é comum criar secret utilizando linha de comando

opaque - secrets mais simples, definido pelo usuário

dockercfg e dockerconfigjson - utilizado para guardar configurações do docker. uteis pra guardar credenciais de acesso ao docker registry

basicauth - guardar certificados tls, chave ssh

ssh-auth - credenciais ssh

tls - certificados tls

token - inicialização do cluster

kubectl get secrets -n namespace nome-secret -o yaml - retorna informações dos secrets em base64

echo -n "secret-value" | base64 -d - decripta o secret

# aula 04
kubectl create secret generic giropops --from-literal=username=dXNlcm5hbWUK --from-literal=password=cGFzc3dvcmQK - cria secret opaque via linha de comando

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: giropops-secret
type: opaque
data:
  username: dXNlcm5hbWUK
  password: cGFzc3dvcmQK
```

# aula 05

utilizando secrets nos pods
```bash
apiVersion: v1
kind: Pod
metadata:
  name: giropops-pod
spec:
  restartPolicy: Never
  containers:
  - name: giropops-pod
    image: ubuntu
    command: ["/bin/bash", "-c"]
    args: 
      - "echo 'sleep 60'"
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: giropops
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: giropops
          key: password
```

set - lista todas as variáveis de ambiente  
kubectl exec -it nome-pod -c nome-container -- env - lista todas as variáveis de ambiente
