<!DOCTYPE html>  
<html lang="ru">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1.0">  
    <title>🕌 Трекер Намаза</title>  
    <style>  
        * { margin: 0; padding: 0; box-sizing: border-box; }  
        body {   
            font-family: 'Segoe UI', Tahoma, sans-serif;   
            max-width: 400px; margin: 0 auto;   
            padding: 20px;   
            background: linear-gradient(135deg, #0f2027, #203a43, #2c5364);   
            color: white;   
            min-height: 100vh;  
        }  
        h1 {   
            text-align: center;   
            margin-bottom: 20px;   
            font-size: 24px;   
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);  
        }  
        #status {   
            text-align: center;   
            font-size: 18px;   
            margin: 20px 0;   
            padding: 15px;   
            background: rgba(255,255,255,0.1);   
            border-radius: 10px;  
        }  
        #times {   
            background: rgba(255,255,255,0.1);   
            padding: 20px;   
            border-radius: 15px;   
            margin: 20px 0;   
            backdrop-filter: blur(10px);  
        }  
        .prayer {   
            display: flex;   
            justify-content: space-between;   
            align-items: center;  
            margin: 15px 0;   
            padding: 15px;   
            background: rgba(255,255,255,0.05);   
            border-radius: 10px;  
            transition: all 0.3s ease;  
        }  
        .prayer:hover { transform: translateY(-2px); box-shadow: 0 5px 15px rgba(0,0,0,0.2); }  
        .prayer-name { font-size: 18px; font-weight: bold; }  
        .prayer-time { font-size: 16px; opacity: 0.9; }  
        button {   
            background: linear-gradient(45deg, #28a745, #20c997);   
            color: white;   
            border: none;   
            padding: 12px 20px;   
            border-radius: 25px;   
            cursor: pointer;   
            font-size: 14px;   
            font-weight: bold;  
            transition: all 0.3s ease;  
            box-shadow: 0 4px 15px rgba(40,167,69,0.4);  
        }  
        button:hover { transform: scale(1.05); box-shadow: 0 6px 20px rgba(40,167,69,0.6); }  
        button.checked {   
            background: linear-gradient(45deg, #ffc107, #ff8f00);   
            box-shadow: 0 4px 15px rgba(255,193,7,0.4);  
        }  
        #refresh {   
            width: 100%;   
            background: linear-gradient(45deg, #007bff, #0056b3);   
            margin-top: 20px;   
            padding: 15px;   
            font-size: 16px;  
        }  
        .next-prayer { animation: pulse 2s infinite; }  
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.7; } }  
    </style>  
</head>  
<body>  
    <h1>🕌 Трекер Намаза - Стамбул</h1>  
    <div id="status">Загрузка времён намаза...</div>  
    <div id="times"></div>  
    <button id="refresh" onclick="loadTimes()">🔄 Обновить времена намаза</button>  
  
    <script>  
        const prayers = ['Fajr', 'Dhuhr', 'Asr', 'Maghrib', 'Isha'];  
        const prayerNamesRu = {  
            Fajr: 'Фаджр ☀️',   
            Dhuhr: 'Зухр 🌞',   
            Asr: 'Аср 🌅',   
            Maghrib: 'Магриб 🌇',   
            Isha: 'Иша 🌙'  
        };  
  
        let todayTimes = {};  
  
        // Загрузка времён намаза для Стамбула  
        async function loadTimes() {  
            const statusEl = document.getElementById('status');  
            statusEl.innerText = '⏳ Загрузка времён намаза...';  
              
            try {  
                // API Aladhan для Стамбула (метод 4 - Карачи)  
                const response = await fetch('http://api.aladhan.com/v1/timingsByCity?city=Istanbul&country=Turkey&method=4');  
                const data = await response.json();  
                  
                if (data.code === 200) {  
                    todayTimes = data.data.timings;  
                    updateDisplay();  
                    statusEl.innerText = `✅ Времена намаза на сегодня загружены`;  
                } else {  
                    throw new Error('Ошибка API');  
                }  
            } catch (e) {  
                statusEl.innerText = '❌ Ошибка загрузки. Проверьте интернет-соединение.';  
                console.error(e);  
            }  
        }  
  
        function updateDisplay() {  
            const timesDiv = document.getElementById('times');  
            const now = new Date();  
            const today = now.toISOString().split('T')[0];  
            let html = '';  
  
            prayers.forEach(prayer => {  
                const timeStr = todayTimes[prayer];  
                const key = `${today}_${prayer}`;  
                const isChecked = localStorage.getItem(key) === 'checked';  
                const btnClass = isChecked ? 'checked' : '';  
                const btnText = isChecked ? '✓ НАМАЗ ОТМОЛЕН' : 'Отметить ✅';  
                  
                html += `  
                    <div class="prayer">  
                        <div>  
                            <div class="prayer-name">${prayerNamesRu[prayer]}</div>  
                            <div class="prayer-time">${timeStr}</div>  
                        </div>  
                        <button class="${btnClass}" onclick="togglePrayer('${today}', '${prayer}', this)">  
                            ${btnText}  
                        </button>  
                    </div>  
                `;  
            });  
  
            timesDiv.innerHTML = html;  
              
            const nowRu = now.toLocaleDateString('ru-RU', {  
                weekday: 'long',  
                year: 'numeric',  
                month: 'long',  
                day: 'numeric'  
            });  
            document.getElementById('status').innerHTML = `📅 ${nowRu}`;  
        }  
  
        function togglePrayer(date, prayer, btn) {  
            const key = `${date}_${prayer}`;  
            const isChecked = localStorage.getItem(key) === 'checked';  
              
            if (isChecked) {  
                localStorage.removeItem(key);  
                btn.classList.remove('checked');  
                btn.textContent = 'Отметить ✅';  
            } else {  
                localStorage.setItem(key, 'checked');  
                btn.classList.add('checked');  
                btn.textContent = '✓ НАМАЗ ОТМОЛЕН';  
            }  
        }  
  
        // Автозагрузка при открытии  
        loadTimes();  
          
        // Обновление каждые 30 минут  
        setInterval(loadTimes, 30 * 60 * 1000);  
    </script>  
</body>  
</html>  
