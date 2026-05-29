# Almanak — SEED

> Uma **SEED**: a receita executavel de um software. Cole `SEED.md` no seu coding agent
> (Claude Code, Codex, Cursor, Gemini CLI) e ele materializa o **Almanak** do zero — uma
> ferramenta para compartilhar HTMLs com o time, versionar e comentar com pins flutuantes
> ancorados no proprio HTML (estilo Figma/Google Docs).

Inspirado no conceito de spec-driven do [OpenAI Symphony](https://github.com/openai/symphony):
distribuir a receita do software, nao o codigo.

## O que e o Almanak
- Login com Google.
- Times: convide membros, todos com acesso.
- Upload de HTML -> versionamento (v1..vN) com autor por versao.
- Link compartilhavel por projeto.
- **Pins:** clique num ponto do HTML renderizado e deixe uma nota EM CIMA do local — o pin e
  o avatar Google de quem comentou; clicar expande o comentario (thread, reply, resolver).
- Painel de comentarios nivel Google Docs: filtros ALL/OPEN/RESOLVED/RECENT, por versao, busca.

## Como germinar
1. Leia [`SETUP.md`](./SETUP.md) — prepare as credenciais (Supabase, Google OAuth, Vercel) no ambiente.
2. Cole no seu agent:
   > Implemente o software descrito em `SEED.md`. Antes, leia `AGENTS.md` e siga as skills em
   > `.claude/skills/` na ordem. Nao declare concluido ate o Definition of Done (SEED §16) passar
   > inteiro, com deploy de producao verificado.
3. O agent provisiona Supabase -> configura Google OAuth -> deploy Vercel -> verifica o fluxo ->
   roda a matriz de aceite em loop ate 10/10 PASS.

## Estrutura
```
SEED.md      # a receita (20 secoes, RFC 2119, dominio, DoD, build loop)
AGENTS.md    # regras do construtor: stack, skills, agentic build protocol, completion bar
SETUP.md     # setup de credenciais do operador
CLAUDE.md / GEMINI.md   # shims para portabilidade entre harnesses
.claude/skills/         # procedimentos: supabase-provision, google-oauth,
                        #   frontend-quality, vercel-deploy, e2e-verify, acceptance-loop
```

## Stack alvo
Next.js (App Router) + Supabase (DB/Auth/Storage, sem ORM) + Google OAuth + deploy Vercel.

---
Construido como SEED — a interface E o produto, e o agent constroi ate passar nos criterios de aceite.
