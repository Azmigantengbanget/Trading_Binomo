<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulator Trading (Versi SVG Final)</title>
    
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');
        :root {
            --bg-dark: #131722;
            --bg-panel: #1e222d;
            --border-color: #2a2e39;
            --text-primary: #d1d4dc;
            --text-secondary: #8a91a0;
            --color-green: #26a69a;
            --color-red: #ef5350;
            --color-yellow: #f9a825;
            --color-win: rgba(38, 166, 154, 0.9);
            --color-loss: rgba(239, 83, 80, 0.9);
            --color-bet-up-bg: rgba(38, 166, 154, 0.2); 
            --color-bet-down-bg: rgba(239, 83, 80, 0.2); 
        }
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            font-family: 'Roboto', sans-serif;
            background-color: var(--bg-dark);
            color: var(--text-primary);
            overflow: hidden;
        }
        .trading-platform { display: flex; width: 100vw; height: 100vh; position: relative; }
        #notification {
            position: absolute; top: 50%; left: 50%;
            transform: translate(-50%, -50%);
            padding: 20px 40px; border-radius: 8px;
            color: white; font-size: 24px; font-weight: 700; text-align: center;
            z-index: 1000; opacity: 1; transition: opacity 0.5s ease-out;
            pointer-events: none;
        }
        #notification.hidden { opacity: 0; }
        #notification.win { background-color: var(--color-win); box-shadow: 0 0 20px var(--color-win); }
        #notification.loss { background-color: var(--color-loss); box-shadow: 0 0 20px var(--color-loss); }
        #notification p { margin: 0; }
        #notification span { font-size: 16px; font-weight: 400; display: block; margin-top: 5px; }
        .left-toolbar {
            width: 60px; background-color: var(--bg-panel);
            border-right: 1px solid var(--border-color);
            padding-top: 20px; display: flex; flex-direction: column; align-items: center;
        }
        .toolbar-item {
            color: var(--text-secondary); text-decoration: none;
            font-size: 22px; margin-bottom: 25px;
        }
        .toolbar-item.active, .toolbar-item:hover { color: var(--color-yellow); }
        .main-content { flex: 1; display: flex; flex-direction: column; }
        .main-header {
            display: flex; align-items: center; padding: 10px 20px;
            background-color: var(--bg-panel); border-bottom: 1px solid var(--border-color);
        }
        .asset-selector .fa-coins { color: var(--color-yellow); margin-right: 8px; }
        .payout-badge {
            background-color: hsla(0,0%,100%,.1); color: var(--color-green);
            font-size: 12px; font-weight: 700; padding: 3px 6px;
            border-radius: 4px; margin-left: 10px;
        }
        .account-info { margin-left: auto; text-align: right; }
        .account-type { font-size: 12px; color: var(--text-secondary); display: block;}
        .balance { font-size: 16px; font-weight: 700; }
        .deposit-button {
            background-color: var(--color-yellow); color: #000;
            border: none; border-radius: 4px; padding: 8px 16px;
            font-weight: 700; margin-left: 20px; cursor: pointer;
        }
        
        #chart-container {
            flex: 1;
            position: relative;
            background-color: var(--bg-dark);
            overflow: hidden; 
            border-left: 1px solid var(--border-color);
            cursor: grab; 
        }
        #chart-container.dragging {
            cursor: grabbing; 
        }

        #candlestick-chart {
            width: 100%;
            height: 100%;
        }

        /* Styles for the custom yellow dot/label indicator on the candle */
        .bet-candle-dot {
            fill: var(--color-yellow);
            stroke: var(--bg-dark); 
            stroke-width: 1;
            pointer-events: none;
        }

        .bet-label-group {
            pointer-events: none; 
            z-index: 20; 
        }
        .bet-label-bg {
            /* fill will be set dynamically based on direction */
            rx: 3; 
            ry: 3;
        }
        .bet-label-bg.up {
            fill: var(--color-green); /* Green for UP bets */
        }
        .bet-label-bg.down {
            fill: var(--color-red); /* Red for DOWN bets */
        }

        .bet-label-text {
            font-family: 'Roboto', sans-serif;
            font-size: 10px;
            font-weight: 500;
            fill: white; /* White text on colored background */
            text-anchor: start; 
            white-space: pre; 
        }


        #current-price-indicator {
            position: absolute;
            height: 1px;
            background-color: var(--color-yellow);
            right: 0;
            transition: top 0.2s linear;
            z-index: 10;
            pointer-events: none;
        }
        #current-price-label {
            position: absolute;
            right: 0;
            background-color: var(--color-yellow);
            color: var(--bg-dark);
            padding: 1px 4px;
            font-size: 12px;
            border-radius: 4px 0 0 4px;
            transition: top 0.2s linear;
            z-index: 10;
            transform: translateY(-50%);
            pointer-events: none;
        }
        .main-footer {
            display: flex; justify-content: space-between; align-items: center;
            padding: 8px 20px; background-color: var(--bg-panel);
            font-size: 12px; color: var(--text-secondary);
            border-top: 1px solid var(--border-color);
            border-left: 1px solid var(--border-color);
        }
        .right-panel {
            width: 280px; background-color: var(--bg-panel);
            border-left: 1px solid var(--border-color); padding: 15px;
            display: flex; flex-direction: column;
        }
        .trade-info-header {
            display: flex; justify-content: space-between; margin-bottom: 20px;
            border-bottom: 1px solid var(--border-color); padding-bottom: 15px;
        }
        .info-box { text-align: center; flex: 1; }
        .info-box .label { font-size: 12px; color: var(--text-secondary); display: block; }
        .info-box .value { 
            font-size: 14px; 
            font-weight: 500; 
            color: var(--text-primary); 
        }
        .info-box .value.profit { color: var(--color-green); } 
        .info-box .value.loss { color: var(--color-red); } 

        .trade-control-box { margin-bottom: 15px; }
        .trade-control-box label { font-size: 14px; color: var(--text-secondary); margin-bottom: 8px; display: block; }
        .input-group {
            display: flex; align-items: center; background-color: var(--bg-dark);
            border: 1px solid var(--border-color); border-radius: 4px;
        }
        .btn-adjust {
            width: 40px; height: 40px; background: none; border: none;
            color: var(--text-primary); font-size: 20px; cursor: pointer;
        }
        .input-group input {
            flex: 1; background: none; border: none; color: var(--text-primary);
            text-align: center; font-size: 16px; font-weight: 500;
            width: 100%; -moz-appearance: textfield;
        }
        input::-webkit-outer-spin-button, input::-webkit-inner-spin-button { -webkit-appearance: none; margin: 0; }
        .currency-symbol { padding-left: 10px; color: var(--text-secondary); }
        .payout-summary {
            display: flex; justify-content: space-between; align-items: center;
            background-color: hsla(0,0%,100%,.05); padding: 10px; border-radius: 4px;
        }
        .payout-value { font-weight: 700; color: var(--text-primary); }
        .payout-percent { color: var(--color-green); font-weight: 700; }
        .majority-opinion { margin-top: auto; }
        .opinion-bar {
            display: flex; height: 25px; border-radius: 4px;
            overflow: hidden; font-size: 12px; font-weight: 700;
        }
        .bar-up {
            background-color: var(--color-green); display: flex;
            align-items: center; justify-content: center; color: white;
        }
        .bar-down {
            background-color: var(--color-red); display: flex;
            align-items: center; justify-content: center; color: white;
        }
        .action-button {
            width: 100%; height: 50px; border: none; border-radius: 4px;
            font-size: 24px; color: white; cursor: pointer;
            transition: transform 0.1s, background-color 0.2s;
        }
        .action-button:disabled { background-color: #555 !important; cursor: not-allowed; }
        .action-button.up { background-color: var(--color-green); margin-top: 15px; margin-bottom: 10px; }
        .action-button.down { background-color: var(--color-red); }
        .action-button:active:not(:disabled) { transform: scale(0.98); }

        /* Styles for the scrolling notification box */
        #trade-notification-box {
            background-color: var(--bg-dark);
            border: 1px solid var(--border-color);
            border-radius: 4px;
            height: 180px; 
            overflow-y: auto; 
            margin-top: 15px; 
            padding: 5px;
            display: flex; 
            flex-direction: column-reverse; 
            gap: 5px; 
            font-size: 12px;
            color: var(--text-primary);
        }

        /* Hide scrollbar for aesthetics (optional) */
        #trade-notification-box::-webkit-scrollbar {
            width: 0px;  
            background: transparent; 
        }
        #trade-notification-box {
            -ms-overflow-style: none;  
            scrollbar-width: none; 
        }

        .trade-notification-item {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 8px 10px;
            border-radius: 4px;
            transition: background-color 0.2s ease, opacity 0.3s ease;
            flex-shrink: 0; 
        }
        .trade-notification-item.up {
            background-color: var(--color-bet-up-bg); 
        }
        .trade-notification-item.down {
            background-color: var(--color-bet-down-bg); 
        }
        .trade-notification-item.completed {
            opacity: 0.7; 
            background-color: hsla(0,0%,100%,.05); 
            color: var(--text-secondary);
        }
        .trade-notification-item.completed.win {
            background-color: var(--color-win); 
        }
        .trade-notification-item.completed.loss {
            background-color: var(--color-loss); 
        }

        .trade-notification-item .info {
            display: flex;
            gap: 5px;
        }
        .trade-notification-item .time-left {
            font-weight: 700;
        }
    </style>
</head>
<body>

    <div class="trading-platform">
        <div id="notification" class="hidden"></div>
        <div class="left-toolbar">
            <a href="#" class="toolbar-item active"><i class="fa-solid fa-chart-line"></i></a>
            <a href="#" class="toolbar-item"><i class="fa-solid fa-clock-rotate-left"></i></a>
            <a href="#" class="toolbar-item"><i class="fa-solid fa-trophy"></i></a>
            <a href="#" class="toolbar-item"><i class="fa-solid fa-gift"></i></a>
            <a href="#" class="toolbar-item"><i class="fa-solid fa-circle-info"></i></a>
        </div>
        <div class="main-content">
            <div class="main-header">
                <div class="asset-selector">
                    <i class="fa-solid fa-coins"></i><span>Crypto IDX</span><span class="payout-badge">83%</span>
                </div>
                <div class="account-info">
                    <span class="account-type">Akun asli</span><span id="balance-display" class="balance"></span>
                </div>
                <button class="deposit-button">Deposit</button>
            </div>
            
            <div id="chart-container">
                <svg id="candlestick-chart"></svg>
                <div id="current-price-indicator">
                    <div id="current-price-label"></div>
                </div>
            </div>

            <div class="main-footer">
                <div id="current-time" class="time-display"></div>
            </div>
        </div>
        <div class="right-panel">
            <div class="trade-info-header">
                <div class="info-box"><span class="label">Total Investasi</span><span class="value" id="active-investment-display">Rp 0,00</span></div>
                <div class="info-box"><span class="label">Pendapatan</span><span class="value" id="active-payout-display">Rp 0,00</span></div>
                <div class="info-box"><span class="label">Waktu tersisa</span><span class="value" id="active-timer-display">00:00</span></div>
            </div>
            <div class="trade-control-box">
                <label>Jumlah</label>
                <div class="input-group">
                    <button id="investment-minus" class="btn-adjust">-</button><span class="currency-symbol">Rp</span>
                    <input type="text" id="investment-amount" value="14.000"><button id="investment-plus" class="btn-adjust">+</button>
                </div>
            </div>
            <div class="trade-control-box">
                <label>Waktu</label>
                <div class="input-group">
                    <button id="time-minus" class="btn-adjust">-</button>
                    <input type="text" id="trade-time" value="00:30" readonly><button id="time-plus" class="btn-adjust">+</button>
                </div>
            </div>
            <div class="trade-control-box payout-summary">
                <label>Pendapatan</label><span id="payout-preview"></span><span class="payout-percent">+83%</span>
            </div>
            
            <div id="trade-notification-box">
                </div>

            <div class="majority-opinion">
                <label>Pendapat mayoritas</label>
                <div class="opinion-bar">
                    <div id="opinion-up" class="bar-up">55%</div><div id="opinion-down" class="bar-down">45%</div>
                </div>
            </div>
            <button id="btn-up" class="action-button up"><i class="fa-solid fa-arrow-up"></i></button>
            <button id="btn-down" class="action-button down"><i class="fa-solid fa-arrow-down"></i></button>
        </div>
    </div>

    <script>
        window.onload = function() {
            // --- CONSTANTS & VARIABLES ---
            const PAYOUT_RATE = 0.83;
            // Updated INITIAL_BALANCE to 1 Miliar (1.000.000.000)
            const INITIAL_BALANCE = 1000000000; 
            // Added 15 seconds to TRADE_DURATIONS
            const TRADE_DURATIONS = [15, 30, 60, 300]; 
            // Default currentDurationIndex to 1 (30 seconds)
            let currentDurationIndex = 1; 
            const CANDLE_INTERVAL = 5000; // New candle every 5 seconds
            const MAX_ACTIVE_BETS = 10; // Maximum number of active bets
            const MAX_NOTIFICATION_ITEMS = 10; // Max items in the scrolling notification box

            let userBalance = INITIAL_BALANCE;
            let activeTrades = []; 
            
            let candleDataHistory = [];
            let currentPrice = 50000; 
            let lastCandleTime = 0;

            // SVG Chart Variables
            const svgChart = document.getElementById('candlestick-chart');
            const svgNS = "http://www.w3.org/2000/svg"; 
            const CHART_HEIGHT = 500; 
            const CHART_WIDTH = 1200; 
            const MIN_PRICE = 49500;  // Original MIN_PRICE for candle mapping
            const MAX_PRICE = 50500;  // Original MAX_PRICE for candle mapping
            const VOLATILITY = 40;    
            const CANDLE_WIDTH = 10;
            const CANDLE_MARGIN = 2;
            const TOTAL_CANDLE_WIDTH = CANDLE_WIDTH + CANDLE_MARGIN;
            const INITIAL_CANDLE_COUNT = Math.floor(CHART_WIDTH / TOTAL_CANDLE_WIDTH) * 2; // Twice visible width for initial pan freedom

            // --- Variables for Chart Panning ---
            let isDragging = false;
            let startDragClientX;
            let startDragClientY;
            // currentViewBox: [minX, minY, width, height] of the currently visible SVG area
            const initialYCenterSvg = mapPriceToY(currentPrice); 
            const initialViewBoxY = initialYCenterSvg - (CHART_HEIGHT / 2); 
            let currentViewBox = [0, initialViewBoxY, CHART_WIDTH, CHART_HEIGHT]; 

            // --- DOM Elements ---
            const notification = document.getElementById('notification');
            const balanceDisplay = document.getElementById('balance-display');
            const priceIndicator = document.getElementById('current-price-indicator');
            const priceLabel = document.getElementById('current-price-label');
            const chartContainer = document.getElementById('chart-container');
            const investmentInput = document.getElementById('investment-amount');
            const timeInput = document.getElementById('trade-time');
            const payoutPreview = document.getElementById('payout-preview');
            const btnUp = document.getElementById('btn-up');
            const btnDown = document.getElementById('btn-down');
            const activePayoutDisplay = document.getElementById('active-payout-display');
            const tradeNotificationBox = document.getElementById('trade-notification-box'); 

            let tradeNotifications = []; 

            // --- HELPER FUNCTIONS FOR NUMBER FORMATTING AND PARSING ---
            
            // Formats a number with dots as thousands separators (e.g., 14000 -> 14.000)
            function formatNumberWithDots(num) {
                // Ensure num is a valid number, default to 0 if not
                const number = typeof num === 'number' && !isNaN(num) ? num : 0;
                return new Intl.NumberFormat('id-ID').format(Math.round(number));
            }

            // Parses a formatted number string (e.g., "14.000") back to a float (e.g., 14000)
            function parseFormattedNumber(str) {
                if (typeof str !== 'string') return str; // Return as is if not a string
                // Remove all dots from the string before parsing
                const cleanedStr = str.replace(/\./g, '');
                const parsed = parseFloat(cleanedStr);
                return isNaN(parsed) ? 0 : parsed; // Return 0 if parsing results in NaN
            }

            // Formats a number as currency (e.g., 14000 -> Rp 14.000)
            function formatCurrency(amount) {
                return `Rp ${formatNumberWithDots(amount)}`;
            }

            // --- CORE CHART FUNCTIONS ---
            
            // Maps a price value to a Y-coordinate on the SVG chart.
            // This mapping is fixed relative to MIN_PRICE and MAX_PRICE.
            function mapPriceToY(price) {
                const priceRange = MAX_PRICE - MIN_PRICE;
                // Ensure price is within the defined MIN_PRICE and MAX_PRICE before mapping
                const clampedPrice = Math.max(MIN_PRICE, Math.min(MAX_PRICE, price));
                const percentage = (clampedPrice - MIN_PRICE) / priceRange;
                return CHART_HEIGHT - (percentage * CHART_HEIGHT); 
            }
            
            // Renders all candlesticks on the SVG chart
            function renderChart() {
                // Clear only candle elements, keep bet indicators as they are managed separately
                const elementsToRemove = Array.from(svgChart.children).filter(el => 
                    !el.classList.contains('bet-candle-dot') &&
                    !el.classList.contains('bet-label-group')
                );
                elementsToRemove.forEach(el => el.remove());

                // Iterate through candle data history and render only those visible in the current viewBox
                candleDataHistory.forEach((data, i) => {
                    const candleXPosition = i * TOTAL_CANDLE_WIDTH;

                    // Check if the candle's X range is within the visible viewBox X range
                    if (candleXPosition + TOTAL_CANDLE_WIDTH > currentViewBox[0] && 
                        candleXPosition < currentViewBox[0] + currentViewBox[2]) {
                        
                        const isUp = data.close >= data.open;
                        const color = isUp ? 'var(--color-green)' : 'var(--color-red)';

                        // Draw Wick (vertical line)
                        const wick = document.createElementNS(svgNS, 'rect');
                        wick.setAttribute('x', candleXPosition + (CANDLE_WIDTH / 2 - 1));
                        wick.setAttribute('y', mapPriceToY(data.high));
                        wick.setAttribute('width', 2);
                        wick.setAttribute('height', Math.abs(mapPriceToY(data.low) - mapPriceToY(data.high)));
                        wick.setAttribute('fill', color);
                        svgChart.appendChild(wick);

                        // Draw Body (rectangle)
                        const body = document.createElementNS(svgNS, 'rect');
                        body.setAttribute('x', candleXPosition);
                        body.setAttribute('y', mapPriceToY(Math.max(data.open, data.close)));
                        body.setAttribute('width', CANDLE_WIDTH);
                        body.setAttribute('height', Math.abs(mapPriceToY(data.open) - mapPriceToY(data.close)) || 1); // Min height 1px
                        body.setAttribute('fill', color);
                        svgChart.appendChild(body);
                    }
                });

                // Set the SVG viewBox based on current pan position
                svgChart.setAttribute('viewBox', currentViewBox.join(' '));

                // Always re-render bet indicators after chart redraw (their positions depend on viewBox)
                renderBetIndicators(); 
            }

            // Generates a new candlestick data point and adds it to history
            function generateCandleData() {
                const open = candleDataHistory.length > 0 ? candleDataHistory[candleDataHistory.length - 1].close : currentPrice;
                let price = open;
                let high = open;
                let low = open;

                for (let i = 0; i < 10; i++) { // Generate 10 micro-steps for price change
                    price += (Math.random() - 0.5) * VOLATILITY;
                    // Clamp price within the fixed MIN_PRICE and MAX_PRICE bounds
                    price = Math.max(MIN_PRICE, Math.min(MAX_PRICE, price)); 
                    high = Math.max(high, price);
                    low = Math.min(low, price);
                }
                
                high = Math.max(MIN_PRICE, Math.min(MAX_PRICE, high)); 
                low = Math.max(MIN_PRICE, Math.min(MAX_PRICE, low)); 

                currentPrice = price; 
                candleDataHistory.push({ open, high, low, close: currentPrice });

                // No auto-scroll of viewBox here for "infinite" pan.
                // New candles will simply extend the history to the right.
                
                renderChart(); 
            }

            // Fills the chart with initial candles to avoid empty space
            function prefillChartWithCandles() {
                for (let i = 0; i < INITIAL_CANDLE_COUNT; i++) { 
                    generateCandleData(); 
                }
                // Initial viewBox X: show the most recent candles, so the right edge of viewbox
                // aligns with the rightmost candle.
                currentViewBox[0] = (candleDataHistory.length * TOTAL_CANDLE_WIDTH) - CHART_WIDTH;
                // Ensure it doesn't go negative if INITIAL_CANDLE_COUNT is small
                currentViewBox[0] = Math.max(0, currentViewBox[0]); 
                
                lastCandleTime = Date.now(); 
                renderChart(); 
            }


            // --- GAME LOOP & TRADING LOGIC ---
            function mainLoop() {
                const now = Date.now();
                if (now - lastCandleTime >= CANDLE_INTERVAL) {
                    lastCandleTime = now;
                    generateCandleData(); 
                }
                
                checkCompletedTrades();
                updateAllDisplays(); 
                
                // Update position of the current price indicator (HTML element)
                const chartHeightPx = chartContainer.clientHeight;
                const priceYInViewBox = mapPriceToY(currentPrice);
                // Convert SVG Y-coordinate to client pixel Y-coordinate considering current pan
                const normalizedY = (priceYInViewBox - currentViewBox[1]) / currentViewBox[3]; 
                const yPosPx = normalizedY * chartHeightPx;

                priceIndicator.style.top = `${yPosPx}px`;
                priceIndicator.style.width = chartContainer.clientWidth + 'px';
                priceLabel.style.top = `${yPosPx}px`;
                priceLabel.textContent = Math.round(currentPrice);

                updateBetIndicatorsPositions(); // Update positions for dots/labels
                updateNotificationCountdowns(); 
            }
            
            // Handles placing a new trade
            function openTrade(direction) {
                // Get the current investment amount, properly parsed (removing dots)
                const investment = parseFormattedNumber(investmentInput.value); 

                if (activeTrades.length >= MAX_ACTIVE_BETS) {
                    showNotification(`Maksimal ${MAX_ACTIVE_BETS} taruhan aktif`, 'loss');
                    return;
                }

                // Check for minimum investment after parsing
                if (isNaN(investment) || investment < 14000) { showNotification("Investasi minimum Rp 14.000", 'loss'); return; }
                if (investment > userBalance) { showNotification("Saldo tidak cukup", 'loss'); return; }

                userBalance -= investment;
                const duration = TRADE_DURATIONS[currentDurationIndex];
                
                const currentCandleIndex = candleDataHistory.length - 1; 

                const newTrade = {
                    id: Date.now() + Math.random(), 
                    investment, direction, duration,
                    startPrice: currentPrice,
                    expiryTime: Date.now() + duration * 1000,
                    candleIndex: currentCandleIndex, 
                    dotElement: null,           
                    labelGroupElement: null,    
                    labelBgElement: null,       
                    labelTextElement: null,     
                    notificationElement: null   
                };
                activeTrades.push(newTrade);

                addTradeNotification(newTrade); 
                renderBetIndicators(); // Re-render to show new bet indicator on chart
                updateAllDisplays(); // Update balance and button states immediately after placing bet
            }

            // Checks for and processes completed trades
            function checkCompletedTrades() {
                const now = Date.now();
                const completedTrades = [];
                const remainingTrades = [];

                activeTrades.forEach(trade => {
                    if (now >= trade.expiryTime) {
                        completedTrades.push(trade);
                    } else {
                        remainingTrades.push(trade);
                    }
                });

                activeTrades = remainingTrades; 

                completedTrades.forEach(trade => {
                    const finalPrice = currentPrice;
                    let status = 'loss';
                    let amount = trade.investment;

                    if (trade.direction === 'up' && finalPrice > trade.startPrice) status = 'win';
                    else if (trade.direction === 'down' && finalPrice < trade.startPrice) status = 'win';
                    
                    if (status === 'win') {
                        const winnings = trade.investment * (1 + PAYOUT_RATE);
                        userBalance += winnings;
                        amount = winnings;
                    }
                    
                    showNotification(status, amount);

                    // Remove SVG elements for completed trades
                    if (trade.dotElement) trade.dotElement.remove(); 
                    if (trade.labelGroupElement) trade.labelGroupElement.remove(); 

                    // Update notification item on completion
                    if (trade.notificationElement) {
                        trade.notificationElement.classList.add('completed');
                        trade.notificationElement.classList.remove('up', 'down'); 
                        if (status === 'win') {
                            trade.notificationElement.classList.add('win');
                        } else {
                            trade.notificationElement.classList.add('loss');
                        }
                        const infoSpan = trade.notificationElement.querySelector('.info span:last-child');
                        if (infoSpan) {
                            infoSpan.textContent = formatCurrency(amount); 
                        }
                        const timeLeftSpan = trade.notificationElement.querySelector('.time-left');
                        if (timeLeftSpan) {
                            timeLeftSpan.textContent = status.toUpperCase(); 
                        }
                    }
                });
                tradeNotifications = tradeNotifications.filter(tn => activeTrades.includes(tn) || tn.completed);
                renderTradeNotifications(); 
                updateAllDisplays(); // Update balance and button states after trade completion
            }

            // Renders/updates all active trade indicators on the SVG chart
            function renderBetIndicators() {
                // Remove all existing bet indicator SVG elements
                svgChart.querySelectorAll('.bet-candle-dot, .bet-label-group').forEach(el => el.remove());

                activeTrades.forEach(trade => {
                    // Only render if the candle associated with the trade is in history
                    if (trade.candleIndex !== undefined && candleDataHistory[trade.candleIndex]) {
                        const candleData = candleDataHistory[trade.candleIndex];
                        const candleX = trade.candleIndex * TOTAL_CANDLE_WIDTH;
                        const dotX = candleX + (CANDLE_WIDTH / 2);
                        const dotY = mapPriceToY(trade.startPrice);

                        // Condition to render dot/label only if it's within the visible portion of the chart
                        const labelApproxWidth = 80; 
                        const labelApproxHeight = 20;

                        if (candleX + TOTAL_CANDLE_WIDTH > currentViewBox[0] && 
                            candleX < currentViewBox[0] + currentViewBox[2] &&
                            dotY > currentViewBox[1] - labelApproxHeight && 
                            dotY < currentViewBox[1] + currentViewBox[3] + labelApproxHeight) 
                        {
                            let dotElement = document.createElementNS(svgNS, 'circle');
                            dotElement.setAttribute('class', 'bet-candle-dot');
                            dotElement.setAttribute('cx', dotX);
                            dotElement.setAttribute('cy', dotY);
                            dotElement.setAttribute('r', 3); 
                            svgChart.appendChild(dotElement);
                            trade.dotElement = dotElement;

                            let labelGroup = document.createElementNS(svgNS, 'g');
                            labelGroup.setAttribute('class', 'bet-label-group');
                            svgChart.appendChild(labelGroup);
                            trade.labelGroupElement = labelGroup;

                            let labelBg = document.createElementNS(svgNS, 'rect');
                            labelBg.setAttribute('class', `bet-label-bg ${trade.direction}`); // Set class for color based on direction
                            labelGroup.appendChild(labelBg);
                            trade.labelBgElement = labelBg;

                            let labelText = document.createElementNS(svgNS, 'text');
                            labelText.setAttribute('class', 'bet-label-text');
                            labelGroup.appendChild(labelText);
                            trade.labelTextElement = labelText;

                            // Set text to only nominal amount
                            labelText.textContent = formatCurrency(trade.investment); 
                            
                            // Get actual text dimensions for background rect
                            const textBBox = labelText.getBBox(); 

                            const labelXOffset = 5; 
                            const labelYOffset = - (textBBox.height / 2) - 2; 

                            labelBg.setAttribute('x', dotX + labelXOffset); 
                            labelBg.setAttribute('y', dotY + labelYOffset); 
                            labelBg.setAttribute('width', textBBox.width + 10); 
                            labelBg.setAttribute('height', textBBox.height + 4); 

                            labelText.setAttribute('x', dotX + labelXOffset + 5); 
                            labelText.setAttribute('y', dotY + labelYOffset + textBBox.height - 2); 
                        } else {
                            // If indicators are outside view, ensure their elements are null so they are not processed
                            if (trade.dotElement) { trade.dotElement.remove(); trade.dotElement = null; }
                            if (trade.labelGroupElement) { trade.labelGroupElement.remove(); trade.labelGroupElement = null; }
                            trade.labelBgElement = null;
                            trade.labelTextElement = null;
                        }
                    }
                });
            }

            // Updates positions and text content of bet indicators for those currently rendered
            function updateBetIndicatorsPositions() {
                activeTrades.forEach(trade => {
                    // Only update if elements exist (i.e., they are currently rendered)
                    if (trade.dotElement && trade.labelGroupElement && trade.labelTextElement && trade.labelBgElement) {
                        const candleX = trade.candleIndex * TOTAL_CANDLE_WIDTH;
                        const dotX = candleX + (CANDLE_WIDTH / 2);
                        const dotY = mapPriceToY(trade.startPrice);

                        trade.dotElement.setAttribute('cx', dotX);
                        trade.dotElement.setAttribute('cy', dotY);

                        // Update label text to only nominal amount (no countdown)
                        trade.labelTextElement.textContent = formatCurrency(trade.investment);

                        const textBBox = trade.labelTextElement.getBBox(); 
                        const labelXOffset = 5; 
                        const labelYOffset = - (textBBox.height / 2) - 2; 

                        trade.labelBgElement.setAttribute('x', dotX + labelXOffset);
                        trade.labelBgElement.setAttribute('y', dotY + labelYOffset);
                        trade.labelBgElement.setAttribute('width', textBBox.width + 10); 
                        trade.labelBgElement.setAttribute('height', textBBox.height + 4); 

                        trade.labelTextElement.setAttribute('x', dotX + labelXOffset + 5);
                        trade.labelTextElement.setAttribute('y', dotY + labelYOffset + textBBox.height - 2);
                    }
                });
            }


            // --- Functions for Trade Notification Box ---
            function addTradeNotification(trade) {
                tradeNotifications.push(trade);

                if (tradeNotifications.length > MAX_NOTIFICATION_ITEMS) {
                    const oldestCompletedIndex = tradeNotifications.findIndex(tn => tn.completed);
                    if (oldestCompletedIndex !== -1) {
                         tradeNotifications.splice(oldestCompletedIndex, 1); 
                    } else {
                         tradeNotifications.shift(); 
                    }
                }
                renderTradeNotifications(); 
            }

            function renderTradeNotifications() {
                tradeNotificationBox.innerHTML = ''; 

                tradeNotifications.slice().reverse().forEach(trade => { 
                    let item = trade.notificationElement;
                    if (!item) {
                        item = document.createElement('div');
                        item.className = `trade-notification-item ${trade.direction}`;
                        item.innerHTML = `
                            <div class="info">
                                <i class="fa-solid fa-arrow-${trade.direction}"></i>
                                <span>Rp ${formatCurrency(trade.investment)}</span>
                            </div>
                            <span class="time-left">${formatTime(trade.duration)}</span>
                        `;
                        trade.notificationElement = item;
                    }
                    
                    if (trade.completed) { 
                        item.classList.add('completed');
                        item.classList.remove('up', 'down'); 
                        item.classList.add(trade.status); 
                        item.querySelector('.time-left').textContent = trade.status.toUpperCase(); 
                        item.querySelector('.info span:last-child').textContent = formatCurrency(trade.finalAmount); 
                    } else {
                        item.classList.remove('completed', 'win', 'loss');
                        item.classList.add(trade.direction); 
                        item.querySelector('.info span:last-child').textContent = formatCurrency(trade.investment);
                        item.querySelector('.time-left').textContent = formatTime(Math.max(0, Math.round((trade.expiryTime - Date.now()) / 1000)));
                    }
                    
                    tradeNotificationBox.appendChild(item);
                });
                tradeNotificationBox.scrollTop = tradeNotificationBox.scrollHeight;
            }

            // Updates countdown text for notification items
            function updateNotificationCountdowns() {
                tradeNotifications.forEach(trade => {
                    if (!trade.completed && trade.notificationElement) {
                        const remainingTime = Math.max(0, Math.round((trade.expiryTime - Date.now()) / 1000));
                        trade.notificationElement.querySelector('.time-left').textContent = formatTime(remainingTime);
                    }
                });
            }

            // --- HELPER & DISPLAY UPDATE FUNCTIONS ---
            function updateAllDisplays() {
                // Display user balance with proper formatting
                balanceDisplay.textContent = formatCurrency(userBalance);
                
                document.getElementById('current-time').textContent = new Date().toLocaleTimeString('en-GB', { timeZone: 'Asia/Jakarta' }) + " GMT+7";
                // Payout preview should also be formatted correctly
                payoutPreview.textContent = formatCurrency((parseFormattedNumber(investmentInput.value) || 0) * (1 + PAYOUT_RATE));
                
                updateActiveTradeInfo();

                // Get the current investment value, properly parsed (removing dots)
                const currentInvestmentValue = parseFormattedNumber(investmentInput.value);

                btnUp.disabled = (currentInvestmentValue <= 0) || (currentInvestmentValue > userBalance) || (activeTrades.length >= MAX_ACTIVE_BETS) || (currentInvestmentValue < 14000);
                btnDown.disabled = (currentInvestmentValue <= 0) || (currentInvestmentValue > userBalance) || (activeTrades.length >= MAX_ACTIVE_BETS) || (currentInvestmentValue < 14000);
            }

            function updateActiveTradeInfo() {
                const activeInvDisp = document.getElementById('active-investment-display');
                const activeTimDisp = document.getElementById('active-timer-display');

                let totalInvestment = 0;
                let potentialProfit = 0;
                let potentialLoss = 0; 
                let hasProfitTrade = false; 
                let minRemainingTime = Infinity;

                activeTrades.forEach(trade => {
                    totalInvestment += trade.investment;
                    const remaining = Math.max(0, Math.round((trade.expiryTime - Date.now()) / 1000));
                    if (remaining < minRemainingTime) {
                        minRemainingTime = remaining;
                    }

                    let isCurrentlyProfitable = false;
                    if (trade.direction === 'up' && currentPrice > trade.startPrice) {
                        isCurrentlyProfitable = true;
                    } else if (trade.direction === 'down' && currentPrice < trade.startPrice) {
                        isCurrentlyProfitable = true;
                    }

                    if (isCurrentlyProfitable) {
                        potentialProfit += trade.investment * PAYOUT_RATE; 
                        hasProfitTrade = true;
                    } else {
                        potentialLoss += trade.investment; 
                    }
                });

                activeInvDisp.textContent = formatCurrency(totalInvestment); 
                activeTimDisp.textContent = activeTrades.length > 0 ? formatTime(minRemainingTime) : "00:00";

                activePayoutDisplay.classList.remove('profit', 'loss'); 

                if (activeTrades.length > 0) {
                    if (hasProfitTrade) {
                        activePayoutDisplay.textContent = formatCurrency(potentialProfit); 
                        activePayoutDisplay.classList.add('profit');
                    } else {
                        activePayoutDisplay.textContent = formatCurrency(-potentialLoss); 
                        activePayoutDisplay.classList.add('loss');
                    }
                } else {
                    activePayoutDisplay.textContent = "Rp 0,00";
                }
            }

            function showNotification(result, amount) {
                notification.innerHTML = `<p>${result.toUpperCase()}</p><span>${formatCurrency(amount)}</span>`; 
                notification.className = result;
                setTimeout(() => { notification.className = 'hidden'; }, 3000);
            }
            // formatCurrency now only adds "Rp " prefix. Actual number formatting is done by formatNumberWithDots
            function formatCurrency(amount) {
                return `Rp ${formatNumberWithDots(amount)}`;
            }
            function formatTime(seconds) {
                const m = Math.floor(seconds / 60).toString().padStart(2, '0');
                const s = (seconds % 60).toString().padStart(2, '0');
                return `${m}:${s}`;
            }

            // --- EVENT LISTENERS ---
            document.getElementById('investment-plus').addEventListener('click', () => {
                let value = parseFormattedNumber(investmentInput.value);
                // Ensure value is a number and add 14000
                investmentInput.value = formatNumberWithDots((value || 0) + 14000); 
                updateAllDisplays();
            });
            document.getElementById('investment-minus').addEventListener('click', () => {
                let value = parseFormattedNumber(investmentInput.value);
                // Ensure value is a number and subtract 14000, min 14000
                investmentInput.value = formatNumberWithDots(Math.max(14000, (value || 0) - 14000)); 
                updateAllDisplays();
            });

            // Format investment input on change/blur
            investmentInput.addEventListener('change', (e) => {
                let value = parseFormattedNumber(e.target.value); 
                e.target.value = formatNumberWithDots(value);
                updateAllDisplays();
            });
            investmentInput.addEventListener('blur', (e) => {
                let value = parseFormattedNumber(e.target.value); 
                e.target.value = formatNumberWithDots(value);
                updateAllDisplays();
            });
            // Initial format for investment amount when page loads
            investmentInput.value = formatNumberWithDots(parseFormattedNumber(investmentInput.value));

            document.getElementById('time-plus').addEventListener('click', () => {
                currentDurationIndex = (currentDurationIndex + 1) % TRADE_DURATIONS.length;
                timeInput.value = formatTime(TRADE_DURATIONS[currentDurationIndex]);
            });
            document.getElementById('time-minus').addEventListener('click', () => {
                currentDurationIndex = (currentDurationIndex - 1 + TRADE_DURATIONS.length) % TRADE_DURATIONS.length;
                timeInput.value = formatTime(TRADE_DURATIONS[currentDurationIndex]);
            });
            btnUp.addEventListener('click', () => openTrade('up'));
            btnDown.addEventListener('click', () => openTrade('down'));
            
            // --- Chart Panning Event Listeners ---
            chartContainer.addEventListener('mousedown', (e) => {
                isDragging = true;
                startDragClientX = e.clientX;
                startDragClientY = e.clientY;
                chartContainer.classList.add('dragging'); 
            });

            chartContainer.addEventListener('mousemove', (e) => {
                if (!isDragging) return;
                e.preventDefault(); 

                const deltaClientX = e.clientX - startDragClientX;
                const deltaClientY = e.clientY - startDragClientY;

                const svgRect = svgChart.getBoundingClientRect();
                const scaleX = currentViewBox[2] / svgRect.width;
                const scaleY = currentViewBox[3] / svgRect.height;

                const deltaViewBoxX = -deltaClientX * scaleX; 
                const deltaViewBoxY = -deltaClientY * scaleY; 

                currentViewBox[0] += deltaViewBoxX;
                currentViewBox[1] += deltaViewBoxY;
                
                // No clamping of viewBox coordinates here for "infinite" pan
                
                startDragClientX = e.clientX;
                startDragClientY = e.clientY;

                renderChart(); 
            });

            chartContainer.addEventListener('mouseup', () => {
                isDragging = false;
                chartContainer.classList.remove('dragging');
            });

            chartContainer.addEventListener('mouseleave', () => {
                isDragging = false;
                chartContainer.classList.remove('dragging');
            });


            // --- INITIALIZATION & MAIN GAME LOOP ---
            svgChart.setAttribute('viewBox', currentViewBox.join(' ')); 
            svgChart.setAttribute('preserveAspectRatio', 'none'); 

            prefillChartWithCandles(); 
            updateAllDisplays(); // Call once at start to set initial formatted balance and payout
            setInterval(mainLoop, 200); 
        };
    </script>
</body>
</html>
