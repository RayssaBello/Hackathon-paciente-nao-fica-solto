# BOARD — o quadro do time

> Cartão sem dono não existe. Marque "peguei" (seu nome em **Dono**) antes de começar. "Pronto" só com `npm test` verde.

## Roteiro do demo (o filtro do backlog)

Exemplo 1 → 2 tarefas no kanban → "não consegui marcar" → card em "Escalada para humano" + aviso no log → "SAIR" em outra tarefa → métricas. Detalhe em `PITCH.md`.

## Feito (verificado em 14/11)

| Cartão | Pronto quando | Evidência |
|---|---|---|
| T001 Contrato: schemas + API | `responseSchema` e validação local com o mesmo formato | `src/schemas.js`, testes T1/T7 |
| T002 Regras determinísticas | Gatilhos, SAIR, ausência, guarda de diagnóstico e de prazo | T3, T4, T5, T8 + extras |
| T003 Adaptador LLM | Cache, backoff 429, fallback, retry de JSON | T6, T7, T9 |
| T004 Agente + persistência | Plano, resposta, tempo, kanban, `tarefas.json` | T1–T9 + teste de persistência |
| T005 Frontend | Kanban, chat, log, métricas, mover por botão e arrasto | Golden path rodado no navegador, console sem erros |
| T006 Gemini real | Chamada real com saída estruturada | `npm run fixtures`: 10 de 10; métricas reais 4/4 e 12/13 |
| T007 Fixtures da demo | Respostas reais salvas em `fixtures/` | `fixtures/*.json` |
| T009 Equipe resolve card escalado | Define prazo no card, tarefa volta ao fluxo, respeita SAIR | 2 testes + fluxo feito na tela |
| T010 Prazo correndo | "Faltam N dias", lembrete, escalada por prazo vencido | 1 teste (dias 7, 14, 21, 31) |
| T011 Gabarito ampliado | 15 condutas e 40 mensagens, com respostas reais gravadas | extração 19/20, classificação 39/40 |
| T012 Fila de prioridade e pacientes fictícios | Urgente no topo; nome fictício no card; filtro | 2 testes + conferido na tela |
| T013 Retorno sem cobrança no dia 0 | Aviso, lembrete faltando 3 dias, depois regra de ausência | 1 teste (dias 0, 15, 27, 29, 31) |
| T014 Gemini decide "já realizado" | Modelo recebe a pergunta do agente e devolve `realizado` | 8/8 com Gemini real + 1 teste; caso da captura concluído pela API |
| T008 Docs | README, CONTEXTO, DECISOES, AGENTS, spec, AUDIT, PITCH | arquivos na raiz |

## A fazer (precisa de gente do time)

### [P001] Ler o edital e fechar os `[VERIFICAR]`
- Dono:
- Pronto quando: rubrica, formato do pitch, horário e formato de submissão e exigência de patrocinador estão em `CONTEXTO.md`

### [P002] Ensaiar o pitch na máquina de apresentação
- Dono:
- Depende de: P001
- Pronto quando: roteiro de `PITCH.md` coube no tempo oficial duas vezes seguidas, com `npm run mock`, por alguém que não construiu

### [P003] Vídeo de backup do golden path
- Dono:
- Pronto quando: gravação de 60–90 s da demo salva na máquina de apresentação e em mais um lugar

### [P004] Repositório
- Dono:
- Pronto quando: `git init`, primeiro commit, remoto criado se o edital pedir. Conferir que `.env` não entrou (`git status` não lista `.env`)

### [P005] Trocar a chave do Gemini depois do evento
- Dono:
- Pronto quando: chave atual revogada no AI Studio (ela circulou por mensagem e captura de tela)

## Opcional (só se sobrar tempo; nada disso está no roteiro)

- Teste automatizado do arrastar-e-soltar
- Com rede liberada: `npm i @google/genai` e zod em `src/schemas.js`

## Travado / ajuda

- Registro do npm e PyPI bloqueados na rede (ver `DECISOES.md` D002). Não bloqueia a demo.
