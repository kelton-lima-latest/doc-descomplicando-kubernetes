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
