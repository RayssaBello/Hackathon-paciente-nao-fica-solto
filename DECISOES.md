# DECISOES — registro de decisão

> O quê, o porquê e a fonte. As decisões de implementação menores estão na tabela "Decisões tomadas" do `README.md`.

## D000 — 14/11 · Perfil do hackathon

- **Decisão:** operar com o briefing do time como fonte; o que depende do edital fica `[VERIFICAR]` em `CONTEXTO.md`.
- **Fonte:** briefing de construção do MVP (mensagem do time, 14/11).
- **Reversível?** Sim. Quando a rubrica chegar, ajustar `PITCH.md`.

## D001 — 14/11 · Metodologia da skill adaptada para um agente só

- **Decisão:** usar da skill `hackathon` o que protege o demo (contexto, spec com aceite testável, teste antes de "pronto", decisões com fonte, audit, pitch). Não usar worktrees, branches por cartão, integrador nem PRs.
- **Motivo:** um agente em um checkout não gera colisão; a própria skill diz que nesse caso é cerimônia.
- **Fonte:** `.claude/skills/hackathon/README.md`, seção "Quando usar / não usar".
- **Afeta:** se mais pessoas ou agentes entrarem, rodar `git init`, criar o remoto e passar a seguir as leis 1, 2 e 6.

## D002 — 14/11 · Zero dependências

- **Decisão:** Node puro. Sem Express, sem zod; Gemini por REST, com uso automático do SDK `@google/genai` se estiver instalado.
- **Motivo:** `registry.npmjs.org` e `pypi.org` recusam conexão nesta rede. Não contornamos o bloqueio por espelhos.
- **Fonte:** `npm install` travou por mais de 5 min; `curl https://registry.npmjs.org` falhou ao conectar (14/11, ~14h20).
- **Alternativas descartadas:** baixar pacotes por CDN (contorna um bloqueio de rede corporativa); Python/FastAPI (PyPI também bloqueado).
- **Reversível?** Sim. Com rede liberada: `npm i @google/genai` e, se quiserem, reescrever `src/schemas.js` com zod.
- **Desvio do briefing:** ele pedia o SDK oficial e zod. O contrato (schema único, validação local, adaptador isolado) foi mantido.

## D003 — 14/11 · Modelo padrão `gemini-flash-lite-latest`

- **Decisão:** trocar o padrão de `gemini-flash-latest` para `gemini-flash-lite-latest`.
- **Motivo e fonte:** teste real em 14/11: `gemini-flash-latest` e `gemini-3.8-flash` devolveram 503/timeout; `gemini-flash-lite-latest` respondeu em ~1 s com JSON válido. `gemini-2.5-flash` devolveu 404 para esta chave.
- **Reversível?** Sim, é uma linha no `.env`.

## D004 — 14/11 · A ação nunca vem do LLM

- **Decisão:** o campo `acao` do LLM é validado mas ignorado; o código deriva a ação da `classificacao`.
- **Fonte:** `npm run fixtures` em 14/11: o Gemini devolveu `{"classificacao":"nao_consegui","acao":"atualizar_status"}` para "não consegui marcar" (está salvo em `fixtures/classificacao.json`).

## D005 — 14/11 · Melhorias de produto depois do MVP

- **Decisão:** acrescentar ao escopo: equipe resolve card escalado (`POST /resolver`), prazo correndo com lembrete e escalada por vencimento, fila de prioridade, paciente fictício por conduta e gabarito ampliado (15 condutas, 40 mensagens).
- **Motivo:** o MVP parava na escalada; faltava mostrar o ciclo fechando e o prazo tendo efeito.
- **Fonte:** pedido do time em 14/11, a partir da lista de melhorias sugerida na sessão.
- **Afeta:** `src/agente.js`, `public/`, `fixtures/gabarito.json`, testes (17 → 22).
- **Desvio do briefing:** o briefing listava só quatro rotas; `/resolver` é nova. A regra "não invente prazo" foi mantida: só um humano define prazo.

## D006 — 14/11 · A avaliação usa respostas gravadas

- **Decisão:** o painel de métricas avalia com as respostas do Gemini gravadas em `fixtures/`, não com chamadas ao vivo.
- **Motivo:** a amostra ampliada pede cerca de 40 chamadas; ao vivo, estouraria o limite por minuto da camada gratuita e travaria a tela por minutos.
- **Fonte:** `npm run fixtures` em 14/11: 45 respostas gravadas, 0 falhas, com 5 s entre chamadas. Resultado: extração 19/20, classificação 36/40, os 4 erros de classificação terminam em escalar para humano.
- **Reversível?** Sim. Regravar com `npm run fixtures` sempre que o prompt, o modelo ou o gabarito mudarem.

## D007 — 14/11 · Retorno não é cobrado no dia 0

- **Decisão:** o acompanhamento passa a depender do tipo. Exame e encaminhamento: pergunta "conseguiu agendar?" no dia 0 e cobra. Retorno: só um aviso no dia 0; lembrete "está confirmado?" faltando 3 dias; depois disso vale a regra de ausência e a de prazo vencido.
- **Motivo:** perguntar "já agendou?" para um retorno de 30 dias no dia da consulta soa errado; o risco do retorno é o paciente não comparecer.
- **Fonte:** observação do time ao usar a demo em 14/11; enunciado do desafio ("se o exame foi marcado ou se o retorno aconteceu").
- **Afeta:** T5 mudou: ao avançar o tempo duas vezes, só o exame escala; o retorno continua em "A fazer". As regras fixas (não consegui, dor, SAIR) seguem valendo para retorno a qualquer momento.
- **Reversível?** Sim: é a função `iniciarFollowUp` em `src/agente.js`.

## D008 — 14/11 · "Já fiz" conclui direto; confirmado não escala sem perguntar

- **Decisão:** (1) resposta classificada como `confirmou` que indica realização ("já fiz", "acabei de voltar do médico", "compareci") leva a tarefa direto para concluída; (2) tarefa em andamento com prazo vencido, cujo paciente não devia resposta, recebe uma conferência ("deu tudo certo?") antes de escalar.
- **Motivo:** no teste do time, o paciente respondeu "confirmado, acabei de voltar do medico" no dia do prazo; a tarefa foi só para "em andamento" e, no avanço seguinte, escalou como prazo vencido. Era um falso alarme para a equipe.
- **Fonte:** capturas de tela do time em 14/11; caso reproduzido em `test/aceitacao.test.js` e conferido com o Gemini real (classificação `confirmou`, tarefa concluída).
- **Limite:** a detecção de "já realizou" é uma lista de expressões em `src/regras.js`. Frase fora da lista só avança um passo; nesse caso a conferência pós-prazo cobre.

## D009 — 14/11 · Confirmação no dia do prazo conclui

- **Decisão:** resposta classificada como `confirmou` no dia do prazo ou depois dele leva a tarefa direto para concluída. Para o "sim" não ser ambíguo, o lembrete do dia do prazo pergunta sempre "Já foi realizado?" (antes do dia, o retorno continua recebendo "Está confirmado?"). A lista de expressões de "já realizou" ganhou "retornei", "cheguei", "saí" e "vim" do médico, da consulta, do exame.
- **Motivo:** no teste do time, "confirmado, retornei agora do medico" no dia 30 só moveu a tarefa para "em andamento" e gerou uma conferência dois dias depois.
- **Fonte:** captura de tela do time em 14/11; caso reproduzido com o Gemini real (classificação `confirmou`, tarefa concluída, nenhuma escalada ao avançar 7 dias).
- **Risco aceito:** quem responde "sim" no dia do prazo querendo dizer "vou hoje" tem a tarefa concluída antes da hora.

## D010 — 14/11 · "Foi realizado" virou regra determinística

- **Decisão:** mensagens que informam realização ("foi realizado", "feito", "fiz", "realizei", "compareci", "voltei do médico") são classificadas como `confirmou` no código, antes do LLM e depois dos gatilhos de escalação. Mensagens com "não" ou "ainda" ficam de fora.
- **Motivo:** o Gemini classificou "foi realizado na parte da manhã" como `pediu_ajuda` e a tarefa foi escalada. O modelo recebe só a mensagem, sem a pergunta do agente, e o prompt manda escolher `pediu_ajuda` na dúvida.
- **Fonte:** captura de tela do time em 14/11 (card t1 escalado com esse motivo); caso reproduzido em `test/aceitacao.test.js`.
- **Alternativa descartada:** enviar a pergunta do agente junto com a mensagem ao LLM. Melhoraria o contexto, mas mudaria o prompt fixado no briefing e invalidaria as fixtures gravadas.
- **Risco aceito:** é uma lista de expressões; uma frase como "fiz a ligação e não atenderam" é barrada pelo "não", mas outras construções podem concluir uma tarefa por engano.

## D011 — 14/11 · Substitui a D010: quem reconhece "já realizado" é o Gemini

- **Decisão:** a D010 (lista de expressões antes do LLM) foi revertida a pedido do time. Agora o modelo recebe a última pergunta do agente junto com a resposta do paciente e devolve o campo `realizado` (booleano). `confirmou` + `realizado: true` conclui a tarefa. A lista de expressões ficou só como reserva para o mock e para quando a API cai.
- **Desvio do briefing:** o system instruction de classificação mantém a frase do briefing e ganhou um complemento; o schema de classificação ganhou `realizado`. O cache de classificação passou a considerar a pergunta do agente.
- **Fonte:** teste com o Gemini real em 14/11 (`gemini-flash-lite-latest`), 8 de 8: "foi realizado na parte da manhã", "confirmado, retornei agora do medico" e "sim" em resposta a "Já foi realizado?" vieram `realizado: true`; "sim", "marquei para sexta" e "agendei ontem" em resposta a "Você já conseguiu agendar?" vieram `realizado: false`.
- **O que continua no código:** gatilhos de escalação, SAIR, ausência, prazo vencido e a ação. O LLM só decide a classificação e o `realizado`, e um erro dele nesse campo no máximo conclui ou deixa de concluir uma tarefa; nunca impede uma escalada por gatilho.
- **Risco aceito:** se o modelo marcar `realizado: true` por engano, a tarefa é concluída sem revisão humana.
- **Efeito medido:** depois de regravar as fixtures com o prompt novo (44 gravadas; 1 falha numa extração, que manteve a resposta anterior, do mesmo prompt), a classificação no gabarito foi de 36/40 para 39/40. O erro restante ("não tenho como pagar o exame" → `pediu_ajuda`) termina em escalar para humano.

## D012 — 14/11 · Redesign aplicado a partir do PDF do time

- **Decisão:** o front passou a seguir o PDF "Paciente Não Fica Solto — Redesign": barra lateral em azul-petróleo, âmbar como cor de ação, títulos em serifa, cinco telas (Visão geral, Conduta, Acompanhamento, Chat, Log e métricas), faixa "equipe acionada" e o logo "Abraço" (o recomendado no PDF), também como favicon.
- **Fonte:** PDF de 7 páginas enviado pelo time em 14/11.
- **Adaptações:** a maquete era anterior a algumas funções. Mantive paciente fictício por conduta, prazo correndo, ação da equipe com prazo, prioridade alta e a avaliação de 15 condutas e 40 mensagens. O filtro do kanban ficou por tipo, como no PDF, com um seletor de paciente ao lado. Os textos dos exemplos são os do briefing, não os ilustrativos da maquete.
- **Fontes tipográficas:** o PDF usa uma serifa e uma sans que viriam da web. Como a demo roda sem rede, o CSS pede Fraunces e Inter e cai para as fontes do sistema (Palatino/Georgia e Segoe UI).
- **API:** `GET /estado` passou a devolver `condutas` e, em cada tarefa, `ultima_classificacao`, usados pelas telas novas.
