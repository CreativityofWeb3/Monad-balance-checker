# Monad-balance-checker
Check your $MON balances 
import React, { useState } from 'react';
import { Search, Wallet, Copy, Loader2, TrendingUp, AlertCircle } from 'lucide-react';

const MonadPortfolioTracker = () => {
  const [address, setAddress] = useState('');
  const [portfolio, setPortfolio] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  // Monad Testnet RPC endpoint
  const MONAD_RPC = 'https://testnet1.monad.xyz';

  const KNOWN_TOKENS = {
    '0x0000000000000000000000000000000000000000': {
      symbol: 'MON',
      name: 'Monad',
      decimals: 18,
      isNative: true,
    },
    // Add more tokens later if you want
  };

  const fetchNativeBalance = async (walletAddress) => {
    try {
      const response = await fetch(MONAD_RPC, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          jsonrpc: '2.0',
          method: 'eth_getBalance',
          params: [walletAddress, 'latest'],
          id: 1,
        }),
      });

      const data = await response.json();
      if (data.error) throw new Error(data.error.message);

      const balanceWei = BigInt(data.result);
      const balance = Number(balanceWei) / 1e18;

      return {
        address: '0x0000000000000000000000000000000000000000',
        symbol: 'MON',
        name: 'Monad',
        balance: balance.toFixed(6),
        decimals: 18,
        isNative: true,
      };
    } catch (err) {
      console.error('Failed to fetch native balance:', err);
      return null;
    }
  };

  const fetchPortfolio = async (walletAddress) => {
    setLoading(true);
    setError('');
    try {
      const tokens = [];

      const nativeBalance = await fetchNativeBalance(walletAddress);
      if (nativeBalance) tokens.push(nativeBalance);

      setPortfolio({
        address: walletAddress,
        tokens: tokens.filter((t) => parseFloat(t.balance) > 0),
        lastUpdated: new Date().toLocaleTimeString(),
      });
    } catch (err) {
      console.error(err);
      setError("Couldn't fetch balances. Check the address or RPC.");
    } finally {
      setLoading(false);
    }
  };

  const handleCheck = () => {
    if (!address.startsWith('0x') || address.length !== 42) {
      setError('Please enter a valid 0x address');
      return;
    }
    fetchPortfolio(address);
  };

  const copyToClipboard = (text) => {
    navigator.clipboard.writeText(text);
  };

  const formatBalance = (balance) => {
    const num = parseFloat(balance);
    if (num === 0) return '0';
    if (num < 0.0001) return '<0.0001';
    if (num < 1) return num.toFixed(6);
    if (num < 1000) return num.toFixed(4);
    return num.toLocaleString(undefined, { maximumFractionDigits: 2 });
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-purple-900 via-blue-900 to-indigo-900 p-4">
      <div className="max-w-4xl mx-auto pt-8">
        <h1 className="text-4xl font-bold text-white mb-6 flex items-center justify-center gap-3">
          <TrendingUp className="text-purple-400" />
          Monad Testnet Portfolio
        </h1>

        {/* Input */}
        <div className="bg-white/10 p-6 rounded-xl mb-6">
          <div className="flex gap-2">
            <input
              type="text"
              value={address}
              onChange={(e) => setAddress(e.target.value)}
              placeholder="Enter wallet address (0x...)"
              className="flex-1 px-4 py-3 rounded-lg bg-white/10 border border-white/20 text-white"
            />
            <button
              onClick={handleCheck}
              disabled={loading}
              className="px-4 bg-purple-500 hover:bg-purple-600 text-white rounded-lg"
            >
              {loading ? <Loader2 className="animate-spin" /> : <Search />}
            </button>
          </div>
          {error && <p className="mt-3 text-red-400">{error}</p>}
        </div>

        {/* Loading */}
        {loading && <p className="text-center text-purple-200">Loading...</p>}
        {/* Portfolio */}
        {portfolio && !loading && (
          <div className="bg-white/10 p-6 rounded-xl text-white">
            <h2 className="text-xl mb-2 flex items-center gap-2">
              <Wallet /> Portfolio
            </h2>
            <p className="text-purple-300 mb-4">{portfolio.address}</p>
            {portfolio.tokens.length > 0 ? (
              portfolio.tokens.map((t, i) => (
                <div key={i} className="flex justify-between py-2 border-b border-white/10">
                  <span>{t.symbol}</span>
                  <span>{formatBalance(t.balance)}</span>
                </div>
              ))
            ) : (
              <p className="text-purple-300">No tokens found.</p>
            )}
          </div>
        )}
      </div>
    </div>
  );
};

export default MonadPortfolioTracker;
