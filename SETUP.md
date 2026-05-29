# SETUP — Germinando a SEED do Almanak

> Equivalente ao "How to use it" do Symphony. Antes de colar a SEED no seu coding agent,
> o **operador** prepara as credenciais no AMBIENTE. A SEED nunca contem secrets (ver
> `SEED.md` Secao 5.1) — ela apenas declara o que precisa e falha alto se faltar.

## Filosofia (igual Symphony)
- **Secret -> ambiente** (env var). Nunca no repo.
- **Config -> arquivo versionado** (`AGENTS.md`).
- **Auth de ferramenta -> CLI pre-autenticado** (`supabase login`, `vercel login`).
- **Faltou credencial REQUIRED? -> o agent para e reporta o bloqueador.** Nao inventa.

## Passo 1 — Contas e ferramentas
- Conta Supabase + Supabase CLI (`supabase login`).
- Conta Vercel + Vercel CLI (`vercel login`).
- Projeto no Google Cloud Console com OAuth consent screen configurado.

## Passo 2 — Credenciais (exportar como env vars)
Crie um `.env.local` (NAO commitar) com:

```
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=        # SOMENTE server-side, nunca no client
SUPABASE_ACCESS_TOKEN=            # sbp_... (criar/configurar projeto via CLI/MCP)

# Google OAuth (via Supabase Auth)
GOOGLE_OAUTH_CLIENT_ID=
GOOGLE_OAUTH_CLIENT_SECRET=

# Vercel
VERCEL_TOKEN=                     # ORG_ID/PROJECT_ID saem de `vercel link`/`vercel pull`
```

Onde obter:
- **Supabase URL/anon/service_role:** Project Settings > API.
- **SUPABASE_ACCESS_TOKEN:** Account > Access Tokens (`sbp_...`).
- **GOOGLE_OAUTH_CLIENT_ID/SECRET:** Google Cloud Console > APIs & Services > Credentials >
  OAuth client ID (Web). Redirect URI = `https://<PROJECT_REF>.supabase.co/auth/v1/callback`.
- **VERCEL_TOKEN:** Vercel > Account Settings > Tokens.

## Passo 3 — Germinar
Cole no seu coding agent (Claude Code, Codex, Cursor, Gemini CLI):

> Implemente o software descrito em `SEED.md`. Antes de comecar, leia `AGENTS.md` e siga as
> skills em `.claude/skills/` na ordem indicada. Nao declare concluido ate o Definition of
> Done (SEED Secao 16) passar inteiro, com deploy de producao verificado.

O agent vai: provisionar Supabase (`supabase-provision`) -> configurar login Google
(`google-oauth`) -> deploy Vercel (`vercel-deploy`) -> verificar fluxo (`e2e-verify`) ->
rodar a matriz de aceite em loop (`acceptance-loop`) ate 10/10 PASS.

## Passo 4 — Validar
Definition of Done atingido quando: os 10 criterios de aceite = PASS e a URL de producao
na Vercel executa o fluxo completo (login Google -> team -> projeto -> upload -> versao ->
compartilhar -> comentar -> resolver).
