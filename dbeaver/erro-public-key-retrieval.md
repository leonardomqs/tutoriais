---
titulo: "Erro: Public Key Retrieval is not allowed (MySQL 8 no DBeaver)"
tags: [dbeaver, mysql, docker, erro]
nivel: intermediario
atualizado: 2026-01-04
---

# Erro: Public Key Retrieval is not allowed (MySQL 8 no DBeaver)

Ao tentar conectar no MySQL 8 rodando via Docker, é comum receber o erro:
> *"Public Key Retrieval is not allowed"*

Isso ocorre devido ao plugin de autenticação padrão `caching_sha2_password` do MySQL 8, que exige uma troca de chave pública RSA segura.

## ✅ Como corrigir

Não é necessário alterar nada no Docker ou no banco. A correção é feita no cliente (DBeaver):

1. Na janela de conexão, vá até a aba **Propriedades do driver** (Driver Properties).
2. Localize a propriedade `allowPublicKeyRetrieval`.
3. Altere o valor para **`TRUE`**.
4. (Recomendado) Localize a propriedade `useSSL` e altere para **`FALSE`**.

**Motivo:** Em ambiente de desenvolvimento local (localhost), habilitar essa recuperação de chave permite que o driver complete o "handshake" de segurança sem a necessidade de configurar certificados SSL complexos.