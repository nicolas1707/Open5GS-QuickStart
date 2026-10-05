# Conteúdo do Repositório

```text
Configuração_Ambiente/
│
├── 1 - Arquitetura_Open5GS.md
├── 2 - Instalação_Open5GS.md
├── 3 - Instalação_WebUI.md
├── 4 - Cadastro_Assinantes_Slices.md
├── 5 - Configuração_Core_5G.md
├── 6 - Instalação_UERANSIM.md
├── 7 - Configuração_UE.md
└── 8 - Inicialização_e_Validação.md
```

Cada documento foi elaborado de forma incremental, explicando não apenas **como** executar cada procedimento, mas também **o que acontece internamente** em cada etapa do funcionamento da rede 5G.

---

# Principais Tópicos Abordados

## Arquitetura 5G SA

- Service Based Architecture (SBA)
- Control Plane
- User Plane
- AMF
- SMF
- UPF
- UDM
- UDR
- AUSF
- NRF
- PCF
- NSSF

## Configuração do Core

- Instalação via Package Manager
- Configuração dos serviços
- Provisionamento de assinantes
- Configuração da WebUI
- Gerenciamento do banco MongoDB

## Rede de Acesso (RAN)

- Configuração do gNodeB
- Configuração do UE
- Interfaces NGAP
- Interface GTP-U
- Simulação utilizando UERANSIM

## Network Slicing

O laboratório implementa um cenário de **Network Slicing**, no qual um único equipamento de usuário estabelece simultaneamente três sessões PDU independentes:

| Slice | SST | DNN |
|--------|----:|-----|
| eMBB | 1 | internet |
| URLLC | 2 | emergency |
| mMTC | 3 | iot |

Cada sessão resulta na criação de uma interface TUN independente no sistema operacional.

---

# Ambiente Utilizado

- Sistema Operacional: Ubuntu 22.04 LTS
- Core: Open5GS
- Banco de Dados: MongoDB
- Interface Web: Open5GS WebUI
- Simulador RAN: UERANSIM
- Arquitetura: 5G Standalone (SA)

---
