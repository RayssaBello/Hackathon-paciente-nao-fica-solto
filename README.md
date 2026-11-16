# Paciente Não Fica Solto

MVP do desafio **05 · Paciente Não Fica Solto** (Hack Inova Mackenzie, 14/11).

Um agente transforma uma conduta médica **fictícia** em tarefas com prazo, faz um follow-up **simulado** com o paciente, classifica a resposta e atualiza um kanban. Quando a situação exige, encaminha para um humano.

> **Limites do protótipo.** Só dados sintéticos. Não dá diagnóstico, prescrição nem orientação clínica. Nenhuma mensagem sai para pessoas reais. Não inventa prazo.

## Como rodar

Requisito: **Node 20.12 ou mais novo**. Não há dependências para instalar (não precisa de `npm install`).

```bash
npm run mock
```

Abre em <http://localhost:3000>, em `MOCK=1`: sem chave e sem rede, usando as respostas salvas em `fixtures/`.

```bash
npm test
```

Roda os 26 testes (T1–T9, extras de segurança e as melhorias), sempre em mock.

### Com o Gemini de verdade

1. Copie `.env.example` para `.env`.
2. Preencha `GEMINI_API_KEY` com a chave do [AI Studio](https://aistudio.google.com/apikey).
3. Confira `GEMINI_MODEL` (padrão: `gemini-flash-lite-latest`) e deixe `MOCK=0`.

```bash
npm start
```

Na barra lateral, abaixo do dia simulado, aparece se o LLM em uso é o Gemini ou o mock. Sem chave, o servidor sobe em mock e avisa no log.

```bash
npm run fixtures
```

Regrava `fixtures/` com respostas reais do Gemini (cerca de 45 chamadas, uma a cada 5 s; leva uns 4 minutos).

O `.env` está no `.gitignore`. A chave nunca vai para o código nem para o repositório.

## Telas

A barra lateral leva a cinco telas. A demo inteira cabe na primeira.

| Tela | O que mostra |
|---|---|
| **Visão geral** | Tudo em um painel: conduta, kanban resumido, linha do tempo e o celular do paciente |
| **1 · Conduta médica** | Campo grande, exemplos, prévia do plano gerado e condutas recebidas |
| **2 · Acompanhamento** | Kanban completo, filtros, detalhe da tarefa com histórico e as ações da equipe |
| **3 · Chat do paciente** | Conversas por tarefa, o celular simulado e como o agente entendeu a última resposta |
| **4 · Log e métricas** | Registro com filtros e como as métricas são calculadas |

Quando há tarefa escalada, uma faixa vermelha aparece com o botão **Equipe: assumir caso**.

## Roteiro da demo

1. Clique em **Exemplo 1** e em **Gerar plano**: aparecem 2 tarefas em "A fazer" (exame em 15 dias, retorno em 30). O exame já sai com a pergunta "conseguiu agendar?"; o retorno só é avisado.
2. No celular do paciente, com a tarefa `t1`, escreva **não consegui marcar** e envie (na tela **Chat do paciente** há botões de atalho com as respostas de teste): o card vai para "Escalada para humano" e o log mostra `AVISO À EQUIPE`.
3. Clique no card `t2` e envie **SAIR**: acompanhamento encerrado.
4. Abra **4 · Log e métricas** na barra lateral e mostre o painel de métricas.
5. **Fechar o ciclo:** **Exemplo 2** cria um card sem prazo em "Escalada para humano". Clique em **Equipe: assumir caso** na faixa vermelha: abre o detalhe da tarefa com o campo de prazo. Digite os dias e clique em **Retomar acompanhamento**: a tarefa volta para "A fazer" e o follow-up recomeça.
6. **Prazo correndo:** na barra lateral, escolha **+7d** e clique em **Avançar 7 dias**. Os cards mostram "faltam N dias"; tarefa em andamento recebe lembrete perto do prazo; prazo vencido vai para a equipe.
7. Extras: **Exemplo 3** (recusa), "estou com dor" (prioridade alta, sobe para o topo da fila) e, em mock, **Simular falha** (JSON inválido ou erro 429).

**Reiniciar demo** limpa o estado.

## Como funciona

```
conduta ──► guarda determinística (pede diagnóstico? recusa)
        ──► LLM extrai tarefas (JSON com schema) ──► validação local
        ──► prazo null? revisão humana : follow-up 1 (simulado)

resposta do paciente ──► regras determinísticas (escalar / SAIR)
                     ──► só se nenhuma regra casar: LLM classifica
                     ──► a AÇÃO é sempre decidida no código
```

| Arquivo | Papel |
|---|---|
| `src/regras.js` | Regras determinísticas: gatilhos de escalação, SAIR, guarda de diagnóstico, guarda de prazo |
| `src/llm.js` | Adaptador do LLM: `extrairTarefas` e `classificarResposta`. Cache, retry, fallback. O resto do código não conhece o Gemini |
| `src/schemas.js` | Um contrato só: `responseSchema` do Gemini e validação local |
| `src/agente.js` | Orquestração, kanban, follow-ups, log, métricas, persistência em `tarefas.json` |
| `src/app.js` | API HTTP e arquivos estáticos |
| `public/` | Página única em HTML/JS puro |
| `fixtures/` | Respostas reais do Gemini salvas, gabarito e amostra rotulada |
| `test/aceitacao.test.js` | T1–T9 e extras |

### API

| Rota | O que faz |
|---|---|
| `POST /plano` `{texto, paciente}` | Extrai tarefas da conduta. `paciente` precisa ser um dos nomes fictícios da lista |
| `POST /resposta` `{tarefa_id, mensagem}` | Classifica a resposta (regras primeiro, LLM depois) |
| `POST /avancar-tempo` `{dias}` | Simula a passagem do tempo (padrão: 2 dias; de 1 a 30) |
| `GET /estado` | Kanban, log e métricas |
| `POST /resolver` `{tarefa_id, prazo_dias}` | A equipe resolve um card escalado: define o prazo e retoma o acompanhamento |
| `POST /mover`, `/avaliar`, `/reiniciar` | Apoio à tela: mover card, avaliar contra o gabarito, limpar estado |

### Regras determinísticas (rodam antes do LLM)

- Mensagem contém "não consegui", "não deu" (com ou sem acento): `nao_consegui`, escala para humano.
- Mensagem contém "ajuda", "urgente", "dor", "piorou": `pediu_ajuda`, escala para humano. "urgente", "dor" e "piorou" marcam prioridade alta.
- Mensagem é exatamente "SAIR" (qualquer caixa): encerra o acompanhamento daquela tarefa.
- Sem resposta a uma pergunta do agente: follow-up 2 na primeira ausência, escalada na segunda.
- **Exame (e encaminhamento) precisa ser marcado:** o agente pergunta no dia 0 e cobra se não houver resposta.
- **Retorno é sobre comparecer:** no dia 0 o agente só avisa ("eu te lembro perto da data") e não cobra. Faltando 3 dias, manda o lembrete "está confirmado?"; a partir daí vale a regra de ausência. Retorno com prazo de até 3 dias pergunta já.
- **"Já fiz" conclui direto, e quem decide é o Gemini:** o modelo recebe a última pergunta do agente junto com a resposta do paciente e devolve, além da classificação, o campo `realizado` (a tarefa já aconteceu?). Com `confirmou` e `realizado: true`, a tarefa vai para "Concluída". Os gatilhos de escalação são do código e vêm antes: "já fiz, mas estou com dor" escala sem passar pelo LLM. Em mock ou com a API fora do ar, uma lista de expressões em `src/regras.js` faz esse papel.
- **Confirmação no dia do prazo conclui:** no dia do prazo o agente pergunta "Já foi realizado?"; um "sim" nesse dia ou depois conclui a tarefa.
- Prazo vencido sem conclusão: escala para humano, mesmo que o paciente tenha pedido SAIR (nenhuma mensagem é enviada a ele).
- Exceção: se o paciente tinha confirmado (tarefa em andamento) e não devia resposta, o agente pergunta uma vez "deu tudo certo?" antes de escalar. Sem resposta, escala.
- Tarefa em andamento (paciente já confirmou) também não é cobrada de novo até faltarem 3 dias para o prazo; aí recebe um lembrete único.
- Nesses casos o LLM não é chamado.

### Proteções em volta do LLM

- **Cache em memória** por texto de entrada.
- **Erro 429:** backoff exponencial (1 s, 2 s), no máximo 2 novas tentativas.
- **Cota estourada ou API fora do ar:** aquela chamada cai para a fixture (ou para o mock heurístico, se o texto não tiver fixture) e o log registra `FALLBACK`.
- **JSON inválido:** 1 retry. Se falhar de novo, a conduta vira um card de revisão manual em "Escalada para humano". A tela não quebra.

## Decisões tomadas (o que o briefing não cobria)

| Decisão | Motivo |
|---|---|
| **Zero dependências.** Sem Express, sem zod, sem `@google/genai` instalado | O registro do npm (e o PyPI) estava bloqueado na rede. O servidor usa `node:http`, a validação é um mini-validador com a interface do zod, e o Gemini é chamado pela API REST oficial |
| O adaptador usa o SDK `@google/genai` **se ele estiver instalado**, senão REST | Com rede liberada, `npm i @google/genai` basta; nenhuma outra linha muda. Os dois caminhos usam `responseMimeType` + `responseSchema` |
| JavaScript, não TypeScript (`llm.js`) | Roda direto no Node, sem build |
| Modelo padrão `gemini-flash-lite-latest` | Em 14/11, `gemini-flash-latest` devolvia 503 (sobrecarga) e timeout; o Flash-Lite respondeu em cerca de 1 s |
| Tarefa sem prazo vai direto para "Escalada para humano", com etiqueta "revisão humana" e sem follow-up | É o jeito mais visível de mandar para revisão sem criar uma quinta coluna |
| A `acao` devolvida pelo LLM é ignorada; o código a recalcula a partir da `classificacao` | Regra de escalação não pode depender do LLM. Nos testes reais o Gemini devolveu `nao_consegui` com `acao: atualizar_status` |
| O prompt de classificação do briefing ganhou um complemento e o schema ganhou o campo `realizado`; o modelo recebe também a pergunta do agente | Sem contexto, o Gemini classificou "foi realizado na parte da manhã" como `pediu_ajuda`. Com contexto, distingue "sim" para "já foi realizado?" de "sim" para "já agendou?" |
| Guarda determinística de diagnóstico antes do LLM | A recusa da Conduta 3 não depende do modelo. É estreita de propósito (pedidos explícitos); o prompt cobre o resto |
| Guarda de prazo: se o LLM devolve prazo e o texto não tem número nem unidade de tempo, o prazo vira `null` | "Não invente prazo" garantido no código |
| "dor" só casa como palavra inteira | Evita escalar "computador" ou "coordenador". Os outros gatilhos casam como trecho |
| `confirmou` avança um passo: A fazer → Em andamento → Concluída. Se a mensagem diz que já foi realizado, conclui direto | Interpretação mais simples do T6; o atalho evita escalar por prazo vencido quem já compareceu |
| Retorno não recebe a pergunta "já conseguiu agendar?" no dia 0; só aviso e lembrete perto da data | Cobrar no dia 0 um retorno de 30 dias não faz sentido; a dor do retorno é o paciente não comparecer |
| SAIR encerra só a tarefa em conversa | O roteiro pede "SAIR em outra tarefa" |
| Mensagem de alerta ("piorou", "urgente"...) escala mesmo depois de SAIR ou de concluída | Segurança primeiro |
| "Avançar tempo" vale 2, 7 ou 15 dias simulados, à escolha | Só para a simulação; 7 e 15 permitem mostrar o prazo vencendo |
| O prazo definido em `/resolver` é de um humano e conta a partir do dia em que foi definido | O agente continua sem inventar prazo |
| Ao resolver um card de paciente que pediu SAIR, a tarefa volta ao fluxo sem novo follow-up | O pedido de saída do paciente vale mais que a retomada |
| Paciente só pode ser um dos 6 nomes fictícios de `fixtures/gabarito.json` | Garante no servidor que nenhum nome real entra |
| A coluna "Escalada" é uma fila: prioridade alta primeiro, depois ordem de chegada | A equipe vê primeiro o que é urgente |
| A avaliação do painel usa as respostas do Gemini gravadas em `fixtures/`, não chamadas ao vivo | 40 chamadas ao vivo estourariam a cota gratuita por minuto. Fica reproduzível e sem custo; `npm run fixtures` regrava |
| Em dúvida ou falha na classificação: `pediu_ajuda` | Errar para o lado do humano |

## Métricas

O gabarito está em `fixtures/gabarito.json`: 15 condutas e 40 mensagens sintéticas, rotuladas à mão.

- **Tarefas corretas x gabarito:** tipo e prazo de cada tarefa, nas 15 condutas, passando pela guarda de diagnóstico, pelo LLM e pela guarda de prazo.
- **Classificações corretas:** as 40 mensagens passam pelo fluxo completo (regras e depois LLM).
- **Escaladas:** cards na coluna "Escalada para humano", com o número de urgentes e de resolvidas pela equipe.

A avaliação roda sozinha ao abrir a tela e usa as respostas reais do Gemini gravadas em `fixtures/` (não gasta cota). O painel mostra de onde veio cada resposta: regra, Gemini gravado ou heurística do mock.

Medido em 14/11 com `gemini-flash-lite-latest` (16 respostas por regra, 39 do Gemini), já com o prompt de classificação que recebe a pergunta do agente:

- **Extração: 19 de 20 tarefas (95%).** O erro: "Paciente orientado. Sem pendências." foi recusada pelo modelo, quando o esperado era "nenhuma tarefa". Não cria tarefa indevida.
- **Classificação: 39 de 40 (98%).** O erro: "não tenho como pagar o exame" foi classificado como `pediu_ajuda` em vez de `nao_consegui`; nos dois casos a tarefa é escalada para humano. Com o prompt anterior, sem a pergunta do agente, eram 36 de 40.

## O que está mockado e o que ficou de fora

- **Mockado:** o envio de follow-ups e as respostas do paciente (tudo simulação na tela); o relógio (botão "avançar tempo"); o LLM quando `MOCK=1`.
- **Fora do escopo:** SMS/WhatsApp real, login, banco de dados, agendamento real, integração com OCI.

## Riscos conhecidos

- **Cota gratuita do Gemini** é baixa por minuto e por dia. Mitigação: cache, regras antes do LLM, fallback para fixture. Se a cota acabar no meio da demo, o log mostra `FALLBACK` e o fluxo continua; para texto livre sem fixture a resposta vem de um mock heurístico simples.
- **Instabilidade do modelo:** vimos 503 e timeout no Flash "cheio". Para a banca, `npm run mock` é o caminho à prova de rede.
- **A amostra de avaliação ainda é pequena** (40 mensagens, 15 condutas, escritas por nós). Mostra o método e os tipos de erro; não é prova de acurácia em mensagens reais.
- **A avaliação reflete o dia em que as fixtures foram gravadas.** Se o modelo mudar, rode `npm run fixtures` de novo.
- **Guarda de diagnóstico estreita:** pedidos indiretos dependem do prompt do LLM.
- **Mensagem de urgência:** o agente só avisa a equipe com prioridade alta; não orienta o paciente (restrição do desafio). O card escalado mostra esse aviso. Num produto real, o texto para esses casos precisa ser definido com a equipe clínica.
- **SAIR vale por tarefa**, não por paciente.
- **Sem autenticação** e estado em um único arquivo: é protótipo local, para um usuário.
- **Arrastar cards** e o **filtro por paciente** não têm teste automatizado (os botões ◀ ▶ e o formulário da equipe foram exercitados).
