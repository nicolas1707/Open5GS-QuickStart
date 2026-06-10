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

Equivalente ao MME do LTE.

Responsabilidades:

- Registro do usuário
- Mobilidade
- Gerenciamento de acesso

```text
gNB
 │
 ▼
AMF
```


## AUSF (Authentication Server Function)

Responsável pela autenticação.

Funções:

- Verificação das credenciais
- Validação do SIM


## UDM (Unified Data Management)

Equivalente ao HSS.

Armazena:

- Dados do assinante
- Perfis
- Informações de autenticação


## UDR (Unified Data Repository)

Banco de dados do 5G Core.

Armazena permanentemente:

- Assinantes
- Políticas
- Configurações


## SMF (Session Management Function)

Responsável pelas sessões de dados.

Funções:

- Criação de sessões PDU
- Atribuição de IP
- Configuração do UPF


## UPF (User Plane Function)

Elemento responsável pelo tráfego de usuário.

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

Equivalente ao PCRF do LTE.

Responsabilidades:

- QoS
- Políticas de tráfego
- Controle de uso
  

## NSSF (Network Slice Selection Function)

Responsável pela seleção de Network Slices.

Exemplo:

```text
Slice IoT
Slice Vídeo
Slice Industrial
```

Permite que diferentes serviços utilizem redes lógicas independentes.


## Fluxo Completo de Registro no 5G SA

```text
UE
 │
 ▼
gNB
 │
 ▼
AMF
 │
 ├── AUSF (Autenticação)
 │
 ├── UDM/UDR (Dados do usuário)
 │
 ├── NSSF (Seleção de Slice)
 │
 └── SMF
         │
         ▼
        UPF
         │
         ▼
     Internet
```

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

| Função | Responsabilidade |
|----------|----------|
| NRF | Descoberta de serviços |
| AMF | Mobilidade e acesso |
| AUSF | Autenticação |
| UDM | Dados do assinante |
| UDR | Banco de dados |
| SMF | Sessões |
| UPF | Tráfego de usuário |
| PCF | Políticas |
| NSSF | Seleção de Slice |

---

# Conceito Principal

O Open5GS implementa tanto o núcleo LTE (EPC) quanto o núcleo 5G SA. Em ambas as arquiteturas existe uma separação clara entre:

- **Control Plane:** decisões da rede.
- **User Plane:** transporte dos dados.

No 5G, essa arquitetura evolui para uma abordagem baseada em serviços (SBA), tornando a rede mais flexível, escalável e preparada para aplicações modernas como Edge Computing e Network Slicing.
