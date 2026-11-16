# CONTEXTO — o perfil deste hackathon

> Preenchido a partir do briefing que o time passou em 14/11. **O edital oficial não foi lido**: tudo que depende dele está marcado `[VERIFICAR]`. Não inventamos rubrica.

## O hackathon

- **Nome:** Hack Inova Mackenzie
- **Desafio:** 05 · Paciente Não Fica Solto (IA na saúde)
- **Formato:** `[VERIFICAR]` presencial/online, tamanho do time · duração: 10 horas de trabalho
- **Data:** 14/11 · horário de submissão e de julgamento: `[VERIFICAR]`
- **Fonte das regras:** briefing do time (prompt de construção do MVP). Edital: `[VERIFICAR]`

## Rubrica oficial

`[VERIFICAR]` Não temos a rubrica. Até ela chegar, o pitch cobre: problema real, solução funcionando, segurança/ética em saúde, uso de IA, viabilidade.

## Requisitos-gate (do briefing)

- [x] Somente dados sintéticos
- [x] Nunca diagnóstico, prescrição ou orientação clínica (recusa registrada no log)
- [x] Nenhuma mensagem para pessoas reais
- [x] Não inventar prazo (`prazo_dias: null` → revisão humana)
- [x] Regras de escalação determinísticas, fora do LLM
- [x] Chave fora do código e do repositório
- [x] Demo roda em `MOCK=1`, testes T1–T9 passando
- [ ] `[VERIFICAR]` Requisitos do edital: repositório público? vídeo? tecnologia de patrocinador obrigatória? (o briefing cita "integração com OCI" como fora do escopo; confirmar se OCI é exigência de alguma trilha)

## Formato de julgamento

`[VERIFICAR]` Pitch ao vivo, expo ou vídeo? O jurado testa o app? Quanto tempo por time? O briefing pede um roteiro de demo de 3 minutos.

## Time e recursos

- Humanos: `[VERIFICAR]`
- Stack: Node puro (sem dependências), HTML/JS, Gemini (AI Studio)

## O que isso muda na metodologia

- **10 horas:** pipeline comprimido. Pesquisa e red team já vieram resolvidos no briefing (escopo, regras e testes de aceite fechados). O tempo vai para build e pitch.
- **Um agente, um checkout:** worktrees, integrador e PRs são cerimônia desnecessária (o próprio README da skill diz isso). Mantivemos: spec antes de código, "pronto" = teste verde, decisões registradas, audit com evidência, demo sempre rodando.
- **O que o Demo keeper protege:** `npm run mock` abrindo em qualquer máquina com Node, sem rede e sem `npm install`.
- **Risco específico:** a rede do evento bloqueia npm e PyPI, e o Gemini "cheio" oscilou (503). Por isso zero dependências e fixtures com respostas reais.

## `[VERIFICAR]` — pendências para o time

- [ ] Rubrica e pesos
- [ ] Formato e tempo do pitch
- [ ] Horário-limite e formato da submissão (repo, vídeo, deck)
- [ ] Exigência de tecnologia de patrocinador (OCI?)
