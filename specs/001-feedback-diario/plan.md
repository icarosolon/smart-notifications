# Plan 001 — Feedback diário de atividades no WhatsApp

Plano técnico (o "como") da Spec 001. Pressupõe a spec escrita e checada.

## Visão geral do fluxo
Um único fluxo no n8n, disparado por agendamento, que: verifica se hoje é dia de envio, calcula a janela de apuração, busca as atividades no Linear, monta a mensagem, e entrega no WhatsApp via Evolution API — só se houver conteúdo.

## Stack desta feature
- **n8n** — orquestração e agendamento.
- **Linear** (API) — fonte das atividades.
- **Evolution API** — envio ao WhatsApp.
- **Nó de código (JavaScript)** no n8n — cálculo determinístico do dia útil/feriados.
- Sem PostgreSQL, Redis ou Chatwoot nesta feature — não há necessidade concreta ainda.

## Passos do fluxo (nós do n8n)

1. **Gatilho de agendamento (Schedule Trigger)**
   - Dispara 08:00, fuso America/Recife.
   - Roda de segunda a sexta (o filtro fino de feriado é no passo 2).

2. **Nó de código — é dia de envio?**
   - Determinístico, não usa IA.
   - Confere se hoje é dia útil (seg–sex e não feriado).
   - Feriados: nacionais + Pernambuco (Data Magna 06/03) + Petrolina (São João Batista 24/06; N. Sra. Rainha dos Anjos 15/08; aniversário da cidade 21/09). Juazeiro-BA não entra.
   - Se não for dia útil, encerra o fluxo sem enviar.

3. **Nó de código — janela de apuração (último dia útil)**
   - Calcula o último dia útil anterior a hoje, recuando sobre fim de semana e feriado.
   - Produz a data (ou intervalo) usada na consulta ao Linear.

4. **Consulta ao Linear**
   - Busca: concluídas no último dia útil; em andamento; próximos passos (previstas).
   - Filtra pelo time/projeto correto.

5. **Nó de decisão — tem conteúdo?**
   - Se as três partes estão vazias, encerra sem enviar.
   - Caso contrário, segue.

6. **Redação da mensagem**
   - Monta o texto com três seções tituladas: Concluído, Em andamento, Próximos passos.
   - Formatação legível para WhatsApp.
   - (Opção: nó de IA só para redigir o texto a partir dos dados — decidir no detalhamento.)

7. **Envio via Evolution API**
   - Destinatário: número pessoal do Ícaro (fase de testes).
   - Trocar pelo ID do grupo só na fase 2, fora desta spec.

## Decisões e trade-offs
- **Cálculo de dia útil em código, não em IA:** regra crítica que não pode variar; determinismo garante repetibilidade.
- **Sem banco nesta feature:** não há estado a persistir hoje. Se no futuro precisar evitar envio duplicado ou guardar histórico, aí entra o PostgreSQL.
- **Redação por IA é opcional:** dá texto mais natural, mas adiciona dependência e custo. Decidir no detalhamento se compensa agora ou se um template fixo basta.

## Riscos
- Indisponibilidade do Linear ou da Evolution API às 08:00.
- Calendário de feriados incorreto quebra o cálculo do dia útil.
- Mudança de fuso/horário de verão (hoje não há horário de verão no Brasil, mas registrar a premissa).

## Pendências para o detalhamento (tasks)
- Definir se a redação usa IA ou template fixo.
- Confirmar a query exata no Linear (status, time, projeto).
- Definir a fonte do calendário de feriados (lista fixa no código vs. API).
