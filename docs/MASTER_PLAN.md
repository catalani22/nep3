# NEP3 — MASTER PLAN DE EXECUÇÃO
> Plano completo e detalhado para construção do Neptune Crypto SaaS Super App  
> **Domínio:** nep3.app | **Marca:** Neptune Crypto SaaS | **Ícone:** 🔱 Tridente  
> Última atualização: Abril 2026

---

## CONTEXTO E ESTADO ATUAL

### O que já existe e funciona

| Produto | Repositório | Status |
|---------|-------------|--------|
| Whale Signals | catalani22/whale-signals-app | ✅ Produção — WhaleTube, Sentinela, XWhale operando |
| Airdrop Pulse | catalani22/Airdrop-Pulse | 🔧 Stage 2 completo — pendências listadas abaixo |
| NEP3 | catalani22/nep3 (este repo) | 📐 Fase de planejamento |

### O que o NEP3 absorve dos projetos existentes

- **Whale Signals** → API de dados de baleias alimenta o **Copy Trade Bot** do NEP3 Command
- **Airdrop Pulse Discovery** → vira o módulo **/airdrop** do NEP3
- **Airdrop Pulse Launch** → vira o módulo **/launch** do NEP3
- Ambos abrem **internamente** no NEP3 (não nova aba), como itens de menu

---

## PRÉ-REQUISITO: CONCLUIR AIRDROP PULSE

**Antes de iniciar o NEP3, o Airdrop Pulse deve estar 100% funcional.**  
Nada mockado. Nada de template. Tudo real.

### O que falta no Airdrop Pulse

#### AP-1: GasSniper — API Real
**Status:** Mock atual (dados fake)  
**Solução:**  
- Etherscan Gas API: `https://api.etherscan.io/api?module=gastracker&action=gasoracle`
- BscScan Gas API: `https://api.bscscan.com/api?module=gastracker&action=gasoracle`
- Base: sem gas tracker nativo → usar `eth_gasPrice` via Infura/Alchemy  
- Solana: `connection.getRecentPrioritizationFees()` da `@solana/web3.js`
- Implementar em `api/gas-tracker.ts` com cascade fallback (Etherscan → Infura → node público)
- Atualizar a UI do GasSniper para consumir endpoint real `/api/gas-tracker`
- WebSocket para atualização em tempo real (ou polling 30s)

#### AP-2: WhaleWatcher — Dados Reais de Carteiras
**Status:** Mock atual  
**Solução:**  
- Helius API (Solana): `GET /v0/addresses/{address}/transactions` — gratuito até 10K req/dia
- Moralis API (EVM): `GET /api/v2.2/{address}/erc20/transfers` — plano free 40K req/mês
- Definir critério de "baleia": wallet com >$100K em ativos
- Feed de transações recentes de carteiras rastreadas
- Alertas on-chain: compra grande → notificação no WhaleWatcher
- Integração com mesma base de dados do Whale Signals (cross-product)
- Implementar em `api/whale-watcher.ts`

#### AP-3: One-Click Strategy (Swap Integrado)
**Status:** Não implementado  
**Solução:**  
- Integrar Li.Fi Widget SDK para swaps cross-chain
- Na página de um token/airdrop: botão "Comprar com Li.Fi"
- Injeta `integrator` fee: 50bps (0.5%) para carteira Neptune
- Documentação: https://docs.li.fi/integrate-li.fi-js/li.fi-widget
- Implementar em `src/components/SwapWidget.tsx`
- Este componente **será reutilizado no NEP3** — construir já pensando nisso

#### AP-4: Admin Panel — Gerenciar Airdrops no Firestore
**Status:** Não existe  
**Solução:**  
- Rota `/admin` com autenticação por email hardcoded (airdroppulse.app@gmail.com)
- CRUD de airdrops: adicionar, editar, aprovar, desaprovar
- Dashboard de projetos lançados: ver tokens, fees coletadas, flags
- Painel de wallets: ver saldo das platform wallets A/B/C
- Painel de rate limits: ver quem está sendo limitado
- Implementar em `src/pages/AdminPanel.tsx` + `api/admin/*.ts`

#### AP-5: Gamificação / XP (MVP)
**Status:** Schema parcial no Firestore  
**Solução (MVP — não over-engineer):**  
- XP por: lançar token (+100 XP), distribuir airdrop (+50 XP), usar GasSniper (+5 XP)
- Badges: "Token Creator", "Airdrop Master", "Whale Tracker"
- Leaderboard semanal (top 10 criadores por volume)
- Implementar em `api/xp.ts` + componente `<UserBadges />` no perfil

#### AP-6: SUI — Stub → Implementação Real
**Status:** Stub (Stage 3 conforme MASTER_GUIDE)  
**Solução:**  
- SUI SDK: `@mysten/sui.js` v1+
- Criar token SUI fungível via Move module padrão
- Integrar Aftermath Finance para bonding curve (se disponível) ou Turbos Finance
- Wallet: Suiet ou SUI Wallet via `@mysten/dapp-kit`
- Taxa: injection na criação do token Move (0.5% para treasury SUI)

#### AP-7: Claim Real de Airdrops
**Status:** Links externos apenas  
**Solução:**  
- Para tokens lançados **dentro** do AP: verificar saldo no contrato MerkleTree
- Botão "Resgatar via Neptune (2% success fee)" — cobra 2% do valor resgatado
- Implementar `api/airdrop-claim.ts`: valida endereço no merkle tree, executa claim, cobra fee
- Para tokens **externos**: link para site oficial (sem fee — não temos controle)

---

## FASE 0: SETUP DO REPOSITÓRIO NEP3

**Objetivo:** Criar a base do projeto antes de começar a codar qualquer feature.

### 0.1 — Criar Projeto Base
```bash
# Na máquina local
gh repo clone catalani22/nep3
cd nep3

# Inicializar com Vite + React + TypeScript
npm create vite@latest . -- --template react-ts
npm install

# Instalar dependências base
npm install tailwindcss @tailwindcss/vite
npm install -D typescript @types/react @types/react-dom

# Configurar Tailwind v4 (mesmo padrão do Airdrop Pulse)
```

### 0.2 — Estrutura de Pastas
```
nep3/
├── src/
│   ├── components/
│   │   ├── layout/        → Sidebar, Header, MobileNav
│   │   ├── swap/          → SwapWidget, RouteDisplay, ChainSelector
│   │   ├── bots/          → BotCard, BotConfig, BotStatus
│   │   ├── defi/          → LendingWidget, YieldCard, StakingPanel
│   │   ├── portfolio/     → PortfolioOverview, PositionCard
│   │   └── shared/        → ChainBadge, TokenIcon, PriceDisplay
│   ├── pages/
│   │   ├── Dashboard.tsx  → Home com overview de posições
│   │   ├── Swap.tsx        → Neptune Swap
│   │   ├── Bridge.tsx      → Cross-chain Bridge
│   │   ├── Bots.tsx        → Neptune Command Hub
│   │   ├── DeFi.tsx        → Lending + Yield
│   │   ├── Airdrop.tsx     → Embed do Airdrop Pulse Discovery
│   │   ├── Launch.tsx      → Embed do Airdrop Pulse Forge
│   │   └── Portfolio.tsx   → Posições unificadas
│   ├── lib/
│   │   ├── api-orchestrator.ts  → Cascade fallback de APIs
│   │   ├── chain-config.ts      → RPCs, explorers, tokens por chain
│   │   ├── fee-injector.ts      → Injeta 0.5% em todas as integrações
│   │   └── wallet-connect.ts    → wagmi + RainbowKit + SUI Wallet
│   ├── hooks/
│   │   ├── useChain.ts          → Chain ativa, switch de chain
│   │   ├── usePortfolio.ts      → Balances multi-chain em tempo real
│   │   └── usePrices.ts         → Preços via DexScreener + CoinGecko cascade
│   └── App.tsx
├── api/
│   ├── swap-route.ts            → Melhor rota (Jupiter / 1inch / Li.Fi)
│   ├── bot-execute.ts           → Executa ordens dos bots
│   ├── portfolio.ts             → Agrega posições de todas as chains
│   └── prices.ts                → Cascade: DexScreener → BirdEye → CoinGecko
├── docs/                        → (esta pasta)
├── vercel.json
└── package.json
```

### 0.3 — Variáveis de Ambiente (Vercel)
```
# Fee wallets (recebem 0.5% de tudo)
NEPTUNE_FEE_WALLET_SOL=...
NEPTUNE_FEE_WALLET_EVM=...     # Base, BNB, ETH, ARB usam mesma EVM
NEPTUNE_FEE_WALLET_SUI=...

# RPCs (Tier 1 → Tier 2 → Tier 3 fallback)
RPC_SOLANA_1=https://rpc.ankr.com/solana/...
RPC_SOLANA_2=https://mainnet.helius-rpc.com/?api-key=...
RPC_SOLANA_3=https://api.mainnet-beta.solana.com

RPC_BNB_1=https://rpc.ankr.com/bsc/...
RPC_BNB_2=https://bsc-dataseed.binance.org
RPC_BNB_3=https://bsc.publicnode.com

RPC_BASE_1=https://rpc.ankr.com/base/...
RPC_BASE_2=https://mainnet.base.org
RPC_BASE_3=https://base.publicnode.com

RPC_ETH_1=https://rpc.ankr.com/eth/...
RPC_ETH_2=https://cloudflare-eth.com
RPC_ETH_3=https://eth.publicnode.com

RPC_ARB_1=https://rpc.ankr.com/arbitrum/...
RPC_ARB_2=https://arb1.arbitrum.io/rpc
RPC_ARB_3=https://arbitrum.publicnode.com

# APIs de dados
DEXSCREENER_API=https://api.dexscreener.com   # sem key, gratuito
BIRDEYE_API_KEY=...                            # free tier
COINGECKO_API_KEY=...                          # free tier
HELIUS_API_KEY=...
MORALIS_API_KEY=...

# Bot execution
BOT_EXECUTOR_SECRET=...    # assina ordens dos bots

# Whale Signals API (cross-product)
WHALE_SIGNALS_API_URL=https://mxvinoeuvquydakickoz.supabase.co/functions/v1
WHALE_SIGNALS_API_KEY=...
```

---

## FASE 1: SHELL DO SUPER APP

**Objetivo:** Interface multi-chain funcionando, navegação completa, wallets conectando.  
**Resultado:** Usuário consegue navegar por todos os módulos (mesmo que alguns sejam placeholder).

### 1.1 — Layout Principal (Sidebar + Header)
- Sidebar fixa com ícones + labels: Swap, Bridge, Bots, DeFi, Airdrop, Launch, Portfolio
- Header: logo NEP3 + tridente, chain selector dropdown, wallet connect button
- Mobile: bottom navigation com 5 ícones principais
- Tema: azul profundo (#0A0E1A) + ciano neon (#00D4FF) + cinza metálico (#8B9CB6)
- Fonte: Inter para UI, Space Grotesk para números/valores

### 1.2 — Chain Selector Multi-Chain
- Dropdown com todas as chains suportadas + ícone + nome + status (mainnet/testnet)
- Tier 1 (launch): Solana 🟣, BNB Chain 🟡, Base 🔵, SUI 🔷
- Tier 2 (fase 2): Ethereum ⚪, Arbitrum 🔵, Polygon 🟣, Avalanche 🔴, TON 💎
- Persiste chain selecionada no localStorage
- `useChain()` hook disponível em qualquer componente

### 1.3 — Wallet Connect Multi-Chain
- EVM: wagmi v2 + RainbowKit v2 (MetaMask, Coinbase, WalletConnect)
- Solana: `@solana/wallet-adapter-react` + Phantom + Backpack
- SUI: `@mysten/dapp-kit` + Suiet + SUI Wallet
- Estado unificado: `useWallet()` retorna `{evmAddress, solAddress, suiAddress}`
- Painel de carteiras: mostra saldo em cada chain conectada

### 1.4 — API Orchestrator (núcleo do sistema)
```typescript
// lib/api-orchestrator.ts
class ApiOrchestrator {
  async getRPC(chain: Chain): Promise<string>
  async getTokenPrice(address: string, chain: Chain): Promise<number>
  async getWalletBalances(address: string, chain: Chain): Promise<Balance[]>
  async getSwapRoute(params: SwapParams): Promise<SwapRoute>
  // Cada método: tenta tier1 → tier2 → tier3, com timeout 3s por tier
}
```

---

## FASE 2: NEPTUNE SWAP (O Motor de Taxa Principal)

**Objetivo:** Swap multi-chain funcionando com 0.5% de fee embutido.  
**Revenue estimado:** Para $1M em volume diário → $5.000/dia.

### 2.1 — Solana: Jupiter Terminal
```typescript
// Inicialização com fee wallet
window.Jupiter.init({
  displayMode: "integrated",
  integratedTargetId: "jupiter-terminal",
  endpoint: RPC_SOLANA,
  platformFeeAndAccounts: {
    feeBps: 50,  // 0.5%
    feeAccounts: new Map([
      // Para cada token, uma fee account associada à NEPTUNE_FEE_WALLET_SOL
    ])
  }
})
```
- Docs: https://terminal.jup.ag
- Fee: 0.5% de TODOS os swaps SOL automaticamente

### 2.2 — EVM: 1inch Fusion+ (Base / BNB / ETH / ARB)
```typescript
// 1inch Swap API v6.0
GET https://api.1inch.dev/swap/v6.0/{chainId}/swap?
  src={tokenIn}&dst={tokenOut}&amount={amount}
  &from={userAddress}&fee=50&referrer={NEPTUNE_FEE_WALLET_EVM}
```
- Chain IDs: Base(8453), BNB(56), ETH(1), ARB(42161), POLY(137)
- Fee: `&fee=50&referrer=` injeta 0.5% para nossa carteira EVM
- API Key: https://portal.1inch.dev (free tier disponível)

### 2.3 — Cross-chain: Li.Fi Widget (Bridge + Swap)
```typescript
import { LiFiWidget } from "@lifi/widget"

// Configuração com fee
<LiFiWidget
  config={{
    integrator: "neptune-crypto-saas",
    fee: 0.005,  // 0.5%
    feeConfig: {
      createAndPublishFeeQuote: true,
      baseFee: { amountUSD: 0, pctOfFromAmount: 0.005 }
    },
    walletManagement: { /* hooks wagmi */ }
  }}
/>
```
- Cobre: ETH ↔ SOL ↔ BNB ↔ Base ↔ ARB ↔ POLY ↔ AVAX
- Fee: 0.5% na bridge completa

### 2.4 — SUI: Aftermath Finance / Turbos Finance
- Aftermath SDK: `@aftermath-finance/sdk`
- `afSdk.Swap().getCompleteTradeRouteGivenAmountIn({ coinInType, coinOutType, coinInAmount, referral: NEPTUNE_FEE_WALLET_SUI })`
- Fee: referral fee configurada no Aftermath (0.5%)

### 2.5 — Interface Unificada de Swap
- Quando usuário está na chain X → mostra swap nativo da chain X
- Quando usuário quer trocar entre chains → exibe Li.Fi Bridge
- Auto-detect: se `tokenIn.chain !== tokenOut.chain` → ativa Li.Fi automaticamente
- Histórico de swaps: salvo no Firestore com TX hash

---

## FASE 3: NEPTUNE COMMAND (Hub de Bots)

**Objetivo:** 5 bots automáticos operando em múltiplas chains.  
**Arquitetura:** Backend Node.js/TypeScript + Supabase Edge Functions (ou Vercel Crons)

### 3.1 — Copy Trade Bot (Estrela Principal)
**Conceito:** Segue carteiras "baleia" detectadas pelo Whale Signals.  
**Integração cross-product:**
```typescript
// Consome a API do Whale Signals (catalani22/whale-signals-app)
GET {WHALE_SIGNALS_API_URL}/twitter-monitor/whale-wallets
// Retorna: [{wallet, chain, lastTx, volume24h, followersCount}]

// Quando whale compra X → Copy Trade Bot compra X também
// com slippage configurável e tamanho de posição em % da carteira do usuário
```
**Fluxo:**
1. Usuário configura: qual(is) baleia(s) seguir, % da carteira por trade, slippage máx, stop-loss
2. Bot monitora transações das baleias em tempo real (Helius webhooks para SOL, Moralis streams para EVM)
3. Quando baleia compra → bot replica proporcionalmente
4. Quando baleia vende → bot sai também (opcional: trailing stop)
5. Fee: 0.3% por execução bem-sucedida (só cobra quando lucra)

### 3.2 — Sniper Bot
**Conceito:** Detecta novo par criado e compra em milissegundos se passar no filtro.  
**Filtros de segurança (evita rug pulls):**
- Liquidez inicial > $10K
- Contrato verificado (quando aplicável)
- Creator wallet sem histórico de rug
- Sem mint authority ativa (Solana)
- Sem honeypot (EVM — checar via honeypot.is API)
**Fontes de detecção:**
- Solana: Helius webhooks (evento `SWAP` em Raydium/Orca)
- EVM: eventos `PairCreated(address,address,address)` via WebSocket RPC
- SUI: eventos on-chain via Mysten Labs RPC
**Fee:** 0.5% no trade de entrada

### 3.3 — DCA Bot (Dollar-Cost Averaging)
**Conceito:** Compra automática de X valor a cada Y horas.  
**Configuração do usuário:**
- Token alvo, chain, valor por ciclo, frequência (1h/6h/24h/7d)
- Limite máximo total (stop automático ao atingir)
- Preço máximo de entrada (não compra se acima de X)
**Execução:** Vercel Crons + assinatura via carteira delegada  
**Fee:** 0.2% por execução

### 3.4 — Grid Trading Bot
**Conceito:** Compra em quedas e vende em altas dentro de um range.  
**Configuração:**
- Range de preço (min / max), número de grades, capital total
- Auto-rebalance quando sai do range
**Chains suportadas:** Solana, Base, BNB (fase 3.4)  
**Fee:** 0.2% por ordem executada

### 3.5 — Polymarket Bot
**Conceito:** Apostas automáticas em mercados de predição.  
**API:** Polymarket CLOB API (`https://clob.polymarket.com`)  
**Estratégias:**
- Segue apostas grandes (>$10K) de wallets históricamente corretas
- Arb entre mercados correlacionados  
**Fee:** 0.5% do lucro líquido (só cobra quando vence)

### 3.6 — Backend dos Bots
```
Supabase (já usado no Whale Signals):
  → Table: bot_configs {user_wallet, bot_type, chain, params, status, created_at}
  → Table: bot_executions {config_id, tx_hash, amount_in, amount_out, fee_charged, status}
  → Edge Function: bot-executor (roda a cada cron, processa configs ativas)
  → Edge Function: bot-monitor (Helius webhook handler para eventos on-chain)
```

---

## FASE 4: NEPTUNE DEEP DEFI (Lending + Yield)

**Objetivo:** Exposição a DeFi sem complexidade. Widget com 1 clique.

### 4.1 — Lending (Empréstimos)
| Protocolo | Chain | Widget/SDK |
|-----------|-------|-----------|
| Aave v3 | Base, BNB, ETH, ARB, POLY | `@aave/contract-helpers` |
| Kamino Finance | Solana | `@kamino-finance/klend-sdk` |
| Marginfi | Solana | `@mrgnlabs/marginfi-client-v2` |
| Venus Protocol | BNB | SDK próprio |

**Interface:** 
- Mostra APY atual de cada protocolo por token
- Compara automaticamente: "Melhores taxas para USDC hoje"
- 1 clique para depositar e começar a render juros
**Fee:** 0.1% de taxa de conveniência na abertura da posição

### 4.2 — Yield Aggregator
| Protocolo | Chain | Estratégia |
|-----------|-------|-----------|
| Meteora Dynamic Pools | Solana | DLMM LP auto-rebalance |
| Beefy Finance | BNB, Base, ARB, POLY | Auto-compound LP |
| Convex Finance | ETH | Curve LP otimizado |
| Kamino Vaults | Solana | CLMM automático |

**Fee:** 10% de performance fee sobre os rendimentos (só cobra sobre o lucro)

### 4.3 — Staking Nativo
- SOL: Stake via Jupiter Stake (mSOL / jitoSOL) 
- BNB: Staking nativo BNB Chain
- ETH: stETH via Lido widget
**Fee:** 0.1% do valor em stake na abertura

---

## FASE 5: NEPTUNE DISCOVERY + NEPTUNE FORGE (Airdrop Pulse)

**Objetivo:** Integrar o Airdrop Pulse como módulos internos do NEP3.

### 5.1 — Integração como módulos internos
**Não redirect — abre dentro do NEP3:**
```tsx
// src/pages/Airdrop.tsx
export default function AirdropPage() {
  return (
    <div className="nep3-module-container">
      <iframe
        src="https://airdroppulse.app"
        className="w-full h-full border-none"
        title="Neptune Discovery — Powered by Airdrop Pulse"
      />
    </div>
  )
}

// Alternativa preferida: compartilhar componentes via npm package privado
// @neptune-crypto/airdrop-pulse-components
```

**Alternativa (melhor UX — fase 5.2):** Mover componentes do AP para um pacote compartilhado e importar nativamente no NEP3 (sem iframe, sem redirect).

### 5.2 — Badge na Home do NEP3
- Banner "🚀 Crie seu Token — Neptune Forge" com CTA → `/launch`
- Banner "🪂 Airdrops Verificados — Neptune Discovery" com CTA → `/airdrop`
- Widgets mini na dashboard mostrando últimos tokens lançados e airdrops ativos

---

## FASE 6: NEPTUNE PORTFOLIO (Dashboard Unificado)

**Objetivo:** Usuário vê TODAS as suas posições em TODAS as chains numa única tela.

### 6.1 — Dados Agregados Multi-Chain
```typescript
// Busca em paralelo todas as chains
const portfolio = await Promise.all([
  getBalances(address, 'solana'),    // Helius
  getBalances(address, 'bnb'),       // Moralis / Ankr
  getBalances(address, 'base'),      // Moralis / Alchemy
  getBalances(address, 'ethereum'),  // Moralis / Alchemy
  getBalances(address, 'sui'),       // Mysten RPC
])
```

### 6.2 — Posições DeFi
- Posições abertas no Aave, Kamino, Meteora (via protocolo APIs)
- P&L de cada posição
- APY atual vs APY na entrada

### 6.3 — Histórico de Transações
- Todas as TXs feitas pelo NEP3 (swaps, bots, deployments)
- Fees pagas (e recebidas pela Neptune — transparência)
- Exportar CSV para imposto de renda

---

## FASE 7: MONETIZAÇÃO E ANALYTICS

### 7.1 — Fee Treasury Dashboard (Admin)
- Ver fees recebidas por dia/semana/mês
- Por produto: Swap, Bridge, Bots, Forge, Discovery
- Por chain: SOL, BNB, Base, ETH, etc.
- Converter fees acumuladas: botão "Consolidar em USDC"

### 7.2 — Métricas de Crescimento
- DAU, MAU, Wallets únicas
- Volume total swapado
- Tokens lançados
- Airdrops distribuídos
- TVL em bots (assets sob gestão)

---

## CRONOGRAMA DE EXECUÇÃO

| Fase | Descrição | Depende de |
|------|-----------|-----------|
| **AP-Completo** | Completar Airdrop Pulse 100% funcional | — |
| **NEP3-F0** | Setup repo, estrutura, design system | AP-Completo |
| **NEP3-F1** | Shell: layout, navigation, wallet connect | NEP3-F0 |
| **NEP3-F2** | Neptune Swap (Jupiter + 1inch + Li.Fi + Aftermath) | NEP3-F1 |
| **NEP3-F3** | Neptune Command (Copy Trade + Sniper + DCA) | NEP3-F2 |
| **NEP3-F4** | Neptune Deep DeFi (Aave + Kamino + Meteora) | NEP3-F2 |
| **NEP3-F5** | Discovery + Forge integrados (Airdrop Pulse) | AP-Completo + NEP3-F1 |
| **NEP3-F6** | Portfolio unificado multi-chain | NEP3-F2 + NEP3-F4 |
| **NEP3-F7** | Admin, analytics, fee dashboard | NEP3-F6 |

---

## PRINCÍPIOS DE DESENVOLVIMENTO

1. **Real ou não faz**: Nada mockado. Nada de `Math.random()` para preços. API real ou não exibe.
2. **Cascade sempre**: Toda integração de API tem pelo menos 2 fallbacks.
3. **Fee em tudo**: Toda transação que passa pelo NEP3 deve ter o hook de fee injertado.
4. **Multi-chain por padrão**: Componentes pensados para N chains desde o início.
5. **Mobile first**: 60%+ dos usuários DeFi usam mobile.
6. **Zero dependência de assinatura**: Plataforma nunca bloqueia feature por plano — só cobra em uso.
7. **Segurança**: Chaves privadas NUNCA no frontend. Toda operação sensível via serverless functions.

---

## REFERÊNCIAS E APIs UTILIZADAS

### APIs Gratuitas (Free Tier)
| API | Uso | Limite Free |
|-----|-----|-------------|
| DexScreener | Preços de tokens | Sem key, generous |
| Jupiter | Swap SOL | Sem key para read |
| 1inch | Swap EVM | Key free no portal |
| Li.Fi | Bridge cross-chain | SDK free |
| Helius | SOL RPC + webhooks | 10K req/dia free |
| Moralis | EVM balances + events | 40K req/mês free |
| Ankr | Multi-chain RPC | 30 req/s free |
| Etherscan / BscScan | Gas tracker | 5 req/s free |
| CoinGecko | Preços, market data | 30 req/min free |
| BirdEye | Tokens Solana | 15K req/dia free |
| Aftermath Finance | SUI swap | SDK gratuito |

### APIs Pagas (Necessárias)
| API | Uso | Custo estimado |
|-----|-----|----------------|
| Helius Pro | Webhooks em tempo real para Sniper | ~$49/mês |
| Moralis Pro | Streams EVM para Sniper | ~$49/mês |
| Alchemy Growth | RPC ETH/Base com WebSocket | ~$49/mês |

*Todos os custos são cobertos com ~$300/dia de volume (60 BPS em fees).*

---

## COMO RETOMAR ESTE PROJETO

```bash
# 1. Clonar
gh repo clone catalani22/nep3
cd nep3

# 2. Ler este documento completo
cat docs/MASTER_PLAN.md

# 3. Verificar estado atual de todos os produtos
# Airdrop Pulse: https://github.com/catalani22/Airdrop-Pulse
# Whale Signals: https://github.com/catalani22/whale-signals-app

# 4. Verificar qual fase está em execução
# Consultar checkpoints em .copilot/session-state/

# 5. Continuar da fase seguinte
```

---

*Documento criado por Alexandre Catalani — Neptune Crypto SaaS*  
*Em colaboração com GitHub Copilot CLI — Abril 2026*
