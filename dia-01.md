# Introdução aos Containers

## O que é um container?
Em resumo, container é **isolamento de recursos** no nível do sistema operacional. Ao contrário de máquinas virtuais, eles compartilham o mesmo kernel do host, mas mantêm o espaço de usuário separado.

### Quais recursos são isolados?
O isolamento garante que o container tenha sua própria visão de:
* **CPU e Memória** (Hardware)
* **Processos** (PID)
* **I/O e Rede**
* **Tabela de usuários**
* **Pontos de montagem** (Filesystem)

---

## Como isso é possível? (A "Mágica" do Kernel)
O isolamento não é uma ferramenta única, mas a combinação de módulos nativos do Kernel Linux:

| Tecnologia | Função Principal |
| :--- | :--- |
| **Cgroups** (Control Groups) | Responsável por **limitar e monitorar** o uso de recursos (Quanto de CPU/RAM o container pode usar). |
| **Namespaces** | Responsável pelo **isolamento de visão**. Garante que um processo não veja o que o outro está fazendo (Separa usuários, redes, mounts). |

### Visualizando a Arquitetura

```mermaid
graph TD
    Kernel[Kernel Linux do Host]
    
    subgraph Container A
        NS1[Namespace: Isolamento]
        CG1[Cgroup: Limite de Recurso]
    end
    
    subgraph Container B
        NS2[Namespace: Isolamento]
        CG2[Cgroup: Limite de Recurso]
    end

    Kernel --> ContainerA
    Kernel --> ContainerB
```

---

## O que é um Container Engine?

Enquanto o Kernel fornece os recursos (Cgroups/Namespaces), o **Container Engine** é a camada de alto nível que gerencia o ciclo de vida da aplicação. Ele atua como um orquestrador local, preparando todo o terreno para a execução.

### Principais Responsabilidades
O Engine abstrai a complexidade de lidar com os recursos brutos, cuidando de:
* **Ciclo de vida:** Criação e destruição de containers.
* **Healthchecks:** Monitoramento da saúde da aplicação.
* **Volumes:** Persistência de dados e montagem de sistemas de arquivos.
* **Rede:** Criação de interfaces virtuais e pontes de comunicação.

> [!IMPORTANT]
> **Camada de Abstração**
> O Container Engine **não conversa diretamente com o Kernel**.
> Ele delega a execução de baixo nível para um *Container Runtime* (como o containerd ou runc). O Engine foca na experiência de uso e na API, não na chamada de sistema (syscall).

### Fluxo de Comunicação

```mermaid
flowchart LR
    User[Usuário / CLI] -->|Comando| Engine[Container Engine]
    Engine -->|Solicita criação| Runtime[Runtime]
    
    subgraph Host[Sistema Operacional]
        Runtime -->|Syscalls| Kernel
        Kernel -->|Isolamento| Container
    end

    style Engine fill:#f9f,stroke:#333,stroke-width:2px
    style Kernel fill:#bbf,stroke:#333,stroke-width:2px
```

---

## O que é um Container Runtime?

O **Container Runtime** é o componente responsável por executar os containers de fato. Ele atua como o intermediário que "conversa" com o Kernel para criar o isolamento e garantir que os processos estejam rodando.

### Classificação dos Runtimes

O ecossistema de containers divide os runtimes em três categorias principais, baseadas na proximidade com o Kernel:

| Tipo | Descrição | Exemplos |
| :--- | :--- | :--- |
| **High Level** | Executado pelo Container Engine. Gerencia o ciclo de vida, imagens e transfere a execução para o Low Level. **Não conversa diretamente com o Kernel.** | `containerd`, `CRI-O` |
| **Low Level** | O "operário". Conversa **diretamente com o Kernel** para configurar *Namespaces* e *Cgroups*. Ele cria o container e depois sai de cena. | `runc` (Padrão de mercado) |
| **Sandbox** | Focado em segurança extrema. Cria um Kernel "falso" ou uma micro-VM para cada container, garantindo isolamento total do Host. | `gvisor`, `Kata Containers` |

> ** Nota:** Na maioria dos setups modernos (como Kubernetes), o `containerd` (High Level) chama o `runc` (Low Level) para criar o container.

### A Camada de Execução Completa

Visualizando onde cada peça se encaixa:

```mermaid
flowchart TD
    Engine["Container Engine (Docker / Podman)"] -->|Chama| High["High Level Runtime (containerd)"]
    High -->|Gerencia imagem e rede| High
    High -->|Solicita criação| Low["Low Level Runtime (runc)"]

    subgraph Kernel_Space["Kernel Space"]
        Low -->|Syscalls| NS["Namespaces & Cgroups"]
    end

    %% Estilos (pensados para GitHub dark theme)
    style Engine fill:#1f2937,stroke:#60a5fa,stroke-width:2px,color:#e5e7eb
    style High fill:#1f2937,stroke:#34d399,stroke-width:2px,color:#e5e7eb
    style Low fill:#3f1d2e,stroke:#f97316,stroke-width:2px,color:#f9fafb
    style NS fill:#172554,stroke:#818cf8,stroke-width:2px,color:#e5e7eb
```

---

## OCI - Open Container Initiative

A **OCI** é um projeto de estrutura de governança aberta, criado para padronizar o ecossistema de containers e evitar o *vendor lock-in* (dependência de um único fornecedor).

### Fundação e Membros
Foi fundada em 2015 por um consórcio de gigantes da tecnologia que decidiram unificar a forma como containers são criados e executados.
* **Membros Fundadores:** Docker, CoreOS, Google, IBM, Microsoft, Red Hat, VMware, entre outros.

### O Propósito (Padronização)
Antes da OCI, cada ferramenta tinha seu formato. A OCI criou especificações abertas para garantir que um container criado em uma ferramenta rodasse em qualquer outra.

| Especificação | O que define? |
| :--- | :--- |
| **Runtime Spec** | Como um "bundle" de filesystem deve ser descompactado e executado. |
| **Image Spec** | Como arquivar e transferir as imagens de container (o formato das *layers*). |

### O papel do `runc`
A OCI não ficou apenas no papel. Eles desenvolveram o **runc** como a implementação de referência das especificações.
* **Linguagem:** Desenvolvido em **Go (Golang)**.
* **Função:** É o runtime padrão que obedece às regras da OCI. É ele que o Docker e o Kubernetes usam "por baixo do capô" para falar com o Kernel.

---

## O que é o Kubernetes? (K8s)

O **Kubernetes**, frequentemente abreviado como K8s, é uma plataforma open-source portátil e extensível para o gerenciamento de cargas de trabalho e serviços containerizados. Ele facilita a configuração declarativa e a automação, atuando como um orquestrador que abstrai a infraestrutura subjacente (seja on-premise, nuvem pública ou híbrida).

## Visão Geral e Arquitetura

O Kubernetes opera sob um modelo de **estado desejado**. O operador define como o sistema deve se comportar (ex: "quero 3 réplicas deste serviço rodando"), e o "Control Plane" do Kubernetes trabalha continuamente para garantir que o **estado atual** da infraestrutura corresponda a esse estado desejado.

### Origem e Governança
Desenvolvido originalmente pelo Google entre 2014 e 2015, o projeto é o sucessor espiritual do sistema interno **Borg** (e posteriormente Omega). O Kubernetes foi escrito em **Go (Golang)** e doado para a **Cloud Native Computing Foundation (CNCF)**, onde se tornou o primeiro projeto graduado da organização.

> [!IMPORTANT]
> O Kubernetes não cria os containers diretamente. Ele orquestra a execução deles através de uma interface padronizada chamada **CRI (Container Runtime Interface)**, delegando a execução para runtimes compatíveis como containerd ou CRI-O.

## Funcionalidades Core

A principal função do Kubernetes é o agendamento (scheduling) e a distribuição de containers em um cluster de máquinas.

| Funcionalidade | Descrição Técnica |
| :--- | :--- |
| **Service Discovery & Load Balancing** | O K8s expõe containers usando nomes DNS ou endereços IP próprios. Se o tráfego for alto, ele realiza o balanceamento de carga para manter a estabilidade. |
| **Storage Orchestration** | Permite a montagem automática de sistemas de armazenamento, sejam locais, de provedores de nuvem (AWS EBS, GPD) ou soluções de rede (NFS, iSCSI). |
| **Automated Rollouts/Rollbacks** | Altera o estado atual para o estado desejado de forma controlada. Se algo falhar, o K8s realiza o rollback automático para a versão anterior. |
| **Self-healing** | Reinicia containers que falharam, substitui e reagenda containers quando nós morrem e mata containers que não respondem ao *health check* definido pelo usuário. |
| **Secret & Config Management** | Armazena e gerencia informações sensíveis (senhas, tokens OAuth, chaves SSH) e configurações de ambiente sem reconstruir as imagens dos containers. |



# Arquitetura do Kubernetes

O Kubernetes opera através de uma arquitetura de cluster distribuído, dividida fundamentalmente em duas categorias de componentes: a camada de gerenciamento (**Control Plane**) e a camada de execução de cargas de trabalho (**Workers**).

Esta separação de responsabilidades garante a escalabilidade, a alta disponibilidade e a organização lógica dos recursos do sistema.

## Control Plane

O **Control Plane** é a entidade responsável pelo controle global do cluster. Ele atua como o "cérebro" da operação, orquestrando decisões e mantendo o alinhamento entre o estado desejado e o estado atual do sistema.

Este nó (ou conjunto de nós) hospeda serviços e containers críticos administrativos que não estão relacionados à aplicação do usuário final, mas sim à infraestrutura do Kubernetes.

### Responsabilidades Primárias
Baseado nas definições de arquitetura, o Control Plane possui as seguintes atribuições mandatórias:

* **Gerenciamento de Saúde:** Monitoramento contínuo dos componentes do cluster.
* **Disponibilidade:** Garantia de que os serviços essenciais permaneçam acessíveis.
* **Gestão de Capacidade:** Avaliação e alocação de recursos computacionais.
* **Armazenamento de Estado:** Persistência de todos os dados de configuração e estado do cluster.

> [!WARNING]
> **Restrição de Execução**
> Por padrão, não é permitido executar aplicações de usuários (workloads) nos nós do Control Plane. Esta é uma medida de segurança e estabilidade para garantir que o consumo de recursos das aplicações não impacte os serviços críticos de gerenciamento do cluster.

## Worker Nodes

Os **Workers** são os nós de trabalho onde as aplicações efetivamente residem. É nesta camada que a carga de trabalho do usuário é processada.

Todos os **Pods** e **Containers** de aplicação são agendados e executados nestes nós, sob a supervisão das diretrizes estabelecidas pelo Control Plane.

## Comparativo de Componentes

A tabela abaixo detalha as distinções funcionais entre os dois tipos de nós no cluster.

| Componente | Função Principal | Tipo de Carga de Trabalho | Responsabilidade Crítica |
| :--- | :--- | :--- | :--- |
| **Control Plane** | Administração e Orquestração | Serviços do Sistema (System Containers) | Armazenar estado e garantir saúde do cluster. |
| **Worker** | Execução e Processamento | Aplicações do Usuário (Pods/Containers) | Rodar a aplicação final. |

## Diagrama de Arquitetura

O diagrama abaixo ilustra a relação hierárquica onde o Control Plane gerencia o estado e a saúde, enquanto os Workers executam as aplicações.

```mermaid
flowchart TD
    subgraph CP [Control Plane]
        direction TB
        Services[Serviços de Administração]
        State[Armazenamento de Estado]
        Health[Verificação de Saúde/Capacidade]
    end

    subgraph WN [Camada de Execução]
        W1[Worker Node 1]
        W2[Worker Node 2]
    end

    subgraph App [Cargas de Trabalho]
        Pods1[Pods & Containers]
        Pods2[Pods & Containers]
    end

    CP -->|Gerencia e Monitora| WN
    W1 -->|Hospeda| Pods1
    W2 -->|Hospeda| Pods2

    style CP fill:#2d3436,stroke:#fff,stroke-width:2px,color:#fff
    style WN fill:#dfe6e9,stroke:#2d3436,stroke-width:2px,color:#2d3436
    style Services fill:#0984e3,stroke:#none,color:#fff
    style State fill:#0984e3,stroke:#none,color:#fff
    style Health fill:#0984e3,stroke:#none,color:#fff
```
---
# Componentes do Control Plane

O Control Plane é composto por um conjunto de processos modulares que operam em sinergia para gerenciar o estado global do cluster Kubernetes. Cada componente possui uma responsabilidade específica na orquestração, armazenamento e tomada de decisões.

Abaixo estão detalhados os quatro componentes fundamentais baseados na arquitetura de referência.

## 1. etcd

O **etcd** atua como o banco de dados do cluster. É um armazenamento do tipo chave-valor (key-value store) consistente e de alta disponibilidade.

### Características Técnicas
* **Persistência de Estado:** É o único local onde o estado do cluster é armazenado definitivamente.
* **Topologia:** Pode ser hospedado diretamente em cada nó do Control Plane ou em um cluster externo dedicado.
* **Redundância:** Devido à sua natureza crítica, exige configuração de réplicas para garantir Alta Disponibilidade (HA).

> [!IMPORTANT]
> **Criticidade do Dados**
> Se o etcd for perdido ou corrompido, o estado do cluster é perdido. Backups regulares e arquitetura redundante são mandatórios para este componente.

## 2. kube-apiserver

O **kube-apiserver** é o componente central de comunicação ("hub"). Ele expõe a API do Kubernetes e atua como o "front-end" para o Control Plane.

### Funcionamento
* **Intermediário Exclusivo:** É o **único** componente que possui permissão para se comunicar diretamente com o **etcd**.
* **Centralização:** Coleta o status de todos os componentes do cluster e centraliza os eventos.
* **Fluxo de Dados:** Recebe requisições, valida e persiste as informações no etcd. Nenhum outro componente acessa o banco de dados sem passar pelo API Server.

## 3. kube-scheduler

O **kube-scheduler** é responsável pelo planejamento e agendamento de cargas de trabalho. Ele decide em qual nó (Worker) um novo Pod deve ser executado.

### Critérios de Decisão
* **Capacidade:** Analisa os recursos disponíveis (CPU/Memória) nos nós.
* **Provisionamento:** Além de agendar Pods, auxilia na criação de volumes.
* **Comunicação:** Consulta constantemente o kube-apiserver para identificar Pods que ainda não foram atribuídos a um nó.

## 4. kube-controller-manager

O **kube-controller-manager** é um daemon que incorpora os loops de controle principais do Kubernetes. Sua função é observar o estado atual e trabalhar para atingir o estado desejado.

### Gerenciamento de Controladores
Ele atua como o "gerente" de diversos controladores específicos, incluindo:
* **Node Controller:** Monitora a saúde dos nós.
* **Deployment Controller:** Gerencia a implementação de aplicações.
* **ReplicaSet Controller:** Garante o número correto de réplicas de um Pod.
* **Pod Controller:** Assegura a integridade do ciclo de vida dos Pods.

## Resumo dos Componentes

| Componente | Função Primária | Dependência Crítica |
| :--- | :--- | :--- |
| **etcd** | Armazenamento de Estado (Banco de Dados) | Requer Alta Disponibilidade. |
| **kube-apiserver** | Comunicação e API Gateway | Único com acesso ao etcd. |
| **kube-scheduler** | Decisão de Alocação (Scheduling) | Depende da capacidade dos nós. |
| **kube-controller-manager** | Manutenção do Estado Desejado | Gerencia múltiplos sub-controladores. |

## Fluxo de Comunicação do Control Plane

O diagrama abaixo ilustra a arquitetura centralizada no API Server e o isolamento do etcd.

```mermaid
flowchart TD
    subgraph Data [Camada de Dados]
        ETCD[(etcd)]
    end

    subgraph Core [Núcleo de Comunicação]
        API[kube-apiserver]
    end

    subgraph Logic [Lógica de Negócio]
        SCHED[kube-scheduler]
        CM[kube-controller-manager]
    end

    %% Definição de Fluxos
    API <-->|Leitura e Escrita Exclusiva| ETCD
    SCHED <-->|Consulta Pods/Nós| API
    CM <-->|Monitora Estado| API

    %% Estilização para Dark Mode
    style ETCD fill:#b00,stroke:#fff,stroke-width:2px,color:#fff
    style API fill:#0984e3,stroke:#fff,stroke-width:2px,color:#fff
    style SCHED fill:#2d3436,stroke:#fff,color:#fff
    style CM fill:#2d3436,stroke:#fff,color:#fff
```
---
# Componentes dos Worker Nodes

Os Worker Nodes são as máquinas (físicas ou virtuais) onde as cargas de trabalho do Kubernetes são efetivamente executadas. Para que um nó possa receber Pods e ser gerenciado pelo cluster, ele deve executar obrigatoriamente os agentes locais descritos abaixo.

## 1. kubelet

O **kubelet** é o principal agente do Kubernetes que roda em cada nó do cluster. Ele atua como o "capitão" do nó, garantindo que os containers descritos nos PodSpecs estejam rodando e saudáveis.

### Responsabilidades
* **Registro de Nó:** Registra o nó junto ao Control Plane, tornando-o disponível para agendamento.
* **Gerenciamento de Ciclo de Vida:** Recebe instruções do **kube-apiserver** e interage com o Container Runtime (ex: Docker, containerd) para iniciar ou parar containers.
* **Monitoramento de Saúde:** Verifica continuamente se os containers estão vivos e prontos (Liveness e Readiness Probes) e reporta o status ao Control Plane.

> [!NOTE]
> **Cardinalidade**
> Existe exatamente uma instância do kubelet rodando em cada nó do cluster. Se o kubelet parar, o Control Plane considera o nó como "NotReady" e para de enviar novos Pods para ele.

## 2. kube-proxy

O **kube-proxy** é um componente de rede que roda em cada nó do cluster. Ele é responsável por manter as regras de rede no host e permitir a comunicação dos Pods.

### Responsabilidades
* **Abstração de Rede:** Gerencia a comunicação entre os Pods e o "mundo exterior" (internet ou redes externas), bem como a comunicação entre serviços dentro do cluster.
* **Roteamento de Tráfego:** Implementa o encaminhamento de pacotes TCP e UDP.
* **Balanceamento de Carga:** Atua na distribuição básica de tráfego para os serviços do Kubernetes.

## Tabela de Funções do Worker

A tabela abaixo diferencia as responsabilidades de execução (Compute) das responsabilidades de comunicação (Network).

| Componente | Categoria | Função Principal | Interação Chave |
| :--- | :--- | :--- | :--- |
| **kubelet** | Computação / Execução | Garantir que containers estejam rodando e saudáveis. | Conversa com **kube-apiserver**. |
| **kube-proxy** | Rede (Networking) | Permitir comunicação dos Pods com redes externas e internas. | Gerencia regras de IP/Porta. |

## Fluxo de Interação no Worker Node

O diagrama abaixo ilustra como os componentes locais interagem com o Control Plane e com o tráfego externo.

```mermaid
flowchart TD
    subgraph CP [Control Plane]
        API[kube-apiserver]
    end

    subgraph Node [Worker Node]
        direction TB
        Kubelet[kubelet]
        Proxy[kube-proxy]
        
        subgraph Workload [Carga de Trabalho]
            Pod1[Pod A]
            Pod2[Pod B]
        end
    end

    subgraph World [Rede Externa / Internet]
        Traffic[Tráfego de Entrada/Saída]
    end

    %% Fluxos de Controle
    API <-->|Instruções e Status| Kubelet
    Kubelet -->|Gerencia| Workload

    %% Fluxos de Rede
    Traffic <-->|Roteamento| Proxy
    Proxy <-->|Redireciona| Workload

    %% Estilização Dark Mode Friendly
    style CP fill:#2d3436,stroke:#fff,color:#fff
    style Kubelet fill:#0984e3,stroke:#fff,color:#fff
    style Proxy fill:#6c5ce7,stroke:#fff,color:#fff
    style Workload fill:#dfe6e9,stroke:#2d3436,color:#2d3436
```
---
# Requisitos de Rede e Portas

A comunicação correta entre os componentes do Kubernetes depende estritamente da liberação de portas específicas nos firewalls ou grupos de segurança (Security Groups) da infraestrutura.

Abaixo estão listadas as portas padrão para o **Control Plane**, **Worker Nodes** e plugins de rede (**CNI**).

## 1. Control Plane (Mestre)

Estas portas devem estar acessíveis para permitir a administração do cluster e a comunicação interna dos componentes de controle.

| Protocolo | Porta | Componente | Descrição | Origem/Destino |
| :--- | :--- | :--- | :--- | :--- |
| **TCP** | `6443` | **kube-apiserver** | Porta da API do Kubernetes. | Todos (Interno/Externo) |
| **TCP** | `2379` - `2380` | **etcd** | API de Cliente e Comunicação entre pares (peers). | `kube-apiserver` e outros nós etcd |
| **TCP** | `10251` | **kube-scheduler** | API do agendador. | Localhost (Self) / Control Plane |
| **TCP** | `10252` | **kube-controller-manager** | API do gerenciador de controladores. | Localhost (Self) / Control Plane |

## 2. Worker Nodes (Trabalhadores)

Estas portas são utilizadas pelos serviços que rodam nos nós de trabalho e para a exposição de aplicações.

| Protocolo | Porta | Componente | Descrição | Origem/Destino |
| :--- | :--- | :--- | :--- | :--- |
| **TCP** | `10250` | **kubelet** | API do Kubelet (Agente do nó). | Control Plane |
| **TCP** | `30000` - `32767` | **NodePort Services** | Intervalo padrão para serviços do tipo NodePort. | Todos (Tráfego Externo) |

## 3. Plugins de Rede (CNI)

As portas abaixo dependem da escolha da interface de rede (CNI) utilizada no cluster. A configuração incorreta destas portas impede a comunicação entre Pods de nós diferentes.

### Weave Net (Wavenet)
| Protocolo | Porta | Descrição |
| :--- | :--- | :--- |
| **TCP** | `6783` | Porta de controle do Weave Net. |
| **UDP** | `6783` - `6784` | Tráfego de dados e encapsulamento (VXLAN). |

### Calico
| Protocolo | Porta | Descrição |
| :--- | :--- | :--- |
| **TCP** | `179` | **BGP** (Border Gateway Protocol). Necessário se o roteamento BGP estiver ativado. |
| **UDP** | `8472` | **VXLAN**. Necessário para o encapsulamento de pacotes entre nós. |

> [!IMPORTANT]
> **Configuração de Firewall para CNI**
> Ao utilizar o **Calico**, certifique-se de liberar o tráfego UDP na porta `8472` para garantir a funcionalidade da rede overlay (VXLAN). Se optar pelo roteamento via **BGP**, a porta TCP `179` é mandatória.
---
# Objetos Fundamentais: Workloads e Services

No Kubernetes, a gestão de aplicações não é feita apenas executando containers isolados. O sistema utiliza uma hierarquia de objetos para garantir escalabilidade, persistência e acessibilidade.

Abaixo detalhamos os três pilares principais: **Pods** (Unidade), **Controllers** (Gerenciamento) e **Services** (Rede).

## 1. Pod

O **Pod** é a menor unidade de gerenciamento e execução dentro do Kubernetes. O sistema não gerencia containers diretamente; ele gerencia Pods.

### Características Técnicas
* **Unicidade:** Cada Pod recebe um endereço IP único dentro do cluster.
* **Multi-Container:** Um Pod pode encapsular um ou mais containers (ex: aplicação principal + sidecar de log).
* **Contexto Compartilhado:** Todos os containers dentro do mesmo Pod compartilham:
    * O mesmo endereço IP e portas (comunicam-se via `localhost`).
    * Os mesmos volumes de armazenamento montados.
    * O mesmo ciclo de vida (nascem e morrem juntos).

## 2. Controllers: Deployment e ReplicaSet

Para garantir que os Pods permaneçam rodando conforme o esperado, utilizamos controladores. Eles monitoram o estado atual e trabalham para atingir o estado declarado.

### Deployment
O **Deployment** é o controlador de nível superior responsável pela gestão declarativa da aplicação.
* **Função:** Define o estado desejado (imagem, recursos, variáveis).
* **Estratégia de Atualização:** Quando um Deployment é editado (ex: troca de versão da imagem), ele cria um **novo ReplicaSet** e escala o antigo para zero gradualmente.
* **Rollback:** Permite reverter para uma versão anterior simplesmente reativando o ReplicaSet antigo que foi desativado.

### ReplicaSet
O **ReplicaSet** é o "braço direito" do Deployment.
* **Função:** Sua única responsabilidade é garantir a **quantidade exata de réplicas** (cópias do Pod) em execução.
* **Automação:** Sempre que um Deployment é criado, um ReplicaSet é gerado automaticamente para gerenciar os Pods daquela versão.

> [!NOTE]
> **Abstração**
> Na prática, você quase nunca cria um ReplicaSet ou um Pod manualmente. Você cria um **Deployment**, e ele gerencia o ReplicaSet, que por sua vez gerencia os Pods.

---

## 3. Service

Como os Pods são efêmeros (podem ser destruídos e recriados com novos IPs), o **Service** atua como uma abstração de rede estável para expor a aplicação.

O Service garante que o tráfego seja roteado para os Pods corretos, independentemente de onde eles estejam rodando ou de qual seja o IP atual deles.

### Tipos de Service

| Tipo | Visibilidade | Descrição |
| :--- | :--- | :--- |
| **ClusterIP** | Interna | Padrão. Expõe o serviço apenas dentro do cluster. Acessível apenas por outras aplicações internas. |
| **NodePort** | Externa (Estática) | Abre uma porta física (range 30000-32767) em **cada nó** do cluster. Permite acesso externo via `<IP-do-No>:<Porta>`. |
| **LoadBalancer** | Externa (Dinâmica) | Provisiona um balanceador de carga nativo do provedor de nuvem (AWS, Azure, GCP) e atribui um IP público fixo. |

---

## Arquitetura de Relacionamento

O diagrama abaixo ilustra a cadeia de responsabilidade: do Deployment até a exposição via Service.

ADD IMAGE < >
