---
name: supabase-provision
description:
  Provisiona o backend Supabase do Almanak — schema, migrations, RLS, Storage e
  Google OAuth. Use ao iniciar o build, antes de qualquer codigo de app, e sempre
  que o schema ou as policies mudarem. Garante os pre-requisitos AUTH, RLS e persistencia.
---

# Supabase Provision

> **A skill NAO cria projeto Supabase.** O operador fornece um projeto existente (free serve)
> via env vars (checkpoint H0 da SEED §16.1). Esta skill apenas CONFIGURA esse projeto. O
> caminho canonico e CLI/SDK do Supabase (portavel). O MCP do Supabase, se presente no ambiente,
> e um **atalho OPCIONAL** para acelerar migrations/sql — nunca um requisito.

## Goals
- Configurar o projeto Supabase FORNECIDO: 6 tabelas de dominio (SEED Secao 6) + RLS habilitado.
- Supabase Auth com Google OAuth funcionando.
- Supabase Storage configurado para guardar os HTMLs (a tabela Version guarda so o path).
- Migrations versionadas e re-aplicaveis.

## Inputs (do ambiente — ver SEED Apendice A)
- `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`,
  `SUPABASE_PROJECT_REF` (projeto ja existente, fornecido pelo operador).
- `SUPABASE_ACCESS_TOKEN` (OPCIONAL) — para habilitar o provider Google via Management API.
- `SEED.md` Secao 6 (Domain Model), 7 (Persistence), 8 (Auth), 12 (RLS).

## Steps
1. **Verificar o projeto fornecido.** Se as env vars REQUIRED (URL/anon/service_role/ref) estiverem
   ausentes -> PARAR no checkpoint H0 e pedir ao operador (SEED §16.1). NAO criar projeto novo.
   - `supabase link --project-ref $SUPABASE_PROJECT_REF` para operar via CLI.
2. Escrever migration SQL em `supabase/migrations/` com as tabelas: `profiles`, `teams`,
   `team_members`, `projects` (com `status`), `options`, `versions` (com `author_id`),
   `comments` (com `pin_x/pin_y`, `parent_id`) — tipos conforme SEED Secao 6.
3. Habilitar RLS em TODAS as tabelas de dominio. Escrever policies:
   - usuario so le/escreve dados de teams onde e membro (`team_members.status = 'active'`).
   - membro de um team NAO acessa dados de outro team.
   - **comentarios/pins:** policy de insert com `with check (author_id = auth.uid())` — o autor
     e sempre o usuario Google logado, nao pode ser forjado pelo client.
   - `service_role` bypassa (uso server-side apenas).
4. Configurar Storage bucket para HTML; policy de acesso restrita a membros do team dono.
5. Configurar Google OAuth no Supabase Auth (Client ID/Secret via env, nunca commitado).
6. Adicionar trigger/handler que cria `profiles` no primeiro login e promove `team_members`
   pendentes (match por `invited_email`).
7. Acessar o cliente sempre via `@supabase/supabase-js` + `@supabase/ssr`. ORM e PROIBIDO.

## Validation
- [ ] `select` em tabela de outro team retorna vazio/negado (RLS provado).
- [ ] Login Google retorna sessao valida e cria profile.
- [ ] Upload de arquivo no Storage retorna path persistivel.
- [ ] Migrations re-aplicaveis sem erro.

## Failure Handling
- RLS ausente em qualquer tabela = bloqueador. Nao prosseguir.
- Service-role key exposta no client = bloqueador de seguranca. Mover para server-side.
