---
name: e2e-verify
description:
  Verifica o fluxo funcional completo do Almanak no browser (login Google, team, projeto,
  upload HTML, versionamento, compartilhamento, comentario, status). Use para produzir
  evidencia PASS/FAIL dos criterios funcionais. Chamada pelo acceptance-loop.
---

# E2E Verify

## Goals
- Executar o fluxo do usuario ponta a ponta e registrar evidencia por criterio.
- Rodar tanto em local quanto contra a URL de producao.

## Inputs
- `SEED.md` Secao 9 (Functional Spec) e Secao 14 (Acceptance Criteria).
- URL alvo (local ou Vercel producao).

## Steps (cobre os 8 criterios funcionais)
1. **AUTH** — acessar rota protegida deslogado -> deve bloquear. Logar via Google -> sessao criada.
2. **TEAM** — criar team, convidar membro por email; simular login do membro -> ve projetos do team.
3. **PROJECT** — criar projeto com nome -> link compartilhavel gerado.
4. **UPLOAD** — subir arquivo HTML numa Option -> versao criada e HTML renderizado (iframe sandbox).
5. **VERSION** — "Add version" na mesma Option -> v2 criada (com autor), v1 preservada, version switcher navega entre v1..vN.
6. **SHARE** — abrir link como 2o membro do team -> ve HTML renderizado.
7. **COMMENT** — criar pin flutuante ancorado no HTML da v2 -> abrir pelo marcador -> responder (thread); "jump to pin" funciona.
8. **STATUS** — alternar open/resolved; painel filtra ALL/OPEN/RESOLVED/RECENT, por versao e busca por autor/corpo.

## Tooling
- Usar Playwright (MCP `playwright` ou `@playwright/test`) para dirigir o browser.
- Google OAuth: usar conta de teste / sessao mockada quando o login interativo nao for automatizavel;
  se nao houver via automatizada, registrar como verificacao manual com screenshot.
- Para cada passo, capturar resultado + screenshot como evidencia.

## Output
Tabela PASS/FAIL por criterio (AUTH..STATUS) com evidencia. Entregar ao acceptance-loop.

## Failure Handling
- Qualquer criterio FAIL -> reportar passo exato + evidencia ao acceptance-loop (nao silenciar).
