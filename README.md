<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Dota 2 Stream Widget Generator</title>
    <style>
        /* Стиль сайта-генератора */
        body { background: #1a1a1a; color: white; font-family: sans-serif; text-align: center; padding-top: 50px; }
        .setup-container { background: #2a2a2a; max-width: 500px; margin: 0 auto; padding: 30px; border-radius: 15px; box-shadow: 0 0 20px rgba(0,0,0,0.5); }
        input { padding: 12px; width: 80%; border-radius: 5px; border: none; margin: 10px 0; font-size: 16px; }
        button { padding: 12px 25px; background: #ed3b1c; color: white; border: none; border-radius: 5px; cursor: pointer; font-weight: bold; }
        .copy-box { background: #111; padding: 15px; margin-top: 20px; word-break: break-all; border: 1px dashed #555; display: none; }
        
        /* Стиль виджета (будет виден только в OBS) */
        #widget-overlay { display: none; text-align: left; background: rgba(10,10,12,0.9); width: 320px; padding: 15px; border-radius: 10px; border-left: 4px solid #ed3b1c; }
        .hero-img { width: 50px; height: 28px; border-radius: 3px; margin-right: 10px; }
        .match-row { display: flex; align-items: center; margin-bottom: 8px; border-bottom: 1px solid #333; padding-bottom: 5px; }
        .win { color: #44ff44; font-weight: bold; margin-right: 10px; }
        .loss { color: #ff4444; font-weight: bold; margin-right: 10px; }
    </style>
</head>
<body>

    <div id="setup-ui" class="setup-container">
        <h1>Dota Stats Widget</h1>
        <p>Введите ваш Dota ID (из Dotabuff или профиля):</p>
        <input type="text" id="dotaIdInput" placeholder="Например: 123456789">
        <br>
        <button onclick="generateLink()">Получить ссылку для OBS</button>
        
        <div id="resultBox" class="copy-box">
            <p style="font-size: 12px; color: #aaa;">Скопируйте эту ссылку и вставьте в "Источник браузера" в OBS:</p>
            <strong id="generatedLink"></strong>
        </div>
    </div>

    <div id="widget-overlay">
        <div id="stats-content">Загрузка матчей...</div>
    </div>

<script>
    // Проверяем, есть ли ID в ссылке (например, ?id=12345)
    const urlParams = new URLSearchParams(window.location.search);
    const playerID = urlParams.get('id');

    if (playerID) {
        // Если ID есть, прячем интерфейс сайта и показываем только виджет
        document.getElementById('setup-ui').style.display = 'none';
        document.getElementById('widget-overlay').style.display = 'block';
        document.body.style.background = 'transparent';
        updateStats(playerID);
        setInterval(() => updateStats(playerID), 120000);
    }

    function generateLink() {
        const id = document.getElementById('dotaIdInput').value.trim();
        if (!id) return alert("Введите ID!");
        
        // Создаем ссылку на основе текущего адреса страницы
        const currentUrl = window.location.href.split('?')[0];
        const finalUrl = `${currentUrl}?id=${id}`;
        
        document.getElementById('generatedLink').innerText = finalUrl;
        document.getElementById('resultBox').style.display = 'block';
    }

    async function updateStats(id) {
        try {
            const matchRes = await fetch(`https://api.opendota.com/api/players/${id}/recentMatches`);
            const matches = await matchRes.json();
            const heroesRes = await fetch('https://api.opendota.com/api/constants/heroes');
            const heroes = await heroesRes.json();

            let html = '';
            matches.slice(0, 5).forEach(match => {
                const heroData = heroes[match.hero_id];
                const heroCleanName = heroData ? heroData.name.replace('npc_dota_hero_', '') : '';
                const imgUrl = `https://cdn.cloudflare.steamstatic.com/apps/dota2/images/dota_react/heroes/${heroCleanName}.png`;
                const isWin = (match.player_slot < 128 && match.radiant_win) || (match.player_slot >= 128 && !match.radiant_win);

                html += `
                    <div class="match-row">
                        <span class="${isWin ? 'win' : 'loss'}">${isWin ? 'W' : 'L'}</span>
                        <img src="${imgUrl}" class="hero-img">
                        <div style="font-size: 13px;">
                            <div style="color:white">${heroData.localized_name}</div>
                            <div style="color:#888">${match.kills}/${match.deaths}/${match.assists}</div>
                        </div>
                    </div>`;
            });
            document.getElementById('stats-content').innerHTML = html;
        } catch (e) {
            document.getElementById('stats-content').innerHTML = "Ошибка ID";
        }
    }
</script>
</body>
</html>
