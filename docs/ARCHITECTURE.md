# NEP3 — ARQUITETURA TÉCNICA

## Stack

| Camada | Tecnologia | Justificativa |
|--------|-----------|---------------|
| Frontend | React 18 + TypeScript + Vite | Mesma stack do Airdrop Pulse — reutilizar conhecimento |
| Estilização | Tailwind v4 (inline hsl) | Mesma convenção do AP — SEM classes de cor |
| Web3 EVM | wagmi v2 + RainbowKit v2 + viem v2 | Padrão da indústria, funciona no AP |
| Web3 Solana | @solana/wallet-adapter-react + @solana/web3.js | Padrão Solana |
| Web3 SUI | @mysten/dapp-kit + @mysten/sui | Oficial SUI Foundation |
| Backend | Vercel Serverless Functions (TypeScript) | Sem servidor, escala automático |
| Database | Supabase (PostgreSQL) — compartilhado com Whale Signals | Já configurado, Edge Functions |
| Auth | Firebase Auth (compartilhado com AP) OU Privy (Web3 native) | TBD |
| Deploy | Vercel (main → produção automática) | Mesmo fluxo do AP |
| Crons | Vercel Crons + Supabase Crons | Bots precisam de crons confiáveis |

## Padrão de Cascata (Hydra Architecture)

Todo acesso externo segue o padrão:

```typescript
class ApiOrchestrator {
  async call<T>(
    providers: Provider[],
    params: unknown,
    timeout = 3000
  ): Promise<T> {
    for (const provider of providers) {
      try {
        const result = await Promise.race([
          provider.fetch(params),
          sleep(timeout).then(() => { throw new Error('timeout') })
        ])
        return result as T
      } catch {
        continue // próximo provider
      }
    }
    throw new Error('All providers failed')
  }
}
```

## Chains Suportadas

### Tier 1 (Launch)
| Chain | ChainID | RPC Primário | Explorer |
|-------|---------|-------------|---------|
| Solana | mainnet-beta | Ankr | solscan.io |
| BNB Chain | 56 | Ankr | bscscan.com |
| Base | 8453 | Ankr | basescan.org |
| SUI | mainnet | Mysten | suiexplorer.com |

### Tier 2 (Fase 2)
| Chain | ChainID | RPC Primário | Explorer |
|-------|---------|-------------|---------|
| Ethereum | 1 | Ankr / Alchemy | etherscan.io |
| Arbitrum One | 42161 | Ankr | arbiscan.io |
| Polygon | 137 | Ankr | polygonscan.com |
| Avalanche C | 43114 | Ankr | snowtrace.io |
| TON | mainnet | TON Center | tonscan.org |

## Fee Injection Pattern

Todo widget/SDK que passa dinheiro DEVE ter o fee hook:

```typescript
// lib/fee-injector.ts
export const NEPTUNE_FEES = {
  swap: 50,       // 50 bps = 0.5%
  bridge: 50,
  bot: 30,        // 30 bps = 0.3%
  claim: 200,     // 200 bps = 2%
  lending: 10,    // 10 bps = 0.1%
} as const

export function getFeeWallet(chain: Chain): string {
  const wallets = {
    solana: process.env.NEPTUNE_FEE_WALLET_SOL!,
    bnb: process.env.NEPTUNE_FEE_WALLET_EVM!,
    base: process.env.NEPTUNE_FEE_WALLET_EVM!,
    ethereum: process.env.NEPTUNE_FEE_WALLET_EVM!,
    arbitrum: process.env.NEPTUNE_FEE_WALLET_EVM!,
    sui: process.env.NEPTUNE_FEE_WALLET_SUI!,
  }
  return wallets[chain]
}
```

## Segurança

1. **Chaves privadas**: JAMAIS no frontend. Apenas variáveis de ambiente Vercel.
2. **Carteiras de fee**: Apenas endereços públicos no código. Chaves privadas fora do repo.
3. **Bot execution**: Cada usuário assina a transação com sua própria carteira — Neptune NUNCA custodia.
4. **Bot configuration**: Serverless function valida params antes de executar.
5. **Rate limiting**: Por IP + por wallet para todos os endpoints de execução.
