# AIRDROP PULSE — ROADMAP PARA 100% FUNCIONAL

> Este documento lista tudo que precisa ser implementado no Airdrop Pulse  
> antes de integrar ao NEP3. Nada mockado. Nada de template. Tudo real.

## Estado Atual (Stage 2 Completo)

✅ Landing page  
✅ LaunchWizard 4 steps (token name, supply, chain, deploy)  
✅ Token deploy: Solana bonding (pump.fun), Solana plain (Token-2022), Base/BNB (Clanker + contrato), SUI stub  
✅ Motor financeiro: 0.5% plataforma + 1% criador  
✅ Ping-pong wallets A/B/C com SHA-256  
✅ Firebase Auth + Firestore  
✅ Creator dashboard (ver tokens criados, fees acumuladas)  
✅ Airdrop dashboard (listar airdrops, status)  
✅ Brand protection (palavras reservadas)  
✅ Rate limit (3 launches/24h/wallet)  
✅ Security audit (3 agentes — Stage 2-B)  
✅ Cron sweep-dormant-fees  
✅ Risk flagging (>15% creator reserve → red flag público)  

---

## O QUE FALTA — POR PRIORIDADE

### AP-1: GasSniper — Dados Reais
**Prioridade:** ALTA (feature visível ao usuário)  
**Status atual:** Mock com dados fake  

**Implementação:**
```typescript
// api/gas-tracker.ts — novo endpoint
// Cascade: Etherscan → BscScan → eth_gasPrice RPC
async function getGasPrice(chain: string) {
  if (chain === 'ethereum' || chain === 'base') {
    try {
      const r = await fetch(`https://api.etherscan.io/api?module=gastracker&action=gasoracle&apikey=${ETHERSCAN_KEY}`)
      return r.json()
    } catch {}
  }
  if (chain === 'bnb') {
    try {
      const r = await fetch(`https://api.bscscan.com/api?module=gastracker&action=gasoracle&apikey=${BSCSCAN_KEY}`)
      return r.json()
    } catch {}
  }
  // Fallback: eth_gasPrice direto no RPC
  const provider = new ethers.JsonRpcProvider(getRPC(chain))
  return provider.getFeeData()
}
// Solana:
const fees = await connection.getRecentPrioritizationFees()
```
**UI:** Atualizar componente GasSniper para chamar `/api/gas-tracker` a cada 30s com polling.

---

### AP-2: WhaleWatcher — Transações Reais
**Prioridade:** ALTA (feature visível ao usuário)  
**Status atual:** Mock/placeholder  

**Implementação:**
```typescript
// api/whale-watcher.ts — novo endpoint
// Solana (Helius):
GET https://api.helius.xyz/v0/addresses/{address}/transactions?api-key={HELIUS_KEY}&limit=20

// EVM (Moralis):
GET https://deep-index.moralis.io/api/v2.2/{address}/erc20/transfers?chain={chain}
Headers: { X-API-Key: MORALIS_KEY }

// Critério "baleia": carteiras com > $100K em ativos ou volume > $50K/dia
// Fonte de carteiras a monitorar: compartilhar base do Whale Signals
```
**UI:** Feed em tempo real de transações grandes, com alertas visuais.

---

### AP-3: One-Click Strategy (Li.Fi Swap Widget)
**Prioridade:** ALTA (gera revenue para o NEP3 também)  
**Status atual:** Não implementado  

**Implementação:**
```tsx
// src/components/SwapWidget.tsx — novo componente
import { LiFiWidget } from "@lifi/widget"

export function SwapWidget({ defaultToken }: { defaultToken?: string }) {
  return (
    <LiFiWidget
      config={{
        integrator: "airdrop-pulse",
        fee: 0.005,  // 0.5%
        toChain: "sol",
        toToken: defaultToken,
        theme: { /* AP colors */}
      }}
    />
  )
}
```
**Uso:** Botão "Comprar via AP" em cada token/airdrop card  
**Nota:** Este componente será reusado no NEP3 — construir com isso em mente.

---

### AP-4: Admin Panel
**Prioridade:** MÉDIA (necessário para gestão de conteúdo)  
**Status atual:** Não existe  

**Implementação:**
```
Rota: /admin (protegida por Firebase Auth — email: airdroppulse.app@gmail.com)

Seções:
├── Tokens: listar todos, aprovar/rejeitar, editar status
├── Airdrops: adicionar novos airdrops manualmente no Firestore
├── Flags: ver tokens com creator reserve > 15%, histórico de rug
├── Wallets: saldo atual das platform wallets A/B/C em cada chain
└── Analytics: tokens por dia, volume total, fees coletadas
```

---

### AP-5: SUI — Implementação Real
**Prioridade:** MÉDIA  
**Status atual:** Stub (marcado como Stage 3 no MASTER_GUIDE)  

**Implementação:**
```typescript
// src/lib/launchToken.ts — substituir stub SUI
import { SuiClient } from "@mysten/sui"
import { Transaction } from "@mysten/sui/transactions"

// Criar token fungível SUI
const tx = new Transaction()
tx.moveCall({
  target: "0x2::coin::create_currency",
  typeArguments: ["0x2::coin::TreasuryCap<T>"],
  arguments: [/* metadata */]
})
const result = await suiClient.signAndExecuteTransaction({ transaction: tx, signer: wallet })
```

---

### AP-6: Airdrop Claim Real (Para Tokens do AP)
**Prioridade:** BAIXA (mas gera 2% fee no NEP3)  
**Status atual:** Links externos para sites de claim  

**Implementação:**
```typescript
// api/airdrop-claim.ts
// Para tokens criados DENTRO do AP:
// 1. Verificar endereço no MerkleTree do contrato
// 2. Calcular valor a receber
// 3. Mostrar botão "Resgatar via Neptune (2% fee)"
// 4. Executar: transfere 98% para usuário, 2% para fee wallet
```
**Contratos necessários:**  
- `MerkleDistributor.sol` para EVM  
- Programa Anchor para Solana

---

### AP-7: Gamificação / XP (MVP)
**Prioridade:** BAIXA (engajamento)  
**Status atual:** Schema parcial no Firestore  

**Implementação MVP:**
```typescript
// XP Events:
// +100 XP: lançar token
// +50 XP: distribuir airdrop
// +10 XP: usar GasSniper
// +5 XP: fazer login diário
// +200 XP: primeiro token com > 100 holders

// Badges:
// "Token Creator" → primeiro token lançado
// "Whale Tracker" → usar WhaleWatcher por 7 dias
// "Airdrop Master" → distribuir para > 100 wallets

// Leaderboard: top 10 criadores da semana por volume
```

---

## ORDEM DE EXECUÇÃO RECOMENDADA

```
1. AP-1: GasSniper real          (2-3h de trabalho)
2. AP-2: WhaleWatcher real       (3-4h de trabalho)
3. AP-3: Li.Fi Swap Widget       (2-3h de trabalho)
4. AP-4: Admin Panel             (4-5h de trabalho)
5. AP-6: SUI real                (3-4h de trabalho)
6. AP-5: Airdrop Claim           (6-8h de trabalho — requer contratos)
7. AP-7: Gamificação MVP         (2-3h de trabalho)
```

**Após completar AP-1 a AP-4 = Airdrop Pulse está 80% completo e pronto para integrar no NEP3.**  
**AP-5, AP-6, AP-7 podem ser finalizados em paralelo com o desenvolvimento do NEP3.**
