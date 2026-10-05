# Configuração do Ambiente

Esta seção reúne toda a documentação necessária para a implantação de um ambiente funcional de uma rede **5G Standalone (5G SA)** utilizando **Open5GS** e **UERANSIM**.

O objetivo é documentar, de forma detalhada, todas as etapas realizadas durante a construção do laboratório, explicando não apenas **como** executar cada procedimento, mas também **o funcionamento interno** dos componentes envolvidos na arquitetura do Core 5G.

Toda a documentação foi desenvolvida com base no **Quickstart oficial do Open5GS**, sendo expandida com explicações técnicas, diagramas, conceitos de redes móveis e observações obtidas durante a implementação prática do ambiente.

---

# Etapas Documentadas
Os documentos foram organizados em ordem cronológica, acompanhando o fluxo natural de implantação do ambiente.

| Etapa | Descrição |
|--------|-----------|
| **1** | Introdução à arquitetura do Open5GS e do Core 5G |
| **2** | Instalação do Open5GS e de suas dependências |
| **3** | Instalação da WebUI e configuração do gerenciamento de assinantes |
| **4** | Cadastro de assinantes e configuração de Network Slices |
| **5** | Instalação e compilação do UERANSIM |
| **6** | Configuração do gNodeB |
| **7** | Configuração do UE |
| **8** | Inicialização, registro e validação do ambiente |

---

# Principais Conceitos Abordados

## Arquitetura 5G SA

- Service Based Architecture (SBA)
- Plano de Controle (Control Plane)
- Plano de Usuário (User Plane)
- Network Functions (AMF, SMF, UPF, UDM, UDR, AUSF, NRF, PCF e NSSF)

---

## Implantação do Core 5G

- Instalação do Open5GS
- Configuração dos serviços
- MongoDB
- WebUI
- Provisionamento de assinantes

---

## Rede de Acesso (RAN)

- Instalação do UERANSIM
- Configuração do gNodeB
- Configuração do UE
- Interface NGAP (N2)
- Interface GTP-U (N3)

---

## Network Slicing

O ambiente documenta a implementação de um cenário de **Network Slicing**, no qual um único UE estabelece simultaneamente três **PDU Sessions**, cada uma associada a uma fatia lógica distinta da rede.

| Slice | SST | DNN |
|--------|----:|-----|
| eMBB | 1 | internet |
| URLLC | 2 | emergency |
| mMTC | 3 | iot |

Cada sessão resulta na criação de uma interface virtual independente (`uesimtun`), permitindo a separação do tráfego entre as diferentes fatias de rede.

---

# Ambiente Utilizado

- **Sistema Operacional:** Ubuntu 22.04 LTS
- **Core 5G:** Open5GS
- **Banco de Dados:** MongoDB
- **Interface Web:** Open5GS WebUI
- **Simulador RAN:** UERANSIM
- **Arquitetura:** 5G Standalone (SA)

---

# Objetivo da Documentação
Além da reprodução do ambiente proposto pelo Quickstart oficial, foram adicionadas explicações detalhadas sobre:

- arquitetura do Core 5G;
- funções de rede (Network Functions);
- autenticação de assinantes;
- criação de PDU Sessions;
- Network Slicing;
- integração entre Open5GS e UERANSIM;
- validação prática do ambiente implementado.
