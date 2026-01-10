# Instalação do PowerShell 7

O Windows vem com o **Windows PowerShell 5.1** (o "clássico", de fundo azul). Ele é um produto "legado" que não recebe mais novos recursos, apenas correções de segurança.

É comum aparecer uma mensagem que sugere instalar o **PowerShell 7** (chamado agora apenas de "PowerShell", ou `pwsh`).

## Aqui estão os motivos técnicos pelos quais você deve instalar:

### 1. Performance e Modernidade
O PowerShell 7 é construído sobre o **.NET Core** (moderno), enquanto o antigo roda no .NET Framework.

- Ele é **muito mais rápido** na inicialização e na execução de scripts.
- Ele é multiplataforma (o mesmo `pwsh` roda no seu Windows e dentro do seu Ubuntu-Labs, se você quiser).

### 2. Recurso "Matador": Paralelismo
Como você mexe com Dados e Automação, vai gostar disso: o PowerShell 7 tem o parâmetro `-Parallel` no comando `ForEach-Object`.

- Antigo (5.1): Processa um arquivo por vez (single-thread).
- Novo (7.x): `... | ForEach-Object -Parallel { ... }` (usa todos os núcleos do seu processador). Para processar logs ou arquivos grandes, a diferença é brutal.

### 3. Coexistência (Risco Zero)
Instalar o PowerShell 7 não substitui nem remove o Windows PowerShell 5.1.

- Eles funcionam lado a lado.
- O executável do antigo é `powershell.exe`.
- O executável do novo é `pwsh.exe`.
- Se algum script antigo (legado) quebrar no 7, você sempre pode rodar no 5.1.

## Como instalar da maneira "Dev"

Não precisa ir no site baixar o `.msi.` Como você está no Windows 11, o jeito mais limpo é usar o gerenciador de pacotes (`winget`) no seu terminal atual:

```PowerShell
winget install Microsoft.PowerShell
```

Após instalar, feche e abra o **Windows Terminal** novamente. Você verá que aparecerá uma nova opção no menu de abas chamada apenas **"PowerShell"** (com ícone preto/cinza escuro), separada do "Windows PowerShell" (ícone azul).

**Sugestão:** Instale e comece a usar o novo (`pwsh`) como seu terminal padrão no Windows. A experiência de autocomplete e colorização de sintaxe é superior.

1. Abra o **Windows Terminal**.
2. Acesse as **Configurações** (clique na setinha `⌄` na barra de abas ou use `Ctrl + ,`).
3. Logo na primeira tela que abrir, chamada **Inicialização** (Startup), a primeira opção é **Perfil padrão**.
4. Clique na lista suspensa e selecione o **PowerShell** (aquele que tem o ícone **preto/cinza escuro**, sem o logo azul do Windows).
    - *Nota: O "Windows PowerShell" é o antigo (azul).*
    - *O "PowerShell" é o novo (preto).*
5. Clique no botão Salvar no canto inferior direito.

## Testando
Feche o Terminal completamente e abra-o novamente. Se tudo deu certo, a aba que abrirá automaticamente já será a do PowerShell 7 (você notará que o carregamento é mais rápido e o prompt pode ser ligeiramente diferente).