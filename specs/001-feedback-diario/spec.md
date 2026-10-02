# Spec 001 — Feedback diário de atividades no WhatsApp

## Contexto e problema
Hoje o feedback diário das atividades do time é montado e enviado manualmente, copiando e colando informação do Linear todo dia útil. Consome tempo, depende de lembrar de fazer, e está sujeito a erro humano e esquecimento. O objetivo é que esse feedback seja gerado e entregue automaticamente toda manhã.

## Objetivo
Entregar, de forma automática e diária, um resumo das atividades cobrindo o que foi concluído no último dia útil, o que está em andamento e os próximos passos, tendo o Linear como fonte da verdade.

## Escopo
**Entra:**
- Geração automática do resumo a partir do Linear.
- Entrega no WhatsApp.
- Na fase de testes, entrega apenas no número pessoal do Ícaro.

**Não entra (por enquanto):**
- As outras quatro categorias: agenda, clima, notícias e devocional.
- Envio para o grupo de desenvolvedores (só após a validação).

## Regras de negócio
- **Horário:** envio às 08:00 em ponto.
- **Dias de envio:** apenas dias úteis. Em feriado, não envia.
- **Dia útil:** segunda a sexta, exceto feriados nacionais, estaduais de Pernambuco e municipais de Petrolina. Juazeiro-BA não entra.
- **Janela de apuração (último dia útil):** o feedback olha para o último dia útil anterior ao envio. Na segunda, olha a sexta; se a véspera foi fim de semana ou feriado, recua até o último dia útil.
- **Envio condicional ao conteúdo:** a mensagem tem três partes — concluído, em andamento e próximos passos. Envia se houver conteúdo em qualquer uma delas. Se as três estiverem vazias, não envia nada naquele dia.

## Critérios de aceitação
- A mensagem é enviada automaticamente às 08:00 em todo dia útil, sem ação manual.
- Em feriados e fins de semana, nenhuma mensagem é enviada.
- O conteúdo reflete corretamente as três partes: concluído (do último dia útil), em andamento e próximos passos.
- O cálculo do último dia útil respeita fins de semana e feriados conforme as regras acima.
- Quando não há nada concluído mas há algo em andamento ou previsto, a mensagem é enviada com as partes disponíveis.
- Quando as três partes estão vazias, nenhuma mensagem é enviada.
- Formatação mínima: as três seções com título claro e itens em lista legível no WhatsApp.

## Fora de escopo e riscos
- Fica de fora qualquer dado financeiro — por decisão do projeto, não trafega pelo WhatsApp.
- Risco: dependência da disponibilidade do Linear e da API do WhatsApp no horário do disparo.
- Risco: precisão do calendário de feriados, que impacta diretamente o cálculo do dia útil.
