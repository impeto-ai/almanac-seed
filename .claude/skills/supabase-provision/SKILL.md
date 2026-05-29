---
name: supabase-provision
description:
  Provisiona o backend Supabase do Almanak — schema, migrations, RLS, Storage e
  Google OAuth. Use ao iniciar o build, antes de qualquer codigo de app, e sempre
  que o schema ou as policies mudarem. Garante os pre-requisitos AUTH, RLS e persistencia.
---

# Supabase Provision

## Goals
- Banco PostgreSQL com as 6 tabelas de dominio da SEED (Secao 6) e RLS habilitado.
- Supabase Auth com Google OAuth funcionando.
- Supabase Storage configurado para guardar os HTMLs (a tabela Version guarda so o path).
- Migrations versionadas e re-aplicaveis.

## Inputs
- `SEED.md` Secao 6 (Domain Model), 7 (Persistence), 8 (Auth), 12 (RLS).
- Env vars do Apendice A da SEED.

## Steps
1. Criar/usar projeto Supabase. Registrar `NEXT_PUBLIC_SUPABASE_URL` e chaves nas env vars (nunca hardcode).
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
