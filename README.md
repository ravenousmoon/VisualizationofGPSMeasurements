<h1> <b> Лабораторна робота №5. Розробка додатку для візуалізації вимірювань GPS </b> </h1>

<b>Мета:</b> Розробити додаток, який зчитує дані з емульованої вимірювальної частини GPS, наданої у вигляді Docker image, та відображає положення об'єкта і супутників на графіку в декартових координатах.

<p><b>Завдання:</b></p>
<ol>
    <li>
        <b>Розробити додаток для відображення положення об'єкта та супутників:</b>
        <ul>
            <li>Розробити веб-додаток, який підключається до WebSocket сервера та зчитує дані про положення супутників і об'єкта.</li>
            <li>Відобразити отримані дані на графіку в декартових координатах. Для цього можна використати бібліотеку Plotly або іншу бібліотеку для роботи з графіками.</li>
        </ul>
    </li>
    <li>
        <b>Обробка та візуалізація даних:</b>
        <ul>
            <li>Обробити дані, отримані через WebSocket, і відобразити положення об'єкта та супутників на графіку.</li>
            <li>Додати можливість зміни параметрів вимірювальної частини GPS за допомогою API запитів.</li>
        </ul>
    </li>
    <li>
        <b>Налаштування графіка:</b>
        <ul>
            <li>Відобразити координати супутників та об'єкта у декартових координатах.</li>
            <li>Використати різні кольори або стилі точок для відображення супутників та об'єкта.</li>
        </ul>
    </li>
</ol>

<p> <b>Передумови виконання:</b> для виконання лабораторної роботи необхідно запустити емулятор вимірювальної частини GPS. Емулятор надається у вигляді Docker-образу під назвою <b>iperekrestov/university/gps-emulation-service</b>. Щоб запустити емулятор, завантажуємо Docker-образ з Docker Hub:</p>
<pre><code>docker pull iperekrestov/university:gps-emulation-service</code></pre>
<p>Запускаємо Docker-контейнер за допомогою команди:</p>
<pre><code>docker run --name gps-emulator -p 4001:4000 iperekrestov/university:gps-emulation-service</code></pre>
<p>Ця команда відкриває порт <b>4001</b> на хост-машині, який відображається на порт <b>4000</b> всередині контейнера, для з'єднання з емульованою вимірювальною частиною GPS.</p>

<p><b>Розробка додатку для відображення положення об'єкта та супутників:</b> Додаток, написаний на <b>HTML</b> та <b>JavaScript</b>, відображає положення супутників та об’єкта в режимі реального часу на декартовій системі координат. Код складається з HTML-розмітки для інтерфейсу та JavaScript-скрипта для встановлення WebSocket-з'єднання, обробки даних і оновлення графіка.</p>
<p>Панель стану (status) відображає стан підключення до WebSocket (<i>підключено/відключено</i>).</p>
<p>Панель керування надає інтерфейс для зміни налаштувань емуляції, зокрема частоти повідомлень, швидкості супутників та швидкості об’єкта. Зміни передаються через API.</p>
<p>Графік Plotly (gpsPlot) показує поточні позиції супутників і об'єкта.</p>
<p>Панель інформації про об’єкт (objectInfo) відображає розраховані координати об’єкта та кількість супутників, які використовуються для обчислення його позиції.</p>

<p><b>Обробка та візуалізація даних:</b> включає отримання інформації від емульованої частини GPS через WebSocket, де передаються координати супутників і об'єкта. Ці дані використовуються для оновлення положення супутників, а за допомогою методу трилатерації обчислюється позиція об'єкта. Якщо є достатньо супутників (не менше трьох), позиція об'єкта визначається на основі відстані до кожного супутника. Дані обробляються в реальному часі, щоб відображати точне місце розташування об'єкта і супутників.</p>

<p><b>Налаштування графіка:</b> включає відображення координат об'єкта та супутників на декартовій площині, використовуючи бібліотеку <b>Plotly</b>. Ось X і Y обмежені від -100 до 100 км, а маркери для супутників і об'єкта відрізняються за кольором і формою, щоб забезпечити чітке розрізнення. Графік автоматично оновлюється, відображаючи поточні координати, і дозволяє відстежувати зміни в реальному часі.</p>

<p>Створюємо додаток для відображення положення об'єкта та супутників:</p>

``` html
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <title>GPS Трекінг</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/plotly.js/2.27.1/plotly.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f0;
            margin: 0;
            padding: 0;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }

        .controls {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-bottom: 20px;
            padding: 15px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .controls label {
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .controls input {
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            width: 100px;
        }

        .controls button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 4px;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        #status {
            padding: 10px;
            font-weight: bold;
            text-align: center;
            margin-bottom: 20px;
            border-radius: 4px;
        }

        .connected { background-color: #4CAF50; color: white; }
        .disconnected { background-color: #f44336; color: white; }

        #gpsPlot {
            width: 100%;
            height: 600px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .info-panel {
            margin-top: 20px;
            padding: 15px;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        #objectInfo {
            margin-top: 10px;
            padding: 10px;
            background-color: #e3f2fd;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div id="status" class="disconnected">Статус: Відключено</div>
        
        <div class="controls">
            <label>
                Частота повідомлень (повід./сек):
                <input type="number" id="messageFrequency" value="2" min="1" max="10">
            </label>
            <label>
                Швидкість супутників (км/год):
                <input type="number" id="satelliteSpeed" value="120" min="10" max="1000">
            </label>
            <label>
                Швидкість об'єкта (км/год):
                <input type="number" id="objectSpeed" value="20" min="1" max="200">
            </label>
            <button onclick="updateConfig()">Оновити конфігурацію</button>
        </div>

        <div id="gpsPlot"></div>
        
        <div class="info-panel">
            <h3>Розрахована позиція об'єкта</h3>
            <div id="objectInfo"></div>
        </div>
    </div>

    <script>
        let socket;
        let satellites = new Map();
        let calculatedPosition = null;
        
        const layout = {
            title: 'GPS Трекінг',
            showlegend: true,
            xaxis: {
                title: 'X координата (км)',
                range: [-100, 100]
            },
            yaxis: {
                title: 'Y координата (км)',
                range: [-100, 100]
            }
        };

        Plotly.newPlot('gpsPlot', [], layout);

        function connectWebSocket() {
            socket = new WebSocket('ws://localhost:4001');

            socket.onopen = () => {
                document.getElementById('status').className = 'connected';
                document.getElementById('status').textContent = 'Статус: Підключено';
            };

            socket.onmessage = (event) => {
                const data = JSON.parse(event.data);
                processGPSData(data);
            };

            socket.onclose = () => {
                document.getElementById('status').className = 'disconnected';
                document.getElementById('status').textContent = 'Статус: Відключено';
                setTimeout(connectWebSocket, 5000);
            };

            socket.onerror = (error) => {
                console.error('Помилка WebSocket:', error);
            };
        }

        function calculateDistance(satellite, receivedAt, sentAt) {
            const timeOfFlight = receivedAt - sentAt; // миллисекунды
            const speedOfLight = 299792.458; // км/с
            return (timeOfFlight / 1000) * speedOfLight;
        }

        function trilaterate(satellites) {
            if (satellites.size < 3) return null;

            let sats = Array.from(satellites.values());
            // Берем первые три спутника для расчета
            let [p1, p2, p3] = sats.slice(0, 3);

            // Расчет расстояний
            let r1 = calculateDistance(p1, p1.receivedAt, p1.sentAt);
            let r2 = calculateDistance(p2, p2.receivedAt, p2.sentAt);
            let r3 = calculateDistance(p3, p3.receivedAt, p3.sentAt);

            // Система уравнений для трилатерации
            let A = 2 * p2.x - 2 * p1.x;
            let B = 2 * p2.y - 2 * p1.y;
            let C = r1 * r1 - r2 * r2 - p1.x * p1.x + p2.x * p2.x - p1.y * p1.y + p2.y * p2.y;
            let D = 2 * p3.x - 2 * p2.x;
            let E = 2 * p3.y - 2 * p2.y;
            let F = r2 * r2 - r3 * r3 - p2.x * p2.x + p3.x * p3.x - p2.y * p2.y + p3.y * p3.y;

            // Решение системы уравнений
            let x = (C * E - F * B) / (E * A - B * D);
            let y = (C * D - A * F) / (B * D - A * E);

            return { x, y };
        }

        function processGPSData(data) {
            // Обновление данных о спутнике
            satellites.set(data.id, {
                x: data.x,
                y: data.y,
                sentAt: data.sentAt,
                receivedAt: data.receivedAt,
                lastUpdate: Date.now()
            });

            // Удаление старых данных (старше 5 секунд)
            const now = Date.now();
            for (let [id, sat] of satellites.entries()) {
                if (now - sat.lastUpdate > 5000) {
                    satellites.delete(id);
                }
            }

            // Расчет позиции объекта
            if (satellites.size >= 3) {
                calculatedPosition = trilaterate(satellites);
                updateObjectInfo();
            }

            updatePlot();
        }

        function updatePlot() {
            const satelliteTrace = {
                x: Array.from(satellites.values()).map(s => s.x),
                y: Array.from(satellites.values()).map(s => s.y),
                mode: 'markers+text',
                type: 'scatter',
                name: 'Супутники',
                marker: { size: 12, color: '#1976D2', symbol: 'diamond' },
                text: Array.from(satellites.keys()).map(id => `Sat ${id.slice(0,4)}`),
                textposition: 'top'
            };

            const traces = [satelliteTrace];

            if (calculatedPosition) {
                const objectTrace = {
                    x: [calculatedPosition.x],
                    y: [calculatedPosition.y],
                    mode: 'markers+text',
                    type: 'scatter',
                    name: 'Об\'єкт',
                    marker: { size: 15, color: '#e91e63', symbol: 'circle' },
                    text: ['Об\'єкт'],
                    textposition: 'top'
                };
                traces.push(objectTrace);
            }

            Plotly.react('gpsPlot', traces, layout);
        }

        function updateObjectInfo() {
            if (!calculatedPosition) return;

            const infoDiv = document.getElementById('objectInfo');
            infoDiv.innerHTML = `
                <strong>Координати об'єкта:</strong><br>
                X: ${calculatedPosition.x.toFixed(2)} км<br>
                Y: ${calculatedPosition.y.toFixed(2)} км<br>
                Кількість видимих супутників: ${satellites.size}
            `;
        }

        async function updateConfig() {
            const config = {
                messageFrequency: parseInt(document.getElementById('messageFrequency').value),
                satelliteSpeed: parseInt(document.getElementById('satelliteSpeed').value),
                objectSpeed: parseInt(document.getElementById('objectSpeed').value)
            };

            try {
                const response = await fetch('http://localhost:4001/config', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(config)
                });

                if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
                console.log('Конфігурація успішно оновлена');
            } catch (error) {
                console.error('Помилка при оновленні конфігурації:', error);
                alert('Помилка при оновленні конфігурації');
            }
        }

        connectWebSocket();
    </script>
</body>
</html>
```

<p>Результат створення GPS:</p>











