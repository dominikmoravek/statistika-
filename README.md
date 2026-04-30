<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>REACTION PRO | Dashboard</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700&family=Rajdhani:wght@300;500;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --neon-green: #00ff41;
            --neon-blue: #00d4ff;
            --dark-bg: #0a0b10;
            --card-bg: #161b22;
        }

        body {
            background-color: var(--dark-bg);
            color: white;
            font-family: 'Rajdhani', sans-serif;
            margin: 0;
            overflow-x: hidden;
        }

        header {
            padding: 20px;
            text-align: center;
            border-bottom: 1px solid #30363d;
            background: rgba(22, 27, 34, 0.8);
            backdrop-filter: blur(10px);
            position: sticky;
            top: 0;
            z-index: 100;
        }

        h1 {
            font-family: 'Orbitron', sans-serif;
            letter-spacing: 5px;
            margin: 0;
            color: var(--neon-blue);
            text-shadow: 0 0 10px var(--neon-blue);
        }

        .container {
            max-width: 1200px;
            margin: 30px auto;
            padding: 0 20px;
        }

        .main-stats {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }

        .card {
            background: var(--card-bg);
            border: 1px solid #30363d;
            border-radius: 15px;
            padding: 25px;
            text-align: center;
            transition: all 0.3s ease;
        }

        .card:hover {
            border-color: var(--neon-blue);
            box-shadow: 0 0 15px rgba(0, 212, 255, 0.1);
        }

        .card h3 {
            margin: 0;
            text-transform: uppercase;
            color: #8b949e;
            font-size: 14px;
            letter-spacing: 2px;
        }

        .card .value {
            font-size: 48px;
            font-weight: 700;
            margin: 10px 0;
            font-family: 'Orbitron', sans-serif;
        }

        #lastTimeValue { color: var(--neon-green); }
        #avgTimeValue { color: var(--neon-blue); }

        .chart-container {
            background: var(--card-bg);
            border-radius: 15px;
            padding: 20px;
            border: 1px solid #30363d;
            height: 400px;
        }

        .btn-connect {
            background: transparent;
            border: 2px solid var(--neon-blue);
            color: var(--neon-blue);
            padding: 12px 30px;
            font-family: 'Orbitron', sans-serif;
            font-weight: bold;
            cursor: pointer;
            border-radius: 50px;
            margin-top: 15px;
            transition: all 0.3s;
        }

        .btn-connect:hover {
            background: var(--neon-blue);
            color: black;
            box-shadow: 0 0 20px var(--neon-blue);
        }

        .status-info { margin-top: 10px; font-size: 14px; letter-spacing: 1px; }
        .dot { height: 10px; width: 10px; background: #ff4136; border-radius: 50%; display: inline-block; margin-right: 8px; }
        .connected .dot { background: var(--neon-green); box-shadow: 0 0 8px var(--neon-green); }
    </style>
</head>
<body>

<header>
    <h1>REACTION <span style="color:white">PRO</span></h1>
    <div class="status-info" id="status"><span class="dot"></span>ODPOJENO</div>
    <button class="btn-connect" id="connectBtn">SPREGOVAT ZAŘÍZENÍ</button>
</header>

<div class="container">
    <div class="main-stats">
        <div class="card">
            <h3>Poslední Reakce</h3>
            <div class="value" id="lastTimeValue">0</div>
            <span style="color:#8b949e">ms</span>
        </div>
        <div class="card">
            <h3>Průměr Kola</h3>
            <div class="value" id="avgTimeValue">0</div>
            <span style="color:#8b949e">ms</span>
        </div>
        <div class="card">
            <h3>Aktivní LED</h3>
            <div class="value" id="activeLedValue">-</div>
            <span style="color:#8b949e">ID kanálu</span>
        </div>
    </div>

    <div class="chart-container">
        <canvas id="reactionChart"></canvas>
    </div>
</div>

<script>
    let times = [];
    let labels = [];
    let chart;
    let lineBuffer = "";

    // Inicializace Grafu
    const ctx = document.getElementById('reactionChart').getContext('2d');
    chart = new Chart(ctx, {
        type: 'line',
        data: {
            labels: labels,
            datasets: [{
                label: 'Reakční čas (ms)',
                data: times,
                borderColor: '#00d4ff',
                backgroundColor: 'rgba(0, 212, 255, 0.1)',
                borderWidth: 3,
                tension: 0.4,
                fill: true,
                pointBackgroundColor: '#00ff41',
                pointRadius: 5
            }]
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: { grid: { color: '#30363d' }, ticks: { color: '#8b949e' } },
                x: { grid: { display: false }, ticks: { color: '#8b949e' } }
            },
            plugins: { legend: { display: false } }
        }
    });

    const connectBtn = document.getElementById('connectBtn');

    connectBtn.addEventListener('click', async () => {
        try {
            const port = await navigator.serial.requestPort();
            await port.open({ baudRate: 115200 });
            
            document.getElementById('status').parentElement.classList.add('connected');
            document.getElementById('status').innerHTML = '<span class="dot"></span>SYSTÉM AKTIVNÍ';
            connectBtn.style.display = 'none';

            const decoder = new TextDecoderStream();
            const inputClosed = port.readable.pipeTo(decoder.writable);
            const inputStream = decoder.readable;
            const reader = inputStream.getReader();

            while (true) {
                const { value, done } = await reader.read();
                if (done) break;
                handleData(value);
            }
        } catch (error) {
            console.error("Chyba:", error);
        }
    });

    function handleData(chunk) {
        lineBuffer += chunk;
        let lines = lineBuffer.split("\r\n");
        lineBuffer = lines.pop(); // Ponechá si rozpracovaný řádek

        lines.forEach(line => {
            line = line.trim();
            if (line.includes(':')) {
                let [key, val] = line.split(':');
                val = parseInt(val);

                if (key === "LED") {
                    document.getElementById('activeLedValue').innerText = val;
                } else if (key === "TIME") {
                    updateChart(val);
                } else if (key === "AVG") {
                    document.getElementById('avgTimeValue').innerText = val;
                }
            }
        });
    }

    function updateChart(newTime) {
        document.getElementById('lastTimeValue').innerText = newTime;
        
        times.push(newTime);
        labels.push(times.length);
        
        if(times.length > 15) {
            times.shift();
            labels.shift();
        }

        const currentAvg = Math.round(times.reduce((a, b) => a + b) / times.length);
        document.getElementById('avgTimeValue').innerText = currentAvg;

        chart.update('none'); // Rychlá aktualizace bez animace pro "speed" efekt
    }
</script>
</body>
</html>
