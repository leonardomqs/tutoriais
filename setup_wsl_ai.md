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

Configuração para rodar imagens e containers no disco secundário.

1. Instale o **Docker Desktop**
   - Marque: **"Use WSL 2 instead of Hyper-V"**

2. Antes de baixar imagens:
   - Vá em **Settings > Resources > Disk image location**
   - Mude de `C:\...` para `D:\Docker` (crie a pasta antes)
   - Clique em **Apply & Restart**

3. Integração:
   - Vá em **Resources > WSL Integration**
   - Ative a chave para **Ubuntu-24.04**

---

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
