# Insync no Linux — Documentação de Uso e Configuração

## 1. O que é o Insync
O Insync é um cliente de sincronização nativo para Linux que permite integrar
**OneDrive**, **Google Drive** e **Dropbox** diretamente ao sistema de arquivos,
funcionando de forma semelhante aos clientes oficiais do Windows e macOS.

No Linux, ele supre a ausência de um cliente oficial do OneDrive, oferecendo:
- Sincronização bidirecional
- Integração direta com o filesystem
- Controle fino de pastas sincronizadas
- Resolução configurável de conflitos

---

## 2. Modelo de licenciamento
- **Licença vitalícia (lifetime)** — pagamento único
- Inclui **1 ano de atualizações e suporte** (Insync Care)
- Após 1 ano:
  - O software continua funcionando normalmente
  - Atualizações e suporte exigem renovação opcional do Care
- Licença vinculada à conta do usuário (não transferível)

---

## 3. Conceito de sincronização
O Insync mantém duas cópias logicamente equivalentes:
- **Cópia local** (Linux Mint)
- **Cópia na nuvem** (OneDrive)

Qualquer ação em um lado pode refletir no outro:
- Criar
- Editar
- Excluir
- Desvincular da sincronização (unsync)

Quando ações incompatíveis ocorrem simultaneamente, surgem **conflitos**.

---

## 4. Tipos de conflitos e impacto prático

### 4.1 Conflito por modificação simultânea
**Cenário:**  
O mesmo arquivo foi editado localmente e na nuvem antes da sincronização.

**Riscos:**
- Sobrescrita silenciosa
- Perda de versões intermediárias

**Opções:**
- `Always ask`  
  O usuário decide manualmente qual versão manter.
- `Keep local changes and upload`  
  A versão local sobrescreve a da nuvem.
- `Keep cloud changes and download`  
  A versão da nuvem sobrescreve a local.

**Recomendação (usuário vindo do Windows):**  
`Keep cloud changes and download`

---

### 4.2 Conflito: edição local vs. exclusão na nuvem
**Cenário:**  
O arquivo foi apagado no OneDrive, mas editado localmente.

**Riscos:**
- Recriar arquivos deletados intencionalmente
- Perder edições recentes

**Opções:**
- `Always ask`
- `Keep local changes and upload the file to the cloud`
- `Remove both local and cloud copy of the file`

**Recomendação:**  
`Always ask`

---

### 4.3 Conflito: edição local vs. unsync
**Cenário:**  
O arquivo foi editado localmente, mas removido da sincronização (unsync).

**Riscos:**
- Perda de alterações locais
- Arquivos ficarem fora da nuvem sem intenção

**Opções:**
- `Always ask`
- `Keep local changes and upload the file to the cloud`
- `Remove local copy of the file and keep the cloud copy as is`

**Recomendação:**  
`Keep local changes and upload the file to the cloud`

---

### 4.4 Conflito: exclusão local vs. edição na nuvem
**Cenário:**  
O arquivo foi apagado localmente, mas editado na nuvem.

**Riscos:**
- Perda de alterações feitas em outro dispositivo
- Divergência de histórico

**Opções:**
- `Always ask`
- `Remove both local and cloud copy of the file`
- `Keep cloud changes and download the file locally`

**Recomendação:**  
`Keep cloud changes and download the file locally`

---

## 5. Estratégia recomendada (perfil consolidado)

**Perfil:**  
Usuário com histórico longo de uso do OneDrive no Windows.

**Princípios adotados:**
- A nuvem é a fonte principal
- Alterações locais não devem ser descartadas sem confirmação
- Exclusões exigem cautela

**Configuração final sugerida:**
- Modificação simultânea → Keep cloud changes and download
- Edição local + exclusão na nuvem → Always ask
- Edição local + unsync → Keep local changes and upload
- Exclusão local + edição na nuvem → Keep cloud changes and download

---

## 6. Reset e recuperação

### 6.1 Reset completo do Insync
Restaura o estado inicial do aplicativo.

```bash
insync quit
rm -rf ~/.config/Insync
insync start
```

**Efeitos**:

- Remove contas configuradas
- Remove preferências e regras de conflito
- Não apaga arquivos locais

---

## 7. Boas práticas

- Realize o primeiro processo de sincronização com atenção, pois ele pode levar bastante tempo dependendo do volume de dados.
- Evite editar, mover ou excluir grandes quantidades de arquivos durante o sync inicial.
- Utilize a opção `Always ask` sempre que houver qualquer dúvida envolvendo exclusões.
- Em caso de comportamento inesperado, valide o estado do arquivo diretamente pela interface web do OneDrive.
- Prefira testar novas configurações em pastas menores antes de aplicá-las ao repositório completo.

---

## 8. Observações operacionais importantes

- O Insync **não é um backup**: ele espelha estados entre o local e a nuvem.
- Uma exclusão confirmada pode se propagar para ambos os lados.
- O histórico de versões depende das políticas do próprio OneDrive.
- Alterações feitas offline serão sincronizadas assim que a conexão for restabelecida.
- Conflitos são resolvidos com base nas regras configuradas previamente.

---

## 9. Estratégias de mitigação de risco

- Mantenha a lixeira do OneDrive habilitada.
- Evite usar múltiplos clientes de sincronização simultaneamente.
- Faça backups periódicos fora da pasta sincronizada.
- Antes de grandes reorganizações de pastas, pause o Insync.
- Após alterações estruturais, reative o sync e monitore os primeiros ciclos.

---

## 10. Conclusão

O Insync é uma solução madura e funcional para integração do OneDrive no Linux.
Quando corretamente configurado, permite uma transição segura do Windows para o
Linux sem comprometer integridade, histórico ou consistência dos dados.

A adoção consciente das regras de conflito e das boas práticas operacionais é
fundamental para evitar perdas acidentais de informação.
