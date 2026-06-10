# Open5GS – Arquitetura (Item 1 do Quickstart)

## O que é o Open5GS?

O Open5GS é uma implementação **open source do Core Network (núcleo da rede móvel)**, suportando:

- LTE / 4G EPC
- 5G NSA (Non-Standalone)
- 5G SA (Standalone)

Ele é responsável por executar as funções que uma operadora utiliza para:

- Autenticar usuários
- Gerenciar mobilidade
- Criar sessões de dados
- Aplicar políticas de QoS
- Encaminhar tráfego para a Internet

---

# Visão Geral da Rede

O Open5GS representa o núcleo da rede móvel.

```text
UE (Celular)
     │
     ▼
eNB/gNB
(Antena)
     │
     ▼
Open5GS Core
     │
     ▼
Internet
```

---

# Modos de Operação

## LTE / 4G EPC

Arquitetura tradicional utilizada nas redes LTE.

```text
UE
 │
 ▼
eNB
 │
 ▼
EPC
 │
 ▼
Internet
```

Principais componentes:

- MME
- HSS
- PCRF
- SGW
- PGW



## 5G NSA (Non-Standalone)

Utiliza rádio 5G, mas mantém o núcleo 4G.

```text
5G NR
   │
   ▼
LTE EPC
```

Características:

- Controle realizado pelo LTE
- Utiliza EPC do 4G
- Facilita a migração para 5G



## 5G SA (Standalone)

Arquitetura totalmente baseada em 5G.

```text
UE
 │
 ▼
gNB
 │
 ▼
5G Core
 │
 ▼
Internet
```

Características:

- Independente do LTE
- Utiliza Service Based Architecture (SBA)
- Menor latência
- Suporte a Network Slicing

---

# Separação de Planos

## CUPS (Control and User Plane Separation)

O Open5GS implementa CUPS.

CUPS significa:

```text
Control Plane
      ≠
User Plane
```

Benefícios:

- Escalabilidade
- Menor latência
- Distribuição geográfica dos gateways
- Edge Computing
  

## Control Plane

Responsável pelas decisões da rede.

Exemplos:

- Autenticação
- Registro do usuário
- Mobilidade
- Gerenciamento de sessões
- Aplicação de políticas



## User Plane

Responsável pelo transporte dos dados.

Exemplos:

- Navegação Web
- Streaming
- Downloads
- Aplicativos


---

# Arquitetura EPC (4G)

## MME (Mobility Management Entity)

Principal elemento de controle da rede LTE.

Responsabilidades:

- Registro do usuário
- Controle de mobilidade
- Paging
- Criação de sessões
- Controle de bearers

```text
eNB
 │
 ▼
MME
```



## HSS (Home Subscriber Server)

Banco de dados dos assinantes.

Armazena:

- IMSI
- Chaves de autenticação
- APNs
- Perfil do usuário

```text
MME
 │
 ▼
HSS
```



## PCRF (Policy and Charging Rules Function)

Controla políticas da rede.

Responsabilidades:

- QoS
- Controle de banda
- Priorização de tráfego
- Regras de cobrança



## SGWC (Serving Gateway Control)

Controla o SGWU.

Funções:

- Controle de sessões
- Gerenciamento de túneis

Não transporta tráfego.



## SGWU (Serving Gateway User)

Responsável pelo transporte dos dados.

```text
eNB
 │
 ▼
SGWU
```



## PGWC (Packet Gateway Control)

Controla o gateway de saída para a Internet.

Funções:

- Criação de sessões
- Atribuição de IP
- Gerenciamento de rotas



## PGWU (Packet Gateway User)

Último elemento antes da Internet.

```text
UE
 │
 ▼
eNB
 │
 ▼
SGWU
 │
 ▼
PGWU
 │
 ▼
Internet
```

---

# Arquitetura 5G Core

O 5G Core utiliza SBA (Service Based Architecture).

Em vez de conexões fixas, os elementos descobrem serviços dinamicamente.


## NRF (Network Repository Function)

Catálogo de serviços da rede.

Todos os elementos registram-se nele.

```text
AMF
SMF
UDM
AUSF
PCF
 │
 ▼
NRF
```

Função:

- Descoberta de serviços
- Registro de funções da rede


## AMF (Access and Mobility Management Function)

O AMF é o principal ponto de entrada do usuário no 5G Core.

Toda comunicação de controle proveniente do gNB chega inicialmente ao AMF. Ele é responsável por gerenciar o acesso do dispositivo à rede, controlar sua mobilidade e coordenar as demais funções do núcleo.

Principais responsabilidades:

- Registro do UE na rede (Registration Procedure)
- Gerenciamento de mobilidade
- Gerenciamento de contexto do usuário
- Encaminhamento de mensagens NAS (Armazenamento Conectado à Rede)
- Seleção das funções necessárias para autenticação e criação de sessões

```text
gNB
 │
 ▼
AMF
```


## AUSF (Authentication Server Function)

O AUSF é responsável por autenticar os usuários da rede.

Quando um dispositivo tenta se registrar, o AMF solicita ao AUSF que valide as credenciais do assinante.

O AUSF executa os procedimentos de autenticação definidos pelo padrão 5G.

Funções:

- Verificação das credenciais
- Validação do SIM

```text
AMF
 │
 ▼
AUSF
 │
 ▼
UDM
```


## UDM (Unified Data Management)

O UDM é responsável por gerenciar as informações dos assinantes.

Ele fornece dados necessários para autenticação, autorização e utilização dos serviços da rede.

O UDM não é um banco de dados; ele atua como uma camada lógica que acessa os dados armazenados no UDR.

Armazena:

- Dados do assinante
- Perfis
- Informações de autenticação


## UDR (Unified Data Repository)

O UDR é o banco de dados central do 5G Core.

Ele armazena permanentemente as informações utilizadas pelas demais funções da rede.

Armazena:

- Assinantes
- Políticas
- Configurações

```text
UDM
 │
 ▼
UDR
```


## SMF (Session Management Function)

O SMF é responsável pelo gerenciamento das sessões de dados dos usuários.

Quando um dispositivo solicita acesso à Internet, o SMF cria uma **PDU Session**, define os parâmetros necessários e seleciona o UPF que será utilizado para transportar os dados.

Funções:

- Criação de sessões PDU (responsável por transportar dados entre o dispositivo (celular, modem, IoT) e a rede de dados externa)
- Atribuição de IP
- Configuração do UPF

```text
AMF
 │
 ▼
SMF
 │
 ▼
UPF
```


## UPF (User Plane Function)

O UPF é o principal elemento do plano de usuário (User Plane).

Todo o tráfego gerado pelos dispositivos passa pelo UPF antes de chegar à Internet ou a redes externas.

```text
gNB
 │
 ▼
UPF
 │
 ▼
Internet
```

Funções:

- Encaminhamento de pacotes
- Aplicação de regras de tráfego
- Interligação com redes externas


## PCF (Policy Control Function)

O PCF é responsável pela definição das políticas da rede.

Ele determina quais regras devem ser aplicadas para cada usuário ou serviço.

O PCF fornece essas informações ao SMF, que as aplica durante a criação das sessões.

Responsabilidades:

- QoS
- Políticas de tráfego
- Controle de uso

```text
PCF
 │
 ▼
SMF
```
  

## NSSF (Network Slice Selection Function)

O NSSF é responsável por selecionar a Network Slice adequada para cada usuário ou serviço.

Uma Slice representa uma rede lógica independente criada sobre a mesma infraestrutura física.

Permite que diferentes serviços utilizem redes lógicas independentes.

Exemplo:

```text
Slice IoT
Slice Vídeo
Slice Industrial
```

```text
UE
 │
 ▼
AMF
 │
 ▼
NSSF
```

---

# Fluxo Completo de Registro no 5G SA

## 1. Registro Inicial

```text
UE
 │
 ▼
gNB
 │
 ▼
AMF
```

O dispositivo solicita acesso à rede.

---

## 2. Autenticação

```text
AMF
 │
 ▼
AUSF
 │
 ▼
UDM
 │
 ▼
UDR
```

As credenciais do assinante são verificadas.

---

## 3. Seleção da Slice

```text
AMF
 │
 ▼
NSSF
```

A slice apropriada é selecionada.

---

## 4. Criação da Sessão

```text
AMF
 │
 ▼
SMF
 │
 ▼
UPF
```

A PDU Session é criada.

---

## 5. Acesso à Internet

```text
UE
 │
 ▼
gNB
 │
 ▼
UPF
 │
 ▼
Internet
```

O tráfego do usuário passa pelo UPF e segue para a rede externa.

---

# Tabela Comparativa LTE x 5G

| LTE (EPC) | 5G Core |
|------------|------------|
| MME | AMF |
| HSS | UDM |
| HSS Database | UDR |
| PCRF | PCF |
| SGW/PGW Control | SMF |
| SGW/PGW User | UPF |
| Não possui NRF | NRF |
| Não possui NSSF | NSSF |

---

# Resumo

## EPC (4G)

| Função | Responsabilidade |
|----------|----------|
| MME | Controle principal |
| HSS | Base de assinantes |
| PCRF | Políticas |
| SGWC | Controle do SGW |
| SGWU | Transporte de dados |
| PGWC | Controle do PGW |
| PGWU | Saída para Internet |


## 5G Core

| Função | Objetivo Principal |
|----------|----------|
| NRF | Registro e descoberta de serviços |
| AMF | Controle de acesso e mobilidade |
| AUSF | Autenticação do usuário |
| UDM | Gerenciamento dos dados do assinante |
| UDR | Armazenamento persistente dos dados |
| SMF | Gerenciamento das sessões PDU |
| UPF | Transporte do tráfego de usuário |
| PCF | Definição de políticas e QoS |
| NSSF | Seleção de Network Slices |

Tudo que toma decisões está no Control Plane; tudo que transporta pacotes está no User Plane.
No Open5GS 5G SA, o UPF é a única Network Function do 5G Core pertencente ao User Plane.

---

# Conceito Principal

O Open5GS implementa tanto o núcleo LTE (EPC) quanto o núcleo 5G SA. Em ambas as arquiteturas existe uma separação clara entre:

- **Control Plane:** decisões da rede.
- **User Plane:** transporte dos dados.

No 5G, essa arquitetura evolui para uma abordagem baseada em serviços (SBA), tornando a rede mais flexível, escalável e preparada para aplicações modernas como Edge Computing e Network Slicing.
