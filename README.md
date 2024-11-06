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










