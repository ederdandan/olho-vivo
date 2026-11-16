# 👁️⚡ Olho Vivo — Jogo de Reflexo & Par Relâmpago (PWA)

> **Jogo web mobile-first de percepção visual e reflexo em tempo real para 1 a 4 jogadores.**  
> Projetado para hospedagem 100% estática no **GitHub Pages**, com suporte a Bots IA, Modo Mesa compartilhado e arquitetura preparada para Multiplayer P2P via WebRTC (PeerJS).

---

## 📋 Índice
1. [Visão Geral & Identidade do Produto](#1-visão-geral--identidade-do-produto)
2. [Arquitetura Atual do Protótipo (`index.html`)](#2-arquitetura-atual-do-protótipo-indexhtml)
3. [Motor Matemático (Geometria Projetiva / Plano de Fano)](#3-motor-matemático-geometria-projetiva--plano-de-fano)
4. [Decisões de UX/UI & Regras de Negócio](#4-decisões-de-uxui--regras-de-negócio)
5. [Sistema de Baralhos Temáticos & Temas Visuais](#5-sistema-de-baralhos-temáticos--temas-visuais)
6. [Roadmap para o MVP Completo (Próximos Passos)](#6-roadmap-para-o-mvp-completo-próximos-passos)
7. [Guia Rápido de Manutenção no Código](#7-guia-rápido-de-manutenção-no-código)

---

## 1. Visão Geral & Identidade do Produto

* **Nome:** Olho Vivo (Par Relâmpago)
* **Objetivo:** Encontrar o único símbolo idêntico entre a carta da mesa (centro) e a sua própria carta antes dos adversários, esvaziando a sua pilha primeiro.
* **Blindagem de Propriedade Intelectual:**
  * O projeto utiliza nome, ícone vetorial próprio (olho estilizado atravessado por raio diagonal) e baralhos autorais baseados em emojis nativos e SVGs geométricos.
  * Evita qualquer referência nominal, gráfica ou estrutural a marcas comerciais registradas.
  * Oferece diferenciais nativos do meio digital: alternância de geometria de carta (**Redonda** vs. **Quadrada Arredondada**), modo rápido de **6 símbolos (31 cartas)**, combos de sequência e universos temáticos dinâmicos.

---

## 2. Arquitetura Atual do Protótipo (`index.html`)

Atualmente, todo o protótipo funciona em **arquitetura Single-File** (`index.html`) sem necessidade de build step (Webpack/Vite), facilitando testes imediatos e deploy direto no GitHub Pages:

* **Estilização:** Tailwind CSS (via CDN) + CSS Custom Properties / classes temáticas (`theme-default`, `theme-classic`, `theme-neon`, `theme-wood`, `theme-clean`).
* **Áudio:** Sintetizador nativo via **Web Audio API** (`playTone` / `playSoundEffect`), eliminando o carregamento de arquivos `.mp3`/`.wav` externos e garantindo resposta sonora instantânea no toque.
* **Fluxo de Telas (SPA leve controlada por `switchTab()`):**
  1. `view-universe` (**1. Universo**): Tela inicial para seleção do conjunto de 57 símbolos antes da partida.
  2. `view-lobby` (**2. Sala 4P**): Configuração da sala, código de convite, QR Code, escolha entre *Online P2P* ou *Modo Mesa*, gerenciamento de até 3 Bots e seletor de dificuldade.
  3. `view-arena` (**3. Arena**): Partida principal em tempo real com HUD de pilhas restantes, contagem regressiva (`3-2-1-JÁ!`), trava punitiva de erro (`1.5s`), sistema de combo (`🔥 2x`) e pódio final com média de tempo de reação.
  4. `view-tabletop` (**4. Mesa**): Modo local *Pass-and-Play* simultâneo para 4 jogadores no mesmo aparelho (cartas rotacionadas nas 4 bordas da tela).
  5. `view-architecture` (**5. Matemática**): Inspetor interativo que comprova a interseção única entre quaisquer duas cartas do baralho.
  * `settings-modal` (**⚙️ Ajustes**): Modal global acessível no cabeçalho para troca de tema visual, formato da carta, densidade do baralho, áudio e compensação de ping.

---

## 3. Motor Matemático (Geometria Projetiva / Plano de Fano)

A geração das cartas não é aleatória. Ela é calculada dinamicamente pela função `generateSpotItDeck(n)` utilizando planos projetivos finitos de ordem prima $n$:

$$N = n^2 + n + 1$$

| Configuração no App | Ordem ($n$) | Símbolos por Carta ($n+1$) | Total de Cartas Geradas | Cartas Usadas na Partida |
| :--- | :---: | :---: | :---: | :---: |
| **Clássico (8 Símbolos)** | `7` | `8` | `57` | `55` (padrão de divisão) |
| **Rápido (6 Símbolos)** | `5` | `6` | `31` | `31` |

* **Propriedade Garantida:** Qualquer par de cartas $(A, B)$ gerado pela matriz possui **exatamente 1 índice em comum** ($|A \cap B| = 1$).
* **Layout Visual (`SYMBOL_LAYOUT_8` e `SYMBOL_LAYOUT_6`):** Coordenadas percentuais `(x, y)` e escalas pré-calculadas para evitar sobreposição de ícones dentro da carta e manter áreas de toque (*hitboxes*) de no mínimo `52px × 52px`.

---

## 4. Decisões de UX/UI & Regras de Negócio

As seguintes mecânicas foram implementadas para resolver problemas clássicos de jogos de reflexo em telas touch:

1. **Preservação do Mapa Mental (`renderCenterCardOnly`):**
   * Quando um Bot pontua antes do jogador, **apenas a carta central é substituída** com animação deslizante. A carta inferior do jogador permanece estática na mesma posição e rotação para não causar fadiga ocular.
2. **Trava Anti-Spam (`triggerPenaltyLock`):**
   * Tocar em um símbolo incorreto bloqueia a carta do jogador por **1.5 segundos** com animação *shake* e vibração (`navigator.vibrate`), impedindo cliques aleatórios frenéticos (*button mashing*).
3. **Confirmação Visual do Par:**
   * Ao acertar, os dois símbolos correspondentes recebem destaque esmeralda por `140ms` antes da transição da carta.
4. **Tensão de Reta Final (`last-card-alert`):**
   * Quando qualquer jogador ou Bot fica com apenas **1 carta restante**, o card no placar e o rodapé pulsam em vermelho (`🔥 ÚLTIMA CARTA!`).

---

## 5. Sistema de Baralhos Temáticos & Temas Visuais

### Universos Disponíveis (`DECK_THEMES`)
Cada tema mapeia os índices `0` a `56` para objetos `{ id, icon/svg, name }`:
* `classic`: **Clássico Variado** (emojis cotidianos, animais, objetos).
* `scifi`: **Galáxia Sci-Fi** (inclui SVGs vetoriais autorais em `CUSTOM_SVGS` + ícones espaciais).
* `magic`: **Contos & Magia** (varinha, sapatinho de cristal em SVG + fantasia medieval).
* `heroes`: **Super Heróis** (escudo, martelo em SVG + ação e quadrinhos).

### Temas Visuais (`setVisualTheme`)
Controlados por classes no `<body>`:
* `theme-default` (Indigo Dark)
* `theme-classic` (Real Roxo & Ouro)
* `theme-neon` (Cyber Neon Arcade)
* `theme-wood` (Tabuleiro Madeira & Creme)
* `theme-clean` (Clean Claro Minimalista)

---

## 6. Roadmap para o MVP Completo (Próximos Passos)

Para evoluir o protótipo atual para a versão final instalável e multiplayer online no GitHub Pages:

### Fase 1 — Transformação em PWA Instalável
- [ ] Criar arquivo `manifest.json` na raiz com `display: "standalone"`, `orientation: "portrait"` e ícones PNG (`192x192` e `512x512`) exportados a partir do SVG oficial da **Opção 3**.
- [ ] Criar `sw.js` (Service Worker) com estratégia *Cache-First* para permitir jogar offline contra Bots e no Modo Mesa.
- [ ] Adicionar `<link rel="manifest" href="./manifest.json">` e o registro do Service Worker no `index.html`.

### Fase 2 — Multiplayer Online Real (WebRTC / PeerJS)
- [ ] Importar `peerjs` via CDN.
- [ ] **Fluxo do Host:** Ao abrir a sala `K9X4`, instanciar `new Peer('olho-vivo-' + roomCode)` e gerenciar o estado oficial do baralho.
- [ ] **Fluxo dos Convidados:** Ler o parâmetro `?sala=XXXX` da URL ao abrir o link/QR Code e conectar diretamente ao ID do Host.
- [ ] **Sincronização & Anti-Latência:**
  - Sincronizar relógio inicial (`ping/pong`) entre Host e Clientes.
  - Aplicar o delay adaptativo do Host (`latency-comp-toggle`) ao renderizar novas cartas centrais.
  - Implementar janela de resolução de `100ms` no Host para comparar o `timestamp` real do clique em caso de empate técnico.

### Fase 3 — Melhorias Futuras (Backlog)
- [ ] **Criador de Baralho Customizado:** Permitir que o usuário monte um baralho próprio escolhendo 57 emojis ou imagens locais (`URL.createObjectURL`) salvas no `localStorage`.
- [ ] **Ranking Local / Histórico:** Salvar recordes de menor tempo médio de reação via `localStorage`.

---

## 7. Guia Rápido de Manutenção no Código

| O que você quer alterar | Onde encontrar no `index.html` |
| :--- | :--- |
| **Adicionar um novo Baralho Temático** | Objeto `DECK_THEMES` (garantir exatamente 57 itens nos índices `0` a `56`) + botão na seção `#view-universe`. |
| **Criar novos ícones vetoriais (SVG)** | Objeto `CUSTOM_SVGS` (usar `viewBox="0 0 36 36"` e `w-[1em] h-[1em]`). |
| **Ajustar velocidade/erro dos Bots** | Constante `DIFFICULTY_CONFIG` (`easy`, `medium`, `hard`). |
| **Mudar tempo de bloqueio por erro** | Função `triggerPenaltyLock()` (variável `let remaining = 1.5;`). |
| **Alterar cores de um Tema Visual** | Bloco `<style>` nas classes `body.theme-*` e função `setVisualTheme()`. |
| **Modificar posição dos símbolos na carta** | Constantes `SYMBOL_LAYOUT_8` e `SYMBOL_LAYOUT_6`. |

---

*Documentação gerada para facilitar a evolução contínua do projeto **Olho Vivo PWA**.*