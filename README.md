import React, { useMemo, useCallback } from 'react';
import './App.css';
import { WalletMultiButton } from '@solana/wallet-adapter-react-ui';
import { ConnectionProvider, WalletProvider, useWallet } from '@solana/wallet-adapter-react';
import { PhantomWalletAdapter } from '@solana/wallet-adapter-wallets';
import { Connection, Transaction, SystemProgram, PublicKey } from '@solana/web3.js';

require('@solana/wallet-adapter-react-ui/styles.css');

const network = 'https://api.devnet.solana.com'; // Devnet for testing
const recipientAddress = '5CEVrxuWiEJZK3BXtvw6nh5iiBZJPeNiBCyfFnqMFTQy'; // Your Solana address for receiving bets

function Home() {
    const { publicKey, sendTransaction } = useWallet();
    
    const placeBet = useCallback(async () => {
        if (!publicKey) {
            alert('Please connect Phantom Wallet first.');
            return;
        }

        const connection = new Connection(network, 'confirmed');
        const recipient = new PublicKey(recipientAddress);

        const transaction = new Transaction().add(
            SystemProgram.transfer({
                fromPubkey: publicKey,
                toPubkey: recipient,
                lamports: 1000000, // 0.001 SOL
            })
        );

        try {
            const signature = await sendTransaction(transaction, connection);
            await connection.confirmTransaction(signature, 'processed');
            alert('✅ Bet placed successfully! Transaction: ' + signature);
        } catch (error) {
            console.error('❌ Error placing bet:', error);
            alert('Error placing bet. Please try again.');
        }
    }, [publicKey, sendTransaction]);

    return (
        <div className="min-h-screen bg-gray-900 text-white flex flex-col items-center justify-center p-6">
            <h1 className="text-5xl font-bold mb-4">🎯 Welcome to SolBet</h1>
            <p className="text-lg mb-6 text-center max-w-xl">
                We all love sports betting — now with <strong>SolBet</strong>, you can place your bets using <strong>Solana</strong>!
            </p>
            <WalletMultiButton />
            <button
                className="mt-6 px-6 py-3 bg-blue-600 hover:bg-blue-700 rounded-xl text-lg font-semibold"
                onClick={placeBet}
                disabled={!publicKey}
            >
                Place a Bet (0.001 SOL)
            </button>
        </div>
    );
}

function App() {
    const wallets = useMemo(() => [new PhantomWalletAdapter()], []);

    return (
        <ConnectionProvider endpoint={network}>
            <WalletProvider wallets={wallets} autoConnect>
                <Home />
            </WalletProvider>
        </ConnectionProvider>
    );
}

export default App;
