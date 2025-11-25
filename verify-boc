#!/usr/bin/env ts-node

/**
 * Standalone BOC Transaction Verifier
 * 
 * Checks that transaction can be found by TON Connect BOC
 * Utilizes TEP-467 normalized hash
 * 
 * Usage:
 *   npx ts-node verify-boc.ts "your_boc_here"
 *   npx ts-node verify-boc.ts "your_boc" --network testnet --api-key "key"
 */

import { TonClient } from '@ton/ton';
import { Cell, loadMessage, Message, beginCell, storeMessage } from '@ton/core';

// ============= CONFIG =============

interface Config {
    boc: string;
    network: 'mainnet' | 'testnet';
    apiKey?: string;
    limit: number;
    verbose: boolean;
}

const ENDPOINTS = {
    mainnet: 'https://toncenter.com/api/v2/jsonRPC',
    testnet: 'https://testnet.toncenter.com/api/v2/jsonRPC',
};

// ============= TEP-467 =============

function getNormalizedExtMessageHash(message: Message): Buffer {
    if (message.info.type !== 'external-in') {
        throw new Error(`Message must be "external-in", got ${message.info.type}`);
    }

    const info = {
        ...message.info,
        src: undefined,
        importFee: BigInt(0),
    };

    const normalizedMessage = {
        ...message,
        init: null,
        info: info,
    } as Message;

    return beginCell()
        .store(storeMessage(normalizedMessage, { forceRef: true }))
        .endCell()
        .hash();
}

// ============= PARSER =============

function parseArgs(): Config {
    const args = process.argv.slice(2);
    
    if (args.length === 0 || args[0] === '--help' || args[0] === '-h') {
        console.log(`
🔍 BOC Transaction Verifier

Usage:
  npx ts-node verify-boc.ts <BOC> [options]

Arguments:
  <BOC>              Transaction BOC in base64 (required)

Options:
  --network <net>    Network: mainnet or testnet (default: mainnet)
  --api-key <key>    TONCenter API key (optional)
  --limit <num>      Number of transactions to check (default: 10)
  --verbose          Show detailed output
  -h, --help         Show this help

Examples:
  npx ts-node verify-boc.ts "te6cck..."
  npx ts-node verify-boc.ts "te6cck..." --network testnet --api-key "your_key"
  npx ts-node verify-boc.ts "te6cck..." --limit 100 --verbose
`);
        process.exit(0);
    }

    const config: Config = {
        boc: args[0],
        network: 'mainnet',
        limit: 10,
        verbose: false,
    };

    for (let i = 1; i < args.length; i++) {
        switch (args[i]) {
            case '--network':
                config.network = args[++i] as 'mainnet' | 'testnet';
                break;
            case '--api-key':
                config.apiKey = args[++i];
                break;
            case '--limit':
                config.limit = parseInt(args[++i], 10);
                break;
            case '--verbose':
                config.verbose = true;
                break;
        }
    }

    return config;
}

// ============= VERIFIER =============

async function verifyBoc(config: Config) {
    try {
        console.log('🔍 BOC Transaction Verifier\n');

        // Step 1: Parse BOC
        if (config.verbose) console.log('📦 Parsing BOC...');
        const cell = Cell.fromBase64(config.boc);
        const message = loadMessage(cell.beginParse());

        if (message.info.type !== 'external-in') {
            throw new Error(`Expected external-in message, got ${message.info.type}`);
        }

        const targetAddress = message.info.dest;
        console.log('✅ BOC parsed');
        console.log('   Address:', targetAddress.toString());

        // Step 2: Calculate normalized hash
        if (config.verbose) console.log('\n🔐 Calculating normalized hash (TEP-467)...');
        const normalizedHash = getNormalizedExtMessageHash(message);
        console.log('   Hash:', normalizedHash.toString('hex'));

        // Step 3: Connect to blockchain
        if (config.verbose) console.log('\n🌐 Connecting to blockchain...');
        const endpoint = ENDPOINTS[config.network];
        const client = new TonClient({
            endpoint,
            apiKey: config.apiKey,
        });
        console.log('   Network:', config.network);
        console.log('   Endpoint:', endpoint);
        if (config.apiKey) {
            console.log('   API Key: ***' + config.apiKey.slice(-4));
        } else {
            console.log('   API Key: none (public endpoint, rate limited)');
        }

        // Step 4: Fetch transactions
        if (config.verbose) console.log(`\n📡 Fetching last ${config.limit} transactions...`);
        const transactions = await client.getTransactions(targetAddress, {
            limit: config.limit,
            archival: true,
        });
        console.log(`   Found: ${transactions.length} transactions`);

        if (transactions.length === 0) {
            console.log('\n⚠️  No transactions found for this address');
            return false;
        }

        // Step 5: Search
        console.log('\n🔎 Searching for matching transaction...\n');
        
        for (let i = 0; i < transactions.length; i++) {
            const tx = transactions[i];
            
            if (config.verbose) {
                console.log(`[${i + 1}/${transactions.length}] LT: ${tx.lt.toString()}, Time: ${new Date(tx.now * 1000).toISOString()}`);
            }
            
            if (tx.inMessage?.info.type !== 'external-in') {
                if (config.verbose) console.log('      Skipped (not external-in)');
                continue;
            }

            const txHash = getNormalizedExtMessageHash(tx.inMessage);
            
            if (txHash.equals(normalizedHash)) {
                console.log('🎉 ✅ TRANSACTION FOUND!\n');
                console.log('=== Details ===');
                console.log('Position:', i + 1, 'of', transactions.length);
                console.log('LT (Logical Time):', tx.lt.toString());
                console.log('Cell Hash:', tx.hash().toString('hex'));
                console.log('Timestamp:', new Date(tx.now * 1000).toISOString());
                console.log('Destination:', tx.inMessage.info.dest.toString());
                
                if (tx.outMessages.size > 0) {
                    console.log('Out messages:', tx.outMessages.size);
                    const firstOut = Array.from(tx.outMessages.values())[0];
                    if (firstOut?.info.type === 'internal') {
                        console.log('First out dest:', firstOut.info.dest.toString());
                    }
                }
                
                console.log('\n✅ Transaction verified on blockchain!');
                return true;
            } else {
                if (config.verbose) console.log('      No match');
            }
        }

        console.log('❌ Transaction not found in the last', config.limit, 'transactions\n');
        console.log('💡 Try:');
        console.log('   - Wait 30 seconds and retry (transaction might not be confirmed yet)');
        console.log('   - Increase --limit to check more transactions');
        console.log('   - Check if BOC is from correct network (testnet vs mainnet)');
        
        return false;

    } catch (error) {
        console.error('\n❌ Error:', error instanceof Error ? error.message : String(error));
        if (config.verbose && error instanceof Error && error.stack) {
            console.error('\nStack trace:', error.stack);
        }
        return false;
    }
}

// ============= MAIN =============

async function main() {
    const config = parseArgs();
    const found = await verifyBoc(config);
    process.exit(found ? 0 : 1);
}

main();

