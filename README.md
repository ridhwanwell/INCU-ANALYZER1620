<!DOCTYPE html>
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

        /* Battery Section */
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

        /* Alarm Section */
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

        /* Sensor Section */
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
        <button id="exportBtn" class="control-btn">Export to Spreadsheet</button>
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

    <!-- Battery Status Section -->
    <div class="battery-section">
        <h2>Battery Status</h2>
        <div class="battery-grid">
            <div class="battery-box" id="batteryCenter">
                <h4>Central Unit</h4>
                <div class="battery-value" id="batteryCenterValue">100%</div>
            </div>
            <div class="battery-box" id="batteryNode1">
                <h4>Sensor Node 1</h4>
                <div class="battery-value" id="batteryNode1Value">100%</div>
            </div>
            <div class="battery-box" id="batteryNode2">
                <h4>Sensor Node 2</h4>
                <div class="battery-value" id="batteryNode2Value">100%</div>
            </div>
            <div class="battery-box" id="batteryNode3">
                <h4>Sensor Node 3</h4>
                <div class="battery-value" id="batteryNode3Value">100%</div>
            </div>
            <div class="battery-box" id="batteryNode4">
                <h4>Sensor Node 4</h4>
                <div class="battery-value" id="batteryNode4Value">100%</div>
            </div>
        </div>
    </div>

    <!-- Alarm Section -->
    <div class="alarm-section">
        <div class="alarm-box" id="alarmBox">
            <h3>System Status</h3>
            <div class="alarm-status" id="alarmStatus">Normal - No Alerts</div>
            <div class="alarm-messages" id="alarmMessages"></div>
        </div>
    </div>

    <!-- Sensor Data Section -->
    <div class="sensor-section">
        <h2>Data Sensor</h2>
        <div class="sensor-grid">
            <div class="sensor-box" id="t1Box">
                <h3>T1</h3>
                <div class="sensor-value" id="t1Value">0.0 °C</div>
            </div>
            <div class="sensor-box" id="t2Box">
                <h3>T2</h3>
                <div class="sensor-value" id="t2Value">0.0 °C</div>
            </div>
            <div class="sensor-box" id="t3Box">
                <h3>T3</h3>
                <div class="sensor-value" id="t3Value">0.0 °C</div>
            </div>
            <div class="sensor-box" id="t4Box">
                <h3>T4</h3>
                <div class="sensor-value" id="t4Value">0.0 °C</div>
            </div>
            <div class="sensor-box" id="t5Box">
                <h3>T5</h3>
                <div class="sensor-value" id="t5Value">0.0 °C</div>
            </div>
            <div class="sensor-box" id="tmBox">
                <h3>TM</h3>
                <div class="sensor-value" id="tmValue">0.0 °C</div>
            </div>
            <div class="sensor-box" id="flowBox">
                <h3>Flow</h3>
                <div class="sensor-value" id="flowValue">0.0 m/s</div>
            </div>
            <div class="sensor-box" id="noiseBox">
                <h3>Noise</h3>
                <div class="sensor-value" id="noiseValue">0.0 dB</div>
            </div>
            <div class="sensor-box" id="rhBox">
                <h3>RH</h3>
                <div class="sensor-value" id="rhValue">0.0 %</div>
            </div>
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
                    <th>Flow (m/s)</th>
                    <th>Noise (dB)</th>
                    <th>RH (%)</th>
                </tr>
            </thead>
            <tbody id="dataTableBody"></tbody>
        </table>
    </div>

    <script>
        // MQTT Configuration
        const options = {
            protocol: 'ws',
            hostname: 'broker.hivemq.com',
            port: 8000,
            path: '/mqtt',
            clean: true,
            connectTimeout: 4000,
            reconnectPeriod: 1000,
            clientId: 'incu_analyzer_' + Math.random().toString(16).substr(2, 8)
        };

        const client = mqtt.connect(options);
        const topic = 'incu/sensors';
        let isRecording = false;
        let timerInterval;
        let dataInterval;
        let remainingTime;
        let tableData = [];
        let currentSensorData = {};

        // MQTT Connection handling
        client.on('connect', () => {
            console.log('Connected to MQTT broker');
            document.getElementById('mqttStatus').className = 'mqtt-status connected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Connected to broker.hivemq.com';
            client.subscribe(topic, (err) => {
                if (!err) {
                    console.log('Subscribed to topic:', topic);
                } else {
                    console.error('Subscription error:', err);
                }
            });
        });

        client.on('error', (error) => {
            console.error('MQTT Error:', error);
            document.getElementById('mqttStatus').className = 'mqtt-status disconnected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Error - ' + error.message;
        });

        client.on('offline', () => {
            document.getElementById('mqttStatus').className = 'mqtt-status disconnected';
            document.getElementById('mqttStatus').textContent = 'MQTT Status: Offline';
        });

        client.on('reconnect', () => {
            console.log('Reconnecting to MQTT broker...');
        });

        // Handle incoming MQTT messages
        client.on('message', (receivedTopic, message) => {
            try {
                const data = JSON.parse(message.toString());
                console.log('Received data:', data);
                currentSensorData = data;
                updateSensorValues(data);
                updateBatteryStatus(data);
                checkAlarms(data);
                highlightUpdatedValues();
            } catch (e) {
                console.error('Error parsing MQTT message:', e);
            }
        });

        function highlightUpdatedValues() {
            const boxes = document.querySelectorAll('.sensor-box');
            boxes.forEach(box => {
                box.classList.add('updated');
                setTimeout(() => box.classList.remove('updated'), 1000);
            });
        }

        function updateSensorValues(data) {
            if (data.t1 !== undefined) document.getElementById('t1Value').textContent = `${parseFloat(data.t1).toFixed(1)} °C`;
            if (data.t2 !== undefined) document.getElementById('t2Value').textContent = `${parseFloat(data.t2).toFixed(1)} °C`;
            if (data.t3 !== undefined) document.getElementById('t3Value').textContent = `${parseFloat(data.t3).toFixed(1)} °C`;
            if (data.t4 !== undefined) document.getElementById('t4Value').textContent = `${parseFloat(data.t4).toFixed(1)} °C`;
            if (data.t5 !== undefined) document.getElementById('t5Value').textContent = `${parseFloat(data.t5).toFixed(1)} °C`;
            if (data.tm !== undefined) document.getElementById('tmValue').textContent = `${parseFloat(data.tm).toFixed(1)} °C`;
            if (data.flow !== undefined) document.getElementById('flowValue').textContent = `${parseFloat(data.flow).toFixed(1)} m/s`;
            if (data.noise !== undefined) document.getElementById('noiseValue').textContent = `${parseFloat(data.noise).toFixed(1)} dB`;
            if (data.rh !== undefined) document.getElementById('rhValue').textContent = `${parseFloat(data.rh).toFixed(1)} %`;
        }

        function updateBatteryStatus(data) {
            // Update central unit battery
            if (data.battery_center !== undefined) {
                const batteryCenter = parseFloat(data.battery_center);
                document.getElementById('batteryCenterValue').textContent = `${batteryCenter.toFixed(0)}%`;
                updateBatteryBox('batteryCenter', batteryCenter);
            }

            // Update node batteries
            if (data.battery_node1 !== undefined) {
                const batteryNode1 = parseFloat(data.battery_node1);
                document.getElementById('batteryNode1Value').textContent = `${batteryNode1.toFixed(0)}%`;
                updateBatteryBox('batteryNode1', batteryNode1);
            }

            if (data.battery_node2 !== undefined) {
                const batteryNode2 = parseFloat(data.battery_node2);
                document.getElementById('batteryNode2Value').textContent = `${batteryNode2.toFixed(0)}%`;
                updateBatteryBox('batteryNode2', batteryNode2);
            }

            if (data.battery_node3 !== undefined) {
                const batteryNode3 = parseFloat(data.battery_node3);
                document.getElementById('batteryNode3Value').textContent = `${batteryNode3.toFixed(0)}%`;
                updateBatteryBox('batteryNode3', batteryNode3);
            }

            if (data.battery_node4 !== undefined) {
                const batteryNode4 = parseFloat(data.battery_node4);
                document.getElementById('batteryNode4Value').textContent = `${batteryNode4.toFixed(0)}%`;
                updateBatteryBox('batteryNode4', batteryNode4);
            }
        }

        function updateBatteryBox(boxId, percentage) {
            const box = document.getElementById(boxId);
            if (percentage < 20) {
                box.classList.add('low');
            } else {
                box.classList.remove('low');
            }
        }

        function checkAlarms(data) {
            const alarmBox = document.getElementById('alarmBox');
            const alarmStatus = document.getElementById('alarmStatus');
            const alarmMessages = document.getElementById('alarmMessages');
            
            let alarms = [];
            let hasError = false;

            // Check temperature difference (T1-T5)
            const temps = [data.t1, data.t2, data.t3, data.t4, data.t5].filter(t => t !== undefined).map(t => parseFloat(t));
            
            if (temps.length >= 2) {
                const maxTemp = Math.max(...temps);
                const minTemp = Math.min(...temps);
                const tempDiff = maxTemp - minTemp;

                if (tempDiff > 2.0) {
                    hasError = true;
                    alarms.push(`⚠️ Temperature variance detected: ${tempDiff.toFixed(1)}°C difference (Max: ${maxTemp.toFixed(1)}°C, Min: ${minTemp.toFixed(1)}°C)`);
                    
                    // Highlight problematic sensors
                    ['t1', 't2', 't3', 't4', 't5'].forEach((sensor, idx) => {
                        const temp = temps[idx];
                        if (temp !== undefined && (temp === maxTemp || temp === minTemp)) {
                            document.getElementById(sensor + 'Box').classList.add('error');
                        } else {
                            document.getElementById(sensor + 'Box').classList.remove('error');
                        }
                    });
                } else {
                    // Remove error highlighting if temperature is normal
                    ['t1', 't2', 't3', 't4', 't5'].forEach(sensor => {
                        document.getElementById(sensor + 'Box').classList.remove('error');
                    });
                }
            }

            // Check for sensor failures (reading 0 or null)
            const sensorChecks = {
                't1': data.t1, 't2': data.t2, 't3': data.t3, 't4': data.t4, 't5': data.t5,
                'tm': data.tm, 'flow': data.flow, 'noise': data.noise, 'rh': data.rh
            };

            Object.entries(sensorChecks).forEach(([sensor, value]) => {
                if (value === undefined || value === null || value === 0) {
                    hasError = true;
                    alarms.push(`⚠️ ${sensor.toUpperCase()} sensor failure detected`);
                    document.getElementById(sensor + 'Box').classList.add('error');
                }
            });

            // Check battery levels
            const batteries = {
                'Central Unit': data.battery_center,
                'Node 1': data.battery_node1,
                'Node 2': data.battery_node2,
                'Node 3': data.battery_node3,
                'Node 4': data.battery_node4
            };

            Object.entries(batteries).forEach(([name, level]) => {
                if (level !== undefined && parseFloat(level) < 20) {
                    hasError = true;
                    alarms.push(`🔋 Low battery warning: ${name} at ${parseFloat(level).toFixed(0)}%`);
                }
            });

            // Update alarm display
            if (hasError) {
                alarmBox.classList.add('alarm-active');
                alarmStatus.textContent = 'ALERT - System Issues Detected!';
                alarmMessages.innerHTML = alarms.join('<br>');
            } else {
                alarmBox.classList.remove('alarm-active');
                alarmStatus.textContent = 'Normal - No Alerts';
                alarmMessages.innerHTML = '';
            }
        }

        function addTableRow(data) {
            const now = new Date();
            const row = {
                date: now.toLocaleDateString('id-ID'),
                time: now.toLocaleTimeString('id-ID'),
                t1: parseFloat(data.t1 || 0),
                t2: parseFloat(data.t2 || 0),
                t3: parseFloat(data.t3 || 0),
                t4: parseFloat(data.t4 || 0),
                t5: parseFloat(data.t5 || 0),
                tm: parseFloat(data.tm || 0),
                flow: parseFloat(data.flow || 0),
                noise: parseFloat(data.noise || 0),
                rh: parseFloat(data.rh || 0)
            };
            
            tableData.push(row);
            
            const tbody = document.getElementById('dataTableBody');
            const tr = document.createElement('tr');
            tr.classList.add('new-row');
            tr.innerHTML = `
                <td>${row.date}</td>
                <td>${row.time}</td>
                <td>${row.t1.toFixed(1)}</td>
                <td>${row.t2.toFixed(1)}</td>
                <td>${row.t3.toFixed(1)}</td>
                <td>${row.t4.toFixed(1)}</td>
                <td>${row.t5.toFixed(1)}</td>
                <td>${row.tm.toFixed(1)}</td>
                <td>${row.flow.toFixed(1)}</td>
                <td>${row.noise.toFixed(1)}</td>
                <td>${row.rh.toFixed(1)}</td>
            `;
            
            tbody.insertBefore(tr, tbody.firstChild);
        }

        function parseTimerInput(timeString) {
            const parts = timeString.split(':');
            if (parts.length !== 3) return 60;
            
            const hours = parseInt(parts[0]) || 0;
            const minutes = parseInt(parts[1]) || 0;
            const seconds = parseInt(parts[2]) || 0;
            
            return (hours * 3600) + (minutes * 60) + seconds;
        }

        function formatTime(seconds) {
            const h = Math.floor(seconds / 3600);
            const m = Math.floor((seconds % 3600) / 60);
            const s = seconds % 60;
            return `${String(h).padStart(2, '0')}:${String(m).padStart(2, '0')}:${String(s).padStart(2, '0')}`;
        }

        function startRecording() {
            if (isRecording) return;
            
            const timerInput = document.getElementById('timerInput').value;
            const intervalSeconds = parseInt(document.getElementById('intervalInput').value) || 2;
            
            remainingTime = parseTimerInput(timerInput);
            if (remainingTime <= 0) {
                alert('Please enter a valid timer duration');
                return;
            }

            isRecording = true;
            document.getElementById('saveBtn').classList.add('active');
            document.getElementById('saveBtn').textContent = 'Stop Saving Data';
            document.getElementById('intervalInput').disabled = true;
            document.getElementById('timerInput').disabled = true;

            // Update timer display
            timerInterval = setInterval(() => {
                remainingTime--;
                document.getElementById('timerDisplay').textContent = formatTime(remainingTime);
                
                if (remainingTime <= 0) {
                    stopRecording();
                }
            }, 1000);

            // Save data at specified intervals
            dataInterval = setInterval(() => {
                if (Object.keys(currentSensorData).length > 0) {
                    addTableRow(currentSensorData);
                }
            }, intervalSeconds * 1000);

            console.log('Recording started');
        }

        function stopRecording() {
            isRecording = false;
            clearInterval(timerInterval);
            clearInterval(dataInterval);
            
            document.getElementById('saveBtn').classList.remove('active');
            document.getElementById('saveBtn').textContent = 'Play Saving Data';
            document.getElementById('intervalInput').disabled = false;
            document.getElementById('timerInput').disabled = false;
            
            console.log('Recording stopped');
        }

        function resetData() {
            if (confirm('Are you sure you want to reset all data? This action cannot be undone.')) {
                tableData = [];
                document.getElementById('dataTableBody').innerHTML = '';
                
                if (isRecording) {
                    stopRecording();
                }
                
                document.getElementById('timerDisplay').textContent = '00:00:00';
                console.log('Data reset completed');
            }
        }

        function exportToSpreadsheet() {
            if (tableData.length === 0) {
                alert('No data to export. Please record some data first.');
                return;
            }

            try {
                // Prepare data for Excel
                const wsData = [
                    ['Date', 'Time', 'T1 (°C)', 'T2 (°C)', 'T3 (°C)', 'T4 (°C)', 'T5 (°C)', 'TM (°C)', 'Flow (m/s)', 'Noise (dB)', 'RH (%)']
                ];

                tableData.forEach(row => {
                    wsData.push([
                        row.date,
                        row.time,
                        row.t1.toFixed(1),
                        row.t2.toFixed(1),
                        row.t3.toFixed(1),
                        row.t4.toFixed(1),
                        row.t5.toFixed(1),
                        row.tm.toFixed(1),
                        row.flow.toFixed(1),
                        row.noise.toFixed(1),
                        row.rh.toFixed(1)
                    ]);
                });

                // Create workbook and worksheet
                const wb = XLSX.utils.book_new();
                const ws = XLSX.utils.aoa_to_sheet(wsData);

                // Set column widths
                ws['!cols'] = [
                    { wch: 12 }, // Date
                    { wch: 12 }, // Time
                    { wch: 10 }, // T1
                    { wch: 10 }, // T2
                    { wch: 10 }, // T3
                    { wch: 10 }, // T4
                    { wch: 10 }, // T5
                    { wch: 10 }, // TM
                    { wch: 12 }, // Flow
                    { wch: 12 }, // Noise
                    { wch: 10 }  // RH
                ];

                // Add worksheet to workbook
                XLSX.utils.book_append_sheet(wb, ws, 'INCU Data');

                // Generate filename with timestamp
                const now = new Date();
                const filename = `INCU_Data_${now.getFullYear()}${String(now.getMonth()+1).padStart(2,'0')}${String(now.getDate()).padStart(2,'0')}_${String(now.getHours()).padStart(2,'0')}${String(now.getMinutes()).padStart(2,'0')}.xlsx`;

                // Export file
                XLSX.writeFile(wb, filename);
                
                console.log('Data exported successfully to:', filename);
                alert('Data exported successfully!');
            } catch (error) {
                console.error('Export error:', error);
                alert('Error exporting data: ' + error.message);
            }
        }

        // Event Listeners
        document.getElementById('saveBtn').addEventListener('click', () => {
            if (isRecording) {
                stopRecording();
            } else {
                startRecording();
            }
        });

        document.getElementById('resetBtn').addEventListener('click', resetData);
        document.getElementById('exportBtn').addEventListener('click', exportToSpreadsheet);

        // Initialize timer display
        document.getElementById('timerDisplay').textContent = document.getElementById('timerInput').value || '00:00:00';

        // Update timer display when input changes
        document.getElementById('timerInput').addEventListener('change', (e) => {
            if (!isRecording) {
                document.getElementById('timerDisplay').textContent = e.target.value || '00:00:00';
            }
        });

        console.log('INCU Analyzer initialized');
        console.log('Connecting to MQTT broker: broker.hivemq.com:8000');
        console.log('Topic:', topic);
    </script>
</body>
</html>
