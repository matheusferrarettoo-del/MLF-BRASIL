# MLF BRASIL Remota — interface inicial

Esta pasta contém a primeira versão visual da Central Remota da MLF BRASIL, preparada para publicação como site estático no GitHub Pages.

## Estado desta entrega

- Interface responsiva para desktop, notebook e celular.
- Navegação entre visão geral, Diretor, tarefas, marketplaces, automações, saúde, histórico e configurações.
- Prévia local de conversa e interações demonstrativas.
- Identificação explícita de dados fictícios e ausência de conexão real.
- Não contém backend, autenticação, sincronização, acesso à extensão ou execução de comandos.

## Publicação no GitHub Pages

1. Faça merge desta pasta na branch principal.
2. No repositório, abra **Settings → Pages**.
3. Selecione **Deploy from a branch**, branch `main`, pasta `/ (root)`.
4. A página existente na raiz permanece como landing page. A Central Remota ficará em `/remota/`.

## Próxima etapa: backend seguro

A interface não deve se conectar diretamente ao `chrome.storage.local` de outro computador. A integração deverá usar um serviço intermediário autenticado e um agente local instalado no computador executor.

Contrato sugerido para a integração:

- `GET /api/health` — estado do servidor e do agente executor.
- `GET /api/overview` — indicadores agregados autorizados.
- `GET /api/tasks` — fila de tarefas e estados.
- `POST /api/chat` — enviar mensagem ao Diretor e receber resposta.
- `POST /api/tasks/{id}/approve` — aprovar uma tarefa pendente.
- `POST /api/commands` — encaminhar comando operacional autorizado.
- `GET /api/events` — eventos em tempo real via SSE ou WebSocket.

## Requisitos de segurança para produção

- Autenticação individual e autorização por função.
- HTTPS obrigatório; MFA recomendado.
- Tokens curtos, revogáveis e armazenados somente no backend.
- Nunca incluir tokens de Shopee, Mercado Livre ou provedores de IA no JavaScript do navegador.
- Lista explícita de comandos permitidos e confirmação reforçada para preço, promoções, Ads, publicação em massa e ações irreversíveis.
- Auditoria imutável de usuário, comando, aprovação, executor e resultado.
- Idempotência, expiração de comandos e proteção contra repetição.
- Não expor portas do computador executor diretamente à internet.

## Observação

GitHub Pages hospeda apenas arquivos estáticos. Ele não executa o backend nem mantém o computador executor conectado. A sincronização real exige um backend e um agente local, que serão entregas separadas.
