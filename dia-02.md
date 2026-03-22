# Anotações: Introdução ao Dia 02 - Descomplicando Kubernetes

## Foco Principal
- O segundo dia é inteiramente dedicado ao **Pod**, a menor unidade dentro do Kubernetes [1].
- O objetivo é dominar como os containers "ganham vida" e operam dentro do cluster [1].

## Tópicos Abordados no Dia
- **Criação de Pods**: Aprendizado prático via linha de comando e através de manifestos (YAML) [1].
- **Parâmetros de Configuração**: Exploração de opções e detalhes técnicos na definição do Pod [1].
- **Multi-container**: Como configurar e gerenciar Pods que rodam mais de um container simultaneamente [1].
- **Especialização**: O conteúdo visa capacitar o aluno a conhecer exatamente tudo o que é necessário sobre a unidade fundamental do cluster [1].


# Anotações de Kubernetes - Aula 02: O que é um Pod

## Definição Fundamental
- **Menor Unidade**: O Pod é a menor unidade lógica e a menor parte com a qual você interage no Kubernetes; você não gerencia contêineres diretamente, mas sim Pods [1].
- **Agrupamento**: É um isolamento virtual onde um ou mais contêineres são agrupados para compartilhar recursos [2, 3].

## Compartilhamento de Recursos e Isolamento
- **Namespaces do Kernel**: Contêineres no mesmo Pod compartilham os mesmos namespaces do Kernel (não confundir com Namespaces do Kubernetes) [3, 4].
- **Rede Única**: Todos os contêineres dentro de um Pod compartilham a **mesma interface de rede** e o mesmo endereço IP [4, 5].
- **Comunicação Interna**: A comunicação entre contêineres do mesmo Pod é feita via **localhost** [5].
- **Comunicação Externa**: A comunicação entre Pods diferentes ocorre através de seus endereços IPs individuais [5].

## Arquitetura e Boas Práticas
- **Mesmo Nó (Node)**: Contêineres de um mesmo Pod **sempre** serão executados no mesmo nó; nunca serão distribuídos entre nós diferentes [6].
- **Flexibilidade**: O uso de múltiplos contêineres (como o padrão **Sidecar**) oferece flexibilidade para tratar a aplicação, como contêineres que monitoram logs ou preparam o ambiente [2, 3].
- **Init Containers**: Existe a possibilidade de usar contêineres de inicialização para preparar algo antes da aplicação principal subir [4].
- **O que evitar**: Não se deve colocar toda a infraestrutura (App, Banco de Dados, Redis) em um único Pod. Se o Pod falhar, toda a aplicação morre de uma vez [6].
 

# Anotações de Kubernetes - Aula 03

## Atalhos e Visualização Básica
- `alias k=kubectl`: Atalho para agilizar a linha de comando [1].
- `k get pods`: Lista pods do namespace `default` por padrão [2].
- `k get pods -A` ou `k get pods --all-namespaces`: Lista todos os pods de todos os namespaces do cluster [3].
- `k get pods -n [namespace]`: Lista pods de um namespace específico (ex: `kube-system` para pods administrativos) [2, 4].

## Formatos de Saída e Monitoramento
- `k get pods -o wide`: Adiciona colunas extras mostrando o **IP do Pod** e o **Node** onde ele está rodando [5].
- `k get pods -o yaml`: Exibe a definição completa do Pod. É útil para entender todos os parâmetros que o Kubernetes usa para manter o Pod ativo e pode servir de base para criar novos manifestos [6, 7].
- `k get pods -w`: Ativa o modo **watch** (observar), que mantém a lista aberta e atualiza em tempo real sempre que um status mudar [5].
- `k get pods -L [nome-label]`: Cria uma coluna personalizada para exibir o valor de uma label específica diretamente na listagem [8].

## Mergulho Profundo com Describe
- `k describe pod [nome-pod]`: Comando essencial para entender detalhes técnicos e depurar erros [4, 9].
- **Pontos cruciais para observar no Describe:**
  - **Labels e Annotations**: Metadados usados para organizar e tomar decisões no cluster [10, 11].
  - **Status e IPs**: Estado atual e endereço na rede interna [11].
  - **Containers**: Imagem utilizada, ID do container e porta configurada [11].
  - **Volumes**: Discos ou tokens montados no Pod [12, 13].
  - **Events**: Seção mais importante para **troubleshooting**, listando cronologicamente tudo o que aconteceu com o Pod (criação, pull de imagem, erros) [12, 14].

## Ajuda Rápida
- `kubectl get pods --help`: Exibe uma lista de parâmetros e exemplos práticos para o comando de listagem [9].

# Anotações de Kubernetes - Aula 04

## Comandos Básicos de Visualização e Gerenciamento
- `kubectl get pods nginx -o yaml`: Lista a definição de criação do Pod no formato YAML.
- `kubectl get pods -o wide`: Retorna detalhes adicionais como o IP do Pod e em qual Node ele está rodando [1, 2].
- `kubectl get pods -L nome-label`: Retorna as labels dos Pods.
- `kubectl --help`: Exibe descrição de comandos e exemplos [3].
- `kubectl delete pods [nome-pod]`: Remove um Pod do cluster [4, 5].

## Criação e Execução
- `kubectl run [nome-pod] --image [imagem] --port [porta]`: Cria um Pod. É recomendável passar a porta (ex: `--port 80`) para serviços como Nginx [6, 7].
- `kubectl run [nome-pod] -it --image [imagem]`: Necessário para imagens como `busybox` ou `alpine` que não possuem um serviço persistente (como um web server) rodando em primeiro plano; sem o modo interativo, o container encerra logo após subir [8-10].

## Interatividade: Attach vs. Exec
- **kubectl attach [nome-pod] -c [container] -i -t**: Anexa você ao **processo principal (PID 1)** que já está em execução. No caso do Nginx, você verá o processo do servidor e não um shell [11-13].
- **kubectl exec -it [nome-pod] -- [comando]**: Cria um **novo processo** dentro do container. É a forma ideal para abrir um shell (ex: `bash`) sem interferir no serviço principal [13, 14].
- **Comando Direto**: Você pode usar o `exec` para rodar comandos sem entrar no container, como `kubectl exec [pod] -- cat /etc/hostname` [15, 16].

## Conceitos Importantes de Rede e Ciclo de Vida
- **Acesso ao IP**: O IP do Pod é acessível apenas **dentro do cluster**. Tentativas de acesso externo (como um `curl` da sua máquina host) falharão [2, 17, 18].
- **Efemeridade**: Alterações feitas diretamente no sistema de arquivos do container (instalação de pacotes ou edição de arquivos) são **perdidas** se o Pod for reiniciado, a menos que haja volumes configurados [19-21].
- **Reinicialização**: Pela política padrão (`Restart Always`), se o processo principal de um container parar, o Kubernetes tentará subi-lo novamente [19, 22].

# Anotações de Kubernetes - Aula 05: Manifestos e Multicontêiner

## Geração de Manifestos (Dry Run)
- `kubectl run [nome] --image [imagem] --dry-run=client -o yaml > pod.yaml`: O `--dry-run=client` apenas simula a criação (fingindo que fez, mas não faz no cluster) [1, 2]. Combinado com `-o yaml`, é a melhor forma de gerar um "template" de manifesto rapidamente [2].

## Anatomia do Manifesto YAML
- **apiVersion**: Define a versão da API (ex: `v1` para objetos estáveis). É crucial saber identificar se a versão mudou para evitar falhas no deploy [3, 4].
- **Kind**: O tipo de objeto (ex: `Pod`) [4].
- **Metadata**: Informações *sobre* o Pod (nome, labels, namespace). Não define o funcionamento técnico, mas sim a identificação [4, 5].
- **Spec**: Especificações técnicas de *como* o Pod deve funcionar (contêineres, volumes, portas) [6].
- **Identação**: O YAML exige **2 espaços**. Um erro comum é usar o `-` (lista) de forma desalinhada, o que invalida o arquivo [5, 7].

## Gerenciamento: Create vs Apply
- `kubectl create -f pod.yaml`: Só funciona se o recurso ainda não existir [8, 9].
- `kubectl apply -f pod.yaml`: Cria se não existir e **atualiza** se já existir [8]. 
- **Limitação Crítica**: O `apply` **não permite adicionar ou remover contêineres** de um Pod em execução. Nesses casos, é obrigatório deletar o Pod e criá-lo novamente [10, 11].

## Logs e Troubleshooting
- `kubectl logs [nome-pod]`: Exibe a saída do contêiner [12].
- `kubectl logs [nome-pod] -c [nome-container]`: Obrigatório em Pods multicontêineres para especificar qual log você deseja ver [13].
- `kubectl logs -f [nome-pod]`: Modo "follow", mostra os logs em tempo real conforme acontecem [13].

## Conceitos de Pods Multicontêiner
- **Compartilhamento de Stack de Rede**: Contêineres no mesmo Pod compartilham o mesmo IP e portas. **Erro comum**: Tentar subir dois servidores web (ex: Nginx e Apache) na porta 80 dentro do mesmo Pod causará falha em um deles [13, 14].
- **Argumentos e Persistência**: Imagens de sistemas puros (Busybox, Ubuntu) "morrem" se não tiverem um processo ativo. Use a seção `args` no YAML para manter o contêiner vivo (ex: `args: ["sleep", "3600"]`) [15, 16].

# Anotações de Kubernetes - Aula 06: Limites de Recursos

## Importância da Gestão de Recursos
- **Segurança e Estabilidade**: Definir limites evita que uma aplicação com erro ("bug") consuma toda a memória do cluster e prejudique outros Pods [1].
- **Dimensionamento**: É o que permite saber o tamanho exato do cluster necessário para suportar suas aplicações [2].

## Configuração no YAML (Bloco `resources`)
- **Requests (Solicitações)**: É o recurso que o Kubernetes **reserva e garante** para o container assim que ele inicia [3, 4].
- **Limits (Limites)**: É o teto **máximo** que o container tem permissão para utilizar [3, 5].
- **Escopo**: A definição de recursos é feita individualmente **por container** e não para o Pod de forma global [6].

## Unidades de Medida
- **CPU**: Pode ser declarada em frações (ex: `0.3` representa 30% de uma CPU) ou em milicores (ex: `300m`) [4, 7].
- **Memória**: Geralmente definida em Megabytes (ex: `128Mi` ou `64Mi`) [8, 9].

## Comportamento e Testes Práticos
- **Comando Describe**: Use `kubectl describe pod` para validar os limites aplicados; o Kubernetes converte automaticamente valores decimais para milicores (ex: 0.5 vira 500m) [4].
- **Estouro de Memória**: Diferente da CPU, se um container tentar usar mais memória que o definido no `limits`, o processo será interrompido ("Fatal") e o container pode sofrer restart [10].
- **Monitoramento Interno**: Comandos como `free -m` dentro do container mostram a memória total do **Node (nó)** e não o limite imposto ao container, o que pode ser enganoso [11].
- **Ferramenta de Stress**: O utilitário `stress` é usado para simular carga e validar se o Kubernetes está limitando corretamente o consumo de CPU e RAM conforme configurado [12, 13].


# Anotações de Kubernetes - Aula 07: Volumes e o tipo EmptyDir

## Conceito de Volumes
- **Persistência Básica**: Sem volumes, qualquer dado criado em um container é perdido se ele for reiniciado ou removido [1, 2].
- **Função**: Um volume é um ponto de montagem injetado no container que aponta para um local fora do sistema de arquivos efêmero [3, 4].

## O Volume tipo EmptyDir
- **Definição**: Inicia como um diretório vazio para armazenar informações do Pod [5, 6].
- **Compartilhamento e Colaboração**: Todos os contêineres no mesmo Pod acessam o mesmo volume, permitindo que troquem arquivos em tempo real (ex: um sidecar processando logs da aplicação principal) [4, 7, 8].
- **Ciclo de Vida**: 
    - Os dados **sobrevivem a restarts** do container [2].
    - Os dados **são excluídos permanentemente** se o Pod for removido ou movido de nó [2, 9].

## Configuração no YAML
- **volumeMounts**: Define o caminho (mountPath) e o nome do volume dentro do container [10, 11].
- **volumes**: Define o volume no nível do Pod [12, 13].
- **Limites**: O parâmetro `sizeLimit` é essencial para evitar que um volume temporário consuma todo o disco do nó host [6, 14].

## Verificação e Troubleshooting
- **kubectl describe**: Valida se o volume foi montado corretamente e seu status (RW - Read/Write por padrão) [15].
- **Comando mount**: Dentro do container (via `exec`), o comando `mount` mostra qual dispositivo está servindo aquele diretório [15, 16].
