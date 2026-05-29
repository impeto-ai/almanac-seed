---
name: frontend-quality
description:
  Padroes de front-end do Almanak — onde a interface E o produto. Use durante toda a
  construcao de UI (login, dashboard, projeto, versoes, comentarios). Diferente do Symphony,
  que trata UI como non-goal, aqui a qualidade visual e funcional e criterio de aceite.
---

# Frontend Quality

> O Almanak nao e um dashboard de observability — e um produto de colaboracao visual. A
> interface precisa parecer production-grade, nao um scaffold generico de IA.

## Direcao visual (referencia do produto real)
- **Estetica clean e editorial.** Fundo off-white/creme, muito respiro/whitespace, layout calmo.
- **Tipografia:** titulos em **serif** (ar editorial, ex: "Individual seed page" em italico serif);
  labels e metadados em **mono UPPERCASE** com letter-spacing (ex: `ALL PROJECTS`, `ACTIVE`, `4 OPTIONS`).
- **Cards** de projeto/opcao: nome em destaque, acao `open ->` discreta, linha de metadados
  (`2 versions · 8 pins · 5 reactions`, `updated 20h ago`), e **stack de avatars circulares
  coloridos** dos membros no rodape do card.
- **Status pill** arredondada com dropdown (ex: `ACTIVE v`).
- **Branding** discreto no topo (`plow • ALMANAC`) e rodape (`INTERNAL · SEEDS · 2026`).
- Paleta sobria; cor forte so em avatars, pins e acoes primarias. Nada pesado/generico.

## Comentarios nivel Google Docs / Figma (core do produto)
- **Painel lateral** "Comments" com: busca por autor/corpo, abas de filtro `ALL / OPEN /
  RESOLVED / RECENT`, filtro por versao (`v1 v2 v3 v4`), e cada item com avatar+autor+tempo+corpo.
- **Pins flutuantes**: comentario ancorado aparece como marcador sobre o HTML renderizado.
  Colapsado, o pin **e o avatar circular da conta Google do autor**. Clicar **expande** o card
  do comentario (autor, "just now", corpo, thread de replies, campo Reply, CLOSE/reply).
  Clicar no comentario do painel da "jump to pin" (rola/destaca a ancora).
- **Threads** (reply) e **reactions** nos comentarios.
- Distinguir visualmente `open` x `resolved` (ex: resolvido recolhido/esmaecido).
- **Aba Activity** (toggle com Comments): feed de eventos "viewed/commented/resolved" por usuario+versao.
- **Atalho** "hold ⌥ to comment" para entrar em modo pin.

## Version switcher
- Breadcrumb `Project / Option / Version` no topo.
- Dropdown "N versions": lista v1..vN com avatar/autor + tempo, badges `LATEST` e `CURRENT`,
  e acao "Add version" (novo upload). Trocar versao re-renderiza o HTML e os pins daquela versao.

## Listagem de projetos
- Titulo serif; filtro de status `ACTIVE / ARCHIVED / SHIPPED`; botao `+ NEW PROJECT`.
- Cada projeto: nome, `N options · N pins`, `open ->`, `updated Xago`.

## Goals
- UI consistente, limpa e responsiva que sustenta o fluxo da SEED (Secao 9 e 11).
- Todos os estados de tela tratados (nunca tela quebrada/vazia sem explicacao).
- Render seguro do HTML do usuario.

## Regras (MUST)
1. **Design consistente** — Tailwind com escala de espacamento, tipografia e paleta coesas.
   Componentes reutilizaveis (botao, card, modal, input) — sem estilos ad-hoc duplicados.
2. **Estados obrigatorios por tela** — toda view com dados MUST tratar: `loading`, `empty`
   (com call-to-action), `error` (com retry) e `success`. Lista vazia nunca aparece "morta".
3. **Responsivo** — mobile-first; o app MUST ser usavel em telas pequenas (sidebar colapsavel,
   sem overflow horizontal).
4. **Feedback de acao** — toda mutacao (criar projeto, upload, comentar, resolver) MUST dar
   feedback imediato (toast/estado otimista/spinner). Sem acao "silenciosa".
5. **Render seguro do HTML** — o HTML enviado pelo usuario MUST ser renderizado em
   `<iframe sandbox>` isolado (sem acesso ao app host). Cross-ref: SEED Secao 9.4 e 12.
6. **Acessibilidade base** — labels em inputs, foco visivel, navegacao por teclado em modais,
   contraste AA. Imagens/avatars com alt.
7. **Navegacao clara** — hierarquia Team > Projeto > Pagina > Versao visivel; breadcrumb ou
   header que situa o usuario. Seletor de versao (v1..vN) obvio.

## Regras (SHOULD)
- Microcopy em PT-BR, direta.
- Skeleton no loading em vez de spinner solto quando houver layout previsivel.
- Transicoes suaves (hover, abertura de modal) sem exagero.
- Painel de comentarios com filtro por status (aberto/resolvido) e por versao bem sinalizado.

## Verificacao (alimenta e2e-verify / acceptance-loop)
- [ ] Cada tela com dados mostra loading + empty + error corretamente.
- [ ] App usavel em viewport mobile (sem overflow, controles acessiveis).
- [ ] HTML do usuario renderiza em iframe sandbox (provar isolamento).
- [ ] Toda acao da feedback visivel.
- [ ] Inputs com label, foco visivel, modal navegavel por teclado.

## Anti-patterns (MUST NOT)
- Tela branca/vazia sem mensagem nem CTA.
- Acao que muda dados sem nenhum feedback.
- Render do HTML do usuario direto no DOM do app (XSS) — sempre iframe sandbox.
- Layout que quebra no mobile.
- Estilos copiados/colados em vez de componente reutilizavel.
