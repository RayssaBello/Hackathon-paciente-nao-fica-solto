# AUDIT-DEMO — 14/11

> Regra do auditor: evidência, não "sim". "Não sei" conta como "não feito". Severidade: P0 mata o demo, P1 sangra na banca, P2 é feio.

## Placar

| Área | Placar | Evidência |
|---|---|---|
| 0 · Gate do edital | 🟡 | Requisitos do briefing cumpridos e testados. Edital não lido: rubrica, formato de pitch e de submissão estão `[VERIFICAR]` em `CONTEXTO.md` |
| 1 · Esqueleto | 🟡 | `npm test`: 17 de 17 verdes (14/11). Golden path rodado no navegador pelo agente que construiu, não por um terceiro. Sem repositório git ainda |
| 2 · O demo | 🟡 | Dados de exemplo embutidos; plano B é o próprio modo mock; botões desabilitam durante a chamada; estados vazios têm texto. Roteiro não ensaiado no tempo oficial; sem vídeo de backup |
| 3 · Camada invisível | ✅ | Ver abaixo |

## Área 3 — o que foi conferido

- **Secret fora do código:** chave só no `.env`, que está no `.gitignore`. Nenhum arquivo de código ou doc contém a chave.
- **`.env` não é servido:** `GET /.env`, `/../.env`, `/%2e%2e/.env` e `/tarefas.json` devolvem 404; `/..%5c.env` devolve 403. O servidor só entrega `public/`.
- **Validação no servidor:** texto vazio → 400; tarefa inexistente → 404; status inválido → 400; corpo acima de 100 kB → 413. Classificação e ação são calculadas no servidor.
- **Sem mass assignment:** as rotas leem só os campos esperados (`texto`, `tarefa_id`, `mensagem`, `status`, `simular`).
- **Erro sem stack trace:** falha interna devolve mensagem genérica em JSON.
- **Rota de IA protegida contra loop:** requisições em série, cache por entrada, regras antes do LLM, no máximo 3 chamadas por entrada no 429 e 2 no JSON inválido.
- **Dependências:** nenhuma, então nada pode mudar de versão no dia.
- **Rede:** o servidor escuta só em `127.0.0.1`.

## P0

Nenhum encontrado no golden path em mock.

## P1

1. **Edital não lido.** Se houver requisito-gate (repo público, vídeo, tecnologia de patrocinador), ele pode desclassificar. Primeiro passo: abrir o edital e preencher `CONTEXTO.md` (cartão P001).
2. **Golden path não foi rodado por alguém de fora.** Primeiro passo: uma pessoa do time segue o `PITCH.md` do zero em outra máquina com Node (cartão P002).
3. **Sem vídeo de backup** (cartão P003).
4. **A chave do Gemini circulou por mensagem e captura de tela.** Tratar como exposta: revogar depois do evento (cartão P005).

## P2

- Arrastar-e-soltar não tem teste automatizado (os botões ◀ ▶ têm cobertura pela rota `/mover`).
- Layout não foi conferido em celular.
- Em modo Gemini, texto livre fora das fixtures cai num mock heurístico simples se a API falhar.

## Riscos aceitos

- Sem SDK oficial nem zod instalados, por bloqueio de rede (`DECISOES.md` D002).
- Amostra de avaliação pequena (3 condutas, 13 mensagens).
- Guarda determinística de diagnóstico estreita; pedidos indiretos dependem do prompt.

## Go / no-go

**Go para demo em `npm run mock`.** O golden path está livre de P0. Antes do pitch, fechar P001 e P002.
