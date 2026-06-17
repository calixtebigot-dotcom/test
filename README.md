<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Défi Logique - Système de Records</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --rouge: #ff4d4d;
            --bleu: #ffffff;
            --jaune: #f1c40f;
            --vert: #2ecc71;
            --sombre: #2c3e50;
            --fond-noir: #121212;
            --carte-grise: #1e1e1e;
        }

        body {
            font-family: 'Segoe UI', sans-serif;
            background: var(--fond-noir);
            color: #ffffff;
            margin: 0;
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            user-select: none;
            padding: 20px;
            box-sizing: border-box;
        }

        .main-wrapper {
            display: flex;
            flex-direction: row;
            gap: 25px;
            max-width: 950px;
            width: 100%;
            align-items: flex-start;
            justify-content: center;
        }

        .container {
            background: var(--carte-grise);
            padding: 40px;
            border-radius: 20px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.5);
            text-align: center;
            flex: 1;
            border: 1px solid #333;
        }

        /* --- STYLE DU CLASSEMENT GLOBAL --- */
        .leaderboard-panel {
            background: var(--carte-grise);
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.5);
            width: 340px;
            min-width: 340px;
            box-sizing: border-box;
            border: 1px solid #333;
        }

        .leaderboard-panel h3 {
            margin-top: 0;
            color: #ffffff;
            border-bottom: 2px solid #333;
            padding-bottom: 10px;
            font-size: 1.25em;
        }

        .leaderboard-table {
            width: 100%;
            border-collapse: collapse;
            text-align: left;
            font-size: 0.9em;
        }

        .leaderboard-table th {
            color: #aaa;
            padding-bottom: 8px;
            font-weight: 600;
        }

        .leaderboard-table td {
            padding: 7px 0;
            border-bottom: 1px solid #2a2a2a;
            color: #ddd;
        }

        .name-col { max-width: 130px; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
        .rank-num-prefix { font-weight: bold; color: #777; margin-right: 8px; display: inline-block; width: 20px; }
        .time-col { color: #888; text-align: center; width: 55px; }
        .score-val { text-align: right; font-weight: bold; color: var(--vert); }

        /* --- ÉCRANS --- */
        .screen { display: none; }
        .active { display: block; }

        /* --- LOBBY --- */
        h1 { color: #ffffff; font-size: 2.2em; margin-bottom: 10px; }
        .record-display {
            background: #252525;
            border: 2px solid #444;
            padding: 15px;
            border-radius: 12px;
            margin: 20px 0;
        }
        .record-value { font-size: 1.5em; font-weight: bold; color: var(--jaune); }

        select, button, input[type="text"] {
            font-size: 16px;
            padding: 12px 25px;
            margin: 10px;
            border-radius: 8px;
            border: 1px solid #444;
            outline: none;
            background: #252525;
            color: white;
        }

        input[type="text"] {
            background: #151515;
            border: 1px solid #555;
            text-align: center;
            width: 200px;
        }
        input[type="text"]:focus {
            border-color: var(--vert);
        }

        button {
            background: var(--vert);
            color: white;
            border: none;
            cursor: pointer;
            font-weight: bold;
            text-transform: uppercase;
            transition: transform 0.2s, background 0.2s;
        }
        button:hover { background: #27ae60; transform: translateY(-2px); }

        /* --- ZONE DE JEU --- */
        .stats-bar {
            display: flex;
            justify-content: space-between;
            margin-bottom: 25px;
        }
        .stat-item {
            padding: 10px 20px;
            border-radius: 10px;
            background: #252525;
            min-width: 120px;
            border: 1px solid #333;
        }
        .stat-label { font-size: 12px; color: #888; text-transform: uppercase; }
        .stat-value { font-size: 20px; font-weight: bold; display: block; }
        
        .points-pos { color: var(--vert); }
        .points-neg { color: var(--rouge); }

        .display-box {
            height: 220px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            margin: 20px 0;
            border: 3px solid #333;
            border-radius: 15px;
            transition: background 0.1s;
            background: #151515;
        }

        .instruction { font-size: 24px; font-weight: bold; margin-bottom: 20px; }

        /* Formes */
        .shape { width: 100px; height: 100px; }
        .circle { border-radius: 50%; }
        .triangle {
            width: 0; height: 0;
            border-left: 50px solid transparent;
            border-right: 50px solid transparent;
            border-bottom: 100px solid;
        }

        .controls-hint { color: #888; font-size: 0.9em; margin-top: 20px; }
        .key { background: #333; padding: 3px 8px; border-radius: 4px; border-bottom: 3px solid #555; font-family: monospace; color: #fff; }

        /* Flash */
        .correct-flash { background-color: rgba(46, 204, 113, 0.15); border-color: var(--vert); }
        .wrong-flash { background-color: rgba(255, 77, 77, 0.15); border-color: var(--rouge); }

        /* --- RÉSULTATS --- */
        .new-record-msg { color: var(--jaune); font-weight: bold; animation: bounce 1s infinite; margin: 10px 0; display: none; }
        @keyframes bounce { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.1); } }

        .chart-wrapper {
            margin-top: 25px;
            height: 220px;
            width: 100%;
        }

        #name-submission-zone {
            margin: 20px 0;
            background: #1a1a1a;
            padding: 15px;
            border-radius: 12px;
            border: 1px solid #333;
        }
    </style>
</head>
<body>

<div class="main-wrapper">
    <div class="container">
        <div id="lobby" class="screen active">
            <h1>Challenge Formes</h1>
            <p style="color: #aaa;">Valide (&rarr;) ou Refuse (&larr;) pour marquer des points.</p>
            
            <div class="record-display">
                <span class="stat-label">Meilleur Score (Record)</span><br>
                <span id="best-score-display" class="record-value">0 pts</span>
            </div>

            <label for="time-select">Durée du test :</label><br>
            <select id="time-select" onchange="onTimeModeChange()">
                <option value="30">30 Secondes</option>
                <option value="120">2 Minutes</option>
                <option value="300">5 Minutes</option>
                <option value="600">10 Minutes</option>
            </select>
            <br>
            <button onclick="startGame()">Lancer la partie</button>
        </div>

        <div id="game-screen" class="screen">
            <div class="stats-bar">
                <div class="stat-item">
                    <span class="stat-label">Temps restant</span>
                    <span id="timer" class="stat-value">--:--</span>
                </div>
                <div id="score-container" class="stat-item">
                    <span class="stat-label">Points actuels</span>
                    <span id="live-points" class="stat-value">0</span>
                </div>
            </div>

            <div id="display-box" class="display-box">
                <div id="instruction" class="instruction">Pret ?</div>
                <div id="shape-el" class="shape"></div>
            </div>

            <div class="controls-hint">
                <span class="key">&larr;</span> Refuser | Valider <span class="key">&rarr;</span>
            </div>
        </div>

        <div id="result-screen" class="screen">
            <h2 id="end-title">Temps écoulé !</h2>
            <div id="new-record-banner" class="new-record-msg">🌟 NOUVEAU RECORD ÉTABLI ! 🌟</div>
            
            <div style="margin: 20px 0;">
                <p>Ton score final : <strong id="final-points" style="font-size: 1.5em; color: var(--vert);">0</strong> points</p>
                <p>Bonnes réponses : <span id="correct-count" style="color:#fff;">0</span> | Précision : <span id="accuracy-pc" style="color:#fff;">0</span>%</p>
            </div>

            <div id="name-submission-zone">
                <label for="username-input" style="display:block; margin-bottom:5px; color:#aaa;">Enregistre ton prénom pour le classement :</label>
                <input type="text" id="username-input" placeholder="Ton Prénom" maxlength="12">
                <button onclick="submitScore()">Enregistrer</button>
            </div>

            <button id="back-lobby-btn" onclick="backToLobby()" style="background: var(--sombre); margin-top: 20px; display: none;">Retour au Lobby</button>

            <div class="chart-wrapper">
                <canvas id="reactionTimesChart"></canvas>
            </div>
        </div>
    </div>

    <div class="leaderboard-panel">
        <h3>🏆 Top 10 Global</h3>
        <table class="leaderboard-table">
            <thead>
                <tr>
                    <th>Prénom</th>
                    <th style="text-align: center;">Durée</th>
                    <th class="score-val">Score</th>
                </tr>
            </thead>
            <tbody id="leaderboard-body">
                </tbody>
        </table>
    </div>
</div>

<script>
    // Config
    const config = {
        colors: [
            { name: 'rouge', hex: '#ff4d4d' },
            { name: 'bleu', hex: '#3399ff' },
            { name: 'jaune', hex: '#f1c40f' }
        ],
        shapes: ['carrée', 'triangle', 'rond'],
        ptsWin: 5,
        ptsLoss: 5
    };

    let gameData = {
        timer: null,
        timeLeft: 0,
        points: 0,
        correct: 0,
        total: 0,
        isActionable: false,
        expected: null,
        currentQuestStartTime: 0, 
        reactionTimesHistory: []  
    };

    let chartInstance = null; 

    window.onload = () => {
        updateRecordDisplay();
        refreshLeaderboardUI();
    };

    function onTimeModeChange() {
        updateRecordDisplay();
        refreshLeaderboardUI();
    }

    function updateRecordDisplay() {
        const time = document.getElementById('time-select').value;
        const record = localStorage.getItem('record_' + time) || 0;
        document.getElementById('best-score-display').textContent = record + " pts";
    }

    function getScoresFromStorage() {
        const stored = localStorage.getItem('global_leaderboard');
        return stored ? JSON.parse(stored) : [];
    }

    function convertTimeLabel(seconds) {
        if (seconds == 30) return "30s";
        return (seconds / 60) + "m";
    }

    // Gestion de la sauvegarde locale du pseudo saisi dans l'input html
    function submitScore() {
        const input = document.getElementById('username-input');
        let username = input.value.trim();
        
        if (!username) {
            username = "Anonyme";
        }

        const timeKey = document.getElementById('time-select').value;
        let scores = getScoresFromStorage();
        
        scores.push({ 
            name: username.substring(0, 12), 
            score: gameData.points, 
            duration: convertTimeLabel(timeKey),
            rawDuration: parseInt(timeKey)
        });
        
        scores.sort((a, b) => b.score - a.score);
        scores = scores.slice(0, 10);
        
        localStorage.setItem('global_leaderboard', JSON.stringify(scores));
        
        refreshLeaderboardUI();

        // On masque le formulaire et on affiche le bouton pour quitter la page
        document.getElementById('name-submission-zone').style.display = 'none';
        document.getElementById('back-lobby-btn').style.display = 'inline-block';
    }

    function refreshLeaderboardUI() {
        const scores = getScoresFromStorage();
        const container = document.getElementById('leaderboard-body');
        container.innerHTML = '';

        for (let i = 0; i < 10; i++) {
            const row = document.createElement('tr');
            
            const tdName = document.createElement('td');
            tdName.className = 'name-col';
            
            const rankSpan = document.createElement('span');
            rankSpan.className = 'rank-num-prefix';
            rankSpan.textContent = (i + 1) + ".";
            
            const nameText = document.createTextNode(scores[i] ? scores[i].name : '---');
            
            tdName.appendChild(rankSpan);
            tdName.appendChild(nameText);

            const tdTime = document.createElement('td');
            tdTime.className = 'time-col';
            tdTime.textContent = scores[i] ? scores[i].duration : '---';

            const tdScore = document.createElement('td');
            tdScore.className = 'score-val';
            tdScore.textContent = scores[i] ? scores[i].score + ' pts' : '---';

            row.appendChild(tdName);
            row.appendChild(tdTime);
            row.appendChild(tdScore);
            container.appendChild(row);
        }
    }

    function startGame() {
        gameData.points = 0;
        gameData.correct = 0;
        gameData.total = 0;
        gameData.timeLeft = parseInt(document.getElementById('time-select').value);
        gameData.isActionable = true;
        gameData.reactionTimesHistory = [];

        // Reset de l'interface des résultats pour une nouvelle tentative
        document.getElementById('name-submission-zone').style.display = 'block';
        document.getElementById('back-lobby-btn').style.display = 'none';
        document.getElementById('username-input').value = '';

        switchScreen('game-screen');
        updatePointsUI();
        startTimer();
        generateQuest();
    }

    function startTimer() {
        updateTimerUI();
        gameData.timer = setInterval(() => {
            gameData.timeLeft--;
            updateTimerUI();
            if (gameData.timeLeft <= 0) endGame();
        }, 1000);
    }

    function generateQuest() {
        const shape = config.shapes[Math.floor(Math.random() * 3)];
        const color = config.colors[Math.floor(Math.random() * 3)];
        const isShapeQuest = Math.random() > 0.5;
        gameData.expected = Math.random() > 0.5;

        let instructionText = "";
        if (isShapeQuest) {
            const targetShape = gameData.expected ? shape : config.shapes.filter(s => s !== shape)[Math.floor(Math.random()*2)];
            instructionText = `Est-ce un ${targetShape} ?`;
        } else {
            const targetColor = gameData.expected ? color.name : config.colors.filter(c => c.name !== color.name)[Math.floor(Math.random()*2)].name;
            instructionText = `Couleur ${targetColor} ?`;
        }

        document.getElementById('instruction').textContent = instructionText;
        drawShape(shape, color.hex);

        gameData.currentQuestStartTime = performance.now();
    }

    function drawShape(type, color) {
        const el = document.getElementById('shape-el');
        el.className = 'shape';
        el.style.backgroundColor = '';
        el.style.borderBottomColor = '';

        if (type === 'carrée') el.style.backgroundColor = color;
        if (type === 'rond') { el.classList.add('circle'); el.style.backgroundColor = color; }
        if (type === 'triangle') { el.classList.add('triangle'); el.style.borderBottomColor = color; }
    }

    window.addEventListener('keydown', (e) => {
        if (!gameData.isActionable) return;
        if (e.key === 'ArrowLeft') handleResponse(false);
        if (e.key === 'ArrowRight') handleResponse(true);
    });

    function handleResponse(userVal) {
        const elapsed = (performance.now() - gameData.currentQuestStartTime) / 1000;
        gameData.reactionTimesHistory.push(parseFloat(elapsed.toFixed(2)));

        gameData.total++;
        const isCorrect = (userVal === gameData.expected);

        if (isCorrect) {
            gameData.points += config.ptsWin;
            gameData.correct++;
            flash('correct-flash');
        } else {
            gameData.points -= config.ptsLoss;
            flash('wrong-flash');
        }

        updatePointsUI();
        generateQuest();
    }

    function flash(cls) {
        const box = document.getElementById('display-box');
        box.classList.add(cls);
        setTimeout(() => box.classList.remove(cls), 150);
    }

    function endGame() {
        clearInterval(gameData.timer);
        gameData.isActionable = false;
        
        const timeKey = document.getElementById('time-select').value;
        const oldRecord = parseInt(localStorage.getItem('record_' + timeKey)) || 0;
        let isNewRecord = false;

        if (gameData.points > oldRecord) {
            localStorage.setItem('record_' + timeKey, gameData.points);
            isNewRecord = true;
        }

        document.getElementById('final-points').textContent = gameData.points;
        document.getElementById('correct-count').textContent = gameData.correct;
        document.getElementById('accuracy-pc').textContent = gameData.total > 0 ? Math.round((gameData.correct / gameData.total) * 100) : 0;
        document.getElementById('new-record-banner').style.display = isNewRecord ? 'block' : 'none';
        
        switchScreen('result-screen');
        renderReactionChart();
    }

    function renderReactionChart() {
        const ctx = document.getElementById('reactionTimesChart').getContext('2d');
        const labels = gameData.reactionTimesHistory.map((_, i) => `Q${i + 1}`);

        if (chartInstance) {
            chartInstance.destroy();
        }

        chartInstance = new Chart(ctx, {
            type: 'line',
            data: {
                labels: labels,
                datasets: [{
                    label: 'Temps de réaction (sec)',
                    data: gameData.reactionTimesHistory,
                    borderColor: '#2ecc71',
                    backgroundColor: 'rgba(46, 204, 113, 0.1)',
                    borderWidth: 2,
                    tension: 0.15,
                    pointRadius: labels.length > 40 ? 0 : 3
                }]
            },
            options: {
                responsive: true,
                maintainAspectRatio: false,
                scales: {
                    y: {
                        beginAtZero: true,
                        grid: { color: '#333' },
                        ticks: { color: '#aaa' },
                        title: { display: true, text: 'Secondes', color: '#aaa', font: { size: 11 } }
                    },
                    x: {
                        grid: { display: false },
                        ticks: { color: '#aaa', maxTicksLimit: 20 }
                    }
                },
                plugins: {
                    legend: { display: true, labels: { color: '#fff', boxWidth: 10, font: { size: 11 } } }
                }
            }
        });
    }

    function updatePointsUI() {
        const el = document.getElementById('live-points');
        el.textContent = gameData.points;
        el.className = 'stat-value ' + (gameData.points >= 0 ? 'points-pos' : 'points-neg');
    }

    function updateTimerUI() {
        const m = Math.floor(gameData.timeLeft / 60);
        const s = gameData.timeLeft % 60;
        document.getElementById('timer').textContent = `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
    }

    function switchScreen(id) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(id).classList.add('active');
    }

    function backToLobby() {
        updateRecordDisplay();
        refreshLeaderboardUI();
        switchScreen('lobby');
    }
</script>

</body>
</html>
