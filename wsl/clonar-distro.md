---
titulo: "Clonar uma distro WSL para estudos"
tags: [wsl, windows, backup]
nivel: intermediario
atualizado: 2026-01-09
---

# Clonar uma distro WSL para estudos

## 1. Checando imagens existentes
```powershell
wsl --list --verbose
```

(ou a versão abreviada: `wsl -l -v`)

## 2. Identifique o nome na coluna "NAME"

Esse comando resultará em algo assim:

```text
 NAME              STATE           VERSION
* Ubuntu-24.04      Stopped         2
  docker-desktop    Stopped         2
```

- O nome que você precisa é exatamente o que aparece na primeira coluna.
- Pode ser um nome personalizado se você já tiver alterado antes. 
  - **Dica:** O asterisco  indica qual é a sua distribuição padrão (a que abre quando você digita apenas wsl).

## 3. Monte o comando de exportação

Agora que você tem o nome (vamos supor que seja `Ubuntu-24.04`), você substitui ele no comando de exportação.

```powershell
wsl --export Ubuntu-24.04 "D:\WSL\backup_base.tar"
```

## Uma recomendação extra (Boa prática)

Antes de rodar o comando de exportação, é saudável (embora não obrigatório) garantir que a distribuição esteja parada para evitar copiar arquivos que estão sendo modificados naquele exato segundo.

Você pode forçar a parada de tudo com: `wsl --shutdown`

E logo em seguida rodar o comando de exportação.

## 4. Criação da nova distro

### 1. Crie a pasta de destino
```powershell
mkdir "D:\WSL\Ubuntu-Labs"
```

### 2. Importe a Distro
Agora, vamos importar o backup para essa nova pasta com o nome que você escolheu. *Atenção aos caminhos: verifique se o arquivo `.tar` está mesmo no local indicado abaixo.*

```powershell
wsl --import Ubuntu-Labs "D:\WSL\Ubuntu-Labs" "D:\WSL\backup_base.tar"
```

- **O que esse comando faz:** Ele pega o seu backup, descompacta e cria um novo disco virtual (`ext4.vhdx`) dentro da pasta `D:\WSL\Ubuntu-Labs`. Como essa pasta está no Dev Drive, você terá a performance acelerada.

### 3. Ajuste o usuário (caso necessário)
Por padrão, distros importadas logam como **root**. Para voltar a usar seu usuário normal (o mesmo que você já usa no trabalho), faça o seguinte:

1. Entre na nova distro:
    ```PowerShell
    wsl -d Ubuntu-Labs
    ```

2. Já dentro do Linux (você verá que o prompt está como # root), crie/edite o arquivo de configuração:

    ```Bash
    nano /etc/wsl.conf
    ```

3. Adicione (ou edite) estas linhas, substituindo `seu_usuario` pelo seu login real:

   ```ini
   [boot]
   systemd=true

   [user]
   default=seu_usuario
   ```
4. Salve (`Ctrl+O`, `Enter`) e saia (`Ctrl+X`).

5. Saia do Linux e reinicie a distro para aplicar:
    ```PowerShell
    exit
    wsl --terminate Ubuntu-Labs
    wsl -d Ubuntu-Labs
    ```

## Informações da distro 

Para saber em qual instalação você se encontra, basta utilizar

```bash
echo $WSL_DISTRO_NAME
```

Se você quiser ver detalhes sobre a versão do Linux em si (para confirmar se é Ubuntu, Debian, etc), use:

```bash
cat /etc/os-release
```

## Dica de Ouro: Diferenciação Visual

Já que você terá dois terminais "gêmeos", é muito fácil se confundir e rodar um comando de teste no ambiente de trabalho (ou vice-versa).

Recomendo fortemente que você mude o **Hostname** ou a **Cor do Prompt** da `Ubuntu-Labs` para bater o olho e saber onde está.

### Para mudar o nome que aparece no terminal (ex: `usuario@UBUNTU-LABS`):

1. Dentro da Ubuntu-Labs, edite o arquivo de configuração do WSL:

    ```Bash
    sudo nano /etc/wsl.conf
    ```
2. Adicione estas linhas no final:
   ```ini
    [network]
    hostname = ubuntu-labs
   ```

   2.1 Pode ocorrer de a seção `[network]` já existir no arquivo, com alguma outra informação. Nesse caso, apenas acrescente o `hostname`, como no exemplo a seguir:

    ```ini
        [network]
        generateResolvConf = false
        hostname = ubuntu-labs
    ```

3. Salve, saia e reinicie a distro (`wsl --terminate Ubuntu-Labs`).

Na próxima vez que entrar, seu prompt provavelmente mudará de `usuario@NomeDoPC` para `usuario@ubuntu-labs`, evitando acidentes!