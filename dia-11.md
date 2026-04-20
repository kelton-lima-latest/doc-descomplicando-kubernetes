# aula 02
ingress - expõe o serviço para fora. consegue ter vários controles, ssl, rota (regra de roteamento), balanceamento de carga, configuração de regras para balanceamento. liberdade para escolher controlador (nginx infress controller, ha proxy, traefik)

cuidado ao escolher ingress - validação em diferentes cenários, boa documentação

possível limitação de requisições por ip

# aula 03
configurando kind

# aula 04
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.15.1/deploy/static/provider/baremetal/deploy.yaml
