# AGENTS.md — leia antes de mexer

## O que estamos construindo

Um agente que transforma uma conduta médica **fictícia** em tarefas com prazo, faz follow-up **simulado**, classifica a resposta do paciente, atualiza um kanban e escala para humano. Golden path do demo: Exemplo 1 → 2 tarefas → "não consegui marcar" → escalada → "SAIR" → métricas.

## Stack

Node puro (>= 20.12), sem dependências. Frontend em HTML/JS puro. LLM: Gemini (AI Studio) atrás de `src/llm.js`.

## Comandos (o contrato de verificação)

```bash
npm run mock    # sobe em MOCK=1, http://localhost:3000
npm start       # sobe com Gemini (precisa de .env)
npm test        # 26 testes (T1–T9, extras, melhorias); precisa estar verde antes de dizer "pronto"
npm run fixtures  # regrava as respostas reais do Gemini (~4 min); rode se mudar prompt ou gabarito
```

## Regras invioláveis do produto

1. Só dados sintéticos. Nenhum dado real de paciente.
2. Nunca diagnóstico, prescrição ou orientação clínica. Pedido assim é recusado e vai para o log.
3. Nenhuma mensagem sai para pessoas reais.
4. Nunca inventar prazo. Sem prazo explícito: `prazo_dias: null` e revisão humana.
5. Regras de escalação ficam em `src/regras.js`, no código. Nunca dependem do LLM.
6. Chave só no `.env` (ignorado pelo git).
7. Nunca cortar: regras de escalação, T3, T4, T7, T8 e a recusa.

## Convenções

- Toda chamada ao LLM passa por `src/llm.js`. Nenhum outro arquivo conhece o Gemini.
- Mudou prompt, schema ou regra: rode `npm test` e refaça o golden path na tela.
- Decisão que muda escopo ou desvia do briefing: registrar em `DECISOES.md` com a fonte.
- Se mais de uma pessoa ou agente for mexer ao mesmo tempo: `git init`, uma branch e uma worktree por tarefa, e marque no `BOARD.md` antes de começar (ver `.claude/skills/hackathon/SKILL.md`).

## Onde está o quê

- `CONTEXTO.md`: perfil do hackathon e pendências `[VERIFICAR]`
- `BOARD.md`: o que está feito e o que falta
- `DECISOES.md`: decisões com motivo e fonte
- `specs/core/spec.md`: critérios de aceite
- `AUDIT.md`: auditoria pré-demo com evidência
- `PITCH.md`: roteiro de 3 minutos e perguntas da banca
