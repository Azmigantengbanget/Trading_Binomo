<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
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
            overflow: hidden; /* Prevent body scroll by default, handled by platform */
            -webkit-user-select: none; /* Disable text selection on touch devices */
            -moz-user-select: none;
            -ms-user-select: none;
            user-select: none;
        }
        .trading-platform {
            display: flex;
            width: 100vw;
            height: 100vh; /* Fixed height for platform, allow internal scrolling */
            position: relative;
            overflow-y: auto; /* Allow the entire platform to scroll vertically */
            overflow-x: hidden; /* Prevent horizontal scroll on the platform */
            -webkit-overflow-scrolling: touch; /* Smooth scrolling on iOS */
        }
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
            flex-shrink: 0;
            height: 100vh; /* Make toolbar full height */
            position: sticky; /* Keep it sticky */
            top: 0;
            left: 0;
        }
        .toolbar-item {
            color: var(--text-secondary); text-decoration: none;
            font-size: 22px; margin-bottom: 25px;
        }
        .toolbar-item.active, .toolbar-item:hover { color: var(--color-yellow); }
        .main-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            min-width: 0;
            height: 100vh; /* Make main content full height */
            flex-grow: 1;
        }
        .main-header {
            display: flex; align-items: center; padding: 10px 20px;
            background-color: var(--bg-panel); border-bottom: 1px solid var(--border-color);
            flex-wrap: wrap;
            gap: 10px;
            flex-shrink: 0;
        }
        .asset-selector {
            font-weight: 700; font-size: 16px; display: flex; align-items: center;
            flex-shrink: 0;
        }
        .asset-selector .fa-coins { color: var(--color-yellow); margin-right: 8px; }
        .payout-badge {
            background-color: hsla(0,0%,100%,.1); color: var(--color-green);
            font-size: 12px; font-weight: 700; padding: 3px 6px;
            border-radius: 4px; margin-left: 10px;
            flex-shrink: 0;
        }
        .account-info {
            margin-left: auto; text-align: right;
            flex-shrink: 0;
        }
        .account-type { font-size: 12px; color: var(--text-secondary); display: block;}
        .balance { font-size: 16px; font-weight: 700; }
        .deposit-button {
            background-color: var(--color-yellow); color: #000;
            border: none; border-radius: 4px; padding: 8px 16px;
            font-weight: 700; margin-left: 20px; cursor: pointer;
            flex-shrink: 0;
        }

        #chart-container {
            flex: 1;
            position: relative;
            background-color: var(--bg-dark);
            overflow: hidden; /* Chart pan/zoom is handled by SVG viewBox */
            border-left: 1px solid var(--border-color);
            cursor: grab;
            touch-action: none; /* Prevent browser touch gestures on chart area for independent pan/zoom */
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
            rx: 3;
            ry: 3;
        }
        .bet-label-bg.up {
            fill: var(--color-green);
        }
        .bet-label-bg.down {
            fill: var(--color-red);
        }

        .bet-label-text {
            font-family: 'Roboto', sans-serif;
            font-size: 10px;
            font-weight: 500;
            fill: white;
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
            flex-shrink: 0;
        }
        .right-panel {
            width: 280px; /* Default for desktop */
            background-color: var(--bg-panel);
            border-left: 1px solid var(--border-color);
            padding: 15px;
            display: flex;
            flex-direction: column;
            flex-shrink: 0;
            height: 100vh; /* Full height on desktop */
        }

        .trade-info-header {
            display: flex; justify-content: space-between; margin-bottom: 20px;
            border-bottom: 1px solid var(--border-color); padding-bottom: 15px;
            flex-wrap: wrap;
            gap: 5px;
            flex-shrink: 0;
        }
        .info-box {
            text-align: center;
            flex: 1;
            min-width: 80px;
        }
        .info-box .label { font-size: 12px; color: var(--text-secondary); display: block; }
        .info-box .value {
            font-size: 14px;
            font-weight: 500;
            color: var(--text-primary);
        }
        .info-box .value.profit { color: var(--color-green); }
        .info-box .value.loss { color: var(--color-red); }

        .trade-control-box { margin-bottom: 15px; flex-shrink: 0; }
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
            flex-shrink: 0;
        }
        .payout-value { font-weight: 700; color: var(--text-primary); }
        .payout-percent { color: var(--color-green); font-weight: 700; }
        .majority-opinion {
            padding-top: 15px;
            flex-shrink: 0;
        }
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
            flex-shrink: 0;
        }
        .action-button:disabled { background-color: #555 !important; cursor: not-allowed; }
        .action-button.up { background-color: var(--color-green); margin-top: 15px; margin-bottom: 10px; }
        .action-button.down { background-color: var(--color-red); margin-bottom: 0; }

        /* Styles for the scrolling notification box */
        #trade-notification-box {
            background-color: var(--bg-dark);
            border: 1px solid var(--border-color);
            border-radius: 4px;
            height: 180px; /* Fixed height for desktop */
            overflow-y: auto;
            margin-top: 15px;
            padding: 5px;
            display: flex;
            flex-direction: column-reverse;
            gap: 5px;
            font-size: 12px;
            color: var(--text-primary);
            flex-grow: 1;
            margin-bottom: 15px;
            flex-shrink: 1;
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

        /* --- MEDIA QUERIES FOR MOBILE LANDSCAPE --- */
        @media (max-width: 1024px) and (orientation: landscape) {
            body {
                font-size: 12px;
                overflow: hidden; /* Prevent body scroll, platform handles it */
            }
            .trading-platform {
                flex-direction: row;
                height: 100vh; /* Set platform to full viewport height */
                overflow-y: hidden; /* Platform does not scroll itself */
                overflow-x: hidden;
                align-items: stretch; /* Stretch children to fill available height */
            }
            .left-toolbar {
                width: 40px;
                padding-top: 5px;
                height: 100%;
            }
            .toolbar-item {
                font-size: 16px;
                margin-bottom: 10px;
            }
            .main-content {
                height: 100%; /* Take full height of platform */
                flex-grow: 1;
            }
            .main-header {
                padding: 5px 8px;
                font-size: 12px;
                flex-wrap: nowrap;
                overflow-x: auto;
                justify-content: flex-start;
                gap: 5px;
                flex-shrink: 0;
            }
            .main-header::-webkit-scrollbar { height: 0px; }
            .main-header { -ms-overflow-style: none; scrollbar-width: none; }

            .asset-selector { font-size: 12px; }
            .payout-badge { font-size: 9px; padding: 1px 3px; margin-left: 5px; }
            .account-info { font-size: 12px; margin-left: auto; }
            .balance { font-size: 12px; }
            .deposit-button { padding: 4px 8px; font-size: 10px; margin-left: 8px; }

            .main-footer {
                padding: 4px 8px;
                font-size: 10px;
                border-left: none;
                flex-shrink: 0;
            }

            .right-panel {
                width: 180px;
                padding: 8px;
                border-left: none;
                flex-direction: column;
                flex-shrink: 0;
                height: 100%; /* Take full height of platform */
                overflow-y: auto; /* Allow right panel to scroll independently */
                scrollbar-width: thin; /* Firefox */
                scrollbar-color: var(--border-color) var(--bg-panel); /* Firefox */
            }
            .right-panel::-webkit-scrollbar { width: 4px; }
            .right-panel::-webkit-scrollbar-track { background: var(--bg-panel); }
            .right-panel::-webkit-scrollbar-thumb { background-color: var(--border-color); border-radius: 2px; border: 1px solid var(--bg-panel); }

            .trade-info-header {
                margin-bottom: 8px;
                padding-bottom: 8px;
                flex-wrap: wrap;
                gap: 3px;
                justify-content: space-between;
                border-bottom: 1px solid var(--border-color);
                flex-shrink: 0;
                order: 1;
            }
            .info-box {
                min-width: unset;
                flex-basis: 48%;
            }
            .info-box .label { font-size: 9px; }
            .info-box .value { font-size: 11px; }

            .trade-control-box {
                margin-bottom: 8px;
                padding-top: 5px;
                flex-shrink: 0;
                order: 2;
            }
            .trade-control-box label { font-size: 11px; margin-bottom: 4px; }
            .input-group { height: 30px; }
            .btn-adjust { width: 30px; height: 30px; font-size: 16px; }
            .input-group input { font-size: 12px; }
            .currency-symbol { padding-left: 5px; }

            .payout-summary {
                padding: 6px;
                margin-bottom: 8px;
                font-size: 11px;
                flex-shrink: 0;
                order: 3;
            }

            /* --- Key layout change for mobile landscape to align buttons at bottom --- */
            #trade-notification-box {
                height: auto;
                max-height: unset; /* Remove max-height to allow full flexible grow */
                margin-top: 8px;
                margin-bottom: 8px;
                flex-grow: 1; /* Allow to take ALL available space */
                flex-shrink: 1;
                order: 4;
            }
            .trade-notification-item {
                padding: 4px 6px;
                font-size: 9px;
            }

            .majority-opinion {
                padding-top: 8px;
                margin-bottom: 8px;
                flex-shrink: 0;
                order: 5;
            }
            .majority-opinion label { font-size: 11px; }
            .opinion-bar { height: 18px; }

            .action-button {
                height: 40px;
                font-size: 18px;
                margin-top: 5px;
                margin-bottom: 0;
                flex-shrink: 0;
                order: 6;
            }
            .action-button.up { margin-bottom: 5px; }

            /* Ensure main-footer aligns with bottom of right-panel - THIS IS THE CRITICAL PART */
            .main-footer {
                align-self: flex-end; /* Align to the bottom of main-content */
                width: 100%; /* Take full width */
                border-bottom: none; /* No bottom border needed */
                border-top: 1px solid var(--border-color); /* Maintain top border */
                padding: 5px 10px; /* Adjust padding */
                /* This should push footer to the very bottom of main-content */
            }
        }
        /* Further adjustments for extremely small mobile screens (e.g., iPhone SE landscape) */
        @media (max-width: 500px) and (orientation: landscape) {
            .left-toolbar { display: none; width: 0; }
            .right-panel { width: 160px; padding: 4px; }
            .main-header { padding: 4px; gap: 3px; }
            .deposit-button { margin-left: 5px; padding: 3px 6px; }
            .asset-selector, .account-info { font-size: 10px; }
            .balance { font-size: 10px; }
            .info-box .label { font-size: 8px; }
            .info-box .value { font-size: 10px; }
            .trade-control-box label { font-size: 9px; }
            .input-group input { font-size: 10px; }
            .btn-adjust { width: 28px; height: 28px; font-size: 14px; }
            .action-button { height: 35px; font-size: 16px; }
            #trade-notification-box { max-height: 80px; }
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
            const INITIAL_BALANCE = 1000000000;
            const TRADE_DURATIONS = [15, 30, 60, 300];
            let currentDurationIndex = 1;
            const CANDLE_INTERVAL = 5000;
            const MAX_ACTIVE_BETS = 10;
            const MAX_NOTIFICATION_ITEMS = 10;

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
            const MIN_PRICE = 49500;
            const MAX_PRICE = 50500;
            const VOLATILITY = 40;
            const CANDLE_WIDTH = 10;
            const CANDLE_MARGIN = 2;
            const TOTAL_CANDLE_WIDTH = CANDLE_WIDTH + CANDLE_MARGIN;
            const INITIAL_CANDLE_COUNT = Math.floor(CHART_WIDTH / TOTAL_CANDLE_WIDTH) * 2;

            // --- Variables for Chart Panning and Zooming ---
            let isDragging = false;
            let lastPointers = new Map(); // Store active pointers for multi-touch (pan/zoom)
            let initialPointersForZoom = new Map(); // Store initial positions for reference on zoom start
            let initialViewBoxOnZoomStart = [0, 0, 0, 0]; // Store viewBox state at start of zoom

            // currentViewBox: [minX, minY, width, height] of the currently visible SVG area
            const initialYCenterSvg = mapPriceToY(currentPrice);
            let currentViewBox = [0, initialYCenterSvg - (CHART_HEIGHT / 2), CHART_WIDTH, CHART_HEIGHT];

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

            function formatNumberWithDots(num) {
                const number = typeof num === 'number' && !isNaN(num) ? num : 0;
                return new Intl.NumberFormat('id-ID').format(Math.round(number));
            }

            function parseFormattedNumber(str) {
                if (typeof str !== 'string') return str;
                const cleanedStr = str.replace(/\./g, '');
                const parsed = parseFloat(cleanedStr);
                return isNaN(parsed) ? 0 : parsed;
            }

            function formatCurrency(amount) {
                return `Rp ${formatNumberWithDots(amount)}`;
            }

            // --- CORE CHART FUNCTIONS ---

            function mapPriceToY(price) {
                const priceRange = MAX_PRICE - MIN_PRICE;
                const clampedPrice = Math.max(MIN_PRICE, Math.min(MAX_PRICE, price));
                const percentage = (clampedPrice - MIN_PRICE) / priceRange;
                return CHART_HEIGHT - (percentage * CHART_HEIGHT);
            }

            function renderChart() {
                const elementsToRemove = Array.from(svgChart.children).filter(el =>
                    !el.classList.contains('bet-candle-dot') &&
                    !el.classList.contains('bet-label-group')
                );
                elementsToRemove.forEach(el => el.remove());

                candleDataHistory.forEach((data, i) => {
                    const candleXPosition = i * TOTAL_CANDLE_WIDTH;

                    if (candleXPosition + TOTAL_CANDLE_WIDTH > currentViewBox[0] &&
                        candleXPosition < currentViewBox[0] + currentViewBox[2]) {

                        const isUp = data.close >= data.open;
                        const color = isUp ? 'var(--color-green)' : 'var(--color-red)';

                        const wick = document.createElementNS(svgNS, 'rect');
                        wick.setAttribute('x', candleXPosition + (CANDLE_WIDTH / 2 - 1));
                        wick.setAttribute('y', mapPriceToY(data.high));
                        wick.setAttribute('width', 2);
                        wick.setAttribute('height', Math.abs(mapPriceToY(data.low) - mapPriceToY(data.high)));
                        wick.setAttribute('fill', color);
                        svgChart.appendChild(wick);

                        const body = document.createElementNS(svgNS, 'rect');
                        body.setAttribute('x', candleXPosition);
                        body.setAttribute('y', mapPriceToY(Math.max(data.open, data.close)));
                        body.setAttribute('width', CANDLE_WIDTH);
                        body.setAttribute('height', Math.abs(mapPriceToY(data.open) - mapPriceToY(data.close)) || 1);
                        body.setAttribute('fill', color);
                        svgChart.appendChild(body);
                    }
                });

                svgChart.setAttribute('viewBox', currentViewBox.join(' '));

                renderBetIndicators();
            }

            function generateCandleData() {
                const open = candleDataHistory.length > 0 ? candleDataHistory[candleDataHistory.length - 1].close : currentPrice;
                let price = open;
                let high = open;
                let low = open;

                for (let i = 0; i < 10; i++) {
                    price += (Math.random() - 0.5) * VOLATILITY;
                    price = Math.max(MIN_PRICE, Math.min(MAX_PRICE, price));
                    high = Math.max(high, price);
                    low = Math.min(low, price);
                }

                high = Math.max(MIN_PRICE, Math.min(MAX_PRICE, high));
                low = Math.max(MIN_PRICE, Math.min(MAX_PRICE, low));

                currentPrice = price;
                candleDataHistory.push({ open, high, low, close: currentPrice });

                renderChart();
            }

            function prefillChartWithCandles() {
                for (let i = 0; i < INITIAL_CANDLE_COUNT; i++) {
                    generateCandleData();
                }
                currentViewBox[0] = (candleDataHistory.length * TOTAL_CANDLE_WIDTH) - CHART_WIDTH;
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

                const chartHeightPx = chartContainer.clientHeight;
                const priceYInViewBox = mapPriceToY(currentPrice);
                const normalizedY = (priceYInViewBox - currentViewBox[1]) / currentViewBox[3];
                const yPosPx = normalizedY * chartHeightPx;

                priceIndicator.style.top = `${yPosPx}px`;
                priceIndicator.style.width = chartContainer.clientWidth + 'px';
                priceLabel.style.top = `${yPosPx}px`;
                priceLabel.textContent = Math.round(currentPrice);

                updateBetIndicatorsPositions();
                updateNotificationCountdowns();
            }

            function openTrade(direction) {
                const investment = parseFormattedNumber(investmentInput.value);

                if (activeTrades.length >= MAX_ACTIVE_BETS) {
                    showNotification(`Maksimal ${MAX_ACTIVE_BETS} taruhan aktif`, 'loss');
                    return;
                }

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
                renderBetIndicators();
                updateAllDisplays();
            }

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

                    if (trade.dotElement) trade.dotElement.remove();
                    if (trade.labelGroupElement) trade.labelGroupElement.remove();

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
                updateAllDisplays();
            }

            function renderBetIndicators() {
                svgChart.querySelectorAll('.bet-candle-dot, .bet-label-group').forEach(el => el.remove());

                activeTrades.forEach(trade => {
                    if (trade.candleIndex !== undefined && candleDataHistory[trade.candleIndex]) {
                        const candleX = trade.candleIndex * TOTAL_CANDLE_WIDTH;
                        const dotX = candleX + (CANDLE_WIDTH / 2);
                        const dotY = mapPriceToY(trade.startPrice);

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
                            labelBg.setAttribute('class', `bet-label-bg ${trade.direction}`);
                            labelGroup.appendChild(labelBg);
                            trade.labelBgElement = labelBg;

                            let labelText = document.createElementNS(svgNS, 'text');
                            labelText.setAttribute('class', 'bet-label-text');
                            labelGroup.appendChild(labelText);
                            trade.labelTextElement = labelText;

                            labelText.textContent = formatCurrency(trade.investment);

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
                            if (trade.dotElement) { trade.dotElement.remove(); trade.dotElement = null; }
                            if (trade.labelGroupElement) { trade.labelGroupElement.remove(); trade.labelGroupElement = null; }
                            trade.labelBgElement = null;
                            trade.labelTextElement = null;
                        }
                    }
                });
            }

            function updateBetIndicatorsPositions() {
                activeTrades.forEach(trade => {
                    if (trade.dotElement && trade.labelGroupElement && trade.labelTextElement && trade.labelBgElement) {
                        const candleX = trade.candleIndex * TOTAL_CANDLE_WIDTH;
                        const dotX = candleX + (CANDLE_WIDTH / 2);
                        const dotY = mapPriceToY(trade.startPrice);

                        trade.dotElement.setAttribute('cx', dotX);
                        trade.dotElement.setAttribute('cy', dotY);

                        trade.labelTextElement.textContent = formatCurrency(trade.investment);

                        const textBBox = trade.labelTextElement.getBBox();
                        const labelXOffset = 5;
                        const labelYOffset = - (textBBox.height / 2) - 2;

                        trade.labelBgElement.setAttribute('x', dotX + labelXOffset);
                        trade.labelBgElement.setAttribute('y', dotY + labelYOffset);
                        trade.labelBgElement.setAttribute('width', textBBox.width + 10);
                        trade.labelBgElement.setAttribute('height', textBBox.height + 4);

                        labelText.setAttribute('x', dotX + labelXOffset + 5);
                        labelText.setAttribute('y', dotY + labelYOffset + textBBox.height - 2);
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
                                <span>Rp ${formatNumberWithDots(trade.investment)}</span>
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
                        item.querySelector('.info span:last-child').textContent = formatNumberWithDots(trade.finalAmount);
                    } else {
                        item.classList.remove('completed', 'win', 'loss');
                        item.classList.add(trade.direction);
                        item.querySelector('.info span:last-child').textContent = formatNumberWithDots(trade.investment);
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
                balanceDisplay.textContent = formatCurrency(userBalance);

                document.getElementById('current-time').textContent = new Date().toLocaleTimeString('en-GB', { timeZone: 'Asia/Jakarta' }) + " GMT+7";
                payoutPreview.textContent = formatCurrency((parseFormattedNumber(investmentInput.value) || 0) * (1 + PAYOUT_RATE));

                updateActiveTradeInfo();

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
            function formatTime(seconds) {
                const m = Math.floor(seconds / 60).toString().padStart(2, '0');
                const s = (seconds % 60).toString().padStart(2, '0');
                return `${m}:${s}`;
            }

            // --- EVENT LISTENERS ---
            document.getElementById('investment-plus').addEventListener('click', () => {
                let value = parseFormattedNumber(investmentInput.value);
                investmentInput.value = formatNumberWithDots((value || 0) + 14000);
                updateAllDisplays();
            });
            document.getElementById('investment-minus').addEventListener('click', () => {
                let value = parseFormattedNumber(investmentInput.value);
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

            // --- Chart Panning & Zooming Event Listeners (using pointer events for better touch support) ---
            let lastPointers = new Map(); // Stores the current positions of active pointers
            let initialPointersForZoom = new Map(); // Stores starting positions of pointers for zoom gestures
            let initialViewBoxOnZoomStart = [...currentViewBox]; // Stores viewBox state at the start of a zoom gesture


            // Helper to convert client coordinates to SVG coordinates
            function clientToSvgCoords(clientX, clientY) {
                const svgRect = svgChart.getBoundingClientRect();
                const svgX = currentViewBox[0] + (clientX - svgRect.left) / svgRect.width * currentViewBox[2];
                const svgY = currentViewBox[1] + (clientY - svgRect.top) / svgRect.height * currentViewBox[3];
                return { x: svgX, y: svgY };
            }

            chartContainer.addEventListener('pointerdown', (e) => {
                isDragging = true;
                e.preventDefault(); // Prevent default browser gestures (like scrolling/zooming)
                chartContainer.setPointerCapture(e.pointerId); // Lock pointer capture for consistent drag/zoom
                lastPointers.set(e.pointerId, { clientX: e.clientX, clientY: e.clientY });

                if (lastPointers.size === 2) { // Start of a two-finger gesture
                    initialViewBoxOnZoomStart = [...currentViewBox]; // Save current viewBox state
                    
                    // Populate initialPointersForZoom with the positions of the two current pointers
                    let count = 0;
                    for (let [id, pos] of lastPointers) {
                        if (count === 0) initialPointersForZoom.set(id, {clientX: pos.clientX, clientY: pos.clientY});
                        else initialPointersForZoom.set(id, {clientX: pos.clientX, clientY: pos.clientY});
                        count++;
                    }
                }
            });

            chartContainer.addEventListener('pointermove', (e) => {
                if (!isDragging) return;
                
                // Update current pointer position in the map
                lastPointers.set(e.pointerId, { clientX: e.clientX, clientY: e.clientY });

                const svgRect = svgChart.getBoundingClientRect();
                
                if (lastPointers.size === 1) { // Single finger drag (Pan)
                    const currentPointer = lastPointers.get(e.pointerId);
                    const prevPointer = { 
                        clientX: currentPointer.clientX - e.movementX, // Use movementX/Y for precise delta
                        clientY: currentPointer.clientY - e.movementY
                    }; 

                    const scaleX = currentViewBox[2] / svgRect.width;
                    const scaleY = currentViewBox[3] / svgRect.height;

                    const deltaViewBoxX = - (currentPointer.clientX - prevPointer.clientX) * scaleX;
                    const deltaViewBoxY = - (currentPointer.clientY - prevPointer.clientY) * scaleY;

                    currentViewBox[0] += deltaViewBoxX;
                    currentViewBox[1] += deltaViewBoxY;

                } else if (lastPointers.size === 2) { // Two fingers (Zoom)
                    let p1Current = null, p2Current = null;
                    let p1Initial = null, p2Initial = null;
                    let count = 0;
                    for (let [id, pos] of lastPointers) {
                        if (count === 0) p1Current = pos;
                        else p2Current = pos;
                        count++;
                    }
                    if (!p1Current || !p2Current) return; // Should not happen

                    p1Initial = initialPointersForZoom.get(p1Current.pointerId);
                    p2Initial = initialPointersForZoom.get(p2Current.pointerId);

                    if (!p1Initial || !p2Initial) { // If initial points not set, set them now
                        initialPointersForZoom.set(p1Current.pointerId, {clientX: p1Current.clientX, clientY: p1Current.clientY});
                        initialPointersForZoom.set(p2Current.pointerId, {clientX: p2Current.clientX, clientY: p2Current.clientY});
                        initialViewBoxOnZoomStart = [...currentViewBox]; // Also reset initial viewBox if gesture was interrupted
                        return; // Skip this move to wait for next stable move
                    }
                    
                    // Calculate current distance between fingers
                    const currentDistance = Math.hypot(p1Current.clientX - p2Current.clientX, p1Current.clientY - p2Current.clientY);
                    // Calculate initial distance between fingers (from when the gesture started)
                    const initialDistance = Math.hypot(p1Initial.clientX - p2Initial.clientX, p1Initial.clientY - p2Initial.clientY);

                    if (initialDistance === 0) return; // Avoid division by zero

                    const zoomFactor = currentDistance / initialDistance;

                    // Clamp zoom factor to avoid extreme zoom levels
                    const minZoomLevel = 0.05; // ViewBox can become 20x larger (zoom out)
                    const maxZoomLevel = 5;    // ViewBox can become 5x smaller (zoom in)
                    
                    let newWidth = initialViewBoxOnZoomStart[2] / zoomFactor;
                    let newHeight = initialViewBoxOnZoomStart[3] / zoomFactor;

                    // Clamp new dimensions to ensure they stay within reasonable bounds
                    newWidth = Math.max(CHART_WIDTH / maxZoomLevel, Math.min(CHART_WIDTH / minZoomLevel, newWidth));
                    newHeight = Math.max(CHART_HEIGHT / maxZoomLevel, Math.min(CHART_HEIGHT / minZoomLevel, newHeight));

                    // Calculate the center point of the two fingers in client coordinates
                    const clientCenterX = (p1Current.clientX + p2Current.clientX) / 2;
                    const clientCenterY = (p1Current.clientY + p2Current.clientY) / 2;

                    // Convert client center to SVG coordinates (relative to the initialViewBoxOnZoomStart)
                    const svgCenterX = initialViewBoxOnZoomStart[0] + (clientCenterX - svgRect.left) / svgRect.width * initialViewBoxOnZoomStart[2];
                    const svgCenterY = initialViewBoxOnZoomStart[1] + (clientCenterY - svgRect.top) / svgRect.height * initialViewBoxOnZoomStart[3];

                    // Update currentViewBox dimensions
                    currentViewBox[2] = newWidth;
                    currentViewBox[3] = newHeight;
                    
                    // Update currentViewBox position to zoom around the center point
                    currentViewBox[0] = svgCenterX - (currentViewBox[2] / 2);
                    currentViewBox[1] = svgCenterY - (currentViewBox[3] / 2);
                }

                renderChart();
            });

            chartContainer.addEventListener('pointerup', (e) => {
                lastPointers.delete(e.pointerId);
                initialPointersForZoom.delete(e.pointerId); // Remove from initial zoom pointers as well

                if (lastPointers.size === 0) { // All fingers lifted
                    isDragging = false;
                    chartContainer.classList.remove('dragging');
                    initialPointersForZoom.clear(); // Clear all initial zoom pointers for next gesture
                } else if (lastPointers.size === 1) { // One finger left after a multi-touch
                    isDragging = true; // Still dragging with one finger (pan)
                    // Reset the single pointer's position to its current state for consistent pan
                    const remainingPointer = lastPointers.values().next().value;
                    lastPointers.set(remainingPointer.pointerId, { clientX: remainingPointer.clientX, clientY: remainingPointer.clientY });
                    initialPointersForZoom.clear(); // Clear zoom state as it's now a pan gesture
                }
                chartContainer.releasePointerCapture(e.pointerId);
            });

            chartContainer.addEventListener('pointercancel', (e) => {
                lastPointers.delete(e.pointerId);
                initialPointersForZoom.delete(e.pointerId);
                if (lastPointers.size === 0) {
                    isDragging = false;
                    chartContainer.classList.remove('dragging');
                    initialPointersForZoom.clear();
                }
                chartContainer.releasePointerCapture(e.pointerId);
            });


            // --- INITIALIZATION & MAIN GAME LOOP ---
            svgChart.setAttribute('viewBox', currentViewBox.join(' '));
            svgChart.setAttribute('preserveAspectRatio', 'none');

            prefillChartWithCandles();
            updateAllDisplays();
            setInterval(mainLoop, 200);
        };
    </script>
</body>
</html>
