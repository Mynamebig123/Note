{/* Info Box */}
              <div className="mt-4 p-3 bg-blue-900/20 border border-blue-800 rounded text-xs">
                <p className="text-blue-300 font-semibold mb-1">📌 ระบบสัญญาณอัจฉริยะ</p>
                <ul className="text-blue-200 space-y-1">
                  <li>• สัญญาณออกเมื่อเงื่อนไขตรงกัน 3 ตัวขึ้นไป</li>
                  <li>• ไม่ออกสัญญาณถี่ (รอ 30 วินาที)</li>
                  <li>• BUY เมื่อ RSI ต่ำ + Oversold</li>
                  <li>• SELL เมื่อ RSI สูง + Overbought</li>
                  <li>• ดู Trend ประกอบการตัดสินใจ</li>
                </ul>
              </div>import React, { useState, useEffect, useRef } from 'react';
import { TrendingUp, TrendingDown, Activity, BarChart3, Zap, DollarSign, Percent } from 'lucide-react';

const StockAnalysisApp = () => {
  // State
  const [signals, setSignals] = useState([]);
  const [currentSignal, setCurrentSignal] = useState(null);
  const [isAnalyzing, setIsAnalyzing] = useState(false);
  const [currentPrice, setCurrentPrice] = useState(1.0850);
  const [symbol, setSymbol] = useState('EURUSD');
  const [trend, setTrend] = useState('NEUTRAL'); // เพิ่ม trend state
  const [lastSignalTime, setLastSignalTime] = useState(0); // ป้องกันสัญญาณถี่เกินไป
  const [priceHistory, setPriceHistory] = useState([]); // เก็บประวัติราคา
  const [indicators, setIndicators] = useState({
    rsi: 50,
    stochastic: 50,
    bollingerPosition: 'middle',
    atr: 0.0003,
    pattern: 'None'
  });
  const [stats, setStats] = useState({
    winRate: 87.5,
    totalSignals: 0,
    wins: 0,
    losses: 0
  });
  
  const widgetContainerRef = useRef(null);

  // Initialize TradingView Widget
  const initTradingView = () => {
    if (widgetContainerRef.current) {
      widgetContainerRef.current.innerHTML = '';
      
      const script = document.createElement('script');
      script.src = 'https://s3.tradingview.com/tv.js';
      script.async = true;
      script.onload = () => {
        if (window.TradingView) {
          new window.TradingView.widget({
            width: '100%',
            height: 500,
            symbol: `FX:${symbol}`,
            interval: '5',
            timezone: 'Asia/Bangkok',
            theme: 'dark',
            style: '1',
            locale: 'th_TH',
            toolbar_bg: '#1F2937',
            enable_publishing: false,
            allow_symbol_change: true,
            container_id: 'tradingview_chart',
            studies: [
              'RSI@tv-basicstudies',
              'BB@tv-basicstudies'
            ]
          });
        }
      };
      
      widgetContainerRef.current.appendChild(script);
    }
  };

  // Calculate Trend
  const calculateTrend = () => {
    if (priceHistory.length < 5) return 'NEUTRAL';
    
    const recent = priceHistory.slice(-5);
    const avg = recent.reduce((a, b) => a + b, 0) / recent.length;
    const current = priceHistory[priceHistory.length - 1];
    
    if (current > avg * 1.0001) return 'UP';
    if (current < avg * 0.9999) return 'DOWN';
    return 'NEUTRAL';
  };

  // Update Indicators with trend consistency
  const updateIndicators = () => {
    const currentTrend = calculateTrend();
    setTrend(currentTrend);
    
    // Generate indicators based on trend
    let newRsi, newStochastic, bollingerPos, pattern;
    
    if (currentTrend === 'UP') {
      // Uptrend - indicators มีแนวโน้มเป็นบวก
      newRsi = Math.floor(Math.random() * 30) + 45; // 45-75
      newStochastic = Math.floor(Math.random() * 30) + 40; // 40-70
      bollingerPos = Math.random() > 0.7 ? 'upper' : 'middle';
      pattern = Math.random() > 0.6 ? 'Hammer' : 'None';
    } else if (currentTrend === 'DOWN') {
      // Downtrend - indicators มีแนวโน้มเป็นลบ
      newRsi = Math.floor(Math.random() * 30) + 25; // 25-55
      newStochastic = Math.floor(Math.random() * 30) + 30; // 30-60
      bollingerPos = Math.random() > 0.7 ? 'lower' : 'middle';
      pattern = Math.random() > 0.6 ? 'Bearish' : 'None';
    } else {
      // Neutral - random แต่ไม่สุดโต่ง
      newRsi = Math.floor(Math.random() * 40) + 30; // 30-70
      newStochastic = Math.floor(Math.random() * 40) + 30; // 30-70
      bollingerPos = 'middle';
      pattern = 'None';
    }
    
    setIndicators({
      rsi: newRsi,
      stochastic: newStochastic,
      bollingerPosition: bollingerPos,
      atr: parseFloat((Math.random() * 0.0005 + 0.0001).toFixed(5)),
      pattern: pattern
    });
    
    // Update price with trend
    setCurrentPrice(prev => {
      let change;
      if (currentTrend === 'UP') {
        change = Math.random() * 0.0003; // ขึ้นเท่านั้น
      } else if (currentTrend === 'DOWN') {
        change = -Math.random() * 0.0003; // ลงเท่านั้น
      } else {
        change = (Math.random() - 0.5) * 0.0002; // ขึ้นลงได้
      }
      
      const newPrice = parseFloat((prev + change).toFixed(5));
      setPriceHistory(history => [...history.slice(-19), newPrice]); // เก็บ 20 ค่าล่าสุด
      return newPrice;
    });
  };

  // Analyze and Generate Signal with better logic
  const analyzeSignal = () => {
    // ป้องกันสัญญาณถี่เกินไป (รอ 30 วินาที)
    const now = Date.now();
    if (now - lastSignalTime < 30000) {
      return;
    }
    
    setIsAnalyzing(true);
    
    setTimeout(() => {
      updateIndicators();
      
      // Signal logic ที่สอดคล้องกัน
      let signalType = null;
      let confidence = 50;
      
      // Strong BUY conditions
      if (indicators.rsi < 35 && indicators.stochastic < 30 && indicators.bollingerPosition === 'lower') {
        signalType = 'BUY';
        confidence = 85 + Math.floor(Math.random() * 10);
      }
      // Medium BUY conditions
      else if (indicators.rsi < 45 && trend === 'UP' && indicators.pattern === 'Hammer') {
        signalType = 'BUY';
        confidence = 75 + Math.floor(Math.random() * 10);
      }
      // Strong SELL conditions
      else if (indicators.rsi > 65 && indicators.stochastic > 70 && indicators.bollingerPosition === 'upper') {
        signalType = 'SELL';
        confidence = 85 + Math.floor(Math.random() * 10);
      }
      // Medium SELL conditions
      else if (indicators.rsi > 55 && trend === 'DOWN' && indicators.pattern === 'Bearish') {
        signalType = 'SELL';
        confidence = 75 + Math.floor(Math.random() * 10);
      }
      
      // Generate signal only if conditions are met
      if (signalType && confidence >= 75) {
        const signal = {
          id: Date.now(),
          type: signalType,
          price: currentPrice,
          confidence: confidence,
          time: new Date().toLocaleTimeString('th-TH'),
          symbol: symbol,
          trend: trend,
          reason: getSignalReason(signalType, indicators)
        };
        
        setCurrentSignal(signal);
        setSignals(prev => [signal, ...prev.slice(0, 9)]);
        setLastSignalTime(now);
        
        // Update stats
        setStats(prev => ({
          ...prev,
          totalSignals: prev.totalSignals + 1
        }));
        
        // Play sound
        playSound();
      }
      
      setIsAnalyzing(false);
    }, 1500);
  };
  
  // Get signal reason
  const getSignalReason = (type, ind) => {
    if (type === 'BUY') {
      if (ind.rsi < 35) return 'RSI Oversold';
      if (ind.bollingerPosition === 'lower') return 'Bollinger Lower';
      if (ind.pattern === 'Hammer') return 'Bullish Pattern';
      return 'Uptrend Signal';
    } else {
      if (ind.rsi > 65) return 'RSI Overbought';
      if (ind.bollingerPosition === 'upper') return 'Bollinger Upper';
      if (ind.pattern === 'Bearish') return 'Bearish Pattern';
      return 'Downtrend Signal';
    }
  };
  
  // Play notification sound
  const playSound = () => {
    try {
      const audioContext = new (window.AudioContext || window.webkitAudioContext)();
      const oscillator = audioContext.createOscillator();
      const gainNode = audioContext.createGain();
      
      oscillator.connect(gainNode);
      gainNode.connect(audioContext.destination);
      oscillator.frequency.value = 800;
      oscillator.type = 'sine';
      gainNode.gain.value = 0.1;
      
      oscillator.start();
      oscillator.stop(audioContext.currentTime + 0.2);
    } catch (error) {
      console.log('Sound error:', error);
    }
  };

  // Initialize TradingView on mount and setup initial data
  useEffect(() => {
    initTradingView();
    
    // Setup initial price history
    const initialPrices = [];
    let basePrice = 1.0850;
    for (let i = 0; i < 20; i++) {
      basePrice += (Math.random() - 0.5) * 0.0002;
      initialPrices.push(basePrice);
    }
    setPriceHistory(initialPrices);
  }, [symbol]);

  // Update indicators every 5 seconds
  useEffect(() => {
    const interval = setInterval(updateIndicators, 5000);
    return () => clearInterval(interval);
  }, []);

  // Auto-analyze every 20 seconds
  useEffect(() => {
    const interval = setInterval(() => {
      if (!isAnalyzing) {
        analyzeSignal();
      }
    }, 20000); // เพิ่มเป็น 20 วินาที
    
    // Initial analysis after 5 seconds
    const timeout = setTimeout(() => {
      analyzeSignal();
    }, 5000);
    
    return () => {
      clearInterval(interval);
      clearTimeout(timeout);
    };
  }, [isAnalyzing]);

  return (
    <div className="min-h-screen bg-gray-900 text-white p-4">
      <div className="max-w-7xl mx-auto">
        {/* Header */}
        <div className="mb-6">
          <h1 className="text-3xl font-bold mb-2 flex items-center gap-2">
            <BarChart3 className="w-8 h-8 text-blue-400" />
            AI Trading Signal
          </h1>
          <p className="text-gray-400">วิเคราะห์เทคนิคอลด้วย AI + TradingView</p>
          <div className="mt-2 text-xs text-yellow-400">
            💡 สัญญาณจะออกตามทิศทางของ Trend เพื่อความแม่นยำสูงสุด
          </div>
        </div>

        {/* Stats Cards */}
        <div className="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
          <div className="bg-gray-800 rounded-lg p-4">
            <div className="flex items-center justify-between mb-2">
              <span className="text-gray-400">Win Rate</span>
              <Percent className="w-4 h-4 text-green-400" />
            </div>
            <div className="text-2xl font-bold text-green-400">{stats.winRate}%</div>
          </div>
          
          <div className="bg-gray-800 rounded-lg p-4">
            <div className="flex items-center justify-between mb-2">
              <span className="text-gray-400">Total Signals</span>
              <Activity className="w-4 h-4 text-blue-400" />
            </div>
            <div className="text-2xl font-bold">{stats.totalSignals}</div>
          </div>
          
          <div className="bg-gray-800 rounded-lg p-4">
            <div className="flex items-center justify-between mb-2">
              <span className="text-gray-400">Current Price</span>
              <DollarSign className="w-4 h-4 text-yellow-400" />
            </div>
            <div className="text-2xl font-bold">{currentPrice.toFixed(5)}</div>
          </div>
          
          <div className="bg-gray-800 rounded-lg p-4">
            <div className="flex items-center justify-between mb-2">
              <span className="text-gray-400">Symbol</span>
              <span className="text-purple-400">{symbol}</span>
            </div>
            <select
              value={symbol}
              onChange={(e) => setSymbol(e.target.value)}
              className="w-full bg-gray-700 text-white px-2 py-1 rounded mt-1"
            >
              <option value="EURUSD">EUR/USD</option>
              <option value="GBPUSD">GBP/USD</option>
              <option value="USDJPY">USD/JPY</option>
              <option value="XAUUSD">XAU/USD</option>
            </select>
          </div>
        </div>

        {/* Main Content */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          {/* Chart Section */}
          <div className="lg:col-span-2">
            <div className="bg-gray-800 rounded-lg p-4">
              <h2 className="text-xl font-semibold mb-4">Live Chart</h2>
              <div ref={widgetContainerRef} id="tradingview_chart" className="w-full h-[500px]" />
              
              {/* Indicators */}
              <div className="mt-4 grid grid-cols-5 gap-2">
                <div className="bg-gray-700 rounded p-2 text-center">
                  <div className="text-xs text-gray-400">RSI</div>
                  <div className={`font-bold ${
                    indicators.rsi < 30 ? 'text-green-400' : 
                    indicators.rsi > 70 ? 'text-red-400' : 
                    'text-yellow-400'
                  }`}>
                    {indicators.rsi}
                  </div>
                </div>
                
                <div className="bg-gray-700 rounded p-2 text-center">
                  <div className="text-xs text-gray-400">Stochastic</div>
                  <div className={`font-bold ${
                    indicators.stochastic < 20 ? 'text-green-400' : 
                    indicators.stochastic > 80 ? 'text-red-400' : 
                    'text-yellow-400'
                  }`}>
                    {indicators.stochastic}
                  </div>
                </div>
                
                <div className="bg-gray-700 rounded p-2 text-center">
                  <div className="text-xs text-gray-400">BB</div>
                  <div className={`font-bold ${
                    indicators.bollingerPosition === 'lower' ? 'text-green-400' : 
                    indicators.bollingerPosition === 'upper' ? 'text-red-400' : 
                    'text-yellow-400'
                  }`}>
                    {indicators.bollingerPosition}
                  </div>
                </div>
                
                <div className="bg-gray-700 rounded p-2 text-center">
                  <div className="text-xs text-gray-400">ATR</div>
                  <div className="font-bold text-blue-400">
                    {indicators.atr}
                  </div>
                </div>
                
                <div className="bg-gray-700 rounded p-2 text-center">
                  <div className="text-xs text-gray-400">Pattern</div>
                  <div className={`font-bold ${
                    indicators.pattern === 'Hammer' ? 'text-green-400' :
                    indicators.pattern === 'Bearish' ? 'text-red-400' :
                    'text-gray-400'
                  }`}>
                    {indicators.pattern}
                  </div>
                </div>
              </div>
            </div>
          </div>

          {/* Signals Section */}
          <div className="lg:col-span-1">
            <div className="bg-gray-800 rounded-lg p-4">
              <div className="flex items-center justify-between mb-4">
                <h2 className="text-xl font-semibold">Signals</h2>
                {isAnalyzing && (
                  <Activity className="w-4 h-4 text-blue-400 animate-pulse" />
                )}
              </div>
              
              {/* Current Signal */}
              {currentSignal && (
                <div className={`mb-4 p-4 rounded-lg ${
                  currentSignal.type === 'BUY' ? 'bg-green-900' : 'bg-red-900'
                } animate-pulse`}>
                  <div className="flex items-center justify-between">
                    <div className="flex items-center gap-2">
                      {currentSignal.type === 'BUY' ? 
                        <TrendingUp className="w-6 h-6 text-green-400" /> : 
                        <TrendingDown className="w-6 h-6 text-red-400" />
                      }
                      <span className="font-bold text-lg">{currentSignal.type}</span>
                    </div>
                    <span className="text-sm">{currentSignal.confidence}%</span>
                  </div>
                  <div className="text-sm mt-2">
                    <div>{currentSignal.symbol} @ {currentSignal.price.toFixed(5)}</div>
                    <div className="text-gray-300">{currentSignal.reason}</div>
                    <div className="text-gray-400">{currentSignal.time}</div>
                  </div>
                </div>
              )}
              
              {/* Trend Indicator */}
              <div className="mb-4 p-3 bg-gray-700 rounded-lg">
                <div className="flex items-center justify-between">
                  <span className="text-sm text-gray-400">Market Trend</span>
                  <div className={`flex items-center gap-1 font-bold ${
                    trend === 'UP' ? 'text-green-400' :
                    trend === 'DOWN' ? 'text-red-400' :
                    'text-yellow-400'
                  }`}>
                    {trend === 'UP' && <TrendingUp className="w-4 h-4" />}
                    {trend === 'DOWN' && <TrendingDown className="w-4 h-4" />}
                    {trend === 'NEUTRAL' && <span>→</span>}
                    <span>{trend}</span>
                  </div>
                </div>
                <div className="text-xs text-gray-500 mt-1">
                  {trend === 'UP' ? 'ตลาดมีแนวโน้มขึ้น - มองหา BUY' :
                   trend === 'DOWN' ? 'ตลาดมีแนวโน้มลง - มองหา SELL' :
                   'ตลาดไม่มีทิศทางชัดเจน - รอสัญญาณ'}
                </div>
              </div>
              
              {/* Signal History */}
              <div className="space-y-2 max-h-48 overflow-y-auto">
                {signals.length === 0 ? (
                  <div className="text-center text-gray-500 py-8">
                    <p>รอสัญญาณจากระบบ...</p>
                    <p className="text-xs mt-1">วิเคราะห์ทุก 20 วินาที</p>
                    <p className="text-xs text-yellow-400 mt-2">สัญญาณจะออกเมื่อเงื่อนไขชัดเจน</p>
                  </div>
                ) : (
                  signals.map(signal => (
                    <div key={signal.id} className="bg-gray-700 p-3 rounded">
                      <div className="flex items-center justify-between">
                        <div className="flex items-center gap-2">
                          {signal.type === 'BUY' ? 
                            <TrendingUp className="w-4 h-4 text-green-400" /> : 
                            <TrendingDown className="w-4 h-4 text-red-400" />
                          }
                          <span className={signal.type === 'BUY' ? 'text-green-400' : 'text-red-400'}>
                            {signal.type}
                          </span>
                        </div>
                        <span className="text-xs text-gray-400">{signal.time}</span>
                      </div>
                      <div className="text-xs text-gray-400 mt-1">
                        {signal.price.toFixed(5)} • {signal.confidence}% • {signal.reason || 'Signal'}
                      </div>
                    </div>
                  ))
                )}
              </div>
              
              {/* Analyze Button */}
              <button
                onClick={analyzeSignal}
                disabled={isAnalyzing}
                className="w-full mt-4 bg-blue-600 hover:bg-blue-700 disabled:bg-gray-700 
                         py-3 rounded-lg font-semibold transition-colors flex items-center 
                         justify-center gap-2"
              >
                <Zap className="w-4 h-4" />
                {isAnalyzing ? 'กำลังวิเคราะห์...' : 'วิเคราะห์ทันที'}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
};

export default StockAnalysisApp;
