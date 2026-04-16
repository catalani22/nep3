# NEP3 — INTEGRAÇÃO DO AIRDROP PULSE

## Conceito

O Airdrop Pulse **não é substituído** pelo NEP3.  
Ele é **absorvido como módulo interno**.

```
airdroppulse.app → continua existindo (produto standalone)
nep3.app/airdrop  → abre Airdrop Pulse internamente (Discovery)
nep3.app/launch   → abre Airdrop Pulse Launch internamente (Forge)
```

## Estratégia de Integração (3 opções em ordem de preferência)

### Opção A: Componentes Compartilhados (Melhor UX)
Criar um pacote npm privado `@neptune-crypto/shared-components` com os componentes do AP.  
Importar no NEP3 nativamente — sem iframe, sem redirect, experiência totalmente fluida.

**Vantagem:** UX perfeita, tema NEP3 aplicado nativamente  
**Desvantagem:** Esforço de refatoração maior

### Opção B: Microfrontend com iframe (Rápido de implementar)
```tsx
// src/pages/Airdrop.tsx no NEP3
export default function AirdropPage() {
  return (
    <iframe
      src="https://airdroppulse.app"
      className="w-full h-screen border-none"
      allow="clipboard-write; wallet; ethereum"
    />
  )
}
```
**Vantagem:** Implementação em minutos  
**Desvantagem:** Dois contextos de wallet, UX não totalmente integrada

### Opção C: Redirect Suave (MVP temporário)
```tsx
// Redireciona mas mantém a URL do NEP3 na barra de endereço
// usando history.replaceState + window.open com target="_self"
// Não recomendado para versão final
```

**Recomendação:** Começar com **Opção B** (iframe) para validar a integração,  
depois migrar para **Opção A** conforme o NEP3 evolui.

## Menu do NEP3 — Estrutura Final

```
Sidebar Navigation:
├── 🏠 Dashboard     → /
├── ⚡ Swap          → /swap
├── 🌉 Bridge        → /bridge
├── 🤖 Bots          → /bots
│   ├── Copy Trade   → /bots/copy-trade
│   ├── Sniper       → /bots/sniper
│   ├── DCA          → /bots/dca
│   ├── Grid         → /bots/grid
│   └── Polymarket   → /bots/polymarket
├── 🌊 DeFi          → /defi
│   ├── Lending      → /defi/lending
│   ├── Yield        → /defi/yield
│   └── Staking      → /defi/staking
├── 🪂 Airdrop       → /airdrop      ← AIRDROP PULSE DISCOVERY
├── 🚀 Launch        → /launch       ← AIRDROP PULSE FORGE
└── 📊 Portfolio     → /portfolio
```

## Badges na Home do NEP3

```tsx
// Dashboard.tsx — seção de destaque para os módulos do AP
<div className="featured-modules">
  <ModuleBanner
    icon="🚀"
    title="Neptune Forge"
    subtitle="Crie seu token multi-chain em 60 segundos"
    cta="Criar Token"
    href="/launch"
    badge="Powered by Airdrop Pulse"
  />
  <ModuleBanner
    icon="🪂"
    title="Neptune Discovery"
    subtitle="Airdrops verificados por IA — resgate com 1 clique"
    cta="Ver Airdrops"
    href="/airdrop"
    badge="Powered by Airdrop Pulse"
  />
</div>
```

## Whale Signals no NEP3

O Whale Signals alimenta o **Copy Trade Bot** do Neptune Command:

```typescript
// Fluxo de dados cross-product
Whale Signals (catalani22/whale-signals-app)
  → Supabase: twitter_whale_alerts table
  → NEP3 Copy Trade Bot: lê alertas de compra de baleias
  → Replica proporcionalmeante na carteira do usuário
```

**API endpoint do Whale Signals para uso interno:**
```
GET {WHALE_SIGNALS_API_URL}/whale-data/recent-buys
Authorization: Bearer {WHALE_SIGNALS_API_KEY}
```

Esta integração é o **diferencial competitivo** do NEP3:  
nenhuma outra plataforma DeFi tem acesso aos dados de baleias que o Whale Signals já coleta.

## Cronograma de Integração

| Etapa | O que fazer |
|-------|------------|
| 1 | Completar Airdrop Pulse 100% funcional (sem mocks) |
| 2 | Criar NEP3 Phase 0 + Phase 1 (shell + navegação) |
| 3 | Integrar AP via iframe em /airdrop e /launch |
| 4 | Testar wallet context entre NEP3 e iframe AP |
| 5 | Adicionar badges na home do NEP3 |
| 6 | Migrar componentes AP para pacote compartilhado (longo prazo) |
