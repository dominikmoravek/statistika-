<!DOCTYPE html>
<html>
<head>
    <title>Reakční Trenažér - Statistiky</title>
    <style>
        body { font-family: sans-serif; text-align: center; background: #f4f4f4; }
        .stats { display: flex; justify-content: center; gap: 20px; margin-top: 20px; }
        .card { background: white; padding: 20px; border-radius: 10px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); min-width: 150px; }
        #log { width: 80%; height: 200px; margin: 20px auto; display: block; }
        button { padding: 10px 20px; font-size: 16px; cursor: pointer; background: #28a745; color: white; border: none; border-radius: 5px; }
    </style>
</head>
<body>
    <h1>Trenažér Postřehu</h1>
    <button id="connect">PŘIPOJIT MICRO:BIT</button>

    <div class="stats">
        <div class="card">
            <h3>Teplota</h3>
            <p id="temp">-- °C</p>
        </div>
        <div class="card">
            <h3>Poslední čas</h3>
            <p id="lastTime">-- ms</p>
        </div>
        <div class="card">
            <h3>Poslední LED</h3>
            <p id="lastLed">--</p>
        </div>
    </div>

    <textarea id="log" readonly placeholder="Zde se objeví surová data z micro:bitu..."></textarea>

    <script>
        let port;
        let reader;
        const btn = document.getElementById('connect');

        btn.addEventListener('click', async () => {
            try {
                port = await navigator.serial.requestPort();
                await port.open({ baudRate: 115200 });
                btn.innerText = "PŘIPOJENO";
                btn.style.background = "#007bff";
                readData();
            } catch (e) {
                console.error("Chyba při připojení:", e);
            }
        });

        async function readData() {
            const decoder = new TextDecoder();
            while (port.readable) {
                reader = port.readable.getReader();
                try {
                    while (true) {
                        const { value, done } = await reader.read();
                        if (done) break;
                        const text = decoder.decode(value);
                        processData(text);
                    }
                } finally {
                    reader.releaseLock();
                }
            }
        }

        function processData(data) {
            document.getElementById('log').value += data;
            document.getElementById('log').scrollTop = document.getElementById('log').scrollHeight;

            // Zpracování teploty (temp:25)
            if (data.includes("temp:")) {
                const t = data.split("temp:")[1].trim();
                document.getElementById('temp').innerText = t + " °C";
            }
            // Zpracování LED (LED=1) - MakeCode serial.writeValue posílá název=hodnota
            if (data.includes("LED=")) {
                const l = data.split("=")[1].trim();
                document.getElementById('lastLed').innerText = "LED " + l;
            }
            // Zpracování Času (TIME=350)
            if (data.includes("TIME=")) {
                const time = data.split("=")[1].trim();
                document.getElementById('lastTime').innerText = time + " ms";
            }
        }
    </script>
</body>
</html>
