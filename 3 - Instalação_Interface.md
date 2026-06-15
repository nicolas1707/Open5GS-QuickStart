# Instalação da WebUI do Open5GS

## Objetivo

A WebUI fornece uma interface gráfica para gerenciamento dos assinantes da rede móvel. 

Um assinante representa um dispositivo autorizado a utilizar a rede móvel. Na prática, ele é a representação digital de um SIM Card (chip) dentro do núcleo 5G.

Quando você cria um assinante na WebUI, está dizendo ao Open5GS: "este usuário existe, possui estas credenciais e pode acessar minha rede".

Embora seja possível gerenciar assinantes utilizando ferramentas de linha de comando, a WebUI simplifica significativamente o processo de: 
- Cadastro de assinantes
- Edição de perfis
- Configuração de APNs
- Gerenciamento de autenticação
- Visualização dos dados armazenados no MongoDB

A WebUI se comunica diretamente com o banco de dados utilizado pelo Open5GS. Dessa forma, qualquer assinante criado através da interface gráfica fica imediatamente disponível para utilização pelo núcleo 5G. 

---

# Arquitetura da WebUI

```text
Navegador
     │
     ▼
Open5GS WebUI
     │
     ▼
MongoDB
     │
     ▼
UDR / UDM
```

A WebUI não se comunica diretamente com o AMF ou com o SMF.

Ela atua principalmente como uma interface para manipulação dos dados armazenados no MongoDB.

O UDR armazena permanentemente os dados dos assinantes e da rede, já o UDM gerencia e fornece acesso aos dados armazenados no UDR.

---

# Instalação do Node.js

O Open5GS WebUI é desenvolvido utilizando Node.js.

Antes da instalação da WebUI, é necessário instalar o ambiente de execução Node.js.

## Atualização da lista de pacotes

```bash
sudo apt update
```

### O que este comando faz?

Atualiza os índices dos repositórios configurados no sistema.

### Impacto no processo

Garante que as versões mais recentes dos pacotes possam ser instaladas.

## Instalação das dependências necessárias

```bash
sudo apt install -y ca-certificates curl gnupg
```

### O que este comando faz?

Instala ferramentas necessárias para acessar repositórios externos de forma segura.

Componentes instalados:

- ca-certificates
- curl
- gnupg

### Impacto no processo

Essas ferramentas serão utilizadas para validar a autenticidade do repositório NodeSource.

Sem elas não é possível adicionar o repositório do Node.js conforme recomendado pelo Open5GS.

## Criação do diretório de chaves

```bash
sudo mkdir -p /etc/apt/keyrings
```

### O que este comando faz?

Cria o diretório utilizado pelo APT para armazenar chaves criptográficas de repositórios externos.

### Impacto no processo

Permite armazenar a chave do NodeSource separadamente das demais chaves do sistema.

## Download da chave do NodeSource

```bash
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
```

### O que este comando faz?

Baixa a chave pública oficial do NodeSource e converte para o formato utilizado pelo APT.

### Impacto no processo

Permite verificar a autenticidade dos pacotes Node.js instalados futuramente.

Sem essa etapa, o Ubuntu bloquearia os pacotes por questões de segurança.

## Definição da versão principal do Node.js

```bash
NODE_MAJOR=20
```

### O que este comando faz?

Cria uma variável contendo a versão principal do Node.js que será utilizada.

### Impacto no processo

Define que o sistema utilizará a série 20.x do Node.js.

Isso facilita futuras atualizações da documentação.

## Adição do repositório NodeSource

```bash
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
```

### O que este comando faz?

Adiciona o repositório oficial do NodeSource ao sistema.

### Impacto no processo

Permite instalar versões modernas do Node.js diretamente da fonte oficial.

O repositório padrão do Ubuntu normalmente possui versões mais antigas. 

## Atualização da lista de pacotes

```bash
sudo apt update
```

### O que este comando faz?

Atualiza novamente os índices dos pacotes.

### Impacto no processo

Agora o Ubuntu passa a enxergar os pacotes disponibilizados pelo NodeSource.

## Instalação do Node.js

```bash
sudo apt install nodejs -y
```

### O que este comando faz?

Instala o ambiente de execução Node.js.

### Impacto no processo

O Node.js é necessário para executar o backend da WebUI do Open5GS.

Sem ele a interface gráfica não pode ser instalada.

---

# Instalação da WebUI

## Execução do instalador oficial

```bash
curl -fsSL https://open5gs.org/open5gs/assets/webui/install | sudo -E bash -
```

### O que este comando faz?

Baixa e executa o script oficial de instalação da WebUI do Open5GS.

### Impacto no processo

O script:

- Instala os pacotes necessários
- Configura a aplicação
- Cria o serviço open5gs-webui
- Conecta a WebUI ao MongoDB

Após a conclusão, a interface gráfica estará disponível para acesso. :contentReference[oaicite:8]{index=8}

---

# Verificação do Serviço

## Consultar o estado da WebUI

```bash
systemctl status open5gs-webui
```

### O que este comando faz?

Verifica se o serviço da WebUI está em execução.

---

# Acesso à Interface

Após a instalação, a WebUI pode ser acessada através do navegador.

```text
http://localhost:9999
```

### Função da Interface

Através dela é possível:

- Adicionar assinantes
- Editar assinantes existentes
- Configurar APNs
- Configurar parâmetros de autenticação
- Gerenciar perfis utilizados pelo núcleo 5G

---

# Relação com o 5G Core

Quando um assinante é criado pela WebUI:

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
  │
  ▼
AUSF
  │
  ▼
AMF
```

Dessa forma, as informações cadastradas passam a ser utilizadas durante:

- Registro do UE
- Autenticação
- Criação de sessões PDU
- Aplicação de políticas

---

# Resultado da Etapa

Ao final desta etapa, o ambiente possuirá:

- Node.js instalado;
- Serviço open5gs-webui instalado;
- Interface gráfica operacional;
- Integração com o MongoDB;
- Ambiente preparado para cadastro de assinantes e configuração do núcleo 5G SA.
