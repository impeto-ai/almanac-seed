---
name: acceptance-loop
description:
  Motor do Definition of Done do Almanak. Executa a Validation Matrix dos 10 criterios de
  aceite, e ITERA (corrige -> re-valida) ate todos = PASS com deploy de producao verificado.
  Use como passo final obrigatorio do build. O agent NAO declara conclusao sem rodar esta skill.
---

# Acceptance Loop

## Goals
- Provar, contra evidencia, que os 10 criterios de aceite da SEED (Secao 14) passam.
- Operar em loop: para cada FAIL, corrigir a implementacao e re-rodar a matriz inteira.
- So declarar Definition of Done quando 10/10 PASS + deploy de producao executa o fluxo completo.

## Inputs
- `SEED.md` Secao 14 (Acceptance Criteria), 15 (Validation Matrix), 16 (Build Loop).
- Skills: `supabase-provision`, `vercel-deploy`, `e2e-verify`.

## The Loop (normativo)
```
repeat:
  1. garantir backend ok        -> supabase-provision (se infra/RLS mudou)
  2. garantir producao no ar     -> vercel-deploy
  3. rodar verificacao funcional -> e2e-verify (criterios AUTH..STATUS)
  4. rodar checks nao-funcionais -> RLS cross-team (crit 9), DEPLOY producao (crit 10)
  5. montar matriz PASS/FAIL dos 10 criterios
  6. se algum FAIL:
       - corrigir a causa raiz
       - voltar ao passo 1 (re-rodar matriz INTEIRA, nao so o item)
  7. se 10/10 PASS e producao verificada:
       - Definition of Done atingido -> parar
```

## Workpad (rastreio obrigatorio)
Manter um checklist persistente atualizado a cada iteracao:
```
## Almanak Build Workpad
### Acceptance Matrix (iteracao N)
- [ ] 1 AUTH      PASS/FAIL  <evidencia>
- [ ] 2 TEAM      ...
- [ ] 3 PROJECT   ...
- [ ] 4 UPLOAD    ...
- [ ] 5 VERSION   ...
- [ ] 6 SHARE     ...
- [ ] 7 COMMENT   ...
- [ ] 8 STATUS    ...
- [ ] 9 RLS       ...
- [ ] 10 DEPLOY   ...
### Fixes aplicados nesta iteracao
- <o que mudou>
```

## Definition of Done (gate final)
- 10/10 criterios = PASS na matriz.
- Deploy de producao na Vercel executa o fluxo completo
  (login Google -> team -> projeto -> upload -> versao -> compartilhar -> comentar -> resolver).
- Nenhum criterio MUST em FAIL.

## Human-in-the-loop (pausa, nao FAIL)
- Criterios que dependem de checkpoint humano (SEED §16.1: H3 AUTH login, H4 smoke producao)
  NAO contam como FAIL enquanto aguardam o humano. Marcar no workpad como `waiting on human: H#`,
  fazer todo o resto automatizavel, e RETOMAR a matriz assim que destravado.
- Nunca "fingir" PASS de um criterio que depende de login interativo nao executado. Marcar
  `BLOCKED/waiting` com honestidade.

## Failure Handling
- Loop infinito de FAIL no mesmo criterio: registrar bloqueador real (secret/permissao faltando)
  no workpad e reportar. Caso contrario, NAO parar — continuar corrigindo.
- Proibido declarar "pronto" sem a matriz 10/10 PASS (criterios com checkpoint humano contam como
  PASS so apos o humano executar o passo e o agent confirmar o resultado).
