---
titulo: "Doppler CLI no WSL: instalação e configuração"
tags: [doppler, wsl, segredos, cli]
nivel: iniciante
atualizado: 2026-04-10
---


# Doppler CLI no WSL: instalação e configuração

Este guia descreve como instalar o Doppler CLI no seu ambiente de desenvolvimento WSL (Ubuntu 24.04 Noble Numbat) para gerenciar segredos de forma segura.

## 1. Instalação via Script Oficial

A forma recomendada é usar o script oficial do Doppler, que gerencia automaticamente a chave GPG, o repositório APT e a instalação do binário:

```bash
curl -Ls --tlsv1.2 --proto "=https" --retry 3 https://cli.doppler.com/install.sh | sudo sh
```

> ⚠️ Não use o método manual de adicionar a chave GPG e o repositório APT separadamente,
> que ainda circula em tutoriais antigos: a URL da chave (`packages.doppler.com/public/cli/gpg...`)
> retorna **404**. Use exclusivamente o script acima.

Após a instalação, confirme que o CLI está disponível:

```bash
doppler --version
```

## 2. Autenticação e Login

Para conectar seu WSL à sua conta do Doppler:

1. Execute o comando de login:
   ```bash
   doppler login
   ```
2. O terminal gerará um código e tentará abrir o navegador no Windows.
3. Certifique-se de estar logado no navegador correto (ex: Edge) para autorizar o acesso.

## 3. Configuração no Projeto

Navegue até a pasta do seu projeto e vincule o ambiente local:

```bash
cd ~/caminho/do/projeto
doppler setup
```

## 4. Automação e Práticas Recomendadas

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
