<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>INCU Analyzer</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mqtt/4.3.7/mqtt.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            padding: 20px;
            background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
            color: #fff;
            min-height: 100vh;
        }

        .title {
            text-align: center;
            font-size: 2.5em;
            margin-bottom: 30px;
            color: #fff;
            text-shadow: 0 0 10px rgba(52, 152, 219, 0.5);
            animation: glow 2s ease-in-out infinite alternate;
        }

        @keyframes glow {
            from { text-shadow: 0 0 10px rgba(52, 152, 219, 0.5); }
            to { text-shadow: 0 0 20px rgba(52, 152, 219, 0.8); }
        }

        .mqtt-status {
            text-align: center;
            margin-bottom: 20px;
            padding: 10px;
            border-radius: 5px;
            transition: all 0.3s ease;
        }

        .mqtt-status.connected {
            background: rgba(46, 204, 113, 0.2);
            color: #2ecc71;
        }

        .mqtt-status.disconnected {
            background: rgba(231, 76, 60, 0.2);
            color: #e74c3c;
        }

        .control-buttons {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 30px;
            flex-wrap: wrap;
        }

        .control-btn {
            padding: 15px 30px;
            font-size: 1.1em;
            border: none;
            border-radius: 8px;
            background: rgba(52, 152, 219, 0.8);
            color: white;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }

        .control-btn:hover {
            transform: translateY(-3px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
            background: rgba(52, 152, 219, 1);
        }

        .control-btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: none;
        }

        .control-btn.active {
            background: rgba(231, 76, 60, 0.8);
            animation: pulse 1.5s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        .input-container {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-bottom: 30px;
            flex-wrap: wrap;
        }

        .input-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }

        .input-group label {
            font-weight: bold;
            color: #fff;
        }

        .input-group input {
            padding: 10px;
            border: 2px solid rgba(52, 152, 219, 0.5);
            border-radius: 5px;
            font-size: 1em;
            background: rgba(255, 255, 255, 0.1);
            color: #fff;
            transition: all 0.3s ease;
        }

        .input-group input:focus {
            outline: none;
            border-color: rgba(52, 152, 219, 1);
            background: rgba(255, 255, 255, 0.2);
        }

        .timer-display {
            text-align: center;
            font-size: 3em;
            margin-bottom: 30px;
            color: #fff;
            text-shadow: 0 0 10px rgba(52, 152, 219, 0.5);
            font-family: 'Courier New', monospace;
        }

        .battery-section {
            max-width: 1200px;
            margin: 0 auto 30px auto;
        }

        .battery-section h2 {
            text-align: center;
            margin-bottom: 15px;
            color: #fff;
        }

        .battery-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 15px;
            margin-bottom: 20px;
        }

        .battery-box {
            background: rgba(255, 255, 255, 0.1);
            padding: 15px;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
            text-align: center;
            transition: all 0.3s ease;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .battery-box.low {
            background: rgba(231, 76, 60, 0.3);
            border-color: rgba(231, 76, 60, 0.5);
            animation: warning-blink 2s infinite;
        }

        @keyframes warning-blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
        }

        .battery-box h4 {
            margin-bottom: 10px;
            color: #fff;
            font-size: 0.9em;
        }

        .battery-value {
            font-size: 1.8em;
            color: #2ecc71;
            font-weight: bold;
        }

        .battery-box.low .battery-value {
            color: #e74c3c;
        }

        .alarm-section {
            max-width: 1200px;
            margin: 0 auto 30px auto;
        }

        .alarm-box {
            background: rgba(46, 204, 113, 0.2);
            padding: 20px;
            border-radius: 10px;
            text-align: center;
            transition: all 0.3s ease;
            border: 2px solid rgba(46, 204, 113, 0.5);
        }

        .alarm-box.alarm-active {
            background: rgba(231, 76, 60, 0.3);
            border-color: rgba(231, 76, 60, 0.8);
            animation: alarm-pulse 1s infinite;
        }

        @keyframes alarm-pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.02); }
        }

        .alarm-box h3 {
            margin-bottom: 10px;
            color: #fff;
        }

        .alarm-status {
            font-size: 1.5em;
            color: #2ecc71;
            font-weight: bold;
        }

        .alarm-box.alarm-active .alarm-status {
            color: #e74c3c;
        }

        .alarm-messages {
            margin-top: 10px;
            font-size: 0.9em;
            color: #fff;
        }

        .sensor-section {
            max-width: 1200px;
            margin: 0 auto 30px auto;
        }

        .sensor-section h2 {
            text-align: center;
            margin-bottom: 15px;
            color: #fff;
        }

        .sensor-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 20px;
            margin-bottom: 30px;
        }

        .sensor-box {
            background: rgba(255, 255, 255, 0.1);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
            text-align: center;
            transition: all 0.3s ease;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .sensor-box:hover {
            transform: translateY(-5px);
            box-shadow: 0 8px 25px rgba(0, 0, 0, 0.3);
            background: rgba(255, 255, 255, 0.15);
        }

        .sensor-box.updated {
            animation: highlight 1s ease-out;
        }

        .sensor-box.error {
            background: rgba(231, 76, 60, 0.3);
            border-color: rgba(231, 76, 60, 0.8);
        }

        @keyframes highlight {
            0% { background: rgba(52, 152, 219, 0.3); }
            100% { background: rgba(255, 255, 255, 0.1); }
        }

        .sensor-box h3 {
            margin-bottom: 10px;
            color: #fff;
        }

        .sensor-value {
            font-size: 1.5em;
            color: #3498db;
            text-shadow: 0 0 5px rgba(52, 152, 219, 0.5);
            transition: all 0.3s ease;
        }

        .temp-selection {
            max-width: 1200px;
            margin: 0 auto 30px auto;
            text-align: center;
        }

        .temp-selection h3 {
            margin-bottom: 15px;
            color: #fff;
        }

        .temp-buttons {
            display: flex;
            justify-content: center;
            gap: 20px;
        }

        .temp-btn {
            padding: 15px 40px;
            font-size: 1.2em;
            border: 2px solid rgba(52, 152, 219, 0.5);
            border-radius: 8px;
            background: rgba(255, 255, 255, 0.1);
            color: white;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .temp-btn.selected {
            background: rgba(52, 152, 219, 0.8);
            border-color: rgba(52, 152, 219, 1);
            box-shadow: 0 4px 15px rgba(52, 152, 219, 0.5);
        }

        .temp-btn:hover {
            transform: translateY(-3px);
            background: rgba(52, 152, 219, 0.6);
        }

        .table-container {
            width: 100%;
            overflow-x: auto;
            margin-top: 20px;
            max-width: 1200px;
            margin-left: auto;
            margin-right: auto;
        }

        .data-table {
            width: 100%;
            border-collapse: separate;
            border-spacing: 0;
            background: rgba(255, 255, 255, 0.1);
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
            border-radius: 10px;
            overflow: hidden;
            min-width: 1000px;
        }

        .data-table th, .data-table td {
            padding: 12px 15px;
            text-align: left;
            border-bottom: 1px solid rgba(255, 255, 255, 0.1);
            white-space: nowrap;
        }

        .data-table th {
            background: rgba(52, 152, 219, 0.8);
            color: white;
            font-weight: bold;
            position: sticky;
            top: 0;
            z-index: 10;
        }

        .data-table tr {
            transition: all 0.3s ease;
        }

        .data-table tr:hover {
            background: rgba(255, 255, 255, 0.15);
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .new-row {
            animation: fadeIn 0.5s ease-out;
        }

        .data-table tr.out-of-tolerance {
            background: rgba(231, 76, 60, 0.4) !important;
        }

        .data-table tr.out-of-tolerance:hover {
            background: rgba(231, 76, 60, 0.5) !important;
        }

        @media (max-width: 768px) {
            .sensor-grid {
                grid-template-columns: repeat(2, 1fr);
            }
            .battery-grid {
                grid-template-columns: repeat(2, 1fr);
            }
        }

        @media (max-width: 480px) {
            .sensor-grid {
                grid-template-columns: 1fr;
            }
            .battery-grid {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <h1 class="title">INCU Analyzer</h1>

    <div id="mqttStatus" class="mqtt-status disconnected">
        MQTT Status: Disconnected
    </div>

    <div class="control-buttons">
        <button id="saveBtn" class="control-btn">Play Saving Data</button>
        <button id="resetBtn" class="control-btn">Reset Data</button>
        <button id="exportBtn" class="control-btn">Export to Excel</button>
    </div>

    <div class="input-container">
        <div class="input-group">
            <label>Interval (seconds)</label>
            <input type="number" id="intervalInput" min="1" value="2">
        </div>
        <div class="input-group">
            <label>Timer (HH:MM:SS)</label>
            <input type="text" id="timerInput" placeholder="00:00:00" pattern="[0-9]{2}:[0-9]{2}:[0-9]{2}" value="00:01:00">
        </div>
    </div>

    <div class="timer-display" id="timerDisplay">00:00:00</div>

    <div class="battery-section">
        <h2>Battery Status</h2>
        <div class="battery-grid">
            <div class="battery-box low" id="batteryCenter">
                <h4>Central Unit</h4>
                <div class="battery-value" id="batteryCenterValue">0%</div>
            </div>
            <div class="battery-box low" id="batteryNode1">
                <h4>Sensor Node 1</h4>
                <div class="battery-value" id="batteryNode1Value">0%</div>
            </div>
            <div class="battery-box low" id="batteryNode2">
                <h4>Sensor Node 2</h4>
                <div class="battery-value" id="batteryNode2Value">0%</div>
            </div>
            <div class="battery-box low" id="batteryNode3">
                <h4>Sensor Node 3</h4>
                <div class="battery-value" id="batteryNode3Value">0%</div>
            </div>
            <div class="battery-box low" id="batteryNode4">
                <h4>Sensor Node 4</h4>
                <div class="battery-value" id="batteryNode4Value">0%</div>
            </div>
        </div>
    </div>

    <div class="alarm-section">
        <div class="alarm-box" id="alarmBox">
            <h3>System Status</h3>
            <div class="alarm-status" id="alarmStatus">Normal - No Alerts</div>
            <div class="alarm-messages" id="alarmMessages"></div>
        </div>
    </div>

    <div class="sensor-section">
        <h2>Data Sensor</h2>
        <div class="sensor-grid">
            <div class="sensor-box" id="t1Box"><h3>T1</h3><div class="sensor-value" id="t1Value">0.0 °C</div></div>
            <div class="sensor-box" id="t2Box"><h3>T2</h3><div class="sensor-value" id="t2Value">0.0 °C</div></div>
            <div class="sensor-box" id="t3Box"><h3>T3</h3><div class="sensor-value" id="t3Value">0.0 °C</div></div>
            <div class="sensor-box" id="t4Box"><h3>T4</h3><div class="sensor-value" id="t4Value">0.0 °C</div></div>
            <div class="sensor-box" id="t5Box"><h3>T5</h3><div class="sensor-value" id="t5Value">0.0 °C</div></div>
            <div class="sensor-box" id="tmBox"><h3>TM</h3><div class="sensor-value" id="tmValue">0.0 °C</div></div>
            <div class="sensor-box" id="rhBox"><h3>HUMIDITY</h3><div class="sensor-value" id="rhValue">0.0 %</div></div>
            <div class="sensor-box" id="flowBox"><h3>AIRFLOW</h3><div class="sensor-value" id="flowValue">0.0 m/s</div></div>
            <div class="sensor-box" id="noiseBox"><h3>NOISE</h3><div class="sensor-value" id="noiseValue">0.0 dB</div></div>
        </div>
    </div>

    <div class="temp-selection">
        <h3>Temperature Setting for Export</h3>
        <div class="temp-buttons">
            <button class="temp-btn selected" id="temp32Btn" onclick="selectTemp(32)">32°C</button>
            <button class="temp-btn" id="temp36Btn" onclick="selectTemp(36)">36°C</button>
        </div>
    </div>

    <div class="table-container">
        <table class="data-table">
            <thead>
                <tr>
                    <th>Date</th>
                    <th>Time</th>
                    <th>T1 (°C)</th>
                    <th>T2 (°C)</th>
                    <th>T3 (°C)</th>
                    <th>T4 (°C)</th>
                    <th>T5 (°C)</th>
                    <th>TM (°C)</th>
                    <th>Humidity (%)</th>
                    <th>Airflow (m/s)</th>
                    <th>Noise (dB)</th>
                </tr>
            </thead>
            <tbody id="dataTableBody"></tbody>
        </table>
    </div>

    <script>
        // ─── MQTT Setup ───────────────────────────────────────────────────────────
        const brokerUrl = 'wss://broker.hivemq.com:8884/mqtt';
        const options = {
            clean: true,
            connectTimeout: 4000,
            reconnectPeriod: 1000,
            clientId: 'incu_analyzer_' + Math.random().toString(16).substr(2, 8),
            keepalive: 60,
            protocolVersion: 4
        };

        const client = mqtt.connect(brokerUrl, options);
        const topic = 'incu/sensors';

        // ─── State ────────────────────────────────────────────────────────────────
        let isRecording = false;
        let tableData = [];
        let currentSensorData = {};
        let lastDataTime = Date.now();
        let selectedTemp = 32;

        // FIX: Use Date-based timing instead of tick counting to survive tab throttling
        let recordingStartWallTime = null;   // Date.now() when recording started
        let totalDurationMs = 0;             // total duration in ms
        let intervalMs = 2000;               // data-capture interval in ms
        let lastCaptureWallTime = null;      // last wall-time we captured a row

        // rAF + visibility-safe tick loop
        let rafHandle = null;
        let tickHandle = null;

        // ─── Helpers ──────────────────────────────────────────────────────────────
        function parseTimerInput(timeString) {
            const parts = timeString.split(':');
            if (parts.length !== 3) return 60;
            const hours   = parseInt(parts[0]) || 0;
            const minutes = parseInt(parts[1]) || 0;
            const seconds = parseInt(parts[2]) || 0;
            return (hours * 3600) + (minutes * 60) + seconds;
        }

        function formatTime(seconds) {
            const h = Math.floor(seconds / 3600);
            const m = Math.floor((seconds % 3600) / 60);
            const s = Math.floor(seconds % 60);
            return `${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`;
        }

        function selectTemp(temp) {
            selectedTemp = temp;
            document.getElementById('temp32Btn').classList.toggle('selected', temp === 32);
            document.getElementById('temp36Btn').classList.toggle('selected', temp === 36);
        }

        // ─── CORE TICK (runs on a real-time setInterval, NOT rAF) ─────────────────
        // Using setInterval with Web Lock / Wake Lock workarounds:
        // The reliable fix is to use Date.now() deltas instead of trusting tick counts.

        function tick() {
            if (!isRecording) return;

            const now = Date.now();
            const elapsed = now - recordingStartWallTime;
            const remaining = totalDurationMs - elapsed;

            // Update countdown display
            if (remaining <= 0) {
                document.getElementById('timerDisplay').textContent = '00:00:00';
                stopRecording();
                return;
            }

            document.getElementById('timerDisplay').textContent = formatTime(Math.ceil(remaining / 1000));

            // Check if it's time to capture a data row
            if (now - lastCaptureWallTime >= intervalMs) {
                lastCaptureWallTime = now;
                if (Object.keys(currentSensorData).length > 0) {
                    addTableRow(currentSensorData);
                }
            }
        }

        // ─── Recording Control ────────────────────────────────────────────────────
        function startRecording() {
            if (isRecording) return;

            const timerInput = document.getElementById('timerInput').value;
            const totalSeconds = parseTimerInput(timerInput);
            if (totalSeconds <= 0) {
                alert('Please enter a valid timer duration');
                return;
            }

            intervalMs = (parseInt(document.getElementById('intervalInput').value) || 2) * 1000;
            totalDurationMs = totalSeconds * 1000;
            recordingStartWallTime = Date.now();
            lastCaptureWallTime = Date.now(); // start capture clock now

            isRecording = true;

            document.getElementById('saveBtn').classList.add('active');
            document.getElementById('saveBtn').textContent = 'Stop Saving Data';
            document.getElementById('intervalInput').disabled = true;
            document.getElementById('timerInput').disabled = true;

            // FIX: Use a short setInterval (200ms) for high-resolution wall-clock checks.
            // This is far more reliable than 1000ms because even when throttled to ~1Hz,
            // we still catch the exact elapsed time via Date.now() — no skipped ticks.
            tickHandle = setInterval(tick, 200);
        }

        function stopRecording() {
            isRecording = false;

            if (tickHandle) { clearInterval(tickHandle); tickHandle = null; }

            document.getElementById('saveBtn').classList.remove('active');
            document.getElementById('saveBtn').textContent = 'Play Saving Data';
            document.getElementById('intervalInput').disabled = false;
            document.getElementById('timerInput').disabled = false;
        }

        function resetData() {
            if (confirm('Are you sure you want to reset all data?')) {
                tableData = [];
                document.getElementById('dataTableBody').innerHTML = '';
                if (isRecording) stopRecording();
                document.getElementById('timerDisplay').textContent =
                    document.getElementById('timerInput').value || '00:00:00';
            }
        }

        // ─── Page Visibility API ──────────────────────────────────────────────────
        // FIX: When tab becomes visible again after being hidden, recalculate elapsed
        // time properly — no data is lost, and the timer is immediately correct.
        document.addEventListener('visibilitychange', () => {
            if (!isRecording) return;

            if (document.hidden) {
                // Tab going hidden — nothing to do, Date.now() keeps ticking
            } else {
                // Tab came back visible — check if we missed any captures
                const now = Date.now();
                const elapsed = now - recordingStartWallTime;

                if (elapsed >= totalDurationMs) {
                    // Timer expired while we were away
                    document.getElementById('timerDisplay').textContent = '00:00:00';
                    stopRecording();
                    return;
                }

                // Catch up on any missed data captures
                // (if interval is 2s and we were gone 10s, capture up to 5 rows)
                const missedCaptures = Math.floor((now - lastCaptureWallTime) / intervalMs);
                if (missedCaptures > 0 && Object.keys(currentSensorData).length > 0) {
                    // Add missed rows with interpolated timestamps
                    for (let i = missedCaptures; i >= 1; i--) {
                        const captureTime = now - (i * intervalMs);
                        addTableRow(currentSensorData, new Date(captureTime));
                    }
                    lastCaptureWallTime = now;
                }

                // Update display immediately
                const remaining = totalDurationMs - elapsed;
                document.getElementById('timerDisplay').textContent = formatTime(Math.ceil(remaining / 1000));
            }
        });

        // ─── Sensor Data ──────────────────────────────────────────────────────────
        function resetSensorDisplay() {
            ['t1','t2','t3','t4','t5','tm'].forEach(id =>
                document.getElementById(id+'Value').textContent = '00.00 °C'
            );
            document.getElementById('flowValue').textContent  = '00.00 m/s';
            document.getElementById('noiseValue').textContent = '00.00 dB';
            document.getElementById('rhValue').textContent    = '00.00 %';
            currentSensorData = {};
        }

        function resetBatteryDisplay() {
            ['Center','Node1','Node2','Node3','Node4'].forEach(name => {
                document.getElementById('battery'+name+'Value').textContent = '0%';
                document.getElementById('battery'+name).classList.add('low');
            });
        }

        // Stale-data watchdog: reset display if no MQTT message for 10s
        setInterval(() => {
            if (Date.now() - lastDataTime > 10000) {
                resetSensorDisplay();
                resetBatteryDisplay();
            }
        }, 5000);

        function highlightUpdatedValues() {
            document.querySelectorAll('.sensor-box').forEach(box => {
                box.classList.add('updated');
                setTimeout(() => box.classList.remove('updated'), 1000);
            });
        }

        function updateSensorValues(data) {
            const fmt = (v, unit) =>
                (v !== undefined && v !== null) ? `${parseFloat(v).toFixed(2)} ${unit}` : `00.00 ${unit}`;

            document.getElementById('t1Value').textContent    = fmt(data.t1,    '°C');
            document.getElementById('t2Value').textContent    = fmt(data.t2,    '°C');
            document.getElementById('t3Value').textContent    = fmt(data.t3,    '°C');
            document.getElementById('t4Value').textContent    = fmt(data.t4,    '°C');
            document.getElementById('t5Value').textContent    = fmt(data.t5,    '°C');
            document.getElementById('tmValue').textContent    = fmt(data.tm,    '°C');
            document.getElementById('flowValue').textContent  = fmt(data.flow,  'm/s');
            document.getElementById('noiseValue').textContent = fmt(data.noise, 'dB');
            document.getElementById('rhValue').textContent    = fmt(data.rh,    '%');
        }

        function updateBatteryStatus(data) {
            const map = {
                battery_center: ['batteryCenter', 'batteryCenterValue'],
                battery_node1:  ['batteryNode1',  'batteryNode1Value'],
                battery_node2:  ['batteryNode2',  'batteryNode2Value'],
                battery_node3:  ['batteryNode3',  'batteryNode3Value'],
                battery_node4:  ['batteryNode4',  'batteryNode4Value'],
            };
            Object.entries(map).forEach(([key, [boxId, valId]]) => {
                if (data[key] !== undefined) {
                    const pct = parseFloat(data[key]);
                    document.getElementById(valId).textContent = `${pct.toFixed(0)}%`;
                    document.getElementById(boxId).classList.toggle('low', pct < 20);
                }
            });
        }

        function checkAlarms(data) {
            const alarmBox      = document.getElementById('alarmBox');
            const alarmStatus   = document.getElementById('alarmStatus');
            const alarmMessages = document.getElementById('alarmMessages');

            let alarms = [];
            const t5 = parseFloat(data.t5 || 0);

            [['T1',data.t1],['T2',data.t2],['T3',data.t3],['T4',data.t4]].forEach(([name,val]) => {
                const v = parseFloat(val || 0);
                if (v > 0 && t5 > 0) {
                    const diff = v - t5;
                    if (v < t5 - 0.8 || v > t5 + 0.8) {
                        alarms.push(`⚠️ ${name} ${diff < 0 ? 'under' : 'over'} temp ${diff < 0 ? '-' : '+'}${Math.abs(diff).toFixed(1)}°C`);
                    }
                }
            });

            const batteries = {
                'Central Unit': data.battery_center,
                'Node 1': data.battery_node1,
                'Node 2': data.battery_node2,
                'Node 3': data.battery_node3,
                'Node 4': data.battery_node4,
            };
            Object.entries(batteries).forEach(([name, level]) => {
                if (level !== undefined && parseFloat(level) < 20)
                    alarms.push(`🔋 Low battery: ${name} ${parseFloat(level).toFixed(0)}%`);
            });

            const hasError = alarms.length > 0;
            alarmBox.classList.toggle('alarm-active', hasError);
            alarmStatus.textContent   = hasError ? 'ALERT!' : 'Normal';
            alarmMessages.innerHTML   = alarms.join('<br>');
        }

        function checkTolerance(data) {
            const t5 = parseFloat(data.t5 || 0);
            for (const key of ['t1','t2','t3','t4']) {
                const v = parseFloat(data[key] || 0);
                if (v > 0 && t5 > 0 && (v < t5 - 0.8 || v > t5 + 0.8)) return true;
            }
            const rh    = parseFloat(data.rh    || 0);
            const tm    = parseFloat(data.tm    || 0);
            const flow  = parseFloat(data.flow  || 0);
            const noise = parseFloat(data.noise || 0);
            if (rh > 0    && (rh < 40 || rh > 65)) return true;
            if (tm >= 40)                            return true;
            if (flow > 0.35)                         return true;
            if (noise >= 65)                         return true;
            return false;
        }

        // FIX: accept optional timestamp so catch-up rows use correct time
        function addTableRow(data, timestamp) {
            const now = timestamp || new Date();
            const row = {
                date:  now.toLocaleDateString('id-ID'),
                time:  now.toLocaleTimeString('id-ID'),
                t1:    parseFloat(data.t1    || 0),
                t2:    parseFloat(data.t2    || 0),
                t3:    parseFloat(data.t3    || 0),
                t4:    parseFloat(data.t4    || 0),
                t5:    parseFloat(data.t5    || 0),
                tm:    parseFloat(data.tm    || 0),
                rh:    parseFloat(data.rh    || 0),
                flow:  parseFloat(data.flow  || 0),
                noise: parseFloat(data.noise || 0),
            };

            tableData.push(row);

            const tbody = document.getElementById('dataTableBody');
            const tr    = document.createElement('tr');
            tr.classList.add('new-row');
            if (checkTolerance(data)) tr.classList.add('out-of-tolerance');

            tr.innerHTML = `
                <td>${row.date}</td>
                <td>${row.time}</td>
                <td>${row.t1.toFixed(2)}</td>
                <td>${row.t2.toFixed(2)}</td>
                <td>${row.t3.toFixed(2)}</td>
                <td>${row.t4.toFixed(2)}</td>
                <td>${row.t5.toFixed(2)}</td>
                <td>${row.tm.toFixed(2)}</td>
                <td>${row.rh.toFixed(2)}</td>
                <td>${row.flow.toFixed(2)}</td>
                <td>${row.noise.toFixed(2)}</td>
            `;
            tbody.insertBefore(tr, tbody.firstChild);
        }

        // ─── MQTT ─────────────────────────────────────────────────────────────────
        client.on('connect', () => {
            document.getElementById('mqttStatus').className   = 'mqtt-status connected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Connected';
            client.subscribe(topic);
        });

        client.on('error', () => {
            document.getElementById('mqttStatus').className   = 'mqtt-status disconnected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Error';
            resetSensorDisplay();
            resetBatteryDisplay();
        });

        client.on('offline', () => {
            document.getElementById('mqttStatus').className   = 'mqtt-status disconnected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Offline';
            resetSensorDisplay();
            resetBatteryDisplay();
        });

        client.on('message', (receivedTopic, message) => {
            try {
                const data = JSON.parse(message.toString());
                lastDataTime = Date.now();
                currentSensorData = data;
                updateSensorValues(data);
                updateBatteryStatus(data);
                checkAlarms(data);
                highlightUpdatedValues();
            } catch (e) {
                console.error('Error parsing MQTT message:', e);
            }
        });

        // ─── Export ───────────────────────────────────────────────────────────────
        function calculateStats(values) {
            if (values.length === 0) return { min: 0, max: 0, mean: 0, stdev: 0 };
            const sorted = [...values].sort((a, b) => a - b);
            const min  = sorted[0];
            const max  = sorted[sorted.length - 1];
            const mean = values.reduce((a, b) => a + b, 0) / values.length;
            const variance = values.reduce((s, v) => s + Math.pow(v - mean, 2), 0) / values.length;
            return { min, max, mean, stdev: Math.sqrt(variance) };
        }

        function exportToExcel() {
            if (tableData.length === 0) {
                alert('No data to export. Please record some data first.');
                return;
            }

            try {
                // --- Sheet 1: Raw Data ---
                const wsData = [['Date','Time','T1 (°C)','T2 (°C)','T3 (°C)','T4 (°C)','T5 (°C)','TM (°C)','Humidity (%)','Airflow (m/s)','Noise (dB)']];
                tableData.forEach(row => {
                    wsData.push([
                        row.date, row.time,
                        +row.t1.toFixed(2), +row.t2.toFixed(2), +row.t3.toFixed(2),
                        +row.t4.toFixed(2), +row.t5.toFixed(2), +row.tm.toFixed(2),
                        +row.rh.toFixed(2), +row.flow.toFixed(2), +row.noise.toFixed(2)
                    ]);
                });

                // --- Sheet 2: Analisis Statistik ---
                const statsMap = {};
                const analysisData = [['ANALISIS STATISTIK'],[],['Parameter','Minimal','Maksimal','STDEV','Mean'],[]];

                ['T1','T2','T3','T4','T5'].forEach(s => {
                    const vals = tableData.map(r => r[s.toLowerCase()]).filter(v => v > 0);
                    if (vals.length) {
                        const st = calculateStats(vals);
                        statsMap[s] = st;
                        analysisData.push([s, +st.min.toFixed(2), +st.max.toFixed(2), +st.stdev.toFixed(2), +st.mean.toFixed(2)]);
                    }
                });

                analysisData.push([]);

                const otherParams = [
                    { name: 'Kelembapan',      key: 'rh' },
                    { name: 'TM (Suhu Matras)', key: 'tm' },
                    { name: 'Airflow',          key: 'flow' },
                    { name: 'Kebisingan',       key: 'noise' },
                ];
                otherParams.forEach(p => {
                    const vals = tableData.map(r => r[p.key]).filter(v => v > 0);
                    if (vals.length) {
                        const st = calculateStats(vals);
                        statsMap[p.name] = st;
                        analysisData.push([p.name, +st.min.toFixed(2), +st.max.toFixed(2), +st.stdev.toFixed(2), +st.mean.toFixed(2)]);
                    }
                });

                // --- Sheet 3: Uncertainty ---
                const uncertaintyData = [
                    ['UNCERTAINTY ANALYSIS'],[],
                    ['TABEL NILAI KETIDAKPASTIAN'],
                    ['Sensor','Suhu 32°C','Suhu 36°C'],
                    ['T1',-0.034,-0.005],['T2',-0.034,0.145],
                    ['T3', 0.006, 0.065],['T4', 0.066,0.135],['T5',-0.024,0.055],
                    [],[],
                    ['TABEL ANALISIS LENGKAP'],
                    ['Setting Alat','STDEV','Mean','Mean Terkoreksi','Koreksi','U95','Koreksi + U95','Toleransi','Hasil'],
                ];

                const ucVals = selectedTemp === 32
                    ? { T1:-0.034, T2:-0.034, T3:0.006, T4:0.066, T5:-0.024 }
                    : { T1:-0.005, T2: 0.145, T3:0.065, T4:0.135, T5: 0.055 };

                const correctedMeans = {};
                ['T1','T2','T3','T4','T5'].forEach(s => {
                    if (statsMap[s]) correctedMeans[s] = statsMap[s].mean + ucVals[s];
                });

                ['T1','T2','T3','T4','T5'].forEach(s => {
                    if (!statsMap[s]) return;
                    const stdev = +statsMap[s].stdev.toFixed(2);
                    const mean  = +statsMap[s].mean.toFixed(2);
                    const meanC = +correctedMeans[s].toFixed(2);

                    if (s === 'T5') {
                        const safeMin = selectedTemp === 32 ? 30.5 : 34.5;
                        const safeMax = selectedTemp === 32 ? 33.5 : 37.5;
                        uncertaintyData.push([s, stdev, mean, meanC, '', '', '', '± 1.5',
                            mean >= safeMin && mean <= safeMax ? 'LOLOS' : 'TIDAK LOLOS']);
                    } else {
                        const corr  = +(correctedMeans[s] - correctedMeans['T5']).toFixed(2);
                        const u95   = 0.52;
                        const cpU95 = +(Math.abs(corr) + Math.abs(u95)).toFixed(2);
                        uncertaintyData.push([s, stdev, mean, meanC, corr, u95, cpU95, 0.8,
                            cpU95 < 0.8 ? 'LOLOS' : 'TIDAK LOLOS']);
                    }
                });

                if (statsMap['Kelembapan']) {
                    const st   = statsMap['Kelembapan'];
                    const mean = +st.mean.toFixed(2);
                    uncertaintyData.push(['Kelembapan', +st.stdev.toFixed(2), mean, '', mean, '', '', '50-60',
                        mean >= 50 && mean <= 60 ? 'LOLOS' : 'TIDAK LOLOS']);
                }

                [
                    { name:'Airflow',          tol:0.35, key:'Airflow' },
                    { name:'Kebisingan',        tol:60,   key:'Kebisingan' },
                    { name:'Temperatur Matras', tol:40,   key:'TM (Suhu Matras)' },
                ].forEach(p => {
                    if (!statsMap[p.key]) return;
                    const st   = statsMap[p.key];
                    const mean = +st.mean.toFixed(2);
                    uncertaintyData.push([p.name, +st.stdev.toFixed(2), mean, '', mean, '', '', p.tol,
                        Math.abs(mean) < p.tol ? 'LOLOS' : 'TIDAK LOLOS']);
                });

                // Build workbook
                const wb = XLSX.utils.book_new();

                const ws1 = XLSX.utils.aoa_to_sheet(wsData);
                ws1['!cols'] = [{wch:12},{wch:12},{wch:10},{wch:10},{wch:10},{wch:10},{wch:10},{wch:10},{wch:12},{wch:12},{wch:10}];

                const ws2 = XLSX.utils.aoa_to_sheet(analysisData);
                ws2['!cols'] = [{wch:20},{wch:12},{wch:12},{wch:12},{wch:12}];

                const ws3 = XLSX.utils.aoa_to_sheet(uncertaintyData);
                ws3['!cols'] = [{wch:18},{wch:12},{wch:12},{wch:16},{wch:12},{wch:12},{wch:15},{wch:12},{wch:12}];

                XLSX.utils.book_append_sheet(wb, ws1, 'Raw Data');
                XLSX.utils.book_append_sheet(wb, ws2, 'Analisis Statistik');
                XLSX.utils.book_append_sheet(wb, ws3, 'Uncertainty');

                const now = new Date();
                const fname = `INCU_Data_${selectedTemp}C_${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}_${String(now.getHours()).padStart(2,'0')}${String(now.getMinutes()).padStart(2,'0')}.xlsx`;
                XLSX.writeFile(wb, fname);
                alert(`Data exported successfully for ${selectedTemp}°C setting!`);
            } catch (error) {
                console.error('Export error:', error);
                alert('Error exporting data: ' + error.message);
            }
        }

        // ─── UI Events ───────────────────────────────────────────────────────────
        document.getElementById('saveBtn').addEventListener('click', () => {
            if (isRecording) stopRecording(); else startRecording();
        });
        document.getElementById('resetBtn').addEventListener('click', resetData);
        document.getElementById('exportBtn').addEventListener('click', exportToExcel);

        document.getElementById('timerDisplay').textContent =
            document.getElementById('timerInput').value || '00:00:00';

        document.getElementById('timerInput').addEventListener('change', e => {
            if (!isRecording)
                document.getElementById('timerDisplay').textContent = e.target.value || '00:00:00';
        });

        console.log('INCU Analyzer initialized (fixed build)');
    </script>
</body>
</html>
