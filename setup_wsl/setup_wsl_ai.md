---
titulo: Como configurar um servidor Linux
tags: [linux, servidor, devops, tutorial]
data: 2023-10-27
nivel: iniciante
---

# Manual Completo: Estação de Trabalho de IA no Windows 11

**Configuração de Alta Performance com Dev Drive, WSL2, Docker e GPU**

**Autor:** Leonardo Garcia Marques

**Data:** Dezembro de 2025

**Hardware:** Notebook com NVMe Secundário e NVIDIA RTX 3050.
---

## 1. Infraestrutura de Armazenamento (O Alicerce)
O objetivo é garantir que toda a leitura/escrita pesada (IOPS) ocorra no sistema de arquivos **ReFS**, otimizado para desenvolvimento, poupando o disco do sistema (C:).

### 1.1. Configuração do Dev Drive
1.  Acesse **Configurações > Sistema > Armazenamento > Configurações Avançadas > Discos e Volumes**.
2.  Exclua qualquer volume existente no **Disco 0** (NVMe secundário).
3.  Selecione "Criar uma Unidade de Desenvolvimento" (**Dev Drive**).
4.  **Configurações Obrigatórias:**
    * Letra: **D:**
    * Sistema de Arquivos: **ReFS**
    * Tamanho: Máximo disponível.

---

## 2. Sistema Operacional: WSL (Ubuntu 24.04)
Instalação do Linux e migração física para o disco rápido.

### 2.1. Instalação
No PowerShell (Admin):
```powershell
wsl --install
# Se a janela do Ubuntu não abrir, instale o "Ubuntu 24.04 LTS" pela Microsoft Store.


## 2.2. Migração para Dev Drive (Crítico)

Por padrão, o Linux fica no `C:`. Execute estes comandos para movê-lo para o `D:` (ReFS):

```powershell
# 1. Parar o WSL
wsl --shutdown

# 2. Criar diretório no disco rápido
mkdir D:\WSL

# 3. Backup da instalação atual
wsl --export Ubuntu-24.04 D:\ubuntu-bkp.tar

# 4. Remover do disco lento (C:)
wsl --unregister Ubuntu-24.04

# 5. Importar no disco rápido (D:)
wsl --import Ubuntu-24.04 D:\WSL D:\ubuntu-bkp.tar

# 6. Limpeza
del D:\ubuntu-bkp.tar
```

---

## 2.3. Correção de Usuário e Systemd

A importação reseta o usuário para `root`. Para corrigir:

1. Acesse o WSL:
   ```bash
   wsl -d Ubuntu-24.04
   ```

2. Edite a configuração:
   ```bash
   nano /etc/wsl.conf
   ```

3. Insira (substituindo `seu_usuario`):

   ```ini
   [boot]
   systemd=true

   [user]
   default=seu_usuario
   ```

4. Reinicie o WSL:
   ```powershell
   wsl --shutdown
   ```

---

## 3. Motor de Containers: Docker Desktop

Embora o instalador do Docker Desktop coloque os arquivos de programa (binários) no Disco C: por padrão, o que realmente consome espaço e precisa de velocidade são as imagens e containers. Vamos configurar para que esse "motor" rode inteiramente no seu Dev Drive (D:).

Siga este roteiro para garantir que o Docker aproveite o ReFS do seu Disco 0.

### Passo 1: Preparar o Terreno
Antes de instalar, vamos criar a pasta onde o "cérebro" do Docker vai morar.

1. Abra o Explorador de Arquivos no seu **DevDrive (D:)**.
2. Crie uma nova pasta chamada Docker.
   * Caminho final: D:\Docker

### Passo 2: Instalação Padrão (mas com atenção)
1. Baixe o **Docker Desktop for Windows** no site oficial.
2. Execute o instalador.
3. Tela de Configuração:
   * Use **WSL 2 instead of Hyper-V (Marcado)**: Isso é essencial. É o que vai permitir que o Docker use o seu Ubuntu e o kernel Linux leve, em vez de criar máquinas virtuais pesadas antigas. Isso é crucial para a performance
   * **Allow Windows Containers... (Desmarcado)**: Mantenha desmarcado. Você vai desenvolver em Linux (Python, IA), então não precisa de containers nativos de Windows Server (que são usados para .NET antigo).
   * Add **shortcut (Marcado)**: Apenas cria o atalho, sem problemas.
4. Ao final, o instalador vai pedir para fazer **Log off** do Windows. Faça isso e entre novamente.

Neste momento, o Docker estará rodando, mas ele criou o disco de dados dele no padrão (Disco C:). Não baixe nada ainda! Vamos movê-lo imediatamente.

### Passo 3: O "Move" Estratégico (Levando para o Dev Drive)
Agora vamos dizer para o Docker: "Pare de usar o C: e use o meu ReFS no D:".

1. Abra o **Docker Desktop** (aceite os termos, pule o login se quiser).
2. Clique no ícone de **Engrenagem** (Settings) no topo direito.
3. No menu lateral, vá em *Resources*.
4. Procure a seção **Disk image location**.
   * Você verá que ele está apontando para `C:\Users\leonardo\AppData...`.
5. Clique em **Browse** e selecione aquela pasta que criamos: D:\Docker.
6. Clique no botão **Apply & Restart**.

**O que acontece agora?** O Docker vai desligar o subsistema, pegar o arquivo de disco virtual (`ext4.vhdx`) que ele criou no C:, movê-lo fisicamente para o `D:\Docker` e ligar novamente.
   * **Ganho**: A partir de agora, quando você fizer um docker pull python, os gigabytes serão gravados no seu SSD Kingston (D:), usando a tecnologia de clonagem de blocos do ReFS.

### Passo 4: Conectar com seu Ubuntu (Integração)
O Docker roda isolado. Precisamos avisar que o seu **Ubuntu-24.04** tem permissão para usar o Docker.

1. Ainda nas configurações (**Settings**) do Docker Desktop.
2. Vá em **Resources** > **WSL Integration**.
3. Marque a caixa: **"Enable integration with my default WSL distro"**.
4. Logo abaixo, se aparecer a opção **Ubuntu-24.04**, ative a chave (switch) para deixá-la azul.
5. Clique em **Apply & Restart**.

### Passo 5: Teste Final (A Prova de Fogo)
Vamos ver se toda essa arquitetura (Windows -> WSL -> Docker -> Dev Drive) está conversando.

1. Abra seu terminal do **Ubuntu** (wsl).
2. Digite:
   ```bash
   docker run hello-world
   ```
3. Se aparecer a mensagem *"Hello from Docker!"*, a comunicação está perfeita.


**Dica de Pro**: Vá na pasta `D:\Docker` pelo Windows Explorer. Você verá um arquivo gigante lá (provavelmente `docker-desktop-data.vhdx`). Esse é o arquivo que vai crescer conforme você trabalha, poupando seu disco C: e voando baixo no ReFS.


Após reiniciar o computador, o Docker Desktop deve iniciar automaticamente (ou você pode abri-lo pelo Menu Iniciar). Ele vai te pedir para aceitar os termos de serviço (Service Agreement) — pode aceitar. Se ele pedir para fazer login, pode pular (Skip) por enquanto se não tiver conta.


**Pare!** ✋ Antes de rodar qualquer coisa, vamos fazer a configuração mais importante para salvar seu espaço e ganhar performance.

### 1. Mover o "Motor" do Docker para o Dev Drive (D:)
Por padrão, o Docker acabou de criar um arquivo de disco no seu C:. Vamos mudá-lo para o D: agora.

1. Abra o **Docker Desktop**.
2. Clique na **Engrenagem** ⚙️ (Settings) no canto superior direito.
3. No menu lateral esquerdo, clique em **Resources**.
4. Procure a opção **"Disk image location".**
5. Ela deve estar mostrando algo como C:\Users\leonardo\AppData....
6. Clique no botão **Browse** e selecione a pasta D:\Docker (se você não criou essa pasta ainda, crie agora dentro do seu DevDrive).
7. Clique em **Apply & Restart**.

*O Docker vai levar alguns segundos para mover o arquivo e reiniciar. Aguarde a luz verde no rodapé do aplicativo voltar.*

### 2. Conectar o Docker ao seu Linux (WSL Integration)
Agora o Docker está rodando no disco rápido, mas o seu Ubuntu ainda não sabe que pode usá-lo.

1. Ainda nas Configurações (⚙️) do Docker Desktop.
2. Vá em **Resources** > **WSL Integration**.
3. Garanta que a opção **"Enable integration with my default WSL distro"** está marcada.
4. Logo abaixo, em "Enable integration with additional distros", procure o seu **Ubuntu-24.04**.
5. **Ative a chave (switch)** ao lado dele para ficar azul.
6. Clique em **Apply & Restart**.

### 3. O Teste Final (A Prova Real)
Agora vamos confirmar se tudo funcionou: Windows, Linux e Docker no disco de alta performance.

1. Abra seu terminal do Ubuntu (digite `wsl` no PowerShell).
2. Rode o comando clássico de teste:

```bash
docker run hello-world
```
Se aparecer uma mensagem começando com **"Hello from Docker!"**, parabéns! 🚀

Você tem agora uma estação de trabalho profissional para IA:

* **SO**: Windows 11 (Disco C - NVMe 1)
* **Dados e Projetos**: Dev Drive ReFS (Disco D - NVMe 0)
* **Ambiente**: Ubuntu 24.04 integrado ao Docker, tudo rodando no Dev Drive.

## 4. Ambiente de IA (Python e GPU)

Configuração *blindada* para evitar conflitos de versão (MKL / symbol errors).

### 4.1. Verificação de Drivers

No terminal do Ubuntu, verifique se a RTX 3050 é detectada:

```bash
nvidia-smi
```

---

### 4.2. Instalação do Miniconda

Não use o Python do sistema. Instale o gerenciador Conda:

```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash
```

> **Nota:** Feche e reabra o terminal.

---

### 4.3. Criação do Ambiente de Pesquisa (`ai-lab`)

Criação do ambiente e instalação do PyTorch.

> **Nota:** Usamos `pip` dentro do conda para evitar o erro `iJIT_NotifyEvent`, comum em instalações puras do conda no WSL.

```bash
# 1. Criar ambiente
conda create -n ai-lab python=3.10 -y
conda activate ai-lab

# 2. Instalar Kernel do Jupyter (essencial para o VS Code)
conda install ipykernel -y

# 3. Instalar PyTorch (CUDA 12.x Stable) via pip
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu124
```

---

### 4.4. Script de Teste

Crie um arquivo `teste_gpu.py` ou rode direto no terminal:

```python
import torch

print(f"GPU Disponível? {torch.cuda.is_available()}")
print(f"Modelo: {torch.cuda.get_device_name(0)}")
```

---

## 5. IDE: VS Code + WSL (Modo Remoto)

Como programar no Windows usando o **“cérebro” do Linux**.

1. Abra o **VS Code** no Windows
2. Instale a extensão **WSL** (Microsoft)
3. Conecte-se ao Linux:
   - Clique no botão azul no canto inferior esquerdo (`><`)
   - Selecione **"Connect to WSL"**

### Extensões no Lado Remoto

Na aba de extensões, instale:
- **Python**
- **Jupyter**

> Use o botão **Install in WSL: Ubuntu-24.04**

---

### 5.1. Usando Jupyter Notebooks

1. Abra um arquivo `.ipynb`
2. No canto superior direito, clique em **Select Kernel**
3. Escolha o kernel do Linux:
   ```
   ai-lab (Python 3.10.x)
/home/seu_usuario/miniconda3/envs/ai-lab/bin/python
   ```

---

## 6. Resumo da Estrutura Final

| Componente     | Localização Física                             | Tecnologia          |
|---------------|-----------------------------------------------|---------------------|
| Windows 11    | Disco C: (NVMe 1)                              | NTFS                |
| Linux (Ubuntu)| `D:\WSL\ext4.vhdx`                           | ReFS (Dev Drive)    |
| Docker Data   | `D:\Docker\disk.vhdx`                        | ReFS (Dev Drive)    |
| Projetos/Git  | `\\wsl$\Ubuntu-24.04\home\seu_usuario`   | ReFS (Dev Drive)    |
| Processamento | NVIDIA RTX 3050                                | CUDA (Pass-through) |
