# 7. Configuração do UE (UERANSIM)

## Objetivo

Após configurar o gNodeB, o próximo passo consiste em configurar o **UE (User Equipment)** do UERANSIM. O UE simula um dispositivo móvel 5G capaz de realizar o procedimento completo de registro (Registration), autenticação (5G-AKA) e estabelecimento de **PDU Sessions** junto ao núcleo Open5GS.

Todas as informações configuradas neste arquivo devem ser consistentes com aquelas cadastradas anteriormente na **WebUI do Open5GS**, especialmente o IMSI, as chaves criptográficas (K e OPc), o PLMN (MCC/MNC), as Network Slices e os DNNs. Qualquer divergência entre esses parâmetros impedirá o registro do UE na rede. 

---

# Localização do Arquivo

O arquivo de configuração encontra-se em:

```text
UERANSIM/config/open5gs-ue.yaml
```

Abra o arquivo utilizando:

```bash
nano config/open5gs-ue.yaml
```

### O que este comando faz?

Abre o arquivo de configuração do UE para edição.

### Impacto no processo

Neste arquivo são definidos:

- identidade do assinante;
- parâmetros de autenticação;
- identificação da operadora;
- fatias de rede autorizadas;
- sessões PDU iniciais;
- algoritmos criptográficos suportados.

---

# Configuração Completa

```yaml
supi: 'imsi-999700000000001'

mcc: '999'

mnc: '70'

protectionScheme: 0

homeNetworkPublicKey: '5a8d38864820197c3394b92613b20b91633cbd897119273bf8e4a6f4eec0a650'

homeNetworkPublicKeyId: 1

routingIndicator: '0000'

key: '8BAF473F2F8FD09487CCCBD7097C6862'

op: 'E8ED3CA456BF83081672A61937F69302'

opType: 'OPC'

amf: '8000'

imei: '356938035643803'

imeiSv: '4370816125816151'

tunNetmask: '255.255.255.0'

useNamespace: false

nsNamePrefix: 'ueransim'

gnbSearchList:
  - 127.0.0.1

configured-nssai:
  - sst: 1
  - sst: 2
  - sst: 3

default-nssai:
  - sst: 1

sessions:
  - type: 'IPv4'
    apn: 'internet'
    slice:
      sst: 1

  - type: 'IPv4'
    apn: 'emergency'
    slice:
      sst: 2

  - type: 'IPv4'
    apn: 'iot'
    slice:
      sst: 3

integrity:
  IA1: true
  IA2: true
  IA3: true

ciphering:
  EA1: true
  EA2: true
  EA3: true

integrityMaxRate:
  uplink: 'full'
  downlink: 'full'
```

---

# Identificação do Assinante

## SUPI

```yaml
supi: 'imsi-999700000000001'
```

### O que significa?

O **SUPI (Subscription Permanent Identifier)** é a identidade permanente do assinante.

No UERANSIM, utiliza-se o formato:

```text
imsi-<IMSI>
```

Nesta configuração:

```text
IMSI = 999700000000001
```

### Impacto no processo

Durante o procedimento de registro, o AMF utiliza esse identificador para localizar o assinante no UDR.

O valor deve ser exatamente igual ao cadastrado anteriormente na WebUI.

---

# PLMN

## MCC

```yaml
mcc: '999'
```

Representa o **Mobile Country Code**.

---

## MNC

```yaml
mnc: '70'
```

Representa o **Mobile Network Code**.

### Impacto no processo

Os valores de MCC e MNC devem coincidir com aqueles configurados no:

- AMF;
- gNodeB;
- cadastro do assinante.

Caso contrário, o UE não reconhecerá a rede como pertencente ao seu operador.

---

# Autenticação

## Subscriber Key

```yaml
key: '8BAF473F2F8FD09487CCCBD7097C6862'
```

É a chave secreta compartilhada entre o USIM e o núcleo 5G.

---

## Operator Code

```yaml
op: 'E8ED3CA456BF83081672A61937F69302'
```

Corresponde ao valor OPc cadastrado anteriormente na WebUI.

---

## OP Type

```yaml
opType: 'OPC'
```

Indica que o valor informado corresponde ao **OPc** e não ao **OP**.

---

## Authentication Management Field

```yaml
amf: '8000'
```

Valor utilizado durante o procedimento de autenticação 5G-AKA.

### Impacto no processo

Esses parâmetros são utilizados pelo AUSF durante a autenticação.

Qualquer divergência entre UE e Open5GS resulta em falha de autenticação e rejeição do registro.

---

# Configuração da Interface TUN

```yaml
tunNetmask: '255.255.255.0'
```

Define a máscara utilizada pelas interfaces virtuais criadas pelo UE.

---

```yaml
useNamespace: false
```

Indica que o UE será executado no namespace padrão do Linux.

---

```yaml
nsNamePrefix: 'ueransim'
```

Define o prefixo utilizado caso namespaces sejam habilitados futuramente.

### Impacto no processo

Como toda a configuração é executada em uma única máquina, não foi necessário utilizar namespaces de rede.

---

# Localização do gNodeB

```yaml
gnbSearchList:
  - 127.0.0.1
```

### O que significa?

Lista dos endereços IP onde o UE procurará um gNodeB.

### Impacto no processo

O endereço deve coincidir com o parâmetro `linkIp` configurado anteriormente no arquivo `open5gs-gnb.yaml`.

Como estamos utilizando apenas uma máquina, foi utilizado o endereço de loopback (`127.0.0.1`). Em ambientes distribuídos, esse endereço deve ser substituído pelo IP real do gNodeB. 

---

# Configuração das Network Slices

## Configured NSSAI

```yaml
configured-nssai:
  - sst: 1
  - sst: 2
  - sst: 3
```

Lista todas as fatias de rede conhecidas pelo UE.

---

## Default NSSAI

```yaml
default-nssai:
  - sst: 1
```

Define a fatia utilizada durante o procedimento inicial de registro.

### Impacto no processo

Após registrar-se utilizando a Slice padrão (SST 1), o UE poderá solicitar sessões adicionais nas demais fatias autorizadas.

---

# Configuração das PDU Sessions

```yaml
sessions:

  - type: 'IPv4'
    apn: 'internet'
    slice:
      sst: 1

  - type: 'IPv4'
    apn: 'emergency'
    slice:
      sst: 2

  - type: 'IPv4'
    apn: 'iot'
    slice:
      sst: 3
```

### O que significa?

Define as sessões de dados que o UE tentará estabelecer automaticamente após concluir o registro na rede.

Cada sessão contém:

- tipo de PDU Session;
- DNN (APN);
- Network Slice desejada.

### Impacto no processo

Após o registro bem-sucedido, o UE solicitará automaticamente três PDU Sessions:

| DNN | SST | Aplicação |
|------|----:|-----------|
| internet | 1 | eMBB |
| emergency | 2 | URLLC |
| iot | 3 | mMTC |

Cada sessão IPv4 bem-sucedida forçará o UERANSIM a criar uma interface virtual de rede no kernel do Linux. Neste cenário, subirão simultaneamente as interfaces `uesimtun0`, `uesimtun1` e `uesimtun2`, cada uma com um IP fornecido pelo Open5GS.

---

# Algoritmos Criptográficos

```yaml
integrity:
  IA1: true
  IA2: true
  IA3: true
```

Define os algoritmos de integridade suportados.

---

```yaml
ciphering:
  EA1: true
  EA2: true
  EA3: true
```

Define os algoritmos de criptografia suportados.

### Impacto no processo

Durante a negociação de segurança, o AMF selecionará um algoritmo suportado tanto pelo UE quanto pelo núcleo.

---

# Resultado da Etapa

Ao final desta configuração:

- O UE possui a mesma identidade cadastrada no Open5GS;
- As credenciais criptográficas coincidem com o cadastro realizado na WebUI;
- O UE conhece as três Network Slices disponíveis;
- Estão configuradas três solicitações automáticas de PDU Session (internet, emergency e iot);
- O simulador está preparado para iniciar o procedimento de registro junto ao gNodeB e ao núcleo 5G.
