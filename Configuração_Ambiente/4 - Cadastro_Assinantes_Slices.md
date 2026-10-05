# Cadastro de Assinantes e Slices (WebUI)

## Objetivo

Com a WebUI instalada e operacional, o próximo passo consiste em realizar o **provisionamento do Equipamento de Usuário (UE)** no banco de dados do núcleo 5G. Esse processo é denominado **cadastro de assinante (Subscriber Provisioning)** e consiste em registrar todas as informações necessárias para que o UE possa ser autenticado e autorizado a utilizar a rede móvel.

No Open5GS, esse cadastro é armazenado no **MongoDB** através das funções **UDR (Unified Data Repository)** e **UDM (Unified Data Management)**. O UDR é responsável pelo armazenamento permanente dos dados do assinante, enquanto o UDM gerencia o acesso e disponibiliza essas informações para as demais funções do núcleo durante os procedimentos de autenticação e registro.

Neste laboratório, o objetivo foi validar o funcionamento do **Network Slicing**, permitindo que um único assinante tivesse autorização para acessar três fatias de rede distintas (SST 1, 2 e 3), cada uma associada a um serviço (DNN/APN) diferente.

---

# 1. Acesso à Interface Web

Acesse a WebUI do Open5GS utilizando um navegador.

```text
http://localhost:9999
```

Utilize as credenciais padrão:

- **Usuário:** admin
- **Senha:** 1423

A documentação oficial informa que a WebUI é utilizada para editar de forma gráfica os dados dos assinantes armazenados no banco de dados do Open5GS, facilitando o provisionamento de UEs durante o desenvolvimento e testes. 

---

# 2. Criação do Assinante (Subscriber)

Na barra lateral esquerda, selecione:

```text
Subscriber
```

Em seguida, clique no botão:

```text
+ Add Subscriber
```

Será aberta a tela de criação de um novo assinante.

O assinante representa o cadastro lógico de um **SIM/USIM** dentro do núcleo 5G. É através desse cadastro que o Open5GS identifica quais dispositivos estão autorizados a utilizar a rede.

---

# 3. Configuração da Identidade e Autenticação

Preencha os seguintes campos.

| Campo | Valor |
|---------|---------|
| IMSI | 999700000000001 |
| Subscriber Key (K) | 8BAF473F2F8FD09487CCCBD7097C6862 |
| USIM Type | OPc |
| Operator Key (OPc) | E8ED3CA456BF83081672A61937F69302 |

## Descrição dos parâmetros

### IMSI

O **International Mobile Subscriber Identity (IMSI)** é o identificador único do assinante dentro da rede móvel.

Durante o procedimento de registro, o UE envia esse identificador para que o núcleo localize o cadastro correspondente no UDR.

---

### Subscriber Key (K)

A chave **K** é a chave secreta compartilhada entre o USIM e o núcleo da rede.

Ela é utilizada pelo algoritmo de autenticação **5G-AKA**, permitindo verificar se o dispositivo realmente pertence ao assinante cadastrado.

---

### OPc

O **Operator Code (OPc)** é um parâmetro criptográfico derivado da chave da operadora.

Ele também participa do algoritmo 5G-AKA e deve possuir exatamente o mesmo valor configurado no UE (UERANSIM).

Qualquer divergência entre **K** e **OPc** resultará na falha da autenticação.

---

# 4. Configuração das Fatias de Rede (Network Slicing)

Ainda na tela do assinante, localize a seção:

```text
Session
```

Para cada fatia de rede, clique em:

```text
Add Slice
```

Foram cadastradas três configurações distintas.

| Serviço | SST | DNN (APN) |
|-----------|----:|-----------|
| eMBB | 1 | internet |
| URLLC | 2 | emergency |
| mMTC | 3 | iot |

O campo **SD (Slice Differentiator)** permaneceu com o valor padrão (`000000`), uma vez que neste laboratório a diferenciação das fatias ocorreu apenas através do **SST (Slice/Service Type)**.

Após concluir o preenchimento, clique em:

```text
Save
```

---

# 5. O que é o SST?

O **SST (Slice/Service Type)** identifica o tipo de serviço que determinada fatia de rede oferece.

Neste laboratório foram utilizados três tipos padronizados pelo 3GPP.

| SST | Serviço | Característica |
|------|----------|----------------|
| 1 | eMBB | Alta largura de banda |
| 2 | URLLC | Baixa latência e alta confiabilidade |
| 3 | mMTC | Comunicação massiva entre dispositivos IoT |

Cada SST foi associado a um **DNN** diferente, permitindo que um mesmo assinante estabelecesse múltiplas sessões de dados independentes.

---

# 6. O que é o DNN?

O **Data Network Name (DNN)** representa a rede de dados que será acessada pelo UE.

No Open5GS, o DNN define para qual rede IP o SMF criará uma **PDU Session**.

Neste laboratório foram utilizados:

| DNN | Finalidade |
|------|------------|
| internet | Serviços convencionais de dados |
| emergency | Serviços críticos de baixa latência |
| iot | Comunicação entre dispositivos IoT |

Quando o UE solicitar uma sessão utilizando determinado DNN, o SMF verificará se esse DNN está autorizado para o assinante.

---

# 7. Armazenamento das Informações

Ao clicar em **Save**, todas as informações são gravadas imediatamente no banco de dados **MongoDB**.

O fluxo interno é:

```text
WebUI
    │
    ▼
MongoDB
    │
    ▼
UDR
    │
    ▼
UDM
```

Essas informações ficam imediatamente disponíveis para os procedimentos de autenticação realizados pelo AUSF e para o gerenciamento de sessões executado pelo SMF. Não é necessário reiniciar os serviços ao adicionar um novo assinante; porém, alterações em um perfil existente podem exigir a reinicialização do AMF/MME para serem aplicadas.

---

# 8. Importância da Configuração

Esta configuração foi fundamental para o correto funcionamento do laboratório pelos seguintes motivos:

- O IMSI identifica unicamente o assinante dentro da rede.
- As chaves **K** e **OPc** permitem que o procedimento de autenticação **5G-AKA** seja concluído com sucesso.
- O cadastro de múltiplos DNNs possibilita ao mesmo assinante estabelecer diferentes **PDU Sessions**.
- O cadastro de múltiplas **Slices (S-NSSAI)** permite que um único UE acesse simultaneamente diferentes serviços de rede.

Caso qualquer um desses parâmetros não coincida com os configurados posteriormente no UERANSIM, o registro do UE será rejeitado durante a autenticação.

---

# Resultado da Etapa

Ao final desta etapa, o Open5GS possui:

- Um assinante válido identificado pelo IMSI **999700000000001**;
- Credenciais criptográficas configuradas para autenticação 5G-AKA;
- Três fatias de rede (SST 1, 2 e 3) autorizadas para o mesmo assinante;
- Três DNNs distintos (`internet`, `emergency` e `iot`) disponíveis para criação de PDU Sessions;
- O banco de dados do núcleo 5G preparado para receber a conexão do gNB e do UE através do simulador UERANSIM.
