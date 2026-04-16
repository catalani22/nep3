# NEP3 — MODELO DE RECEITA

## Princípio

> "O usuário paga conforme ganha. A Neptune fisga uma fração de cada onda."

100% free para usar. Zero assinatura. Zero KYC.  
Revenue = **volume transacionado × fee**.

## Tabela Completa de Fees

| Serviço | Fee | Coleta | Estimativa ($1M/dia volume) |
|---------|-----|--------|-----------------------------|
| Swap nativo (por chain) | 0.5% | Automático no widget | $5.000/dia |
| Bridge cross-chain | 0.5% | Li.Fi fee config | Incluso acima |
| Copy Trade Bot | 0.3% por execução | Serverless deduct | $1.500/dia |
| Sniper Bot | 0.5% na entrada | Serverless deduct | $2.500/dia |
| DCA Bot | 0.2% por ordem | Serverless deduct | $1.000/dia |
| Grid Bot | 0.2% por ordem executada | Serverless deduct | $500/dia |
| Polymarket Bot | 0.5% do lucro | Serverless deduct | $250/dia |
| Token Launch (Forge) | 0.5% perpétuo no volume | Contrato/SDK | Composto |
| Airdrop Claim | 2% success fee | Serverless deduct | $1.000/dia |
| Lending opening | 0.1% do valor | Serverless deduct | $200/dia |
| Yield entry | 10% do rendimento | Serverless deduct | $100/dia |

## Como as Fees São Coletadas (por serviço)

### Swap — Jupiter (Solana)
```typescript
platformFeeAndAccounts: {
  feeBps: 50,
  feeAccounts: feeAccountsMap  // fee account derivada da NEPTUNE_FEE_WALLET_SOL
}
// Jupiter envia 0.5% automaticamente para nossa fee account
```

### Swap — 1inch (EVM)
```
GET /swap/v6.0/{chainId}/swap?...&fee=50&referrer={NEPTUNE_FEE_WALLET_EVM}
// 1inch envia 0.5% para nossa wallet EVM automaticamente
```

### Bridge — Li.Fi
```typescript
config: { fee: 0.005 }  // 0.5% deduzido do valor bridgeado
```

### Bots (Copy Trade, Sniper, DCA, Grid)
```typescript
// Serverless function calcula fee ANTES de executar a ordem
const feeAmount = tradeAmount * 0.003  // 0.3%
// Transfere feeAmount para NEPTUNE_FEE_WALLET antes de submeter TX
```

### Token Launch (Forge) — Perpétuo
```
Solana bonding: pump.fun creator fee → nossa wallet
Solana plain: Token-2022 TransferFeeConfig 50bps hardcoded → imutável
EVM bonding: Clanker rewards.recipients 50bps → nossa wallet EVM
EVM plain: AirdropPulseToken.sol bytecode 50bps → imutável
SUI: Move module com transfer fee hardcoded
```

### Airdrop Claim
```typescript
// Quando usuário clica "Resgatar via Neptune"
const claimValue = getClaimableAmount(wallet, merkleRoot)
const fee = claimValue * 0.02  // 2%
const userReceives = claimValue - fee
// Executa claim no contrato, split automático
```

## Treasury Multi-Chain

Todas as fees são recebidas em:
- `NEPTUNE_FEE_WALLET_SOL` — acumula SOL + SPL tokens
- `NEPTUNE_FEE_WALLET_EVM` — acumula ETH/BNB/BASE tokens + USDC/USDT
- `NEPTUNE_FEE_WALLET_SUI` — acumula SUI tokens

**Consolidação mensal:** Converter tudo para USDC/USDT via swap interno  
(usando o próprio Neptune Swap — sem pagar fee para si mesmo, claro).

## Projeção de Receita

| Volume Diário | Fee Diária | Fee Mensal | Fee Anual |
|--------------|------------|------------|-----------|
| $100K | $500 | $15K | $180K |
| $1M | $5.000 | $150K | $1.8M |
| $10M | $50.000 | $1.5M | $18M |
| $100M | $500.000 | $15M | $180M |

*Considerando fee média ponderada de 0.5% em todos os produtos.*

## Comparação Competitiva de Modelo

| Plataforma | Modelo | Fee |
|-----------|--------|-----|
| Uniswap | Fee no swap | 0.05–1% (vai para LPs) |
| Jupiter | Fee configurável | 0% deles, integrador configura |
| 1inch | Referral program | 0.5–1% (dev mode) |
| **NEP3** | **0.5% em TUDO** | **100% para Neptune** |
| Binance | Taxa de trading | 0.1% (+ desconto BNB) |

*A vantagem do NEP3: diversificação. Revenue de 5 fontes diferentes simultaneamente.*
