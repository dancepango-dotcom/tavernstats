<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dota 2 Stats Widget Generator</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&family=Orbitron:wght@700&display=swap');

        body {
            background: #0a0a0c;
            color: #e0e0e0;
            font-family: 'Roboto', sans-serif;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .generator-card {
            background: #141417;
            padding: 50px 40px; /* Увеличил вертикальные отступы для баланса */
            border-radius: 24px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.7);
            text-align: center;
            max-width: 420px;
            width: 90%;
            border: 1px solid #2a2a2e;
        }

        h1 { 
            font-family: 'Orbitron', sans-serif; 
            font-size: 24px; 
            color: #ed3b1c; 
            margin: 0 0 15px 0; 
            letter-spacing: 2px; 
        }

        .description { 
            color: #888; 
            font-size: 14px; 
            margin-bottom: 30px; 
            line-height: 1.5; 
        }

        input {
            width: 100%;
            padding: 14px;
            background: #000;
            border: 1px solid #333;
            border-radius: 10px;
            color: white;
            font-size: 16px;
            box-sizing: border-box;
            margin-bottom: 12px;
            transition: 0.3s;
            text-align: center;
        }
        input:focus { border-color: #ed3b1c; outline: none; }

        button {
            width: 100%;
            padding: 14px;
            background: #ed3b1c;
            color: white;
            border: none;
            border-radius: 10px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: 0.2s;
        }
        button:hover { background: #ff4526; transform: translateY(-1px); }

        .result-area {
            margin-top: 25px;
            padding: 15px;
            background: #000;
            border-radius: 10px;
            display: none;
            border: 1px solid #ed3b1c;
        }
        .result-link { 
            word-break: break-all; 
            color: #44ff44; 
            font-size: 12px; 
            display: block; 
            margin-bottom: 12px;
            font-family: monospace;
        }
        .copy-btn {
            background: #333;
            font-size: 12px;
            padding: 8px 15px;
            width: auto;
            display: inline-block;
        }

        .footer {
            margin-top: 40px;
            padding-top: 20px;
            border-top: 1px solid #222;
            font-size: 13px;
        }
        .dev-name { color: #fff; font-weight: bold; }
        .tg-link { color: #ed3b1c; text-decoration: none; font-weight: bold; }
        .offer { display: block; margin-top: 10px; color: #666; font-style: italic; }

        /* ВИДЖЕТ (В OBS) */
        #widget-container { display: none; background: rgba(0, 0, 0, 0.8); padding: 6px 10px; border-radius: 6px; }
        .match-row { display: flex; align-items: center; padding: 4px 0; border-bottom: 1px solid rgba(255,255,255,0.05); gap: 8px; }
        .match-row:last-child { border: none; }
        .hero-img { width: 38px; height: 21px; border-radius: 2px; }
        .res-badge { width: 16px; height: 16px; border-radius: 2px; display: flex; align-items: center; justify-content: center; font-weight: bold; font-size: 10px; flex-shrink: 0; }
        .win-bg { background: #44ff44; color: #000; }
        .loss-bg { background: #ff4444; color: #000; }
        .kda { font-size: 12px; color: white; white-space: nowrap; font-family: sans-serif; }
    </style>
</head>
<body>

    <div id="ui-layer" class="generator-card">
        <h1>DOTA 2 WIDGET</h1>
        <p class="description">Введи свой ID, чтобы получить персональную ссылку на статистику последних 5 игр для OBS.</p>
        
        <input type="text" id="idInput" placeholder="Ваш Dota ID (например: 12345678)">
        <button onclick="generate()">Сгенерировать ссылку</button>

        <div id="resultBox" class="result-area">
            <span class="result-link" id="linkOut"></span>
            <button class="copy-btn" onclick="copyLink()">Копировать в буфер</button>
        </div>

        <div class="footer">
            <span class="dev-name">Разработчик: DignussQ</span> | TG: <a href="https://t.me/valdisdm" class="tg-link" target="_blank">@valdisdm</a>
            <span class="offer">Могу разработать индивидуальный виджет под твой стрим. Пиши!</span>
        </div>
    </div>

    <div id="widget-container">
        <div id="stats-content"></div>
    </div>

<script>
    const params = new URLSearchParams(window.location.search);
    const pid = params.get('id');

    if (pid) {
        document.getElementById('ui-layer').style.display = 'none';
        document.getElementById('widget-container').style.display = 'block';
        document.body.style.background = 'transparent';
        document.body.style.justifyContent = 'flex-start';
        document.body.style.alignItems = 'flex-start';
        loadStats(pid);
        setInterval(() => loadStats(pid), 60000);
    }

    function generate() {
        const val = document.getElementById('idInput').value.trim();
        if (!val) return alert("Введите ваш Dota ID");
        const baseUrl = window.location.href.split('?')[0];
        const url = baseUrl + "?id=" + val;
        document.getElementById('linkOut').innerText = url;
        document.getElementById('resultBox').style.display = 'block';
    }

    function copyLink() {
        const text = document.getElementById('linkOut').innerText;
        navigator.clipboard.writeText(text).then(() => {
            alert("Ссылка успешно скопирована!");
        }).catch(err => {
            console.error('Ошибка при копировании: ', err);
        });
    }

    async function loadStats(id) {
        try {
            const r = await fetch(`https://api.opendota.com/api/players/${id}/recentMatches`);
            const matches = await r.json();
            const hR = await fetch('https://api.opendota.com/api/constants/heroes');
            const heroes = await hR.json();

            let h = '';
            matches.slice(0, 5).forEach(m => {
                const hero = heroes[m.hero_id];
                const hName = hero ? hero.name.replace('npc_dota_hero_', '') : '';
                const img = `https://cdn.cloudflare.steamstatic.com/apps/dota2/images/dota_react/heroes/${hName}.png`;
                const win = (m.player_slot < 128 && m.radiant_win) || (m.player_slot >= 128 && !m.radiant_win);
                
                h += `<div class="match-row">
                    <div class="res-badge ${win ? 'win-bg' : 'loss-bg'}">${win ? 'W' : 'L'}</div>
                    <img src="${img}" class="hero-img">
                    <div class="kda"><b>${m.kills}/${m.deaths}/${m.assists}</b></div>
                </div>`;
            });
            document.getElementById('stats-content').innerHTML = h;
        } catch (e) { console.error(e); }
    }
</script>
</body>
</html>
