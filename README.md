import React, { useCallback, useMemo } from 'react';
import './App.css';
import { Connection, PublicKey, Transaction, SystemProgram } from '@solana/web3.js';
import {
    WalletAdapterNetwork,
} from '@solana/wallet-adapter-base';
import {
    WalletModalProvider,
    WalletMultiButton
} from '@solana/wallet-adapter-react-ui';
import {
    ConnectionProvider,
    WalletProvider,
    useWallet
} from '@solana/wallet-adapter-react';
import {
    PhantomWalletAdapter
} from '@solana/wallet-adapter-wallets';

require('@solana/wallet-adapter-react-ui/styles.css');

const network = WalletAdapterNetwork.Devnet;
const endpoint = 'https://api.devnet.solana.com';

const recipientAddress = '5CEVrxuWiEJZK3BXtvw6nh5iiBZJPeNiBCyfFnqMFTQy'; // Jouw wallet

function AppContent() {
    const { publicKey, sendTransaction } = useWallet();

    const placeBet = useCallback(async () => {
        if (!publicKey) {
            alert('Connect Phantom Wallet first');
            return;
        }

        const connection = new Connection(endpoint, 'confirmed');
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
            alert('✅ Inzet geplaatst! Transactie: ' + signature);
        } catch (error) {
            console.error('❌ Fout bij inzetten:', error);
        }
    }, [publicKey, sendTransaction]);

    return (
        <div className="App">
            <h1>⚽ Solana Sportweddenschap</h1>
            <WalletMultiButton />
            <br /><br />
            <button onClick={placeBet} disabled={!publicKey}>
                Plaats inzet (0.001 SOL)
            </button>
        </div>
    );
}

function App() {
    const wallets = useMemo(() => [new PhantomWalletAdapter()], []);

    return (
        <ConnectionProvider endpoint={endpoint}>
            <WalletProvider wallets={wallets} autoConnect>
                <WalletModalProvider>
                    <AppContent />
                </WalletModalProvider>
            </WalletProvider>
        </ConnectionProvider>
    );
}

export default App;
