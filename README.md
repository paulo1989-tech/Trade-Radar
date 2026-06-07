import { useState, useEffect, useCallback, useRef } from "react";

const STOCKS = [
  { ticker: "NVDA", name: "NVIDIA Corp", sector: "AI/Tech" },
  { ticker: "META", name: "Meta Platforms", sector: "AI/Tech" },
  { ticker: "MSFT", name: "Microsoft Corp", sector: "AI/Tech" },
  { ticker: "GOOGL", name: "Alphabet Inc", sector: "AI/Tech" },
  { ticker: "AMD", name: "Advanced Micro Devices", sector: "AI/Tech" },
  { ticker: "PLTR", name: "Palantir Technologies", sector: "AI/Tech" },
  { ticker: "SMCI", name: "Super Micro Computer", sector: "AI/Tech" },
  { ticker: "TSLA", name: "Tesla Inc", sector: "Tech/EV" },
  { ticker: "AAPL", name: "Apple Inc", sector: "Tech" },
  { ticker: "AMZN", name: "Amazon.com", sector: "Tech/Cloud" },
  { ticker: "CRM", name: "Salesforce Inc", sector: "AI/SaaS" },
  { ticker: "SNOW", name: "Snowflake Inc", sector: "AI/Data" },
  { ticker: "CRWD", name: "CrowdStrike Holdings", sector: "Cybersec" },
  { ticker: "ARM", name: "Arm Holdings", sector: "AI/Chips" },
  { ticker: "MRVL", name: "Marvell Technology", sector: "AI/Chips" },
];

function generateStockData(ticker) {
  const base = {
    NVDA: 875, META: 520, MSFT: 415, GOOGL: 178, AMD: 162,
    PLTR: 28, SMCI: 48, TSLA: 248, AAPL: 212, AMZN: 195,
    CRM: 298, SNOW: 145, CRWD: 318, ARM: 132, MRVL: 88
  };
  const price = base[ticker] || 100;
  const change = (Math.random() - 0.4) * 6;
  const volumeMultiplier = Math.random() * 4 + 0.5;
  const rsi = Math.random() * 60 + 30;
  const rsVsMarket = (Math.random() - 0.3) * 15;
  const newsScore = Math.random() * 100;
  const redditScore = Math.random() * 100;
  const breakout = Math.random() > 0.6;
  const currentPrice = price * (1 + change / 100);

  // Score calculation
  let score = 0;
  let reasons = [];
  let bullishFactors = 0;

  if (volumeMultiplier > 1.5) { score += 20; bullishFactors++; reasons.push(`Volumen ${volumeMultiplier.toFixed(1)}x Ã¼ber Ã˜`); }
  if (breakout) { score += 25; bullishFactors++; reasons.push("Kursausbruch Ã¼ber Widerstand"); }
  if (rsVsMarket > 3) { score += 20; bullishFactors++; reasons.push(`RS: +${rsVsMarket.toFixed(1)}% vs. S&P500`); }
  if (newsScore > 65) { score += 15; bullishFactors++; reasons.push("Positive Nachrichtenlage"); }
  if (redditScore > 60) { score += 10; bullishFactors++; reasons.push("Bullishe Reddit-Stimmung"); }
  if (rsi > 55 && rsi < 75) { score += 10; bullishFactors++; reasons.push(`RSI ${rsi.toFixed(0)} â€“ Momentum stark`); }

  // Signal type
  let signal = "WATCH";
  if (score >= 70 && bullishFactors >= 4) signal = "STRONG";
  else if (score >= 45 && bullishFactors >= 3) signal = "WATCH";
  else signal = "AVOID";

  const entry = currentPrice;
  const stopLoss = entry * (1 - (signal === "STRONG" ? 0.04 : 0.06));
  const target = entry * (1 + (signal === "STRONG" ? 0.12 : 0.08));
  const crv = ((target - entry) / (entry - stopLoss)).toFixed(1);

  return {
    ticker,
    price: currentPrice,
    change,
    volume: volumeMultiplier,
    rsi,
    rsVsMarket,
    newsScore,
    redditScore,
    breakout,
    score: Math.min(score, 98),
    signal,
    reasons,
    entry: entry.toFixed(2),
    stopLoss: stopLoss.toFixed(2),
    target: target.toFixed(2),
    crv,
    bullishFactors,
    timestamp: new Date(),
  };
}

const STYLES = `
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;600;700;800&display=swap');

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: #050a0f;
    color: #e0f0ff;
    font-family: 'Syne', sans-serif;
    min-height: 100vh;
  }

  .app {
    min-height: 100vh;
    background: radial-gradient(ellipse at 20% 0%, #001a2e 0%, #050a0f 50%),
                radial-gradient(ellipse at 80% 100%, #001428 0%, transparent 60%);
    padding: 0;
  }

  /* HEADER */
  .header {
    border-bottom: 1px solid rgba(0, 180, 255, 0.15);
    padding: 20px 32px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    background: rgba(0, 10, 20, 0.8);
    backdrop-filter: blur(12px);
    position: sticky;
    top: 0;
    z-index: 100;
  }
  .logo {
    display: flex;
    align-items: center;
    gap: 12px;
  }
  .logo-icon {
    width: 40px; height: 40px;
    background: linear-gradient(135deg, #00b4ff, #0066ff);
    border-radius: 10px;
    display: flex; align-items: center; justify-content: center;
    font-size: 20px;
    box-shadow: 0 0 20px rgba(0, 180, 255, 0.4);
    animation: pulse-glow 2s ease-in-out infinite;
  }
  @keyframes pulse-glow {
    0%, 100% { box-shadow: 0 0 20px rgba(0, 180, 255, 0.4); }
    50% { box-shadow: 0 0 35px rgba(0, 180, 255, 0.7); }
  }
  .logo-text { font-size: 20px; font-weight: 800; letter-spacing: -0.5px; }
  .logo-text span { color: #00b4ff; }

  .header-right {
    display: flex;
    align-items: center;
    gap: 16px;
  }
  .live-badge {
    display: flex;
    align-items: center;
    gap: 6px;
    background: rgba(0, 255, 100, 0.1);
    border: 1px solid rgba(0, 255, 100, 0.3);
    padding: 6px 12px;
    border-radius: 20px;
    font-size: 12px;
    font-weight: 700;
    color: #00ff64;
    font-family: 'Space Mono', monospace;
    letter-spacing: 1px;
  }
  .live-dot {
    width: 7px; height: 7px;
    background: #00ff64;
    border-radius: 50%;
    animation: blink 1.2s ease-in-out infinite;
  }
  @keyframes blink {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.2; }
  }
  .scan-btn {
    background: linear-gradient(135deg, #00b4ff, #0066ff);
    border: none;
    color: #fff;
    padding: 8px 20px;
    border-radius: 8px;
    font-family: 'Syne', sans-serif;
    font-weight: 700;
    font-size: 13px;
    cursor: pointer;
    transition: all 0.2s;
    letter-spacing: 0.5px;
  }
  .scan-btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 6px 20px rgba(0, 180, 255, 0.4);
  }
  .scan-btn:disabled {
    opacity: 0.6;
    cursor: not-allowed;
    transform: none;
  }

  /* MAIN LAYOUT */
  .main { padding: 28px 32px; max-width: 1600px; margin: 0 auto; }

  /* STATS BAR */
  .stats-bar {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 16px;
    margin-bottom: 28px;
  }
  .stat-card {
    background: rgba(255,255,255,0.03);
    border: 1px solid rgba(255,255,255,0.07);
    border-radius: 14px;
    padding: 18px 22px;
    position: relative;
    overflow: hidden;
  }
  .stat-card::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
    background: linear-gradient(90deg, transparent, #00b4ff, transparent);
    opacity: 0.5;
  }
  .stat-label {
    font-size: 11px;
    color: #4a7a9b;
    letter-spacing: 1.5px;
    text-transform: uppercase;
    margin-bottom: 8px;
    font-family: 'Space Mono', monospace;
  }
  .stat-value {
    font-size: 28px;
    font-weight: 800;
    line-height: 1;
  }
  .stat-value.green { color: #00ff64; }
  .stat-value.yellow { color: #ffcc00; }
  .stat-value.red { color: #ff4466; }
  .stat-value.blue { color: #00b4ff; }

  /* GRID */
  .grid { display: grid; grid-template-columns: 1fr 380px; gap: 24px; }

  /* SIGNAL LIST */
  .panel {
    background: rgba(255,255,255,0.02);
    border: 1px solid rgba(255,255,255,0.07);
    border-radius: 18px;
    overflow: hidden;
  }
  .panel-header {
    padding: 18px 24px;
    border-bottom: 1px solid rgba(255,255,255,0.06);
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .panel-title {
    font-size: 15px;
    font-weight: 700;
    color: #c0d8f0;
    letter-spacing: 0.3px;
  }
  .panel-sub { font-size: 12px; color: #3a6a8a; font-family: 'Space Mono', monospace; }

  /* FILTER TABS */
  .filter-tabs {
    display: flex;
    gap: 8px;
    padding: 16px 24px;
    border-bottom: 1px solid rgba(255,255,255,0.06);
  }
  .tab {
    padding: 6px 14px;
    border-radius: 20px;
    font-size: 12px;
    font-weight: 700;
    cursor: pointer;
    border: 1px solid rgba(255,255,255,0.1);
    background: transparent;
    color: #5a8aaa;
    transition: all 0.2s;
    font-family: 'Syne', sans-serif;
    letter-spacing: 0.3px;
  }
  .tab.active-all { background: rgba(0,180,255,0.15); border-color: rgba(0,180,255,0.4); color: #00b4ff; }
  .tab.active-strong { background: rgba(0,255,100,0.12); border-color: rgba(0,255,100,0.35); color: #00ff64; }
  .tab.active-watch { background: rgba(255,200,0,0.12); border-color: rgba(255,200,0,0.35); color: #ffcc00; }
  .tab.active-avoid { background: rgba(255,50,80,0.12); border-color: rgba(255,50,80,0.3); color: #ff4466; }

  /* STOCK ROWS */
  .stock-list { overflow-y: auto; max-height: 600px; }
  .stock-row {
    padding: 16px 24px;
    border-bottom: 1px solid rgba(255,255,255,0.04);
    display: grid;
    grid-template-columns: 80px 1fr auto auto;
    gap: 16px;
    align-items: center;
    cursor: pointer;
    transition: background 0.15s;
    position: relative;
  }
  .stock-row:hover { background: rgba(255,255,255,0.025); }
  .stock-row.selected { background: rgba(0, 180, 255, 0.06); }
  .stock-row.selected::before {
    content: '';
    position: absolute;
    left: 0; top: 0; bottom: 0;
    width: 3px;
    background: #00b4ff;
  }

  .signal-badge {
    padding: 4px 10px;
    border-radius: 6px;
    font-size: 10px;
    font-weight: 700;
    letter-spacing: 1px;
    text-align: center;
    font-family: 'Space Mono', monospace;
  }
  .badge-STRONG { background: rgba(0,255,100,0.15); color: #00ff64; border: 1px solid rgba(0,255,100,0.3); }
  .badge-WATCH { background: rgba(255,200,0,0.12); color: #ffcc00; border: 1px solid rgba(255,200,0,0.3); }
  .badge-AVOID { background: rgba(255,50,80,0.1); color: #ff4466; border: 1px solid rgba(255,50,80,0.25); }

  .ticker { font-weight: 800; font-size: 15px; color: #e0f0ff; }
  .stock-name { font-size: 11px; color: #3a6a8a; margin-top: 2px; }
  .sector-tag { font-size: 10px; color: #0080cc; font-family: 'Space Mono', monospace; }

  .price-col { text-align: right; }
  .price { font-size: 14px; font-weight: 700; font-family: 'Space Mono', monospace; color: #c0d8f0; }
  .change { font-size: 12px; font-family: 'Space Mono', monospace; }
  .change.pos { color: #00ff64; }
  .change.neg { color: #ff4466; }

  .score-col { text-align: right; }
  .score-ring {
    width: 44px; height: 44px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 800;
    font-size: 13px;
    font-family: 'Space Mono', monospace;
    margin-left: auto;
  }
  .ring-strong { background: rgba(0,255,100,0.1); border: 2px solid #00ff64; color: #00ff64; box-shadow: 0 0 12px rgba(0,255,100,0.25); }
  .ring-watch { background: rgba(255,200,0,0.1); border: 2px solid #ffcc00; color: #ffcc00; }
  .ring-avoid { background: rgba(255,50,80,0.08); border: 2px solid #ff4466; color: #ff4466; }

  /* DETAIL PANEL */
  .detail-panel { display: flex; flex-direction: column; gap: 16px; }

  .detail-card {
    background: rgba(255,255,255,0.02);
    border: 1px solid rgba(255,255,255,0.07);
    border-radius: 18px;
    overflow: hidden;
  }

  .detail-header {
    padding: 20px 22px 16px;
    border-bottom: 1px solid rgba(255,255,255,0.06);
  }
  .detail-ticker {
    font-size: 28px;
    font-weight: 800;
    letter-spacing: -0.5px;
  }
  .detail-name { color: #3a6a8a; font-size: 13px; margin-top: 2px; }
  .detail-price-row {
    display: flex;
    align-items: baseline;
    gap: 12px;
    margin-top: 12px;
  }
  .detail-price {
    font-size: 36px;
    font-weight: 800;
    font-family: 'Space Mono', monospace;
    color: #e0f0ff;
  }

  /* TRADE LEVELS */
  .trade-levels { padding: 18px 22px; display: flex; flex-direction: column; gap: 10px; }
  .level-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 10px 14px;
    border-radius: 10px;
  }
  .level-entry { background: rgba(0,180,255,0.08); border: 1px solid rgba(0,180,255,0.2); }
  .level-stop { background: rgba(255,50,80,0.07); border: 1px solid rgba(255,50,80,0.2); }
  .level-target { background: rgba(0,255,100,0.07); border: 1px solid rgba(0,255,100,0.2); }
  .level-label { font-size: 11px; font-weight: 700; letter-spacing: 1px; text-transform: uppercase; font-family: 'Space Mono', monospace; }
  .label-entry { color: #00b4ff; }
  .label-stop { color: #ff4466; }
  .label-target { color: #00ff64; }
  .level-value { font-size: 16px; font-weight: 800; font-family: 'Space Mono', monospace; color: #e0f0ff; }

  .crv-row {
    display: flex;
    justify-content: space-between;
    padding: 12px 14px;
    background: rgba(255,255,255,0.03);
    border-radius: 10px;
    border: 1px solid rgba(255,255,255,0.07);
  }
  .crv-label { font-size: 12px; color: #4a7a9b; font-family: 'Space Mono', monospace; }
  .crv-value { font-size: 15px; font-weight: 800; color: #00b4ff; font-family: 'Space Mono', monospace; }

  /* FACTORS */
  .factors { padding: 18px 22px; }
  .factors-title { font-size: 11px; color: #3a6a8a; letter-spacing: 1.5px; text-transform: uppercase; margin-bottom: 12px; font-family: 'Space Mono', monospace; }
  .factor-item {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 6px 0;
    font-size: 12px;
    color: #8abacc;
    border-bottom: 1px solid rgba(255,255,255,0.04);
  }
  .factor-item:last-child { border-bottom: none; }
  .factor-icon { font-size: 13px; }

  /* RADAR MINI */
  .radar-bars { padding: 0 22px 18px; display: flex; flex-direction: column; gap: 8px; }
  .radar-bar-row { display: flex; align-items: center; gap: 10px; }
  .bar-label { font-size: 10px; color: #4a7a9b; width: 70px; font-family: 'Space Mono', monospace; text-transform: uppercase; flex-shrink: 0; }
  .bar-track { flex: 1; height: 6px; background: rgba(255,255,255,0.05); border-radius: 3px; overflow: hidden; }
  .bar-fill { height: 100%; border-radius: 3px; transition: width 0.5s ease; }
  .bar-fill.green { background: linear-gradient(90deg, #00cc50, #00ff64); }
  .bar-fill.yellow { background: linear-gradient(90deg, #cc9900, #ffcc00); }
  .bar-fill.orange { background: linear-gradient(90deg, #cc5500, #ff8800); }
  .bar-fill.blue { background: linear-gradient(90deg, #0066cc, #00b4ff); }
  .bar-pct { font-size: 10px; color: #3a6a8a; width: 28px; text-align: right; font-family: 'Space Mono', monospace; }

  /* NOTIFICATIONS */
  .notif-panel { padding: 18px 22px; }
  .notif-title { font-size: 11px; color: #3a6a8a; letter-spacing: 1.5px; text-transform: uppercase; margin-bottom: 12px; font-family: 'Space Mono', monospace; }
  .notif-item {
    background: rgba(0,255,100,0.06);
    border: 1px solid rgba(0,255,100,0.2);
    border-radius: 10px;
    padding: 12px 14px;
    margin-bottom: 8px;
    animation: slideIn 0.4s ease;
  }
  @keyframes slideIn {
    from { opacity: 0; transform: translateY(-8px); }
    to { opacity: 1; transform: translateY(0); }
  }
  .notif-ticker { font-size: 13px; font-weight: 800; color: #00ff64; }
  .notif-text { font-size: 11px; color: #5a9a7a; margin-top: 3px; }
  .notif-time { font-size: 10px; color: #3a6a8a; font-family: 'Space Mono', monospace; margin-top: 4px; }

  /* SCANNING OVERLAY */
  .scan-overlay {
    position: fixed;
    inset: 0;
    background: rgba(0,5,15,0.85);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 200;
    backdrop-filter: blur(4px);
  }
  .scan-box {
    text-align: center;
    padding: 48px;
  }
  .scan-spinner {
    width: 80px; height: 80px;
    border: 3px solid rgba(0,180,255,0.15);
    border-top-color: #00b4ff;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
    margin: 0 auto 24px;
  }
  @keyframes spin { to { transform: rotate(360deg); } }
  .scan-text { font-size: 18px; font-weight: 700; color: #00b4ff; }
  .scan-sub { font-size: 13px; color: #3a6a8a; margin-top: 8px; font-family: 'Space Mono', monospace; }

  /* EMPTY STATE */
  .empty { padding: 48px; text-align: center; color: #3a6a8a; }
  .empty-icon { font-size: 36px; margin-bottom: 12px; }
  .empty-text { font-size: 14px; }

  /* SCROLLBAR */
  ::-webkit-scrollbar { width: 4px; }
  ::-webkit-scrollbar-track { background: transparent; }
  ::-webkit-scrollbar-thumb { background: rgba(0,180,255,0.2); border-radius: 2px; }

  /* ALERT BADGE */
  .alert-dot {
    display: inline-block;
    width: 8px; height: 8px;
    background: #00ff64;
    border-radius: 50%;
    margin-right: 6px;
    animation: blink 1s infinite;
  }

  /* RESPONSIVE */
  @media (max-width: 900px) {
    .grid { grid-template-columns: 1fr; }
    .stats-bar { grid-template-columns: repeat(2, 1fr); }
    .main { padding: 16px; }
  }
`;

export default function TradingRadar() {
  const [stocks, setStocks] = useState([]);
  const [selected, setSelected] = useState(null);
  const [filter, setFilter] = useState("ALL");
  const [scanning, setScanning] = useState(false);
  const [notifications, setNotifications] = useState([]);
  const [lastScan, setLastScan] = useState(null);
  const notifRef = useRef([]);
  const [aiAnalysis, setAiAnalysis] = useState("");
  const [loadingAI, setLoadingAI] = useState(false);

  const runScan = useCallback(async () => {
    setScanning(true);
    await new Promise(r => setTimeout(r, 1800));

    const data = STOCKS.map(s => ({ ...s, ...generateStockData(s.ticker) }));
    data.sort((a, b) => b.score - a.score);
    setStocks(data);
    setLastScan(new Date());

    // Notifications for STRONG signals
    const strong = data.filter(s => s.signal === "STRONG");
    const newNotifs = strong.slice(0, 3).map(s => ({
      id: Date.now() + Math.random(),
      ticker: s.ticker,
      text: `Score ${s.score} â€” ${s.reasons[0]}`,
      time: new Date().toLocaleTimeString("de-DE", { hour: "2-digit", minute: "2-digit" }),
    }));

    if (newNotifs.length > 0) {
      notifRef.current = [...newNotifs, ...notifRef.current].slice(0, 5);
      setNotifications([...notifRef.current]);

      // Browser notification
      if ("Notification" in window && Notification.permission === "granted") {
        newNotifs.forEach(n => {
          new Notification(`ðŸš€ ${n.ticker} â€“ Starkes Signal!`, { body: n.text });
        });
      }
    }

    if (!selected && data.length > 0) setSelected(data[0]);
    setScanning(false);
  }, [selected]);

  useEffect(() => {
    runScan();
    const interval = setInterval(runScan, 45000);
    return () => clearInterval(interval);
  }, []);

  // Request notification permission
  useEffect(() => {
    if ("Notification" in window && Notification.permission === "default") {
      Notification.requestPermission();
    }
  }, []);

  const fetchAIAnalysis = async (stock) => {
    if (!stock) return;
    setLoadingAI(true);
    setAiAnalysis("");
    try {
      const prompt = `Du bist ein professioneller Aktienanalyst. Analysiere folgendes Trading-Signal kurz und prÃ¤zise auf Deutsch (max. 3 SÃ¤tze):
Aktie: ${stock.ticker} (${stock.name})
Kurs: $${stock.price.toFixed(2)}, Ã„nderung: ${stock.change.toFixed(2)}%
Volumen: ${stock.volume.toFixed(1)}x Ã¼ber Durchschnitt
RSI: ${stock.rsi.toFixed(0)}, RS vs. Markt: ${stock.rsVsMarket.toFixed(1)}%
Score: ${stock.score}/100, Signal: ${stock.signal}
Faktoren: ${stock.reasons.join(", ")}
Einstieg: $${stock.entry}, Stop-Loss: $${stock.stopLoss}, Ziel: $${stock.target}, CRV: ${stock.crv}

Gib eine knappe Trading-EinschÃ¤tzung: Warum ist dieses Setup interessant (oder nicht)? Was sollte der Trader beachten?`;

      const response = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-20250514",
          max_tokens: 1000,
          messages: [{ role: "user", content: prompt }],
        }),
      });
      const data = await response.json();
      const text = data.content?.map(c => c.text || "").join("") || "Keine Analyse verfÃ¼gbar.";
      setAiAnalysis(text);
    } catch {
      setAiAnalysis("KI-Analyse konnte nicht geladen werden.");
    }
    setLoadingAI(false);
  };

  useEffect(() => {
    if (selected) fetchAIAnalysis(selected);
  }, [selected?.ticker]);

  const filtered = stocks.filter(s => filter === "ALL" || s.signal === filter);
  const strongCount = stocks.filter(s => s.signal === "STRONG").length;
  const watchCount = stocks.filter(s => s.signal === "WATCH").length;
  const avoidCount = stocks.filter(s => s.signal === "AVOID").length;
  const avgScore = stocks.length ? Math.round(stocks.reduce((a, b) => a + b.score, 0) / stocks.length) : 0;

  const getScoreColor = (score) => score >= 70 ? "green" : score >= 45 ? "yellow" : "orange";
  const getRingClass = (signal) => `ring-${signal.toLowerCase()}`;

  return (
    <>
      <style>{STYLES}</style>
      <div className="app">
        {scanning && (
          <div className="scan-overlay">
            <div className="scan-box">
              <div className="scan-spinner" />
              <div className="scan-text">ðŸ” Markt wird gescannt â€¦</div>
              <div className="scan-sub">Analysiere {STOCKS.length} AI/Tech-Aktien</div>
            </div>
          </div>
        )}

        {/* HEADER */}
        <div className="header">
          <div className="logo">
            <div className="logo-icon">âš¡</div>
            <div>
              <div className="logo-text">Momentum<span>Radar</span></div>
            </div>
          </div>
          <div className="header-right">
            <div className="live-badge">
              <div className="live-dot" />
              LIVE
            </div>
            {lastScan && (
              <span style={{ fontSize: 11, color: "#3a6a8a", fontFamily: "Space Mono, monospace" }}>
                Scan: {lastScan.toLocaleTimeString("de-DE")}
              </span>
            )}
            <button className="scan-btn" onClick={runScan} disabled={scanning}>
              {scanning ? "Scanntâ€¦" : "â†» Neu scannen"}
            </button>
          </div>
        </div>

        <div className="main">
          {/* STATS BAR */}
          <div className="stats-bar">
            <div className="stat-card">
              <div className="stat-label">Starke Signale</div>
              <div className="stat-value green">{strongCount}</div>
            </div>
            <div className="stat-card">
              <div className="stat-label">Beobachten</div>
              <div className="stat-value yellow">{watchCount}</div>
            </div>
            <div className="stat-card">
              <div className="stat-label">Finger weg</div>
              <div className="stat-value red">{avoidCount}</div>
            </div>
            <div className="stat-card">
              <div className="stat-label">Ã˜ Radar-Score</div>
              <div className="stat-value blue">{avgScore}</div>
            </div>
          </div>

          {/* GRID */}
          <div className="grid">
            {/* LEFT: SIGNAL LIST */}
            <div>
              <div className="panel">
                <div className="panel-header">
                  <div className="panel-title">ðŸ“¡ Signal-Radar</div>
                  <div className="panel-sub">{filtered.length} Aktien</div>
                </div>
                <div className="filter-tabs">
                  {["ALL","STRONG","WATCH","AVOID"].map(f => (
                    <button
                      key={f}
                      className={`tab ${filter === f ? `active-${f.toLowerCase()}` : ""}`}
                      onClick={() => setFilter(f)}
                    >
                      {f === "ALL" ? "Alle" : f === "STRONG" ? "ðŸŸ¢ Strong" : f === "WATCH" ? "ðŸŸ¡ Watch" : "ðŸ”´ Avoid"}
                    </button>
                  ))}
                </div>
                <div className="stock-list">
                  {filtered.length === 0 && (
                    <div className="empty">
                      <div className="empty-icon">ðŸ“­</div>
                      <div className="empty-text">Keine Signale in dieser Kategorie</div>
                    </div>
                  )}
                  {filtered.map(stock => (
                    <div
                      key={stock.ticker}
                      className={`stock-row ${selected?.ticker === stock.ticker ? "selected" : ""}`}
                      onClick={() => setSelected(stock)}
                    >
                      <div>
                        <div className={`signal-badge badge-${stock.signal}`}>
                          {stock.signal === "STRONG" ? "ðŸŸ¢" : stock.signal === "WATCH" ? "ðŸŸ¡" : "ðŸ”´"} {stock.signal}
                        </div>
                      </div>
                      <div>
                        <div className="ticker">{stock.ticker}</div>
                        <div className="stock-name">{stock.name}</div>
                        <div className="sector-tag">{stock.sector}</div>
                      </div>
                      <div className="price-col">
                        <div className="price">${parseFloat(stock.price).toFixed(2)}</div>
                        <div className={`change ${stock.change >= 0 ? "pos" : "neg"}`}>
                          {stock.change >= 0 ? "â–²" : "â–¼"} {Math.abs(stock.change).toFixed(2)}%
                        </div>
                      </div>
                      <div className="score-col">
                        <div className={`score-ring ${getRingClass(stock.signal)}`}>
                          {stock.score}
                        </div>
                      </div>
                    </div>
                  ))}
                </div>
              </div>
            </div>

            {/* RIGHT: DETAIL + NOTIFICATIONS */}
            <div className="detail-panel">
              {/* DETAIL CARD */}
              {selected ? (
                <div className="detail-card">
                  <div className="detail-header">
                    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start" }}>
                      <div>
                        <div className={`detail-ticker ${selected.signal === "STRONG" ? "green" : selected.signal === "WATCH" ? "yellow" : "red"}`}
                          style={{ color: selected.signal === "STRONG" ? "#00ff64" : selected.signal === "WATCH" ? "#ffcc00" : "#ff4466" }}>
                          {selected.ticker}
                        </div>
                        <div className="detail-name">{selected.name}</div>
                      </div>
                      <div className={`score-ring ${getRingClass(selected.signal)}`} style={{ width: 52, height: 52, fontSize: 16 }}>
                        {selected.score}
                      </div>
                    </div>
                    <div className="detail-price-row">
                      <div className="detail-price">${parseFloat(selected.price).toFixed(2)}</div>
                      <div className={`change ${selected.change >= 0 ? "pos" : "neg"}`} style={{ fontSize: 16, fontFamily: "Space Mono, monospace" }}>
                        {selected.change >= 0 ? "â–²" : "â–¼"} {Math.abs(selected.change).toFixed(2)}%
                      </div>
                    </div>
                  </div>

                  {/* TRADE LEVELS */}
                  <div className="trade-levels">
                    <div className="level-row level-entry">
                      <span className="level-label label-entry">Einstieg</span>
                      <span className="level-value">${selected.entry}</span>
                    </div>
                    <div className="level-row level-stop">
                      <span className="level-label label-stop">Stop-Loss</span>
                      <span className="level-value">${selected.stopLoss}</span>
                    </div>
                    <div className="level-row level-target">
                      <span className="level-label label-target">Kursziel</span>
                      <span className="level-value">${selected.target}</span>
                    </div>
                    <div className="crv-row">
                      <span className="crv-label">Chance : Risiko</span>
                      <span className="crv-value">{selected.crv} : 1</span>
                    </div>
                  </div>

                  {/* RADAR BARS */}
                  <div className="radar-bars">
                    {[
                      { label: "Volumen", val: Math.min((selected.volume / 4) * 100, 100), color: selected.volume > 2 ? "green" : "yellow" },
                      { label: "RSI", val: selected.rsi, color: selected.rsi > 55 ? "green" : "yellow" },
                      { label: "RS/Markt", val: Math.min(Math.max((selected.rsVsMarket + 10) / 20 * 100, 0), 100), color: selected.rsVsMarket > 3 ? "green" : "orange" },
                      { label: "News", val: selected.newsScore, color: selected.newsScore > 65 ? "green" : "yellow" },
                      { label: "Reddit", val: selected.redditScore, color: selected.redditScore > 60 ? "green" : "orange" },
                    ].map(b => (
                      <div key={b.label} className="radar-bar-row">
                        <div className="bar-label">{b.label}</div>
                        <div className="bar-track">
                          <div className={`bar-fill ${b.color}`} style={{ width: `${b.val}%` }} />
                        </div>
                        <div className="bar-pct">{Math.round(b.val)}</div>
                      </div>
                    ))}
                  </div>

                  {/* FACTORS */}
                  <div className="factors">
                    <div className="factors-title">SignalbegrÃ¼ndung</div>
                    {selected.reasons.map((r, i) => (
                      <div key={i} className="factor-item">
                        <span className="factor-icon">âœ“</span>
                        <span>{r}</span>
                      </div>
                    ))}
                    {selected.reasons.length === 0 && (
                      <div style={{ color: "#ff4466", fontSize: 12 }}>âš  Keine bullishen Faktoren erkannt</div>
                    )}
                  </div>

                  {/* AI ANALYSIS */}
                  <div style={{ padding: "0 22px 18px" }}>
                    <div className="factors-title" style={{ marginBottom: 10 }}>ðŸ¤– KI-Analyse</div>
                    <div style={{
                      background: "rgba(0,180,255,0.05)",
                      border: "1px solid rgba(0,180,255,0.15)",
                      borderRadius: 10,
                      padding: "12px 14px",
                      fontSize: 12,
                      color: "#8ab8cc",
                      lineHeight: 1.6,
                      minHeight: 60,
                    }}>
                      {loadingAI ? (
                        <span style={{ color: "#3a6a8a" }}>Analysiere â€¦</span>
                      ) : aiAnalysis || "â€”"}
                    </div>
                  </div>
                </div>
              ) : (
                <div className="detail-card" style={{ padding: 48, textAlign: "center", color: "#3a6a8a" }}>
                  <div style={{ fontSize: 32, marginBottom: 12 }}>ðŸ“Š</div>
                  <div>Aktie auswÃ¤hlen fÃ¼r Details</div>
                </div>
              )}

              {/* NOTIFICATIONS */}
              {notifications.length > 0 && (
                <div className="detail-card">
                  <div className="notif-panel">
                    <div className="notif-title">
                      <span className="alert-dot" />
                      Benachrichtigungen
                    </div>
                    {notifications.map(n => (
                      <div key={n.id} className="notif-item">
                        <div className="notif-ticker">ðŸš€ {n.ticker}</div>
                        <div className="notif-text">{n.text}</div>
                        <div className="notif-time">{n.time}</div>
                      </div>
                    ))}
                  </div>
                </div>
              )}
            </div>
          </div>
        </div>
      </div>
    </>
  );
}
