# Tasks 001 — Feedback diário via WhatsApp

> Spec: `specs/001-feedback-diario/spec.md` · Plano: `specs/001-feedback-diario/plan.md`
> Infra existente: n8n e Evolution API (v2.3.7, instância `ICASST`, conectada).
> Formato: `- [ ] Tnnn [P?] descrição`. `[P]` = pode ser feita em paralelo com a anterior.

---

## Decisões fechadas (atualizam as pendências do plano)

| Pendência do plano | Decisão |
|---|---|
| Redação | **Template fixo**, sem IA. |
| Query no Linear | **Sem filtro de projeto nem de time** — todas as issues atribuídas a mim (cobre Unimed e projetos pessoais). |
| Concluído | Issues minhas com `completedAt` dentro da janela de apuração. |
| Em andamento | Issues minhas em status do tipo `started` (**In Progress** e **In Review**). |
| Próximos | Issues minhas em **Todo**, mais as do **Backlog** com a label **Improvement**. |
| Feriados | **Lista fixa no código**, revisada uma vez por ano. |

Nomes conferidos no Linear em 2026-10-07: os times Nexa e univsf têm os mesmos status (Backlog, Todo, In Progress, In Review, Done, Canceled, Duplicate). A label se chama **Improvement**, no singular.

---

## Fase 1 — Credenciais e prova de envio

- [ ] T001 Gerar uma API key pessoal no Linear (Settings › Security & access › Personal API keys) e cadastrar no n8n como credencial **Header Auth** (`Authorization: <key>`). Nunca colar a key dentro de nó.
- [ ] T002 [P] Cadastrar no n8n a credencial da Evolution API como **Header Auth** (`apikey: <key da instância>`).
- [ ] T003 Prova de envio isolada: workflow descartável com um nó HTTP Request enviando "teste" para o seu número pela instância `ICASST`. Conferir na documentação da v2.3.7 o endpoint e o corpo exatos (esperado: `POST /message/sendText/ICASST` com `number` e `text`). **Só seguir para a Fase 2 com a mensagem recebida.**
- [ ] T004 [P] Guardar o número de destino numa variável do n8n (`SN_DESTINO`), não no nó. Na fase de testes, é o seu número; o grupo de DEVs entra trocando só essa variável.

## Fase 2 — O fluxo (workflow `SN-001 Feedback diário`, criado **inativo**)

- [ ] T005 Nas configurações do workflow, timezone **America/Recife**. Nó **Schedule Trigger**: cron `0 8 * * 1-5`.
- [ ] T006 Nó **Code "é dia de envio?"**: se hoje está na lista de feriados, encerra sem enviar. Lista inicial:
  - Nacionais 2026–2027: 01/01, 21/04, 01/05, 07/09, 12/10, 02/11, 15/11, 20/11, 25/12; Sexta-feira Santa em **03/04/2026** e **26/03/2027**.
  - Pernambuco: 06/03 (Data Magna).
  - Petrolina: 24/06, 15/08, 21/09.
  - Comentário no topo do nó: "revisar todo dezembro; datas móveis por ano".
- [ ] T007 Nó **Code "janela de apuração"**: recua a partir de ontem, pulando sábado, domingo e feriado, até achar o último dia útil. Saída: `inicio` e `fim` em ISO (00:00 e 24:00 de Recife convertidos para UTC). Aceita uma **data de referência opcional** para teste; sem ela, usa hoje.
- [ ] T008 Nó **HTTP Request "Linear"**: `POST https://api.linear.app/graphql`, **uma única query** com três blocos (aliases): `concluidas`, `andamento`, `proximos`, todos filtrados por "atribuída a mim", com os critérios da tabela de decisões. Campos: `identifier`, `title`. Validar a sintaxe dos filtros no explorer da API do Linear antes de colar no nó, principalmente o filtro de label do bloco `proximos`.
- [ ] T009 Nó **IF "tem conteúdo?"**: segue se qualquer um dos três blocos tiver ao menos um item; senão, encerra sem enviar.
- [ ] T010 Nó **Code "template"**: monta a mensagem. Seção vazia mostra "Nenhum item". Formato:

  ```
  *Feedback — ter, 06/10*

  *Concluído*
  • NEX-12 Título da issue

  *Em andamento*
  • UNIVSF-40 Título da issue

  *Próximos*
  Nenhum item
  ```

  A data no título é a da **janela de apuração**, não a de hoje.
- [ ] T011 Nó **HTTP Request "WhatsApp"**: o mesmo envio validado no T003, com `number` = `SN_DESTINO` e `text` = saída do template.

## Fase 3 — Validação

- [ ] T012 Rodar manualmente com a data de referência do T007 em quatro casos, conferindo a janela calculada e se houve ou não envio:
  1. uma **segunda** comum → janela é a sexta;
  2. a segunda depois de uma sexta feriado (ex.: **06/04/2026**; 03/04 foi Sexta-feira Santa) → janela é a quinta 02/04;
  3. um **feriado** (ex.: 20/11/2026) → não envia;
  4. uma data sem nenhuma issue na janela e nada em andamento ou próximo → não envia.
- [ ] T013 Ativar o workflow e acompanhar **três dias úteis seguidos** no seu número. Registrar qualquer divergência como issue no Linear.

## Fase 4 — Encerramento

- [ ] T014 Atualizar a seção de pendências do `plan.md` com a tabela de decisões acima.
- [ ] T015 Mover a issue da Spec 001 no Linear para Done. A ida para o grupo de DEVs é outra decisão, fora desta spec.

---

## Fora destas tarefas

- PostgreSQL, Redis e Chatwoot: não entram.
- Limite de itens por seção: sem limite por ora. Se a mensagem ficar longa demais na prática, decidir o corte numa revisão.
- Carnaval e Corpus Christi **não** estão na lista: são ponto facultativo nacional, não feriado. Se Petrolina os decretar como feriado municipal, entram na revisão anual.