# aula 02
Quando pods são excluídos, dados não ficam persistentes

Volumes - faz com que dados sejam persistentes

emptydir - volumes efêmeros, que desaparecem quando o pods morre

volumes persistentes - armazena as informações dos pods, mesmo que o pod deixe de existir

# aula 03
storageclass - gerenciamento dos persistent volumes (pvs). forma de automatizar criação de pvs. categorizar volumes
exemplos - utilização por tipos de discos: nfs, discos lentos, s3 (aws)

pvc - solicitação de disco para aplicação (construtor da aplicação)

# aula 04
kubectl get storageclass

sempre verificar quem é storageclass padrão. quando não especificado, vai para padrão

# aula 05
pv - bloco onde se deseja armazenar informação
pvc - solicitação de armazenamento

tipos de pv:
host path - armazenamento local. não utilizar em ambiente de produção
armazenamento em rede - nfs, iscsi, armazenamento em nuvem

kubectl get persistentvolume

accessmode - modo de acesso aos discos, disponível para vários apps, um pod por vez
ReadWriteOnde - disponível para letura e escrita somente por um único nó

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  labels:
    stogare: lento
  name: meu-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: /mnt/data
  storageClassName: giropops
```
