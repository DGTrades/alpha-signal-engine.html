# alpha-signal-engine.html
Ai bot 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ALPHA SIGNAL ENGINE</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Space+Grotesk:wght@500;600;700&display=swap');
        body { font-family: 'Inter', system-ui, sans-serif; }
        .title-font { font-family: 'Space Grotesk', sans-serif; }
        .pulse { animation: pulse 2s infinite; }
        @keyframes pulse { 0%,100% {opacity:1} 50% {opacity:0.4} }
    </style>
</head>
<body class="bg-[#050d17] text-[#c8d8e8] min-h-screen pb-12">
    <div class="max-w-3xl mx-auto">
        <header class="bg-gradient-to-b from-[#0a1a2e] to-[#050d17] border-b border-[#0d2540] sticky top-0 z-50 px-6 py-5">
            <div class="flex items-center justify-between">
                <div class="flex items-center gap-x-3">
                    <div class="w-3 h-3 rounded-full bg-[#00ff88] shadow-[0_0_12px_#00ff88] pulse"></div>
                    <h1 class="title-font text-2xl font-bold tracking-tighter text-white">ALPHA SIGNAL ENGINE</h1>
                </div>
                <div id="chart-preview-header" class="hidden items-center gap-x-3 bg-[#0d2540] border border-[#1a3a5a] rounded-xl px-4 py-2">
                    <img id="header-thumb" class="w-10 h-7 object-cover rounded border border-[#1a3a5a]" alt="">
                    <div>
                        <div class="text-[#00bfff] text-xs font-bold">CHART LOADED</div>
                        <div id="header-filename" class="text-[#445566] text-[10px] truncate max-w-[140px]"></div>
                    </div>
                    <button onclick="clearChart()" class="text-[#556677] hover:text-white text-xl">×</button>
                </div>
            </div>
        </header>

        <div class="px-6 pt-8">
            <!-- Chart Upload -->
            <div class="mb-8">
                <label class="block text-[#556677] text-xs uppercase tracking-[1.5px] mb-3">Chart Screenshot</label>
                <div id="drop-zone" class="border-2 border-dashed border-[#1a2d40] hover:border-[#00bfff] rounded-2xl p-12 text-center cursor-pointer transition-all bg-[#07121d]"
                     onclick="document.getElementById('file-input').click()">
                    <input type="file" id="file-input" accept="image/*" class="hidden" onchange="handleFileSelect(event)">
                    <div id="upload-content">
                        <div class="text-6xl mb-6">📸</div>
                        <p class="text-[#8899aa] text-xl">Drop chart or click to upload</p>
                        <p class="text-[#334455] mt-2">PNG • JPG • WEBP</p>
                    </div>
                    <div id="preview-content" class="hidden">
                        <img id="chart-preview" class="mx-auto max-h-64 rounded-2xl border border-[#1a3a5a]" alt="">
                        <p class="text-[#00ff88] mt-4">✓ Chart ready for analysis</p>
                    </div>
                </div>
            </div>

            <!-- Controls -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                    <label class="block text-[#556677] text-xs uppercase tracking-widest mb-3">INSTRUMENT</label>
                    <div class="flex flex-wrap gap-2" id="pairs-container"></div>
                </div>
                <div class="grid grid-cols-2 gap-6">
                    <div>
                        <label class="block text-[#556677] text-xs uppercase tracking-widest mb-3">TIMEFRAME</label>
                        <div class="flex flex-wrap gap-2" id="timeframes-container"></div>
                    </div>
                    <div>
                        <label class="block text-[#556677] text-xs uppercase tracking-widest mb-3">SESSION</label>
                        <select id="session-select" class="w-full bg-[#0d1b2a] border border-[#1a2d40] rounded-2xl px-5 py-4 text-sm">
                            <option>London Open</option>
                            <option selected>NY Open</option>
                            <option>NY Killzone</option>
                            <option>Asian Range</option>
                            <option>Custom</option>
                        </select>
                    </div>
                </div>
            </div>

            <!-- Strategies -->
            <div class="mt-8">
                <div class="flex justify-between mb-4">
                    <label class="text-[#556677] text-xs uppercase tracking-widest">STRATEGIES</label>
                    <div class="text-xs flex gap-4">
                        <button onclick="selectAllStrategies()" class="text-[#00ff88]">All</button>
                        <button onclick="clearStrategies()" class="text-[#ff4560]">Clear</button>
                    </div>
                </div>
                <div class="grid grid-cols-2 gap-3" id="strategies-container"></div>
            </div>

            <!-- Notes -->
            <div class="mt-8">
                <label class="block text-[#556677] text-xs uppercase tracking-widest mb-3">NOTES</label>
                <textarea id="notes" rows="3" class="w-full bg-[#0d1b2a] border border-[#1a2d40] rounded-2xl px-5 py-4 text-sm" placeholder="Additional context..."></textarea>
            </div>

            <button onclick="generateSignal()" id="generate-btn"
                class="mt-10 w-full py-7 rounded-3xl font-bold text-xl tracking-wider bg-gradient-to-r from-[#00c864] to-[#00ff88] text-black hover:brightness-110">
                ⚡ GENERATE SIGNAL
            </button>

            <div id="error-msg" class="hidden mt-4 p-4 bg-red-950 border border-red-800 rounded-2xl text-red-400"></div>

            <!-- Signal Output -->
            <div id="signal-container" class="mt-12"></div>

            <!-- History -->
            <div id="history-section" class="mt-16 hidden">
                <h3 class="text-[#334455] text-xs uppercase tracking-widest mb-6">Previous Signals</h3>
                <div id="history-list" class="space-y-4"></div>
            </div>
        </div>
    </div>

    <script>
        // Data & Variables
        const STRATEGIES = [{id:"bor",label:"Break & Retest",icon:"⚡"},{id:"liq",label:"Liquidity Sweep",icon:"🌊"},{id:"rev",label:"Reversal Pattern",icon:"🔄"},{id:"ny5",label:"NY 5-Min ORB",icon:"🗽"},{id:"fvg",label:"FVG + IFVG",icon:"🎯"},{id:"ob",label:"Order Block",icon:"📦"},{id:"choch",label:"CHoCH / MSB",icon:"🔀"},{id:"pd",label:"Premium/Discount",icon:"💹"}];
        const PAIRS = ["NQ","ES","EURUSD","GBPUSD","BTCUSD","GOLD","SPY","QQQ"];
        const TIMEFRAMES = ["1m","5m","15m","1H","4H"];

        let currentPair = "NQ", currentTimeframe = "5m", selectedStrategies = ["bor","fvg","ny5"], chartFile = null, signalsHistory = [];

        function renderPairs() {
            const c = document.getElementById('pairs-container');
            c.innerHTML = '';
            PAIRS.forEach(p => {
                const b = document.createElement('button');
                b.className = `px-6 py-3 text-sm rounded-2xl transition-all ${currentPair===p ? 'bg-[#00ff8818] border border-[#00ff88] text-[#00ff88]' : 'bg-[#0d1b2a] border border-[#1a2d40] text-[#8899aa]'}`;
                b.textContent = p;
                b.onclick = () => { currentPair = p; renderPairs(); };
                c.appendChild(b);
            });
        }

        function renderTimeframes() {
            const c = document.getElementById('timeframes-container');
            c.innerHTML = '';
            TIMEFRAMES.forEach(tf => {
                const b = document.createElement('button');
                b.className = `px-5 py-3 text-sm rounded-2xl transition-all ${currentTimeframe===tf ? 'bg-[#00bfff18] border border-[#00bfff] text-[#00bfff]' : 'bg-[#0d1b2a] border border-[#1a2d40] text-[#8899aa]'}`;
                b.textContent = tf;
                b.onclick = () => { currentTimeframe = tf; renderTimeframes(); };
                c.appendChild(b);
            });
        }

        function renderStrategies() {
            const c = document.getElementById('strategies-container');
            c.innerHTML = '';
            STRATEGIES.forEach(s => {
                const active = selectedStrategies.includes(s.id);
                const btn = document.createElement('button');
                btn.className = `flex items-center gap-3 px-5 py-5 rounded-2xl text-left text-sm transition-all ${active ? 'bg-[#0d2540] border border-[#00bfff55] text-white' : 'bg-[#07121d] text-[#445566]'}`;
                btn.innerHTML = `<span class="text-2xl">${s.icon}</span><span>${s.label}</span>${active ? '<span class="ml-auto text-[#00ff88]">✓</span>' : ''}`;
                btn.onclick = () => {
                    if (selectedStrategies.includes(s.id)) selectedStrategies = selectedStrategies.filter(id => id !== s.id);
                    else selectedStrategies.push(s.id);
                    renderStrategies();
                };
                c.appendChild(btn);
            });
        }

        function handleFileSelect(e) {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = ev => {
                chartFile = {url: ev.target.result, name: file.name};
                document.getElementById('upload-content').classList.add('hidden');
                document.getElementById('preview-content').classList.remove('hidden');
                document.getElementById('chart-preview').src = chartFile.url;
                
                const header = document.getElementById('chart-preview-header');
                document.getElementById('header-thumb').src = chartFile.url;
                document.getElementById('header-filename').textContent = chartFile.name;
                header.classList.remove('hidden');
            };
            reader.readAsDataURL(file);
        }

        function clearChart() {
            chartFile = null;
            document.getElementById('upload-content').classList.remove('hidden');
            document.getElementById('preview-content').classList.add('hidden');
            document.getElementById('chart-preview-header').classList.add('hidden');
        }

        function showError(msg) {
            const el = document.getElementById('error-msg');
            el.textContent = msg;
            el.classList.remove('hidden');
            setTimeout(() => el.classList.add('hidden'), 4000);
        }

        function generateMockSignal() {
            const isBuy = Math.random() > 0.45;
            const conf = Math.floor(68 + Math.random() * 28);
            return {
                signal: isBuy ? "BUY" : "SELL",
                bias: isBuy ? "BULLISH" : "BEARISH",
                entry_price: "Current price / FVG reclaim",
                stop_loss: isBuy ? "Below sweep low" : "Above recent high",
                take_profit_1: "1:2 RR - Next liquidity",
                take_profit_2: "1:3.5 RR",
                take_profit_3: "Runner to HTF",
                risk_reward: isBuy ? "1:3.8" : "1:4.2",
                confidence: conf,
                strategy_used: "ICT Confluence",
                reasoning: `Excellent ${isBuy ? "bullish" : "bearish"} structure on ${currentTimeframe}. Price respected key ${selectedStrategies.includes("fvg") ? "FVG" : "order block"}. Strong displacement visible.`,
                invalidation: "Loss of structure / break of opposite liquidity",
                chart_observations: "Clear FVG + displacement + liquidity sweep confluence."
            };
        }

        function generateSignal() {
            if (selectedStrategies.length === 0) return showError("Select at least one strategy");

            const btn = document.getElementById('generate-btn');
            btn.disabled = true;
            btn.innerHTML = '🔍 ANALYZING...';

            setTimeout(() => {
                const sig = generateMockSignal();
                sig.pair = currentPair;
                sig.timeframe = currentTimeframe;

                const container = document.getElementById('signal-container');
                container.innerHTML = `
                    <div class="bg-[#0d1b2a] border-l-4 border-l-[#00ff88] rounded-3xl p-8">
                        <div class="flex justify-between items-start">
                            <div>
                                <span class="inline-block px-8 py-2 rounded-3xl text-xl font-bold bg-[#00ff8822] text-[#00ff88]">${sig.signal}</span>
                                <span class="ml-4 text-sm">${sig.bias}</span>
                            </div>
                            <div class="text-right">
                                <div class="text-xs opacity-60">R:R</div>
                                <div class="text-4xl font-black text-[#00ff88]">${sig.risk_reward}</div>
                            </div>
                        </div>
                        <div class="mt-8 grid grid-cols-2 gap-4">
                            <div class="bg-[#07121d] p-5 rounded-2xl"><div class="text-xs opacity-60">ENTRY</div><div class="font-medium">${sig.entry_price}</div></div>
                            <div class="bg-[#07121d] p-5 rounded-2xl"><div class="text-xs opacity-60">STOP LOSS</div><div class="text-red-400">${sig.stop_loss}</div></div>
                        </div>
                        <div class="mt-6 text-sm leading-relaxed bg-[#07121d] p-6 rounded-2xl">${sig.reasoning}</div>
                        ${chartFile ? `<img src="${chartFile.url}" class="mt-8 w-full rounded-2xl border border-[#1a2d40]">` : ''}
                    </div>
                `;
                container.scrollIntoView({behavior: "smooth"});

                signalsHistory.unshift(sig);
                if (signalsHistory.length > 6) signalsHistory.pop();
                renderHistory();

                btn.disabled = false;
                btn.innerHTML = '⚡ GENERATE SIGNAL';
            }, 1200);
        }

        function renderHistory() {
            const container = document.getElementById('history-list');
            container.innerHTML = '';
            document.getElementById('history-section').classList.remove('hidden');

            signalsHistory.slice(1).forEach(sig => {
                const div = document.createElement('div');
                div.className = "flex justify-between bg-[#07121d] rounded-2xl p-5 border border-[#0d1b2a]";
                div.innerHTML = `
                    <div class="flex items-center gap-4">
                        <span class="px-5 py-1 rounded-3xl text-sm font-bold ${sig.signal==='BUY'?'bg-[#00ff8822] text-[#00ff88]':'bg-[#ff456022] text-[#ff4560]'}">${sig.signal}</span>
                        <div>${sig.pair} ${sig.timeframe}</div>
                    </div>
                    <div class="text-right">
                        <div class="font-black text-xl">${sig.risk_reward}</div>
                        <div class="text-xs text-[#00ff88]">${sig.confidence}%</div>
                    </div>
                `;
                container.appendChild(div);
            });
        }

        function selectAllStrategies() { selectedStrategies = STRATEGIES.map(s=>s.id); renderStrategies(); }
        function clearStrategies() { selectedStrategies = []; renderStrategies(); }

        // Init
        window.onload = () => {
            renderPairs();
            renderTimeframes();
            renderStrategies();
        };
    </script>
</body>
</html>
