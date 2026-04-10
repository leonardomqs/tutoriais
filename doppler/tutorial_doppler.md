# Tutorial: Instalação e Configuração do Doppler CLI (WSL Ubuntu 24.04)

Este guia descreve como instalar o Doppler CLI no seu ambiente de desenvolvimento WSL (Ubuntu 24.04 Noble Numbat) para gerenciar segredos de forma segura.

## 1. Instalação das Dependências

Antes de adicionar o repositório, garanta que o sistema possui as ferramentas necessárias para lidar com chaves GPG e repositórios HTTPS.

```bash
sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
```

## 2. Adição da Chave GPG e Repositório Oficial

No Ubuntu 24.04, é recomendado armazenar as chaves em `/usr/share/keyrings` para maior segurança e evitar avisos de "legacy keyring".

Adicione a chave GPG do Doppler:

```bash
curl -sLf --retry 3 --retry-delay 3 https://packages.doppler.com/public/cli/gpg.823B70958367733E.key \
  | sudo gpg --dearmor -o /usr/share/keyrings/doppler-archive-keyring.gpg
```

Adicione o repositório oficial à lista de fontes do APT:

```bash
echo "deb [signed-by=/usr/share/keyrings/doppler-archive-keyring.gpg] https://packages.doppler.com/public/cli/deb/debian any-version main" \
  | sudo tee /etc/apt/sources.list.d/doppler-cli.list
```

## 3. Instalação do CLI

Atualize a lista de pacotes e instale o binário:

```bash
sudo apt-get update && sudo apt-get install -y doppler
```

## 4. Autenticação e Login

Para conectar seu WSL à sua conta do Doppler:

1. Execute o comando de login:
```bash
   doppler login
```
2. O terminal gerará um código e tentará abrir o navegador no Windows.
3. Certifique-se de estar logado no navegador correto (ex: Edge) para autorizar o acesso.

## 5. Configuração no Projeto

Navegue até a pasta do seu projeto e vincule o ambiente local:

```bash
cd ~/caminho/do/projeto
doppler setup
```

## 6. Automação e Práticas Recomendadas

### Uso com Python/FastAPI

Sempre execute seus scripts através do Doppler para injetar as variáveis de ambiente sem precisar de arquivos `.env` locais:

```bash
doppler run -- python main.py
```

### Dica para o PATH

Para que seus scripts personalizados funcionem de qualquer lugar, adicione sua pasta de scripts ao seu `~/.bashrc`:

```bash
echo 'export PATH="$HOME/scripts:$PATH"' >> ~/.bashrc
source ~/.bashrc
```