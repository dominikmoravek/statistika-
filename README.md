<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>STATISTIKA REAKCÍ</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Rajdhani:wght@300;500;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --neon-green: #00ff41;
            --neon-blue: #00d4ff;
            --neon-red: #ff3131;
            --dark-bg: #0a0b10;
            --card-bg: #161b22;
        }

        body {
            background-color: var(--dark-bg);
            color: white;
            font-family: 'Rajdhani', sans-serif;
            margin: 0;
            padding: 0;
        }

        header {
            padding: 30px;
            text-align: center;
            background: rgba(22, 27, 34, 0.9);
            border-bottom: 2px solid var(--neon-blue);
        }

        h1 {
            font-family: 'Orbitron', sans-serif;
            letter-spacing: 8px;
            margin: 0;
            color: white;
            text-transform: uppercase;
        }

        .container {
            max-width: 1200px;
            margin: 30px auto;
            padding: 0 20px;
        }

        .main-stats {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .card {
            background: var(--card-bg);
            border: 1px solid #30363d;
            border-radius: 12px;
            padding: 20px;
            text-align: center;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }

        .card h3 {
            margin: 0;
            color: #8b949e;
            font-size: 14px;
            letter-spacing: 1px;
            text-transform: uppercase;
        }

        .card .value {
            font-size: 40px;
            font-weight: 700;
            margin: 10px 0;
            font-family: 'Orbitron', sans-serif;
        }

        #lastTime { color: white; }
        #avgTime { color: var(--neon-blue); }
        #fastestTime { color: var(--neon-green); }
        #slowestTime { color: var(--neon-red); }

        .chart-container {
            background: var(--card-bg);
            border-radius: 12px;
            padding: 20px;
            border: 1px solid #30363d;
            height: 350px;
        }

        .btn-connect {
            background: var(--neon-blue);
            color: black;
            border: none;
            padding: 15px 40px;
            font-family: 'Orbitron', sans-serif;
            font-weight: bold;
            cursor: pointer;
            border-radius: 5px;
            margin: 20px auto;
            display: block;
            transition: 0.3s;
        }

        .btn-connect:hover {
            box-shadow: 0 0 20px var(--neon-blue);
            transform: scale(1.05);
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
            <span>ms</span>
        </div>
        <div class="card">
            <h3>Průměr</h3>
            <div class="value" id="avgTime">0</div>
            <span>ms</span>
        </div>
        <div class="card">
            <h3>Nejrychlejší</h3>
            <div class="value" id="fastestTime">0</div>
            <span>ms</span>
        </div>
        <div class="card">
            <h3>Nejpomalejší</h3>
            <div class="value" id="slowestTime">0</div>
            <span>ms</span>
        </div>
    </div>

    <div class="chart-container">
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
                backgroundColor: 'rgba(0, 212, 255, 0.1)',
                borderWidth: 3,
                fill: true,
                tension: 0.3
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            plugins: { legend: { display: false } },
            scales: {
                y: { grid: { color: '#30363d' }, ticks: { color: '#8b949e' } },
                x: { display: false }
            }
        }
    });

    document.getElementById('connectBtn').addEventListener('click', async () => {
        try {
            const port = await navigator.serial.requestPort();
            await port.open({ baudRate: 115200 });
            document.getElementById('connectBtn').style.display = 'none';

            const decoder = new TextDecoderStream();
            port.readable.pipeTo(decoder.writable);
            const reader = decoder.readable.getReader();

            while (true) {
                const { value, done } = await reader.read();
                if (done) break;
                process(value);
            }
        } catch (e) { console.error(e); }
    });

    function process(chunk) {
        lineBuffer += chunk;
        let lines = lineBuffer.split("\r\n");
        lineBuffer = lines.pop();

        lines.forEach(line => {
            if (line.includes('TIME:')) {
                let val = parseInt(line.split(':')[1]);
                update(val);
            }
        });
    }

    function update(val) {
        // Poslední čas
        document.getElementById('lastTime').innerText = val;

        // Statistiky
        times.push(val);
        if (val < best) best = val;
        if (val > worst) worst = val;
        
        const avg = Math.round(times.reduce((a, b) => a + b) / times.length);

        document.getElementById('fastestTime').innerText = best;
        document.getElementById('slowestTime').innerText = worst;
        document.getElementById('avgTime').innerText = avg;

        // Graf
        chart.data.labels.push("");
        chart.data.datasets[0].data.push(val);
        if (chart.data.datasets[0].data.length > 20) {
            chart.data.labels.shift();
            chart.data.datasets[0].data.shift();
        }
        chart.update();
    }
</script>
</body>
</html>
