---
name: google-oauth
description:
  Configura o login Google do Almanak ponta a ponta — Google Cloud Console (OAuth consent +
  Client ID) e o casamento dos redirect URIs com Supabase e Vercel. Use junto com
  supabase-provision. Esta e a fonte #1 de bug ("funciona local, quebra em producao").
---

# Google OAuth (via Supabase)

> O Google OAuth no Almanak e intermediado pelo Supabase Auth. Mas a criacao do OAuth Client
> vive FORA do Supabase (Google Cloud Console) e os redirect URIs cruzam Supabase + Vercel.
> Errar o redirect URI e a falha mais comum. Esta skill existe para blindar isso.

## Goals
- OAuth 2.0 Client ID criado no Google Cloud Console.
- Provider Google habilitado no Supabase com Client ID/Secret (via env, nunca commitado).
- Redirect URIs corretos para LOCAL e PRODUCAO.
- `signInWithOAuth({ provider: 'google' })` funcionando em ambos.

## Steps — Google Cloud Console
1. Criar/selecionar projeto no Google Cloud Console.
2. APIs & Services > OAuth consent screen > configurar (External).
3. APIs & Services > Credentials > Create Credentials > OAuth client ID.
4. Application type: **Web application**.
5. **Authorized redirect URIs** — adicionar o callback do SUPABASE (nao da app):
   ```
   https://<PROJECT_REF>.supabase.co/auth/v1/callback
   ```
6. Copiar **Client ID** e **Client Secret**.

## Steps — Supabase
7. Authentication > Providers > Google: habilitar e colar Client ID/Secret.
   - Via Management API: `external_google_enabled: true`, `external_google_client_id`, `external_google_secret`.
   - Local (`config.toml`): `[auth.external.google] enabled=true client_id=... secret=env(...)`.
8. Configurar **Site URL** e **Redirect URLs** no Supabase Auth:
   - Site URL = URL de producao da Vercel (ex: `https://almanak.vercel.app`).
   - Additional Redirect URLs = `http://localhost:3000/**` e `https://<vercel-url>/**`.

## Steps — App (Next.js)
9. Login:
   ```ts
   await supabase.auth.signInWithOAuth({
     provider: 'google',
     options: { redirectTo: `${origin}/auth/callback` }
   })
   ```
10. Rota de callback `/auth/callback` faz `exchangeCodeForSession(code)` (PKCE) e redireciona logado.

## Validation
- [ ] Login Google funciona em LOCAL (`localhost:3000`).
- [ ] Login Google funciona em PRODUCAO (URL Vercel) — teste explicito, e o que costuma quebrar.
- [ ] Apos login, sessao valida + profile criado (ver supabase-provision trigger).

## Failure Handling
- `redirect_uri_mismatch` -> o URI no Google Console nao bate com `https://<ref>.supabase.co/auth/v1/callback`. Corrigir no Google.
- Funciona local mas quebra em producao -> faltou a URL da Vercel em Site URL / Redirect URLs do Supabase.
- Client Secret commitado -> bloqueador de seguranca. Mover para env/secret manager.
