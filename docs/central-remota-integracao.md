# Central Remota — plano de integração segura

## Objetivo

Conectar a Central Remota ao assistente que já opera no computador principal. Não criar um segundo assistente nem alterar os módulos existentes de SKUs, MDF, custos, Shopee ou Mercado Livre.

## O que existe hoje

- `miniapp/index.html` e `miniapp/app.js`: interface Telegram Mini App.
- A Mini App usa `Telegram.WebApp.sendData()` com um payload JSON versionado: `{ "v": 1, "command": "..." }`.
- `remota/index.html`: painel web responsivo demonstrativo.
- `MLF_MATHEUS`: aplicação web com Firebase Authentication e Cloud Firestore no projeto existente `mlf-decor`.

## O que ainda não está conectado

- Recepção de `web_app_data` pelo bot Telegram (webhook ou processo equivalente).
- Validação criptográfica de `Telegram.WebApp.initData` no servidor.
- Serviço autenticado para registrar e consultar comandos.
- Agente local no computador executor.
- Adaptador que entregue a mensagem ao assistente atual e devolva a resposta.
- Estados reais de conexão, execução e histórico na interface.

A chamada `sendData()` apenas entrega os dados ao bot no fluxo Telegram; ela não envia o comando diretamente ao PC nem ao Firebase.

## Arquitetura-alvo

```text
Telegram Mini App / Painel web
          |
          | HTTPS + identidade validada
          v
Backend da Central Remota
          |
          | Firestore: fila e eventos por usuário
          v
Agente local no PC (conexão de saída)
          |
          | adaptador para o assistente já existente
          v
Assistente atual / extensão / sistema MLF
          |
          | resultado + logs
          v
Backend -> Firestore -> Mini App / Painel
```

O computador executor inicia a conexão de saída. Não publicar portas locais do computador na internet.

## Contrato inicial de comando

Payload recebido da Mini App:

```json
{
  "v": 1,
  "command": "verifique os anúncios com problemas..."
}
```

O backend deve criar um registro com campos equivalentes a:

```json
{
  "ownerUid": "UID autenticado",
  "source": "telegram",
  "command": "texto do usuário",
  "status": "pending",
  "createdAt": "server timestamp",
  "expiresAt": "timestamp com validade curta",
  "idempotencyKey": "identificador único",
  "requiresApproval": false
}
```

Estados permitidos: `pending`, `claimed`, `awaiting_approval`, `running`, `succeeded`, `failed`, `cancelled`, `expired`.

O agente deve reivindicar o comando atomicamente, validar usuário, validade e lista de ações permitidas, e registrar início, resultado e erro. Comandos expirados ou já reivindicados não podem ser executados novamente.

## Proteções obrigatórias

1. Nunca confiar em `initDataUnsafe` como prova de identidade. O backend valida a assinatura de `initData` usando o token do bot, verifica `auth_date` e aplica proteção contra repetição.
2. Não aceitar `ownerUid`, papel, permissões ou estado de aprovação informados pelo navegador. O servidor deriva a identidade após validar a sessão.
3. Firestore Security Rules com menor privilégio e isolamento por UID. O navegador não pode ler ou gravar filas de outros usuários nem se atribuir papel mestre.
4. Segredos do bot, credenciais de marketplace e chaves privadas ficam somente no backend/agente seguro, nunca em arquivos estáticos, GitHub Pages ou JavaScript público.
5. Começar com comandos de consulta. Alterações de preço, anúncios, campanhas, Ads, exclusões e operações em massa exigem aprovação explícita e auditável.
6. Registrar autor, origem, comando, aprovação, agente executor, horários, resultado e identificador de idempotência.
7. HTTPS obrigatório, tokens curtos/revogáveis, limites de frequência e tamanho, expiração de comandos e logs sem credenciais.
8. Não mudar regras atuais do Firestore sem antes obter a versão implantada, mapear todos os caminhos utilizados e testar no Emulator Suite.

## Coleções novas sugeridas (não criar em produção sem revisar as regras)

- `remote_commands/{commandId}`: fila de comandos com proprietário, estado e expiração.
- `remote_agents/{agentId}`: presença, versão e heartbeat do agente, com leitura restrita.
- `remote_events/{eventId}`: trilha de auditoria e eventos exibíveis no painel.

Os documentos devem incluir `ownerUid`; as regras devem limitar cada usuário ao próprio escopo. A autorização mestre deve vir de uma fonte confiável do servidor ou de custom claims, não de um campo que o próprio cliente possa editar.

## Integração com o assistente existente

Antes de implementar o adaptador, localizar no código do agente/extensão atual:
- ponto de entrada usado para receber comandos;
- formato de mensagens e contexto de conversa;
- mecanismo atual de confirmação de ações sensíveis;
- como respostas, erros e anexos são devolvidos.

O adaptador remoto deve chamar o mesmo ponto de entrada já usado pelo assistente local. Não duplicar prompt, histórico, ferramentas ou lógica de execução.

## Próximos arquivos necessários para concluir a integração

- Código-fonte/ZIP da extensão Chrome atual (incluindo `manifest.json`, service worker/background, scripts e módulos do assistente).
- Código do bot Telegram ou informação sobre onde está hospedado e como recebe atualizações.
- Regras Firestore atualmente implantadas (`firestore.rules`) e, se existirem, `firebase.json` e código de Functions.

Sem esses arquivos, não é seguro inventar o adaptador nem alterar o banco real. Esta especificação é documentação de arquitetura, não uma implementação funcional do backend.
