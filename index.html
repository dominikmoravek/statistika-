<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>STATISTIKA</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Rajdhani:wght@300;500;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --neon-green: #00ff41;
            --neon-blue: #00d4ff;
            --neon-red: #ff3131;
            --dark-bg: #0d1117;
            --card-bg: #161b22;
        }

        /* Jemné posouvání stránky */
        html { scroll-behavior: smooth; }

        body {
            background-color: var(--dark-bg);
            color: white;
            font-family: 'Rajdhani', sans-serif;
            margin: 0;
            padding: 0;
            overflow-x: hidden;
        }

        /* Animace pro náběh stránky */
        @keyframes fadeInUp {
            from { opacity: 0; transform: translateY(30px); }
            to { opacity: 1; transform: translateY(0); }
        }

        header {
            padding: 40px 20px;
            text-align: center;
            background: linear-gradient(180deg, #161b22 0%, var(--dark-bg) 100%);
            border-bottom: 1px solid #30363d;
            animation: fadeInUp 0.8s ease-out;
        }

        h1 {
            font-family: 'Orbitron', sans-serif;
            letter-spacing: 12px;
            margin: 0;
            color: white;
            text-shadow: 0 0 20px rgba(0, 212, 255, 0.5);
        }

        .container {
            max-width: 1100px;
            margin: -20px auto 50px;
            padding: 0 20px;
            animation: fadeInUp 1s ease-out;
        }

        .main-stats {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .card {
            background: var(--card-bg);
            border: 1px solid #30363d;
            border-radius: 16px;
            padding: 25px;
            text-align: center;
            transition: all 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            position: relative;
            overflow: hidden;
        }

        .card:hover {
            transform: scale(1.05);
            border-color: var(--neon-blue);
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }

        .card h3 {
            margin: 0;
            color: #8b949e;
            font-size: 13px;
            letter-spacing: 2px;
            text-transform: uppercase;
        }

        .card .value {
            font-size: 45px;
            font-weight: 700;
            margin: 15px 0;
            font-family: 'Orbitron', sans-serif;
            transition: 0.3s;
        }

        #lastTime { color: white; }
        #avgTime { color: var(--neon-blue); }
        #fastestTime { color: var(--neon-green); }
        #slowestTime { color: var(--neon-red); }

        .chart-section {
            background: var(--card-bg);
            border-radius: 16px;
            padding: 25px;
            border: 1px solid #30363d;
            height: 400px;
            animation: fadeInUp 1.2s ease-out;
        }

        .btn-connect {
            background: transparent;
            color: var(--neon-blue);
            border: 2px solid var(--neon-blue);
            padding: 15px 45px;
            font-family: 'Orbitron', sans-serif;
            font-size: 14px;
            font-weight: bold;
            cursor: pointer;
            border-radius: 8px;
            margin: 30px auto;
            display: block;
            transition: 0.3s;
            text-transform: uppercase;
        }

        .btn-connect:hover {
            background: var(--neon-blue);
            color: black;
            box-shadow: 0 0 30px rgba(0, 212, 255, 0.4);
        }

        /* Pulsování pro "živý" efekt */
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }
        .live-dot {
            height: 8px; width: 8px; background: var(--neon-green);
            border-radius: 50%; display: inline-block; margin-right: 10px;
            animation: pulse 1.5s infinite;
        }
    </style>
</head>
<body>

<header>
    <h1>STATISTIKA</h1>
    <button class="btn-connect" id="connectBtn">PŘIPOJIT TRENAŽÉR</button>
</header>

<div class="container">
    <div class="main-stats">
        <div class="card">
            <h3>Poslední</h3>
            <div class="value" id="lastTime">0</div>
            <span style="color:#58a6ff">ms</span>
        </div>
        <div class="card">
            <h3>Průměr</h3>
            <div class="value" id="avgTime">0</div>
            <span style="color:#58a6ff">ms</span>
        </div>
        <div class="card">
            <h3>Nejrychlejší</h3>
            <div class="value" id="fastestTime">0</div>
            <span style="color:#58a6ff">ms</span>
        </div>
        <div class="card">
            <h3>Nejpomalejší</h3>
            <div class="value" id="slowestTime">0</div>
            <span style="color:#58a6ff">ms</span>
        </div>
    </div>

    <div class="chart-section">
        <canvas id="reactionChart"></canvas>
    </div>
</div>

<script>
    let times = [];
    let best = Infinity;
    let worst = 0;
    let lineBuffer = "";

    const ctx = document.getElementById('reactionChart').getContext('2d');
    const chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: [],
            datasets: [{
                data: [],
                borderColor: '#00d4ff',
                backgroundColor: 'rgba(0, 212, 255, 0.05)',
                borderWidth: 3,
                fill: true,
                tension: 0.4,
                pointRadius: 4,
                pointBackgroundColor: '#00d4ff'
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
                y: { 
                    grid: { color: 'rgba(255,255,255,0.05)' },
                    ticks: { color: '#8b949e', font: { family: 'Rajdhani' } }
                },
                x: { display: false }
            }
        }
    });

    document.getElementById('connectBtn').addEventListener('click', async () => {
        try {
            const port = await navigator.serial.requestPort();
            await port.open({ baudRate: 115200 });
            document.getElementById('connectBtn').innerHTML = '<span class="live-dot"></span>PROPOJENO';
            document.getElementById('connectBtn').style.borderColor = 'var(--neon-green)';
            document.getElementById('connectBtn').style.color = 'var(--neon-green)';

            const decoder = new TextDecoderStream();
            port.readable.pipeTo(decoder.writable);
            const reader = decoder.readable.getReader();

            while (true) {
                const { value, done } = await reader.read();
                if (done) break;
                processData(value);
            }
        } catch (e) { console.error("Chyba připojení:", e); }
    });

    function processData(chunk) {
        lineBuffer += chunk;
        let lines = lineBuffer.split("\r\n");
        lineBuffer = lines.pop();

        lines.forEach(line => {
            if (line.includes('TIME:')) {
                let val = parseInt(line.split(':')[1]);
                if(!isNaN(val)) updateStats(val);
            }
        });
    }

    function updateStats(val) {
        document.getElementById('lastTime').innerText = val;

        times.push(val);
        if (val < best) best = val;
        if (val > worst) worst = val;
        
        const avg = Math.round(times.reduce((a, b) => a + b) / times.length);

        document.getElementById('fastestTime').innerText = best;
        document.getElementById('slowestTime').innerText = worst;
        document.getElementById('avgTime').innerText = avg;

        chart.data.labels.push("");
        chart.data.datasets[0].data.push(val);
        if (chart.data.datasets[0].data.length > 25) {
            chart.data.labels.shift();
            chart.data.datasets[0].data.shift();
        }
        chart.update();
    }
</script>
</body>
</html>
