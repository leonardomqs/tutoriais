---
titulo: "Estação de trabalho de IA no Windows 11"
tags: [wsl, windows, docker, gpu, cuda, dev-drive]
nivel: avancado
atualizado: 2026-08-21
---

# Estação de trabalho de IA no Windows 11

Configuração de alta performance com **Dev Drive (ReFS)**, **WSL2**, **Docker** e **GPU NVIDIA**.

> **Hardware de referência:** notebook com NVMe secundário e NVIDIA RTX 3050.
> **Ambiente validado:** Windows 11 Pro + Ubuntu 24.04 LTS.

---

## 1. Infraestrutura de armazenamento (o alicerce)

O objetivo é garantir que toda a leitura/escrita pesada (IOPS) ocorra no sistema de
arquivos **ReFS**, otimizado para desenvolvimento, poupando o disco do sistema (C:).

Premissa: um Disco **D:**, NVMe secundário, com o Windows instalado na unidade **C:**.

> ⚠️ Os passos de 1.1 **apagam tudo** que estiver no disco secundário. Confirme pelo
> tamanho que é o disco certo e que não há nada a recuperar nele.

### 1.1. Limpar o disco secundário

Necessário apenas se o disco já tiver partições. Se ele já estiver como espaço não
alocado, pule para a etapa 1.2.

1. Abra o **Prompt de Comando** ou o **PowerShell** como **Administrador**.
2. Entre no utilitário de disco:

   ```powershell
   diskpart
   ```

3. Já dentro do `diskpart`:

   ```text
   select disk 0     <- confirme pelo tamanho (~465 GB) antes de continuar
   clean
   exit
   ```

Ao final, o disco volta a ser um único bloco de espaço não alocado.

### 1.2. Criar o Dev Drive

1. Acesse **Configurações → Sistema → Armazenamento → Configurações avançadas de
   armazenamento → Discos e volumes**.
2. Se o disco aparecer como "Não inicializado", inicialize-o como **GPT**.
3. Clique em **"Criar unidade de desenvolvimento"**.
4. Configure:

   | Opção | Valor |
   | --- | --- |
   | Letra | **D:** (ou a de sua preferência) |
   | Sistema de arquivos | **ReFS** |
   | Tamanho | O máximo disponível |
   | Rótulo | `DevDrive` |

### 1.3. Redirecionar os caches do pip e do Hugging Face

Por padrão o Python grava downloads temporários no C:, dentro de `AppData`. Apontar esses
caches para o Dev Drive economiza espaço no sistema e acelera as instalações — o que pesa
bastante quando você baixa gigabytes de modelos.

1. Crie as pastas `D:\Cache\Pip` e `D:\Cache\HuggingFace`.
2. No Windows, procure por **"Editar as variáveis de ambiente do sistema"**.
3. Clique em **Variáveis de Ambiente**.
4. Em **Variáveis de usuário**, clique em **Novo** e adicione:

   | Nome | Valor |
   | --- | --- |
   | `PIP_CACHE_DIR` | `D:\Cache\Pip` |
   | `HF_HOME` | `D:\Cache\HuggingFace` |

---

## 2. Sistema operacional: WSL (Ubuntu 24.04)

Instalação do Linux e migração física para o disco rápido.

### 2.1. Instalação

No PowerShell (como Administrador):

```powershell
wsl --install
```

> Se a janela do Ubuntu não abrir sozinha, instale o **Ubuntu 24.04 LTS** pela Microsoft Store.

### 2.2. Migração para o Dev Drive (crítico)

Por padrão o Linux fica no `C:`. Estes comandos movem a distro para o `D:` (ReFS):

> ⚠️ O passo 4 usa `wsl --unregister`, que **apaga a distro e tudo que estiver dentro dela**.
> Só execute depois de confirmar que o `.tar` do passo 3 foi gerado corretamente.

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

### 2.3. Correção de usuário e systemd

A importação reseta o usuário para `root`. Para corrigir:

1. Acesse o WSL:

   ```bash
   wsl -d Ubuntu-24.04
   ```

2. Edite a configuração:

   ```bash
   nano /etc/wsl.conf
   ```

3. Insira, substituindo `seu_usuario`:

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

## 3. Motor de containers: Docker Desktop

O instalador do Docker Desktop coloca os binários no C: por padrão, e isso não é problema.
O que consome espaço e precisa de velocidade são as **imagens e containers** — é esse
"motor" que vamos mover para o Dev Drive.

### 3.1. Preparar o terreno

Antes de instalar, crie a pasta onde os dados do Docker vão morar:

1. Abra o Explorador de Arquivos no seu **Dev Drive (D:)**.
2. Crie uma pasta chamada `Docker`. Caminho final: `D:\Docker`.

### 3.2. Instalar

1. Baixe o **Docker Desktop for Windows** no site oficial.
2. Execute o instalador.
3. Na tela de configuração:

   | Opção | Estado | Por quê |
   | --- | --- | --- |
   | Use WSL 2 instead of Hyper-V | ✅ Marcado | Faz o Docker usar seu Ubuntu e o kernel Linux leve, em vez de VMs pesadas. É o que garante a performance. |
   | Allow Windows Containers | ❌ Desmarcado | Você vai desenvolver em Linux; containers nativos de Windows Server só servem para .NET antigo. |
   | Add shortcut to desktop | ✅ Marcado | Só cria o atalho. |

4. Ao final, o instalador pede para fazer **log off** do Windows. Faça e entre novamente.
5. Ao abrir, aceite os termos de serviço. Se pedir login, pode pular (**Skip**).

> ✋ **Pare aqui.** Neste momento o Docker já criou o disco de dados dele no C:.
> Não baixe nenhuma imagem ainda — mova o motor primeiro, na etapa 3.3.

### 3.3. Mover o motor para o Dev Drive

1. No **Docker Desktop**, clique na engrenagem ⚙️ (**Settings**), no canto superior direito.
2. No menu lateral, vá em **Resources**.
3. Localize **Disk image location** — ele deve estar apontando para
   `C:\Users\<seu_usuario>\AppData...`.
4. Clique em **Browse** e selecione a pasta `D:\Docker`.
5. Clique em **Apply & Restart**.

O Docker desliga o subsistema, move o disco virtual (`ext4.vhdx`) fisicamente para
`D:\Docker` e religa. Aguarde a luz verde voltar no rodapé do aplicativo.

**Ganho:** a partir daqui, todo `docker pull` grava os gigabytes no NVMe do Dev Drive,
usando a clonagem de blocos do ReFS.

### 3.4. Conectar o Docker ao Ubuntu (WSL Integration)

O Docker roda isolado — é preciso autorizar sua distro a usá-lo.

1. Ainda em **Settings**, vá em **Resources** → **WSL Integration**.
2. Marque **"Enable integration with my default WSL distro"**.
3. Em *"Enable integration with additional distros"*, localize **Ubuntu-24.04** e ative a
   chave ao lado (fica azul).
4. Clique em **Apply & Restart**.

### 3.5. Teste final

Confirma toda a cadeia: Windows → WSL → Docker → Dev Drive.

1. Abra o terminal do Ubuntu (digite `wsl` no PowerShell).
2. Rode:

   ```bash
   docker run hello-world
   ```

Se aparecer **"Hello from Docker!"**, a comunicação está correta.

> **Dica:** abra `D:\Docker` pelo Explorador de Arquivos. O arquivo grande que está lá
> (`docker-desktop-data.vhdx`) é o que vai crescer conforme você trabalha — no disco rápido,
> e não no C:.

Neste ponto você tem:

* **SO:** Windows 11 (Disco C — NVMe 1)
* **Dados e projetos:** Dev Drive ReFS (Disco D — NVMe 0)
* **Ambiente:** Ubuntu 24.04 integrado ao Docker, tudo rodando no Dev Drive

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

Como programar no Windows usando o **"cérebro" do Linux**.

### 5.1. Conectar ao WSL

1. Abra o **VS Code** no Windows
2. Instale a extensão **WSL** (Microsoft)
3. Conecte-se ao Linux:
   - Clique no botão azul no canto inferior esquerdo (`><`)
   - Selecione **"Connect to WSL"**

### 5.2. Extensões no lado remoto

Na aba de extensões, instale:
- **Python**
- **Jupyter**

> Use o botão **Install in WSL: Ubuntu-24.04**

---

### 5.3. Usando Jupyter Notebooks

1. Abra um arquivo `.ipynb`
2. No canto superior direito, clique em **Select Kernel**
3. Escolha o kernel do Linux:
   ```text
   ai-lab (Python 3.10.x) /home/seu_usuario/miniconda3/envs/ai-lab/bin/python
   ```



## 6. Resumo da Estrutura Final

| Componente    | Localização Física                      | Tecnologia          |
|---------------|-----------------------------------------|---------------------|
| Windows 11    | Disco C: (NVMe 1)                       | NTFS                |
| Linux (Ubuntu)| `D:\WSL\ext4.vhdx`                      | ReFS (Dev Drive)    |
| Docker Data   | `D:\Docker\docker-desktop-data.vhdx`    | ReFS (Dev Drive)    |
| Projetos/Git  | `\\wsl$\Ubuntu-24.04\home\seu_usuario`  | ReFS (Dev Drive)    |
| Processamento | NVIDIA RTX 3050                         | CUDA (Pass-through) |
