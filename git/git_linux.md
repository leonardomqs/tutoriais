---
titulo: Como configurar git no ambiente linux
tags: [git, linux, tutorial]
data: 20225-12-30
nivel: iniciante
---

# CONFIGURAÇÃO DO GIT EM AMBIENTE LINUX

## 1. Instalar o Git

```bash
sudo apt update
sudo apt install git -y
```

Verifique a versão instalada:

```bash
git --version
```

## 2. Configurar usuário e e-mail

Essas informações vão aparecer nos commits:

```bash
git config --global user.name "Seu Nome"
git config --global user.email "seuemail@exemplo.com"
```

Verifique:

```bash
git config --list
```
## 3. Gerar chave SSH (recomendado para GitHub)

O GitHub recomenda autenticação por **SSH** (mais seguro do que HTTPS com senha).

**Gerar uma chave:**

```bash
ssh-keygen -t ed25519 -C "seuemail@exemplo.com"
```

Se seu sistema não suportar ed25519, use rsa:

```bash
ssh-keygen -t rsa -b 4096 -C "seuemail@exemplo.com"
```

Quando pedir o local para salvar, apenas pressione Enter.
Você pode colocar uma senha se quiser (opcional).

## 4. Adicionar chave SSH ao agente

Ative o **ssh-agent**:

```bash
eval "$(ssh-agent -s)"
```

Adicione sua chave:

```bash
ssh-add ~/.ssh/id_ed25519
```

(se tiver usado rsa, troque o nome do arquivo)


## 5. Copiar chave pública para o GitHub

Copie a chave:

```bash
cat ~/.ssh/id_ed25519.pub
```
Ou:

```bash
xclip -sel clip < ~/.ssh/id_ed25519.pub
```

(se tiver o xclip instalado, já copia para a área de transferência)

Depois, vá no GitHub:


* Perfil → **Settings** → **SSH and GPG keys** → **New SSH key**
* Cole a chave lá.

## 6. Testar a conexão

```bash
ssh -T git@github.com
```

Se tudo der certo, você verá algo como:

```vbnet
Hi seu-usuario! You've successfully authenticated, but GitHub does not provide shell access.
```

## 7. Clonar ou conectar repositórios

Agora você pode clonar repositórios via SSH:

```bash
git clone git@github.com:usuario/repositorio.git
```

Ou, se já tem um repositório local, conectar ao GitHub:

```bash
git remote add origin git@github.com:usuario/repositorio.git
```

E enviar:

```bash
git push -u origin main
```

(se a branch principal for master, substitua main por master).