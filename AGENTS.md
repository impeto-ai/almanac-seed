# AGENTS.md — Regras do Construtor (Almanak)

> Este arquivo e a **fonte de verdade das regras** que qualquer coding agent DEVE seguir ao
> materializar a `SEED.md`. Padrao aberto (Agentic AI Foundation) — lido nativamente por
> Codex, Cursor, Copilot, Windsurf, Amp, Devin. Claude Code le via shim `CLAUDE.md`,
> Gemini via `GEMINI.md`. Leia `SEED.md` para O QUE construir; este arquivo define O COMO.

## Ordem de leitura obrigatoria
1. `SEED.md` — a receita (o que o software faz + criterios de aceite + build loop).
2. Este `AGENTS.md` — regras de stack e qualidade.
3. `rules/*.md` — regras detalhadas por tema (se presentes).

## Stack (MUST)
- **Framework:** Next.js (App Router) + TypeScript. SPA-experience servida na Vercel.
- **Banco/Auth/Storage:** Supabase. Acesso EXCLUSIVO via `@supabase/supabase-js` +
  `@supabase/ssr`. **ORM (Prisma, Drizzle, TypeORM, etc) e PROIBIDO.**
- **Auth:** Google OAuth via Supabase Auth. Nenhum outro provider.
- **Deploy:** Vercel (producao). Build MUST passar.
- **UI:** Tailwind CSS. A interface E o produto — seguir a skill `frontend-quality`
  (estados loading/empty/error, responsivo, feedback de acao, render seguro em iframe sandbox,
  acessibilidade base). Qualidade de front e criterio de aceite, nao acessorio.

## Regras de seguranca (MUST)
- RLS habilitado em TODAS as tabelas de dominio. Sem RLS = nao concluido.
- `SUPABASE_SERVICE_ROLE_KEY` SOMENTE server-side. NUNCA exposto ao client/bundle.
- Segredos sempre em env vars (ver Apendice A da SEED). Hardcode de segredo = PROIBIDO.
- HTML enviado por usuario renderizado em `<iframe sandbox>` (evitar XSS no app host).

## Convencoes de codigo (SHOULD)
- Estrutura por feature: `app/(rotas)`, `lib/supabase/`, `components/`, `supabase/migrations/`.
- Server Components por padrao; Client Components so quando ha interatividade.
- Acesso a dados em modulos dedicados (`lib/queries/*`), nao espalhado em componentes.
- Migracoes SQL versionadas em `supabase/migrations/` (uma intencao por arquivo).
- Nomes em ingles; mensagens de UI podem ser PT-BR.

## Definition of Done (MUST — espelha Secao 16 da SEED)
O agent NAO PODE declarar conclusao ate:
1. Os 10 criterios de aceite (Secao 14 da SEED) = `PASS` na Validation Matrix.
2. Deploy de producao na Vercel respondendo o fluxo completo
   (login Google -> team -> projeto -> upload -> versao -> compartilhar -> comentar -> resolver).
3. RLS validado (acesso cross-team negado).

Se qualquer item falhar, o agent DEVE corrigir e re-executar a matriz inteira. Operar em loop
ate verde. Auto-verificacao e obrigatoria.

## Skills (procedimentos obrigatorios)

Este projeto define skills em `.claude/skills/` (nativas no Claude Code). Qualquer agent —
mesmo sem suporte nativo a skills — DEVE ler e seguir os `SKILL.md` correspondentes como
procedimento. Ordem de uso no build:

1. **`supabase-provision`** — primeiro. Backend, Storage, RLS, migrations.
2. **`google-oauth`** — login Google (Google Cloud Console + Supabase + redirect URIs local/prod).
3. **`frontend-quality`** — durante toda a construcao de UI (a interface E o produto).
4. **`vercel-deploy`** — deploy de producao + env vars + smoke.
5. **`e2e-verify`** — verificacao funcional no browser (criterios AUTH..STATUS).
6. **`acceptance-loop`** — ultimo e obrigatorio. Roda a Validation Matrix e ITERA ate 10/10 PASS.

Credenciais: o agent le secrets do AMBIENTE (Apendice A da SEED), nunca do repo. Se faltar
credencial REQUIRED, parar e reportar o bloqueador (escape hatch). Ver SEED Secao 5.1.

As skills do tipo Symphony (commit/push/pull/land/linear) NAO se aplicam: Almanak e um build
unico germinado da SEED, sem ciclo de PR/tracker.

## Agentic Build Protocol (como operar em loop E2E)

> Espelha o `WORKFLOW.md` do Symphony, adaptado para um build unico (sem tracker/PR). E o que
> faz o agent CONSTRUIR ponta a ponta sozinho, em vez de implementar um pedaco e parar.

### Postura (MUST)
- Esta e uma sessao de construcao **autonoma/unattended**. **Nunca** peca ao humano para
  executar passos de follow-up.
- So pare cedo por **bloqueador real**: credencial/permissao/secret REQUIRED ausente (ver SEED
  §5.1). Nesse caso, registre o bloqueador no workpad (o que falta, por que bloqueia, acao exata
  para destravar) e pare. Caso contrario, **continue ate o Definition of Done**.
- **Checkpoints humanos sao diferentes de bloqueadores** (ver SEED §16.1, H1-H4): sao pausas
  PLANEJADAS (OAuth interativo etc). No checkpoint, faca tudo o mais que e automatizavel, pause
  com instrucao exata (`waiting on human: H#`), e **retome o loop** quando o humano agir. Nao
  encerre a sessao tratando checkpoint como fracasso.
- A mensagem final reporta apenas acoes concluidas e bloqueadores. Sem "proximos passos para o usuario".

### Step 0 — Kickoff: determinar estado e rotear
1. Ler `SEED.md` (receita), este `AGENTS.md` (regras) e as skills em `.claude/skills/`.
2. Detectar o estado atual do build: o repo ja tem app? Supabase provisionado? Deploy no ar?
3. Rotear:
   - Nada feito -> comecar do `supabase-provision`.
   - Parcial -> **continuar de onde parou** (ver Continuation). NAO refazer o que ja esta PASS.
   - Tudo implementado -> ir direto ao `acceptance-loop`.

### Plan-first (MUST)
- Antes de implementar, escrever o plano hierarquico no **workpad** e fazer um self-review dele.
- Investir esforco em planejamento e desenho de verificacao ANTES de codar.
- Espelhar a Validation Matrix (SEED §14) no workpad como checklist de aceite obrigatorio.

### Continuation (retomar sem refazer)
- Ao reiniciar/retomar: ler o workpad, marcar o que ja esta concluido/PASS e **retomar do
  primeiro item incompleto**. Nao repetir investigacao/validacao ja feita salvo se o codigo mudou.
- Tratar a matriz de aceite como o mapa: itens PASS ficam; itens FAIL voltam para o loop.

## Completion Bar (espelha WORKFLOW.md do Symphony)

O agent NAO declara conclusao ate TODOS abaixo:
- [ ] `supabase-provision` concluida; RLS habilitado e provado (acesso cross-team negado).
- [ ] App buildando e deployada em URL de producao na Vercel.
- [ ] `e2e-verify` executada; criterios funcionais AUTH..STATUS = PASS com evidencia.
- [ ] `acceptance-loop` fechada com matriz 10/10 PASS.
- [ ] Login Google funcionando em PRODUCAO (nao so local).
- [ ] Qualidade de UI conforme `frontend-quality` (estados loading/empty/error, responsivo,
      iframe sandbox, feedback de acao) verificada.

## Workpad (rastreio de progresso obrigatorio)

Manter um checklist persistente vivo durante todo o build (plan / acceptance matrix / fixes /
notes), atualizado a cada milestone. Nunca deixar item concluido sem marcar. E a fonte de
verdade do progresso — garante que nada se perde se a sessao reiniciar.

## Provisionamento — ferramenta flexivel, artefato fixo
- Para configurar o Supabase, use o que estiver disponivel: **CLI/SDK OU MCP**. A ferramenta e livre.
- **MUST (gate de portabilidade):** as migrations DEVEM ficar versionadas como SQL em
  `supabase/migrations/`, re-aplicaveis por qualquer um via `supabase db push`. O que reproduz no
  ambiente do avaliador e o SQL versionado no repo, nao a ferramenta usada para aplicar.
- Se nao houver projeto Supabase definido, PERGUNTAR ao operador qual usar ou se vai criar um novo
  (checkpoint H0). Nao criar projeto sem confirmacao.

## O que NAO fazer (MUST NOT)
- Introduzir ORM.
- Outro provedor de auth alem de Google.
- Outro banco alem de Supabase.
- Outro alvo de deploy alem de Vercel.
- Expor service-role key no client.
- Declarar pronto sem rodar a Validation Matrix.
