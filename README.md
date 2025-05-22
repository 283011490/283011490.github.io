<!DOCTYPE html>
<html>
<head>
    <!-- 核心依赖 -->
    <script src="https://cdn.plot.ly/plotly-2.24.1.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/axios@1.6.7/dist/axios.min.js"></script>
    
    <!-- 样式配置 -->
    <style>
        .alert-box { 
            position: absolute; 
            top: 20px; 
            right: 20px; 
            padding: 15px;
            border-radius: 8px;
            font-weight: bold;
        }
        .bullish { background: #d4edda; color: #155724; }
        .bearish { background: #f8d7da; color: #721c24; }
    </style>
</head>
<body>
    <!-- 控制面板 -->
    <div id="controls">
        <select id="interval">
            <option value="1s">1秒</option>
            <option value="1m">1分钟</option>
            <option value="15m">15分钟</option>
            <option value="1h">1小时</option>
            <option value="1d">日线</option>
        </select>
        
        <div id="symbol-buttons">
            <!-- 动态生成币种按钮 -->
        </div>
    </div>

    <!-- K线容器 -->
    <div id="kline-container"></div>

    <script>
        // 核心逻辑
        const BINANCE_WS = "wss://stream.binance.com:9443/ws/";
        let activeCharts = {};

        // 初始化币种列表
        async function initSymbols() {
            const response = await axios.get('https://api.binance.com/api/v3/exchangeInfo');
            response.data.symbols.slice(0,20).forEach(symbol => { // 示例显示前20个币种
                const btn = document.createElement('button');
                btn.textContent = symbol.symbol;
                btn.onclick = () => toggleSymbolChart(symbol.symbol);
                document.getElementById('symbol-buttons').appendChild(btn);
            });
        }

        // K线图生成逻辑
        function renderKline(symbol, data) {
            const trace = {
                x: data.map(d => new Date(d[0])),
                close: data.map(d => parseFloat(d[4])),
                high: data.map(d => parseFloat(d[2])),
                low: data.map(d => parseFloat(d[3])),
                open: data.map(d => parseFloat(d[1])),
                type: 'candlestick',
                name: symbol
            };
            
            const layout = {
                title: `${symbol} 实时K线`,
                annotations: [{
                    x: 0.95, y: 0.95,
                    xref: 'paper', yref: 'paper',
                    text: generateRecommendation(data),
                    showarrow: false,
                    font: { size: 14 }
                }]
            };

            Plotly.newPlot(`chart-${symbol}`, [trace], layout);
        }

        // 实时数据订阅
        function subscribeRealTime(symbol) {
            const ws = new WebSocket(`${BINANCE_WS}${symbol.toLowerCase()}@kline_1s`);
            ws.onmessage = (event) => {
                const kline = JSON.parse(event.data).k;
                updateChartData(symbol, kline);
            };
        }

        // 生成交易建议（示例逻辑）
        function generateRecommendation(data) {
            const closes = data.slice(-30).map(d => parseFloat(d[4]));
            const momentum = closes[closes.length-1] - closes[0];
            const confidence = Math.min(Math.abs(momentum)*100, 95).toFixed(1);
            
            return momentum > 0 ? 
                `▲ 做多建议（信心指数 ${confidence}%）` : 
                `▼ 做空建议（信心指数 ${confidence}%）`;
        }

        // 初始化执行
        initSymbols();
    </script>
</body>
</html>
