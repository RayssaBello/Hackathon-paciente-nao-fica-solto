# PITCH — roteiro de 3 minutos

> Ajustar à rubrica oficial quando ela chegar (`CONTEXTO.md`, `[VERIFICAR]`). Rodar com `npm run mock`: funciona sem internet.

## Antes de subir

1. `npm run mock` e abrir <http://localhost:3000>. A demo acontece na tela **Visão geral**.
2. Clicar em **Reiniciar demo**.
3. Zoom do navegador em 110–125% para a banca ler o log.

## Roteiro

| Tempo | O que falar | O que fazer na tela |
|---|---|---|
| 0:00–0:25 | "O paciente sai da consulta com uma conduta: fazer um exame, voltar em 30 dias. Daí em diante ninguém acompanha. Se ele não marcou, ninguém fica sabendo." | Tela parada no estado vazio |
| 0:25–0:55 | "A equipe cola a conduta. O agente extrai só o que está escrito e cria as tarefas com prazo. O exame precisa ser marcado, então ele já pergunta ao paciente. O retorno ele só avisa, e volta a lembrar perto da data. As mensagens aqui são simuladas." | **Exemplo 1** → **Gerar plano**. Apontar os 2 cards e a mensagem no chat |
| 0:55–1:25 | "O paciente responde que não conseguiu marcar. Isso não passa pela IA: é regra fixa no código. O card vai para um humano e a equipe é avisada." | Escrever **não consegui marcar** no celular e enviar. Apontar o card vermelho e o `AVISO À EQUIPE` no log |
| 1:25–1:45 | "Se o paciente não quer mais mensagens, ele escreve SAIR e o acompanhamento para." | Clicar no card `t2` → escrever **SAIR** e enviar |
| 1:45–2:20 | "O agente não inventa prazo: sem data, o card vai para a equipe. Quem define o prazo é uma pessoa. Ela digita, clica em Retomar, e o acompanhamento volta a andar. E ele não dá diagnóstico: recusa e registra." | **Exemplo 2** → Gerar plano (card "sem prazo"). **Equipe: assumir caso** → digitar `20` → **Retomar acompanhamento**. Voltar para **Visão geral**. **Exemplo 3** → Gerar plano (linha `RECUSA` no log) |
| 2:20–2:45 | "O prazo corre. Perto do vencimento o agente lembra o paciente; venceu, avisa a equipe. E medimos: 95% das tarefas certas e 98% das respostas, em 15 condutas e 40 mensagens. O único erro de classificação cai do lado seguro: chama um humano." | **+7d** → **Avançar 7 dias** (cards mostram "Dia 7 de 15"). Abrir **4 · Log e métricas** e apontar os números |
| 2:45–3:00 | "Tudo aqui é dado sintético e nenhuma mensagem sai do sistema. O próximo passo é ligar num canal real e na agenda, com a mesma regra: na dúvida, humano." | Voltar ao kanban |

## Plano B

- **Sem internet ou Gemini fora:** a demo já roda em mock. Se estiver em modo Gemini e a API cair, o log mostra `FALLBACK` e o fluxo segue.
- **Notebook travou:** vídeo de backup (cartão P003 do `BOARD.md`).
- **Estado sujo:** botão **Reiniciar demo**.

## Perguntas prováveis da banca

| Pergunta | Resposta, com evidência |
|---|---|
| E se a IA errar a classificação? | O que é crítico não passa por ela: "não consegui", "ajuda", "urgente", "dor", "piorou" e SAIR são regras no código (`src/regras.js`). No resto, a dúvida vira `pediu_ajuda`. A ação é sempre calculada pelo código: nos testes o Gemini devolveu uma ação incoerente e o sistema ignorou |
| E se a IA inventar um prazo? | O prompt proíbe e o código confere: prazo sem número nem unidade de tempo no texto é descartado e a tarefa vai para revisão humana (tem teste) |
| E se o Gemini cair ou a cota acabar? | Cache, backoff no 429 e fallback para respostas salvas, tudo registrado no log. Dá para mostrar ao vivo com "Simular falha" |
| Isso usa dado de paciente? | Não. Só dados sintéticos e nada é enviado a ninguém |
| Como sabem que funciona? | 26 testes automatizados (`npm test`), incluindo os 9 de aceitação, e métricas contra gabarito |
| O paciente com dor recebe orientação? | Não. O agente não orienta: escala para um humano com prioridade alta. O texto para esses casos precisa ser definido com a equipe clínica |
| Qual o tamanho da avaliação? | 15 condutas e 40 mensagens sintéticas, com as respostas reais do Gemini gravadas. Mostra o método e os tipos de erro; não é prova de acurácia em mensagens reais |
| E depois que escala, o que acontece? | A equipe resolve no próprio card: define o prazo, retoma o acompanhamento ou conclui. Se o paciente pediu SAIR, a tarefa volta sem novas mensagens |
| Como a equipe sabe o que é urgente? | A coluna de escaladas é uma fila: "dor", "piorou" e "urgente" sobem para o topo, em vermelho |
| Por que o retorno não é cobrado no começo? | Porque a dor do retorno é faltar, não marcar. O agente avisa no dia 0, lembra faltando 3 dias e, se o paciente sumir ou o prazo vencer, chama a equipe |
| O que falta para produção? | Canal real (WhatsApp/SMS) com consentimento, integração com agenda, login, banco, avaliação com volume e revisão clínica dos textos |
