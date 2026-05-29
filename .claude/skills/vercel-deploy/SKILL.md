---
name: vercel-deploy
description:
  Faz o deploy de producao do Almanak na Vercel e roda o smoke test do fluxo completo.
  Use quando a implementacao estiver pronta para validar em producao e sempre que o
  acceptance-loop exigir re-deploy. Garante o gate critico DEPLOY.
---

# Vercel Deploy

## Goals
- App acessivel em URL publica de producao na Vercel.
- Env vars (Apendice A da SEED) configuradas na Vercel e no Supabase.
- Smoke test do fluxo ponta a ponta passando em producao.

## Inputs
- `SEED.md` Secao 5 (Prerequisites) e 19 (Deployment Contract).
- Build local verde antes de qualquer deploy.

## Steps
1. Garantir `next build` local passa sem erro.
2. Configurar env vars de PRODUCAO na Vercel (uma a uma):
   ```
   vercel env add NEXT_PUBLIC_SUPABASE_URL production
   vercel env add NEXT_PUBLIC_SUPABASE_ANON_KEY production
   vercel env add SUPABASE_SERVICE_ROLE_KEY production      # server-only
   vercel env add GOOGLE_OAUTH_CLIENT_ID production
   vercel env add GOOGLE_OAUTH_CLIENT_SECRET production
   ```
3. Adicionar a URL de producao da Vercel como Redirect URL no Supabase Auth (Site URL + Redirect URLs)
   — coordenar com a skill `google-oauth`.
4. Build + deploy de producao via CLI:
   ```
   vercel pull --yes --environment=production --token=$VERCEL_TOKEN
   vercel build --prod --token=$VERCEL_TOKEN
   vercel deploy --prebuilt --prod --token=$VERCEL_TOKEN
   ```
   (ou git push na branch de producao com o projeto conectado ao repo).
5. Rodar smoke test contra a URL de producao (delegar a `e2e-verify`).

## Validation
- [ ] URL de producao responde 200 na home.
- [ ] Login Google funciona em producao (nao so local).
- [ ] Fluxo completo executa em producao via `e2e-verify`.

## Failure Handling
- Build falha = corrigir antes de deploy.
- Auth quebra em producao (mas funciona local) = checar redirect URLs no Google/Supabase.
- Variavel faltando em producao = adicionar na Vercel e re-deploy.
