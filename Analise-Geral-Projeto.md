# Análise Técnica — EasyCraft RPG Online Idle

> **Data da Análise**: Abril 2026
> **Versão**: 0.4.8 (Frontend e Backend)

---

## 1. Visão Geral

EasyCraft é um RPG Online Idle de fantasia medieval que roda no navegador (PWA) e como aplicação desktop (Electron). Projeto full-stack em TypeScript com renderização 3D, multiplayer em tempo real e 20+ sistemas de gameplay interconectados.

| Métrica | Valor |
| :------ | :---- |
| Entidades no banco | 49 |
| Módulos backend | 35 |
| Componentes React | 329 |
| Views no dashboard | 67 |
| Serviços frontend | 42 |
| Zustand stores | 12 |
| Namespaces i18n | 20+ |
| Idiomas suportados | PT, EN |

---

## 2. Stack Tecnológico

### Frontend

| Tecnologia | Versão | Função |
| :--------- | :----- | :----- |
| React | 18.2 | UI framework |
| TypeScript | 5.2 | Tipagem estática |
| Vite | 7.1 | Build tool |
| Three.js | 0.159 | Renderização 3D |
| @react-three/fiber | 8.15 | React bindings para Three.js |
| @react-three/drei | 9.88 | Helpers e abstrações 3D |
| Zustand | 4.4 | State management |
| TailwindCSS | 3.3 | Estilização utility-first |
| Socket.IO Client | 4.7 | WebSocket client |
| Axios | 1.6 | HTTP client |
| React Router | 6.20 | Roteamento (HashRouter) |
| i18next | 26.0 | Internacionalização |
| react-i18next | 17.0 | React bindings i18n |
| Lucide React | 0.294 | Ícones |
| Electron | 40.0 | Desktop build (Windows) |
| vite-plugin-pwa | 1.2 | Progressive Web App |
| Vitest | 4.0 | Test runner |
| ESLint | 8.57 | Linting (flat config) |
| Prettier | 3.6 | Formatação |

### Backend

| Tecnologia | Versão | Função |
| :--------- | :----- | :----- |
| Node.js | >= 18 | Runtime |
| Express | 4.18 | HTTP framework |
| TypeScript | 5.3 | Tipagem estática |
| TypeORM | 0.3 | ORM (Active Record + Data Mapper) |
| PostgreSQL | 15 | Banco relacional |
| Redis | 7 | Cache e sessões |
| Socket.IO | 4.7 | WebSocket server |
| Zod | 3.22 | Validação de input |
| Pino | 10.1 | Structured logging |
| Helmet | 7.1 | Security headers |
| JWT | 9.0 | Autenticação stateless |
| bcryptjs | 2.4 | Hashing de senhas |
| Jest | - | Test runner backend |

### Infraestrutura

| Componente | Stack |
| :--------- | :---- |
| Containerização | Docker Compose (dev, prod, SSL) |
| Reverse Proxy | Nginx com terminação SSL |
| Banco de Dados | PostgreSQL 15 Alpine (200 max conn, 256MB shared buffers) |
| Cache | Redis 7 Alpine |
| Build Desktop | electron-builder (NSIS, Windows) |
| PWA Cache | Workbox — assets 3D até 50MB, imagens, áudio |

---

## 3. Arquitetura

### Diagrama de Comunicação

```
┌──────────────────────────────────────────────────────┐
│                     CLIENTE                          │
│  React 18 + Three.js + Zustand + Socket.IO Client    │
│  (Browser PWA / Electron Desktop)                    │
└──────────┬───────────────────┬───────────────────────┘
           │ HTTP REST          │ WebSocket
           ▼                    ▼
┌──────────────────────────────────────────────────────┐
│                     SERVIDOR                         │
│  Express 4 + Socket.IO 4 + TypeORM 0.3               │
│  35 módulos de domínio                               │
│                                                      │
│  ┌─────────────────┐  ┌────────────────────────────┐ │
│  │   PostgreSQL 15  │  │       Redis 7              │ │
│  │   49 entidades   │  │  Cache + Sessions          │ │
│  └─────────────────┘  └────────────────────────────┘ │
└──────────────────────────────────────────────────────┘
```

### Padrão Backend

```
Routes (Express Router)
  └── Controller (validação Zod + formatação de response)
       └── Service (lógica de negócio, acesso ao banco)
            └── TypeORM Repository (queries e persistência)
```

- **Services** são singletons exportados
- **Formato de response** padronizado: `{ success: boolean, data?: T, error?: string }`
- **Schema management** via `applySchemaPatches()` — patches incrementais no DataSource, sem migrations tradicionais
- **Entidades** registradas centralmente em `ALL_ENTITIES` array
- **express-async-errors** importado globalmente — handlers async não precisam de try/catch wrapper

### Padrão Frontend

```
Zustand Store (estado global reativo)
  └── Service (chamadas HTTP/WebSocket)
       └── Component React (UI + Three.js)
```

- **Estilização**: Tailwind-only + `cva` (variantes) + `cn()` (merge classes) — tema medieval customizado
- **3D**: React Three Fiber + Drei para vila, batalha e previews
- **Roteamento**: HashRouter (compatível com Electron e PWA)
- **Persistência**: Stores persistem via `safeStorage` wrapper

---

## 4. Módulos Backend (35)

| Módulo | Descrição |
| :----- | :-------- |
| `achievement` | Conquistas com rastreamento de progresso e recompensas |
| `admin` | Painel administrativo com CRUD de todas as entidades, logs e backup |
| `assets` | Gerenciamento de assets (upload de imagens/modelos) |
| `auth` | Autenticação JWT, registro e login |
| `battle` | Combate por turnos — IA de inimigos, skills, buffs, poções |
| `bonus` | Multiplicadores globais do servidor (XP, gold, drop) |
| `character` | CRUD de personagens, atributos, stats derivados |
| `chat` | Chat em tempo real (global, guilda, DM) via Socket.IO rooms |
| `classes` | Classes primárias e secundárias com bônus |
| `crafting` | Receitas, ingredientes, chance de sucesso, XP de crafting |
| `crypt` | Masmorras (dungeon) com andares e progressão |
| `daily-rewards` | Recompensas diárias com milestones progressivos |
| `enhancement` | Encantamento de equipamentos (nível, sucesso, regressão, durabilidade) |
| `feature-unlock` | Desbloqueio progressivo de features por nível e tutorial |
| `friend` | Requisições de amizade, lista, status online/offline |
| `gathering` | Sessões de coleta de recursos, worker de processamento |
| `guild` | Guildas com membros, ranks, banco, convites, habilidades de guilda |
| `inventory` | Gerenciamento de inventário, equipar/desequipar, destruição |
| `item` | Definições de itens do jogo |
| `mail` | Correio do jogo (admin mail e player mail) |
| `marketplace` | Mercado entre jogadores com comissão e histórico |
| `npc-buyer` | NPCs que compram itens dos jogadores |
| `npc-shop` | Lojas de NPCs |
| `pet` | Sistema de pets (templates, captura, gerenciamento) |
| `pvp` | Duelos, desafios ranqueados e estatísticas PvP |
| `quest` | Missões com objetivos, progresso e recompensas |
| `ranking` | Rankings por múltiplas métricas |
| `realtime` | Serviço WebSocket central (rooms, broadcast, presença) |
| `shop-rental` | Loja do jogador — aluguel de espaço, vendas |
| `skills` | Habilidades por classe, upgrade, cooldowns |
| `status` | Sistema de buffs/debuffs com duração por turnos |
| `support` | Tickets de suporte in-game |
| `system` | Configurações globais do servidor |
| `tutorials` | Sistema de tutoriais com passos guiados e triggers |
| `village` | Sincronização multiplayer da vila (posições, nameplates) |

---

## 5. Entidades do Banco de Dados (49)

### Core

| Entidade | Descrição |
| :------- | :-------- |
| `User` | Conta do usuário (auth) |
| `Character` | Personagem do jogador |
| `CharacterStats` | Atributos e stats derivados |

### Classes e Habilidades

| Entidade | Descrição |
| :------- | :-------- |
| `ClassDefinition` | Definições de classes do jogo |
| `CharacterClass` | Classe ativa do personagem |
| `ClassSecondaryMap` | Mapeamento de classes secundárias |
| `Skill` | Habilidades disponíveis |
| `CharacterSkill` | Habilidades aprendidas pelo personagem |

### Itens e Inventário

| Entidade | Descrição |
| :------- | :-------- |
| `Item` | Definições de itens |
| `Inventory` | Itens no inventário (com enchant_level e durability) |
| `Equipment` | Equipamentos equipados por slot |

### Combate

| Entidade | Descrição |
| :------- | :-------- |
| `Battle` | Registro de batalhas |
| `Enemy` | Definições de inimigos (stats, skills, loot) |
| `IdleBattleSession` | Sessões de batalha automática |
| `CryptSpot` | Spots de masmorras |

### Economia

| Entidade | Descrição |
| :------- | :-------- |
| `CraftingRecipe` | Receitas de crafting |
| `GatherSession` | Sessões de coleta de recursos |
| `MarketplaceListing` | Anúncios no marketplace |
| `MarketplaceTransaction` | Transações realizadas |
| `NpcBuyerOrder` | Ordens de compra de NPCs |
| `PlayerShopSlot` | Slots da loja do jogador |

### Quests e Progressão

| Entidade | Descrição |
| :------- | :-------- |
| `Quest` | Definições de missões |
| `CharacterQuest` | Progresso de missões do personagem |
| `Achievement` | Definições de conquistas |
| `CharacterAchievement` | Conquistas desbloqueadas |
| `FeatureUnlock` | Features desbloqueadas por personagem |
| `TutorialState` | Estado do tutorial |
| `DailyRewardConfig` | Configuração de recompensas diárias |
| `CharacterDailyReward` | Recompensas diárias coletadas |
| `DailyRewardMilestone` | Milestones de login |
| `CharacterMilestoneReward` | Milestones alcançados |

### Social

| Entidade | Descrição |
| :------- | :-------- |
| `Guild` | Guildas |
| `GuildMember` | Membros e ranks |
| `GuildInvite` | Convites pendentes |
| `GuildJoinRequest` | Requisições de entrada |
| `GuildMessage` | Mensagens no chat de guilda |
| `GuildBankLog` | Log do banco da guilda |
| `GuildSkill` | Habilidades da guilda (8 skills passivas) |
| `Friendship` | Amizades |
| `FriendRequest` | Requisições de amizade |

### PvP

| Entidade | Descrição |
| :------- | :-------- |
| `PvpBattle` | Batalhas PvP |
| `PvpChallenge` | Desafios PvP |
| `PvpStats` | Estatísticas de PvP |

### Pets

| Entidade | Descrição |
| :------- | :-------- |
| `Pet` | Pet ativo do jogador |
| `PetTemplate` | Templates de pets disponíveis |
| `PendingPet` | Pets aguardando captura |

### Comunicação e Sistema

| Entidade | Descrição |
| :------- | :-------- |
| `AdminMail` | Mensagens administrativas |
| `CharacterMail` | Correio do jogador |
| `SupportTicket` | Tickets de suporte |
| `AdminLog` | Logs de ações administrativas |

---

## 6. Frontend — Componentes e Views

### Estrutura de Componentes (329 arquivos .tsx)

| Diretório | Função |
| :-------- | :----- |
| `dashboard/views/` | 67 views principais do jogo |
| `admin/` | Painel administrativo completo |
| `battle/` | Cenas de combate 3D + HUD |
| `character/` | Criação e preview 3D |
| `chat/` | Chat em tempo real |
| `common/` | Componentes reutilizáveis |
| `crypt/` | Interface de masmorras |
| `dialogue/` | Fila de diálogos com NPCs |
| `friends/` | Sistema de amigos |
| `gathering/` | Coleta de recursos (cena 3D) |
| `guild/` | Guildas, membros, habilidades |
| `hud/` | HUD do jogo (HP, MP, atalhos) |
| `layout/` | Layout base + navegação |
| `login/` | Tela de login e registro |
| `npc-shop/` | Lojas de NPC |
| `pvp/` | Interface PvP |
| `session/` | Gestão de sessão |
| `support/` | Suporte in-game |
| `town/` | Vila 3D interativa |
| `tutorial/` | Tutorial guiado |
| `ui/` | Componentes base (ForgePanel, botões, modais) |

### Views Principais

As views são exibidas dentro do dashboard e representam as telas do jogo:

HomeView, BattleView, PvpView, InventoryView, CraftingView, GatheringView, BlacksmithView, CryptView, QuestsView, SkillsView, ClassesView, StatsView, GuildView, ChatView, MailboxView, MarketplaceView, NpcShopView, RankingView, PetsView, AchievementsView, SettingsView, AppearanceView, DailyRewardsView, GeneralStatusCard, entre outros.

### Stores Zustand (12)

| Store | Responsabilidade |
| :---- | :--------------- |
| `authStore` | Sessão, token, usuário logado |
| `characterStore` | Dados do personagem ativo |
| `inventoryNewStore` | Inventário, equipamentos, filtros |
| `craftingNewStore` | Receitas, crafting, favoritos |
| `questNewStore` | Quests, progresso, recompensas |
| `statusNewStore` | Buffs, debuffs, efeitos ativos |
| `achievementStore` | Conquistas e progresso |
| `assetStore` | Cache de assets 3D e imagens |
| `dailyRewardsStore` | Recompensas diárias, milestones |
| `dialogueQueueStore` | Fila de diálogos com NPCs |
| `featureUnlockStore` | Features desbloqueadas |
| `mailStore` | Correio do jogo |

### Serviços (42)

Camada de comunicação entre stores e API. Cada serviço encapsula chamadas HTTP (Axios) e/ou eventos WebSocket (Socket.IO) para um domínio específico: `achievement`, `auth`, `battle`, `battle-manual`, `battle-potion`, `battle-state`, `character`, `chat`, `classes`, `crafting`, `daily-rewards`, `enhancement`, `feature-unlock`, `friend`, `gathering`, `guild`, `inventory`, `mail`, `marketplace`, `npc-buyer`, `npc-shop`, `pet`, `pvp`, `quest`, `ranking`, `realtime`, `shop-rental`, `skills`, `support`, `tutorials`, entre outros.

---

## 7. Sistema de Internacionalização (i18n)

Implementado com `i18next` + `react-i18next`, suportando português (PT) e inglês (EN).

| Aspecto | Detalhe |
| :------ | :------ |
| Framework | i18next 26 + react-i18next 17 |
| Detecção de idioma | localStorage (`ec_lang`) → navigator |
| Namespaces | 20+ (auth, common, battle, inventory, crafting, guild, pvp, pets, etc.) |
| Dados de jogo | Colunas `name_en`/`description_en` nas tabelas de game data |
| Carregamento | Bundled (sem suspense, sem lazy loading) |

---

## 8. Sistema de Feature Unlock

Desbloqueio progressivo de funcionalidades. 20 features controladas por nível e tutorial.

### Sempre Desbloqueadas

`settings`, `chat`, `friends`

### Desbloqueio por Nível

| Nível | Feature |
| :---- | :------ |
| 2 | Stats |
| 3 | Skills |
| 4 | Ferreiro (Enhancement) |
| 5 | Classes |
| 8 | Pets |
| 10 | NPC Shop |
| 11 | Ranking |
| 12 | PvP |
| 13 | Guild |
| 14 | Appearance |
| 15 | Cripta |

### Desbloqueio por Tutorial

| Tutorial Concluído | Feature Desbloqueada |
| :----------------- | :------------------- |
| `home_battle_intro` | Inventory |
| `inventory_intro` | Quests |
| `quests_intro` | Battle |
| `stats_intro` | Gathering |
| `gathering_select_intro` | Crafting |
| `crafting_intro` | Marketplace |

---

## 9. Sistemas de Jogo — Detalhamento Técnico

### 9.1 Sistema de Combate

- **Batalha manual por turnos** — ações: ataque básico, skills, itens, fugir
- **IA de inimigos** — sistema de decisão com priorização de ações e uso de skills
- **Buffs/Debuffs** — duração por turnos, aplicáveis a jogador e inimigos
- **Poções** — uso manual ou configuração de uso automático por threshold de HP
- **Batalha idle** — modo automático com lógica autônoma e interface de acompanhamento
- **Cena 3D** — visualização de combate com modelos animados e efeitos visuais

### 9.2 Sistema de Enhancement (Encantamento)

- **Enchant level** — níveis de encantamento nos equipamentos
- **Custo** — `baseValue x level x 200`, arredondado para 500
- **Taxa de sucesso** — de 92% (nível 1) a 20% (nível alto)
- **Regressão** — nível 6-7: 25% chance; nível 8+: 50% chance
- **Durabilidade** — `applyDurabilityLoss()` pós-batalha (-1 a -2); durabilidade 0 = item sem bônus

### 9.3 Habilidades de Guilda

8 habilidades passivas que beneficiam todos os membros:

| Categoria | Skills |
| :-------- | :----- |
| Combate | COMBAT_ATK, COMBAT_DEF, COMBAT_HP |
| Recompensas | REWARD_XP, REWARD_GOLD, REWARD_DROP |
| Infraestrutura | INFRA_BANK, INFRA_MEMBERS |

- **Custo**: `base x targetLevel x 2` (em gold da guilda)
- **XP de guilda**: 10% do XP ganho em batalha pelos membros
- **Pontos**: guildLevel - 1 - pontosGastos

### 9.4 Sistema de Pets

- **Templates** de pets definidos no banco (`PetTemplate`)
- **Captura** via sistema de pending pets (`PendingPet`)
- **Gerenciamento** — pet ativo, stats, evolução
- **Feature unlock** — disponível a partir do nível 8

### 9.5 Recompensas Diárias

- **Check-in diário** com recompensas incrementais
- **Milestones** — recompensas bônus ao atingir X dias consecutivos
- **4 entidades** dedicadas: DailyRewardConfig, CharacterDailyReward, DailyRewardMilestone, CharacterMilestoneReward

### 9.6 Economia

- **Crafting** — 6 categorias (Weapons, Armor, Consumables, Materials, Enhancements, Accessories), batch crafting, sistema de favoritos
- **Gathering** — sessões configuráveis com cena 3D, progresso em tempo real, probabilidade de itens
- **Marketplace** — anúncios com comissão, busca/filtros, histórico de transações
- **NPC Buyer** — NPCs que compram itens com preços dinâmicos
- **Shop Rental** — jogadores alugam espaço para vender itens

---

## 10. Renderização 3D

### Tecnologias

- **Three.js 0.159** como engine de renderização
- **React Three Fiber** para integração declarativa com React
- **React Three Drei** para helpers (OrbitControls, Environment, etc.)

### Cenas 3D

| Cena | Uso |
| :--- | :-- |
| Vila (TownScene3D) | Hub principal com movimento, NPCs animados, objetos interativos, multiplayer |
| Batalha (BattleScene3D) | Combate com modelos animados, efeitos visuais, iluminação dinâmica |
| Preview (CharacterPreview3D) | Modelo do personagem com attachments de equipamentos |
| Gathering | Cena de coleta de recursos com barra de progresso |
| Editor (Admin) | Edição 3D da vila com posicionamento de objetos |

### Otimizações 3D

- Cache de modelos GLB via Map
- PWA cache de assets 3D (até 50MB por arquivo)
- Sistema de attachments (armas em bones: RightHand, LeftHand, Head)
- Animações via AnimationMixer

---

## 11. Comunicação em Tempo Real

### Socket.IO

- **Autenticação obrigatória** em todas as conexões (JWT + verificação de ownership)
- **Rooms** separadas por contexto: chat global, guilda, batalha, vila
- **Eventos** organizados por domínio: `character:*`, `chat:*`, `guild:*`, `marketplace:*`, `village:*`

### Casos de Uso

| Funcionalidade | Protocolo |
| :------------- | :-------- |
| Chat (global, guilda, DM) | WebSocket |
| Presença online | WebSocket |
| Posições na vila | WebSocket |
| Notificações de marketplace | WebSocket |
| Stats atualizados | WebSocket |
| Guild notifications | WebSocket |
| Operações CRUD | HTTP REST |
| Upload de assets | HTTP REST (multipart) |
| Autenticação | HTTP REST |

---

## 12. PWA e Desktop

### Progressive Web App

- **Plugin**: vite-plugin-pwa + Workbox
- **Estratégias de cache**:
  - Imagens (png, jpg, svg, webp): CacheFirst, 1 ano, max 500 entries
  - Modelos 3D (glb, gltf, bin): CacheFirst, 1 ano, max 200 entries
  - Áudio (mp3, wav, ogg): CacheFirst, 1 ano, max 300 entries
- **Manifest**: standalone, tema #1a1510, ícone 512x512

### Electron (Desktop)

- **Versão**: Electron 40
- **Builder**: electron-builder (NSIS para Windows)
- **App ID**: `com.easycraft.online`
- **Entry**: `electron/main.js`

---

## 13. Métricas do Projeto

| Categoria | Quantidade |
| :-------- | :--------- |
| Entidades TypeORM | 49 |
| Módulos backend | 35 |
| Componentes React (.tsx) | 329 |
| Views no dashboard | 67 |
| Serviços frontend | 42 |
| Zustand stores | 12 |
| Namespaces i18n | 20+ |
| Endpoints REST | 35+ routers |
| Docker Compose files | 3 (dev, prod, SSL) |

### Painel Administrativo

O admin cobre CRUD completo para todas as entidades do jogo, com:

- Gerenciamento de itens, inimigos, quests, crafting, dungeons, skills, classes
- Gerenciamento de jogadores, personagens e sessões
- Controle de marketplace e economia
- Administração de guildas e PvP
- Configurações globais (multiplicadores, status do servidor)
- Backup e restore do banco de dados
- Logs de auditoria com filtros
- Editor 3D da vila

---

## 14. Qualidade de Código

| Aspecto | Implementação |
| :------ | :------------ |
| Tipagem | TypeScript strict mode (frontend e backend) |
| Linting | ESLint 8 flat config, `--max-warnings 0` |
| Formatação | Prettier |
| Testes backend | Jest + ts-jest |
| Testes frontend | Vitest + @vitest/coverage-v8 + JSDOM |
| Validação de input | Zod em todas as rotas do backend |
| Segurança | Helmet, CORS, bcryptjs, JWT, prepared statements (TypeORM) |
| Logging | Pino (structured JSON logging) |
| Versionamento | Semântico via standard-version |

---

**Versão**: 0.4.8
**Última atualização**: Abril 2026
