# 🔍 TON BOC Verifier

Verify TON transactions by BOC (Bag of Cells) via command line.

## 🚀 Quick Start

```bash
cd /path/to/standalone-boc-verifier
npm install
npx ts-node verify-boc.ts "your_boc_here"
```

## 📋 Commands

```bash
# Mainnet (public API)
npx ts-node verify-boc.ts "te6cck..."

# With API key
npx ts-node verify-boc.ts "te6cck..." --api-key "your_key"

# Testnet
npx ts-node verify-boc.ts "te6cck..." --network testnet --api-key "key"

# Check more transactions
npx ts-node verify-boc.ts "te6cck..." --limit 50
```

## 🎯 Parameters

| Flag        | Description           | Default   |
| ----------- | --------------------- | --------- |
| `--network` | `mainnet` / `testnet` | `mainnet` |
| `--api-key` | TONCenter API key     | -         |
| `--limit`   | Transaction count     | `10`      |

## 📊 Output

```
✅ TRANSACTION FOUND!

Position: 2 of 10
LT: 62414865000001
Timestamp: 2025-10-10T11:27:28.000Z
Hash: 66e36b29...47dc9374

✅ Verified on blockchain!
```

## 💡 Tips

-   **Not found?** Increase `--limit` or wait 30 seconds
-   **API key:** Free at [tonconsole.com](https://tonconsole.com)
-   **TEP-467:** [Normalized Hash](https://github.com/ton-blockchain/TEPs/blob/master/text/0467-normalized-message-hash.md)
