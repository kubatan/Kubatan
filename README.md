import React, { useMemo, useCallback, useState } from 'react';
import './App.css';
import { WalletMultiButton } from '@solana/wallet-adapter-react-ui';
import { ConnectionProvider, WalletProvider, useWallet } from '@solana/wallet-adapter-react';
import { PhantomWalletAdapter } from '@solana/wallet-adapter-wallets';
import { Connection, Transaction, SystemProgram, PublicKey } from '@solana/web3.js';

require('@solana/wallet-adapter-react-ui/styles.css');

const network = 'https://api.devnet.solana.com'; // Devnet for testing
const recipientAddress = '5CEVrxuWiEJZK3BXtvw6nh5iiBZJPeNiBCyfFnqMFTQy'; // Your Solana address for receiving bets

const sports = [
    {
        name: 'Football',
        events: ['Match 1', 'Match 2', 'Match 3'],
        betOptions: ['Win', 'Lose', 'Over 2.5 Goals', 'Under 2.5 Goals'],
    },
    {
        name: 'Basketball',
        events: ['Game A', 'Game B', 'Game C'],
        betOptions: ['Win', 'Lose', 'Over 150 Points', 'Under 150 Points'],
    },
    {
        name: 'Tennis',
        events: ['Tournament 1', 'Tournament 2', 'Tournament 3'],
        betOptions: ['Win', 'Lose'],
    },
];

function Home() {
    const { publicKey, sendTransaction } = useWallet();
    const [betAmount, setBetAmount] = useState(0.01); // Default to 0.01 SOL
    const [selectedSport, setSelectedSport] = useState(sports[0].name);
    const [selectedEvent, setSelectedEvent] = useState(sports[0].events[0]);
    const [selectedBetOption, setSelectedBetOption] = useState(sports[0].betOptions[0]);
    const [errorMessage, setErrorMessage] = useState('');

    const handleBetAmountChange = (e) => {
        const value = parseFloat(e.target.value);
        if (value < 0.01 || value > 10) {
            setErrorMessage('Bet amount must be between 0.01 SOL and 10 SOL');
        } else {
            setErrorMessage('');
        }
        setBetAmount(value);
    };

    const handleSportChange = (e) => {
        const sport = e.target.value;
        setSelectedSport(sport);
        setSelectedEvent(sports.find(s => s.name === sport).events[0]); // Reset to first event of selected sport
        setSelectedBetOption(sports.find(s => s.name === sport).betOptions[0]); // Reset to first bet option
    };

    const handleEventChange = (e) => {
        setSelectedEvent(e.target.value);
    };

    const handleBetOptionChange = (e) => {
        setSelectedBetOption(e.target.value);
    };

    const placeBet = useCallback(async () => {
        if (!publicKey) {
            alert('Please connect Phantom Wallet first.');
            return;
        }

        // Validate bet amount
        if (betAmount < 0.01 || betAmount > 10) {
            alert('Bet amount must be between 0.01 SOL and 10 SOL');
            return;
        }

        const connection = new Connection(network, 'confirmed');
        const recipient = new PublicKey(recipientAddress);

        // Convert SOL to lamports
        const lamports = betAmount * 1000000000; // 1 SOL = 1 billion lamports

        const transaction = new Transaction().add(
            SystemProgram.transfer({
                fromPubkey: publicKey,
                toPubkey: recipient,
                lamports: lamports,
            })
        );

        try {
            const signature = await sendTransaction(transaction, connection);
            await connection.confirmTransaction(signature, 'processed');
            alert(`✅ Bet placed successfully on ${selectedSport} event: ${selectedEvent} with bet: ${selectedBetOption}! Transaction: ${signature}`);
        } catch (error) {
            console.error('❌ Error placing bet:', error);
            alert('Error placing bet. Please try again.');
        }
    }, [publicKey, sendTransaction, betAmount, selectedSport, selectedEvent, selectedBetOption]);

    return (
        <div className="min-h-screen bg-gradient-to-r from-blue-700 to-purple-800 text-white flex flex-col items-center justify-center p-6">
            <h1 className="text-5xl font-bold mb-4 text-center">🎯 Welcome to SolBet</h1>
            <p className="text-lg mb-6 text-center max-w-xl">
                We all love sports betting — now with <strong>SolBet</strong>, you can place your bets using <strong>Solana</strong>!
            </p>
            <WalletMultiButton className="mb-6" />
            
            {/* Sport Selection */}
            <div className="mb-6">
                <label htmlFor="sport" className="block text-lg font-semibold mb-2">Choose Sport:</label>
                <select
                    id="sport"
                    value={selectedSport}
                    onChange={handleSportChange}
                    className="p-3 rounded-lg bg-gray-900 text-white text-xl"
                >
                    {sports.map((sport) => (
                        <option key={sport.name} value={sport.name}>{sport.name}</option>
                    ))}
                </select>
            </div>

            {/* Event Selection */}
            <div className="mb-6">
                <label htmlFor="event" className="block text-lg font-semibold mb-2">Choose Event:</label>
                <select
                    id="event"
                    value={selectedEvent}
                    onChange={handleEventChange}
                    className="p-3 rounded-lg bg-gray-900 text-white text-xl"
                >
                    {sports.find(s => s.name === selectedSport)?.events.map((event) => (
                        <option key={event} value={event}>{event}</option>
                    ))}
                </select>
            </div>

            {/* Bet Option Selection */}
            <div className="mb-6">
                <label htmlFor="betOption" className="block text-lg font-semibold mb-2">Choose Bet Type:</label>
                <select
                    id="betOption"
                    value={selectedBetOption}
                    onChange={handleBetOptionChange}
                    className="p-3 rounded-lg bg-gray-900 text-white text-xl"
                >
                    {sports.find(s => s.name === selectedSport)?.betOptions.map((option) => (
                        <option key={option} value={option}>{option}</option>
                    ))}
                </select>
            </div>

            {/* Bet Amount */}
            <div className="mb-6">
                <label htmlFor="betAmount" className="block text-lg font-semibold mb-2">Enter Bet Amount (SOL):</label>
                <input
                    id="betAmount"
                    type="number"
                    step="0.01"
                    value={betAmount}
                    onChange={handleBetAmountChange}
                    min="0.01"
                    max="10"
                    className="p-3 rounded-lg bg-gray-900 text-white text-xl"
                />
                {errorMessage && (
                    <p className="text-red-500 mt-2">{errorMessage}</p>
                )}
            </div>

            <button
                className="mt-6 px-6 py-3 bg-blue-600 hover:bg-blue-700 rounded-xl text-lg font-semibold shadow-lg"
                onClick={placeBet}
                disabled={!publicKey || betAmount < 0.01 || betAmount > 10}
            >
                Place a Bet ({betAmount} SOL)
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
