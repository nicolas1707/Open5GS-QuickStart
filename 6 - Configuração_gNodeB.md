# 6. Configuração do gNodeB (UERANSIM)

## Objetivo

Após a instalação do UERANSIM, o próximo passo consiste em configurar o **gNodeB (gNB)**, responsável por simular a estação rádio-base (RAN) de uma rede 5G Standalone (SA).

O gNodeB é o elemento que estabelece a comunicação entre o **UE** e o **5G Core**, utilizando:

- **Interface N2 (NGAP/SCTP)** para comunicação com o **AMF**;
- **Interface N3 (GTP-U)** para transporte dos dados entre o gNodeB e o **UPF**.

Além disso, o gNodeB informa ao núcleo quais **Network Slices (S-NSSAIs)** são suportadas pela célula, permitindo que os UEs solicitem sessões de dados compatíveis com essas fatias. 

---

# Localização do Arquivo

O arquivo de configuração encontra-se em:

```text
UERANSIM/config/open5gs-gnb.yaml
```

Abra o arquivo utilizando:

```bash
nano config/open5gs-gnb.yaml
```

### O que este comando faz?

Abre o arquivo de configuração do gNodeB para edição.

### Impacto no processo

Todas as características da estação rádio-base simulada são definidas neste arquivo, incluindo:

- identidade da rede;
- parâmetros da célula;
- endereço do AMF;
- interfaces de comunicação;
- slices suportadas.

---

# Configuração Completa

```yaml
mcc: '999'

mnc: '70'

nci: '0x000000010'

idLength: 32

tac: 1

linkIp: 127.0.0.1

ngapIp: 127.0.0.1

gtpIp: 127.0.0.1

amfConfigs:

  - address: 127.0.0.5
    port: 38412

slices:

  - sst: 1
  - sst: 2
  - sst: 3

ignoreStreamIds: true
```

---

# Explicação de Cada Parâmetro

## MCC

```yaml
mcc: '999'
```

### O que significa?

**Mobile Country Code**.

Representa o código do país da rede móvel.

### Impacto no processo

Deve ser exatamente igual ao configurado anteriormente no **AMF**.

Caso exista divergência entre UE, gNB e Open5GS, o registro será rejeitado.

---

## MNC

```yaml
mnc: '70'
```

### O que significa?

**Mobile Network Code**.

Identifica a operadora dentro do país.

### Impacto no processo

Também deve coincidir exatamente com o valor configurado no Open5GS.

---

## NCI

```yaml
nci: '0x000000010'
```

### O que significa?

**NR Cell Identity**.

É o identificador único da célula 5G.

Cada gNodeB transmite esse identificador para que o UE saiba em qual célula está conectado.

### Impacto no processo

Permite identificar unicamente a célula durante o procedimento de registro.

---

## idLength

```yaml
idLength: 32
```

### O que significa?

Quantidade de bits utilizados para representar o identificador do gNodeB.

### Impacto no processo

Em ambientes de laboratório, o valor padrão **32 bits** é suficiente.

---

## TAC

```yaml
tac: 1
```

### O que significa?

**Tracking Area Code**.

Define a área de rastreamento da célula.

### Impacto no processo

O TAC deve coincidir com o configurado no AMF.

Caso contrário, o UE poderá ser rejeitado durante o registro. 

---

## linkIp

```yaml
linkIp: 127.0.0.1
```

### O que significa?

Endereço IP utilizado pelo simulador para a comunicação entre o **UE** e o **gNodeB**.

Trata-se de uma interface utilizada apenas pela simulação do enlace rádio (Radio Link Simulation).

### Impacto no processo

Como todo o processo está sendo executado na mesma máquina, utiliza-se o endereço de loopback.

Caso UE e gNB estejam em computadores diferentes, esse endereço deverá ser alterado para o IP real da máquina do gNodeB.

---

## ngapIp

```yaml
ngapIp: 127.0.0.1
```

### O que significa?

Endereço local utilizado pelo gNodeB para estabelecer a interface **N2**.

A interface N2 utiliza:

- protocolo NGAP;
- transporte SCTP.

### Impacto no processo

O AMF enviará todas as mensagens NGAP para esse endereço.

Em ambientes distribuídos, deve ser substituído pelo IP real da máquina do gNodeB.

---

## gtpIp

```yaml
gtpIp: 127.0.0.1
```

### O que significa?

Endereço local utilizado pela interface **N3**.

Essa interface transporta o tráfego dos usuários utilizando **GTP-U**.

### Impacto no processo

Após a criação das PDU Sessions, será através deste endereço que os túneis uesimtun receberão os pacotes encaminhados pelo UPF. É vital que este endereço consiga alcançar o IP definido na interface do UPF do Open5GS. Neste laboratório local, o loopback cumpriu essa função na perfeição.

---

# Configuração do AMF

```yaml
amfConfigs:

  - address: 127.0.0.5
    port: 38412
```

### O que significa?

Define o endereço do **AMF** ao qual o gNodeB deverá se conectar.

### Impacto no processo

Ao iniciar o gNodeB:

1. é criada uma conexão SCTP;
2. é enviado um **NG Setup Request**;
3. o AMF responde com um **NG Setup Response**.

Somente após essa etapa o gNodeB poderá aceitar registros de UEs.

---

# Configuração das Slices

```yaml
slices:

  - sst: 1
  - sst: 2
  - sst: 3
```

### O que significa?

Lista todas as **Network Slices** suportadas pela célula.

Nesta configuração:

| SST | Serviço |
|------|----------|
| 1 | eMBB |
| 2 | URLLC |
| 3 | mMTC |

### Impacto no processo

Durante o registro do UE, o gNodeB informa ao AMF quais fatias estão disponíveis.

Caso uma Slice solicitada pelo UE não esteja presente nessa lista, a criação da PDU Session poderá ser rejeitada. 

---

# ignoreStreamIds

```yaml
ignoreStreamIds: true
```

### O que significa?

Instrui o UERANSIM a ignorar diferenças na numeração dos fluxos SCTP.

### Impacto no processo

A documentação oficial recomenda utilizar este parâmetro com o Open5GS, pois algumas implementações do Core utilizam identificadores de stream diferentes dos esperados pelo simulador. Isso evita falhas durante a sinalização NGAP. 

---

# Relação entre o gNodeB e o Open5GS

Após a configuração, o fluxo esperado é:

```text
UE
 │
 │ (Radio Link Simulation)
 ▼
gNodeB (UERANSIM)
 │
 │ N2 (NGAP / SCTP)
 ▼
AMF
 │
 │ SBA
 ▼
5G Core
```

Após o estabelecimento da conexão N2, o gNodeB estará preparado para encaminhar solicitações de registro dos UEs ao núcleo 5G.

---

# Resultado da Etapa

Ao final desta configuração:

- O gNodeB conhece a identidade da rede (MCC, MNC e TAC);
- Está configurado para conectar-se ao AMF do Open5GS pela interface N2;
- Está preparado para encaminhar o tráfego do plano de usuário pela interface N3;
- Declara suporte às três Network Slices (SST 1, 2 e 3);
- Está pronto para aceitar conexões do UE, cuja configuração será realizada na próxima etapa.
