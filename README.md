<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>VPN Service</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
        }

        .container {
            max-width: 400px;
            margin: 0 auto;
            background: white;
            border-radius: 15px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }

        .header {
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            color: white;
            padding: 30px 20px;
            text-align: center;
        }

        .header h1 {
            font-size: 24px;
            margin-bottom: 5px;
        }

        .header p {
            opacity: 0.9;
            font-size: 14px;
        }

        .status {
            padding: 20px;
            text-align: center;
            border-bottom: 1px solid #eee;
        }

        .status-indicator {
            width: 12px;
            height: 12px;
            background: #ff4757;
            border-radius: 50%;
            display: inline-block;
            margin-right: 10px;
            animation: pulse 2s infinite;
        }

        .status.connected .status-indicator {
            background: #2ed573;
        }

        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }

        .servers {
            padding: 20px;
        }

        .server-list {
            list-style: none;
        }

        .server-item {
            padding: 15px;
            border: 2px solid #f1f2f6;
            border-radius: 10px;
            margin-bottom: 10px;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
        }

        .server-item:hover {
            border-color: #4facfe;
            background: #f8fafb;
        }

        .server-item.active {
            border-color: #4facfe;
            background: #e3f2fd;
        }

        .flag {
            width: 24px;
            height: 24px;
            border-radius: 50%;
            background: #ddd;
            margin-right: 15px;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 12px;
        }

        .server-info {
            flex: 1;
        }

        .server-name {
            font-weight: 600;
            margin-bottom: 2px;
        }

        .server-ping {
            font-size: 12px;
            color: #666;
        }

        .connect-btn {
            display: block;
            width: 100%;
            padding: 15px;
            background: linear-gradient(135deg, #4facfe 0%, #00f2fe 100%);
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-top: 20px;
        }

        .connect-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(79, 172, 254, 0.3);
        }

        .connect-btn:active {
            transform: translateY(0);
        }

        .stats {
            padding: 20px;
            background: #f8fafb;
            border-top: 1px solid #eee;
        }

        .stat-item {
            display: flex;
            justify-content: space-between;
            margin-bottom: 8px;
            font-size: 14px;
        }

        .stat-value {
            font-weight: 600;
            color: #2f3542;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Secure VPN</h1>
            <p>Защитите свою приватность</p>
        </div>

        <div class="status" id="status">
            <div class="status-indicator"></div>
            <span id="status-text">Не подключено</span>
        </div>

        <div class="servers">
            <h3 style="margin-bottom: 15px;">Выберите сервер</h3>
            <ul class="server-list">
                <li class="server-item active" data-server="usa">
                    <div class="flag">🇺🇸</div>
                    <div class="server-info">
                        <div class="server-name">США - Нью-Йорк</div>
                        <div class="server-ping">Пинг: 45ms</div>
                    </div>
                </li>
                <li class="server-item" data-server="germany">
                    <div class="flag">🇩🇪</div>
                    <div class="server-info">
                        <div class="server-name">Германия - Франкфурт</div>
                        <div class="server-ping">Пинг: 28ms</div>
                    </div>
                </li>
                <li class="server-item" data-server="japan">
                    <div class="flag">🇯🇵</div>
                    <div class="server-info">
                        <div class="server-name">Япония - Токио</div>
                        <div class="server-ping">Пинг: 185ms</div>
                    </div>
                </li>
                <li class="server-item" data-server="singapore">
                    <div class="flag">🇸🇬</div>
                    <div class="server-info">
                        <div class="server-name">Сингапур</div>
                        <div class="server-ping">Пинг: 210ms</div>
                    </div>
                </li>
            </ul>

            <button class="connect-btn" id="connectBtn">Подключиться</button>
        </div>

        <div class="stats">
            <div class="stat-item">
                <span>Загружено:</span>
                <span class="stat-value" id="upload">0 MB</span>
            </div>
            <div class="stat-item">
                <span>Скачано:</span>
                <span class="stat-value" id="download">0 MB</span>
            </div>
            <div class="stat-item">
                <span>Длительность:</span>
                <span class="stat-value" id="duration">00:00:00</span>
            </div>
        </div>
    </div>

    <script>
        let isConnected = false;
        let startTime = null;
        let timerInterval = null;

        const connectBtn = document.getElementById('connectBtn');
        const statusDiv = document.getElementById('status');
        const statusText = document.getElementById('status-text');
        const serverItems = document.querySelectorAll('.server-item');
        const uploadEl = document.getElementById('upload');
        const downloadEl = document.getElementById('download');
        const durationEl = document.getElementById('duration');

        let currentServer = 'usa';

        // Выбор сервера
        serverItems.forEach(item => {
            item.addEventListener('click', () => {
                serverItems.forEach(i => i.classList.remove('active'));
                item.classList.add('active');
                currentServer = item.dataset.server;
            });
        });

        // Подключение/отключение
        connectBtn.addEventListener('click', () => {
            if (!isConnected) {
                // Имитация подключения
                connectBtn.textContent = 'Подключение...';
                connectBtn.disabled = true;

                setTimeout(() => {
                    isConnected = true;
                    statusDiv.classList.add('connected');
                    statusText.textContent = 'Подключено - ' + getServerName(currentServer);
                    connectBtn.textContent = 'Отключиться';
                    connectBtn.disabled = false;
                    
                    // Запуск таймера
                    startTime = new Date();
                    startTimer();
                    
                    // Имитация трафика
                    simulateTraffic();
                }, 2000);
            } else {
                // Отключение
                isConnected = false;
                statusDiv.classList.remove('connected');
                statusText.textContent = 'Не подключено';
                connectBtn.textContent = 'Подключиться';
                
                // Остановка таймера
                clearInterval(timerInterval);
                durationEl.textContent = '00:00:00';
            }
        });

        function getServerName(serverCode) {
            const names = {
                'usa': 'США - Нью-Йорк',
                'germany': 'Германия - Франкфурт',
                'japan': 'Япония - Токио',
                'singapore': 'Сингапур'
            };
            return names[serverCode] || 'Неизвестно';
        }

        function startTimer() {
            clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                const now = new Date();
                const diff = now - startTime;
                const hours = Math.floor(diff / 3600000);
                const minutes = Math.floor((diff % 3600000) / 60000);
                const seconds = Math.floor((diff % 60000) / 1000);
                
                durationEl.textContent = 
                    `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
            }, 1000);
        }

        function simulateTraffic() {
            let upload = 0;
            let download = 0;
            
            setInterval(() => {
                if (isConnected) {
                    upload += Math.random() * 0.5;
                    download += Math.random() * 0.8;
                    
                    uploadEl.textContent = upload.toFixed(1) + ' MB';
                    downloadEl.textContent = download.toFixed(1) + ' MB';
                }
            }, 1000);
        }
    </script>
</body>
</html>
