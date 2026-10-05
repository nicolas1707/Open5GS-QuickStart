# 5. Instalação do UERANSIM

## Objetivo

Após a configuração do núcleo 5G (Open5GS), o próximo passo consiste em instalar o **UERANSIM**, um simulador de **RAN (Radio Access Network)** e **UE (User Equipment)** para redes 5G Standalone (SA).

O UERANSIM permite simular:

- Um **gNodeB (gNB)**, representando a estação rádio-base 5G;
- Um **UE**, representando um dispositivo móvel 5G;
- O procedimento completo de registro (Registration);
- A criação de **PDU Sessions**;
- A troca de mensagens NAS e NGAP com o Open5GS.

Dessa forma, é possível validar todo o funcionamento do núcleo 5G sem a necessidade de equipamentos físicos, como SDRs (Software Defined Radios) ou estações rádio comerciais.

---

# Requisitos

O UERANSIM possui os seguintes requisitos mínimos:

- Ubuntu 16.04 ou superior;
- GCC 9 ou superior;
- G++ 9 ou superior;
- CMake 3.17 ou superior;
- Suporte ao protocolo SCTP.

Como o ambiente utilizado foi o **Ubuntu 22.04 (Jammy Jellyfish)**, todos os requisitos são atendidos pelo sistema operacional. Entretanto, a documentação oficial recomenda instalar uma versão recente do **CMake**, pois versões antigas distribuídas pelo APT podem não atender aos requisitos de compilação das versões mais novas do UERANSIM. 

---

# 1. Atualização dos Repositórios

Antes da instalação, atualize a lista de pacotes disponíveis.

```bash
sudo apt update
```

### O que este comando faz?

Atualiza o índice de pacotes dos repositórios configurados no sistema.

Nenhum software é instalado ou atualizado nesta etapa.

### Impacto no processo

Garante que todas as dependências do UERANSIM serão obtidas em suas versões mais recentes disponíveis para o Ubuntu.

---

# 2. Instalação das Dependências

Instale todas as bibliotecas necessárias para compilação e execução do UERANSIM.

```bash
sudo apt install -y make gcc g++ libsctp-dev lksctp-tools iproute2 git
```

### O que este comando faz?

Instala as principais dependências utilizadas pelo projeto.

### Impacto no processo

Cada pacote possui uma função específica durante a compilação ou execução.

| Pacote | Finalidade |
|----------|------------|
| make | Automatiza a compilação do projeto. |
| gcc | Compilador da linguagem C. |
| g++ | Compilador da linguagem C++. |
| libsctp-dev | Biblioteca de desenvolvimento do protocolo SCTP. |
| lksctp-tools | Ferramentas de teste e suporte ao SCTP em Linux. |
| iproute2 | Utilitários de configuração de interfaces de rede (comando `ip`). |
| git | Download do código-fonte do UERANSIM. |

O protocolo **SCTP (Stream Control Transmission Protocol)** merece destaque, pois é utilizado pela interface **N2 (NGAP)** para estabelecer a comunicação entre o gNodeB e o AMF do Open5GS. Sem essa biblioteca, o UERANSIM não consegue estabelecer a conexão de sinalização com o núcleo 5G.

---

# 3. Instalação do CMake

A documentação oficial recomenda utilizar uma versão recente do CMake.

No Ubuntu, a forma recomendada é através do Snap.

```bash
sudo snap install cmake --classic
```

### O que este comando faz?

Instala uma versão atualizada do **CMake**, responsável por gerar os arquivos de compilação utilizados pelo projeto.

### Impacto no processo

Versões antigas do CMake distribuídas pelo APT podem não atender aos requisitos mínimos do UERANSIM. A utilização da versão disponibilizada via Snap evita problemas de compatibilidade durante a compilação. 

---

# 4. Download do Código-Fonte

Faça o download do projeto diretamente do repositório oficial.

```bash
git clone https://github.com/aligungr/UERANSIM
```

### O que este comando faz?

Clona todo o código-fonte do projeto UERANSIM para o computador.

### Impacto no processo

Permite compilar localmente a versão oficial do simulador, incluindo seus arquivos de configuração e exemplos.

---

# 5. Acessar o Diretório do Projeto

```bash
cd UERANSIM
```

### O que este comando faz?

Entra na pasta criada pelo Git Clone.

### Impacto no processo

Todos os comandos de compilação serão executados a partir desse diretório.

---

# 6. Compilação do Projeto

Compile o código-fonte.

```bash
make
```

### O que este comando faz?

Executa automaticamente o processo de compilação do UERANSIM utilizando o arquivo **Makefile** presente no projeto.

Durante essa etapa, o código-fonte é convertido em executáveis capazes de simular o gNodeB e o UE.

### Impacto no processo

Ao final da compilação, será criada a pasta:

```text
build/
```

Nela estarão disponíveis os principais executáveis:

| Executável | Função |
|------------|---------|
| nr-gnb | Simulador do gNodeB (RAN 5G) |
| nr-ue | Simulador do Equipamento de Usuário (UE) |
| nr-cli | Interface de linha de comando para gerenciamento do simulador |
| nr-binder | Ferramenta para associação da conectividade do UE a aplicações Linux |
| libdevbnd.so | Biblioteca utilizada pelo `nr-binder` |

Os programas utilizados durante este processo serão apenas:

- **nr-gnb**
- **nr-ue**

Os demais executáveis são opcionais e destinados a cenários específicos de integração.

---

# 7. Verificação da Compilação

Para verificar se a compilação foi concluída corretamente, liste o conteúdo da pasta `build`.

```bash
ls build
```

### O que este comando faz?

Lista todos os arquivos gerados durante a compilação.

### Impacto no processo

Espera-se encontrar, pelo menos, os seguintes executáveis:

```text
nr-gnb
nr-ue
nr-cli
nr-binder
libdevbnd.so
```

Caso esses arquivos estejam presentes, a compilação foi concluída com sucesso.

---

# Resultado da Etapa

Ao final desta etapa:

- O ambiente possui todas as dependências necessárias para executar o UERANSIM;
- O código-fonte oficial foi obtido diretamente do GitHub;
- O projeto foi compilado com sucesso;
- Os executáveis **nr-gnb** e **nr-ue** estão disponíveis para utilização;
- O ambiente está preparado para a configuração do gNodeB e do UE, que será realizada na próxima etapa.
