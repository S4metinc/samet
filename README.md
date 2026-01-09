
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>sametINC | Bio</title>
    <style>
        /* Modern ve Lüks Fontlar */
        @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@100;400;800&family=Cinzel:wght@400;700&display=swap');

        * { margin: 0; padding: 0; box-sizing: border-box; }

        body, html {
            width: 100%;
            height: 100%;
            background-color: #000;
            overflow: hidden;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'JetBrains Mono', monospace;
            cursor: none; /* Gizemli hava için imleci gizle */
        }

        /* Arka Plan: Fırtınalı Gece */
        .background-storm {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(rgba(0,0,0,0.92), rgba(0,0,0,0.85)), 
                        url('https://images.unsplash.com/photo-1431440869543-efaf3388c585?q=80&w=2070&auto=format&fit=crop');
            background-size: cover;
            background-position: center;
            filter: grayscale(100%) brightness(5%) contrast(170%) blur(2px);
            z-index: 0;
            transform: scale(1.1);
            transition: filter 0.1s ease-out;
        }

        /* Kanvas Katmanları */
        canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }

        #data-canvas { z-index: 5; opacity: 0.08; }
        #rain-canvas { z-index: 10; opacity: 0.3; }

        .glass-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: radial-gradient(circle, transparent 20%, #000 100%);
            z-index: 15;
        }

        /* Logo ve İçerik */
        .logo-container {
            position: relative;
            z-index: 30;
            text-align: center;
            display: flex;
            flex-direction: column;
            align-items: center;
            animation: intro-anim 2s ease-out;
        }

        .logo-text {
            display: flex;
            align-items: center;
            gap: 15px;
            color: #fff;
            text-transform: uppercase;
        }

        .brand-name {
            font-family: 'Cinzel', serif;
            font-weight: 400;
            font-size: clamp(2rem, 8vw, 6.5rem); /* Mobil uyumlu boyut */
            letter-spacing: 0.5em;
            color: #ffffff;
            background: linear-gradient(90deg, #333 0%, #fff 50%, #333 100%);
            background-size: 200% auto;
            -webkit-background-clip: text;
            background-clip: text;
            -webkit-text-fill-color: transparent;
            filter: drop-shadow(0 0 20px rgba(255, 255, 255, 0.2));
            animation: text-flow 12s linear infinite;
        }

        .inc-tag {
            font-family: 'JetBrains Mono', monospace;
            font-weight: 800;
            font-size: clamp(0.6rem, 2vw, 1.2rem);
            border-left: 2px solid rgba(255, 255, 255, 0.6);
            padding: 5px 15px;
            color: #fff;
            letter-spacing: 0.3em;
            opacity: 0.6;
        }

        .status-line {
            width: 150px;
            height: 1px;
            background: rgba(255, 255, 255, 0.1);
            margin-top: 40px;
            position: relative;
            overflow: hidden;
        }

        .status-line::after {
            content: '';
            position: absolute;
            left: -100%;
            width: 100%;
            height: 100%;
            background: linear-gradient(90deg, transparent, rgba(255,255,255,0.5), transparent);
            animation: scan 4s infinite;
        }

        .lightning-bolt {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: #fff;
            opacity: 0;
            z-index: 20;
            pointer-events: none;
        }

        /* Animasyonlar */
        @keyframes text-flow { to { background-position: 200% center; } }
        @keyframes scan { 0% { left: -100%; } 100% { left: 100%; } }
        @keyframes intro-anim {
            from { opacity: 0; filter: blur(20px); transform: translateY(20px); }
            to { opacity: 1; filter: blur(0); transform: translateY(0); }
        }

        .logo-container {
            animation: float 10s ease-in-out infinite alternate;
        }

        @keyframes float {
            from { transform: translateY(0); }
            to { transform: translateY(-15px); }
        }

        /* Biyografi Bilgi Yazısı */
        .footer-info {
            position: absolute;
            bottom: 30px;
            font-size: 10px;
            color: rgba(255,255,255,0.15);
            letter-spacing: 4px;
            text-transform: uppercase;
            z-index: 25;
        }

    </style>
</head>
<body>

    <div class="background-storm" id="bg"></div>
    <div class="lightning-bolt" id="flash"></div>
    <div class="glass-overlay"></div>
    
    <canvas id="data-canvas"></canvas>
    <canvas id="rain-canvas"></canvas>

    <div class="logo-container">
        <div class="logo-text">
            <span class="brand-name">samet</span>
            <span class="inc-tag">INC</span>
        </div>
        <div class="status-line"></div>
    </div>

    <div class="footer-info">System Secure // Night Ops // Established 2024</div>

    <script>
        const rCanvas = document.getElementById('rain-canvas');
        const dCanvas = document.getElementById('data-canvas');
        const rCtx = rCanvas.getContext('2d');
        const dCtx = dCanvas.getContext('2d');
        const flash = document.getElementById('flash');
        const bg = document.getElementById('bg');

        // Değişken isimlerini çakışmayı önlemek için güncelledik
        let canvasW, canvasH, rainDrops = [], matrixData = [];

        function setup() {
            canvasW = window.innerWidth;
            canvasH = window.innerHeight;
            rCanvas.width = dCanvas.width = canvasW;
            rCanvas.height = dCanvas.height = canvasH;

            rainDrops = [];
            for(let i=0; i<100; i++) {
                rainDrops.push({
                    x: Math.random() * canvasW, 
                    y: Math.random() * canvasH, 
                    s: Math.random() * 2 + 1
                });
            }

            matrixData = Array(Math.floor(canvasW / 25)).fill(0);
        }

        function draw() {
            // Dijital Veri Akışı (Matrix)
            dCtx.fillStyle = "rgba(0,0,0,0.15)";
            dCtx.fillRect(0, 0, canvasW, canvasH);
            dCtx.fillStyle = "rgba(255,255,255,0.1)";
            dCtx.font = "12px monospace";
            matrixData.forEach((y, i) => {
                dCtx.fillText(Math.floor(Math.random() * 2), i * 25, y * 20);
                if(y * 20 > canvasH && Math.random() > 0.98) {
                    matrixData[i] = 0;
                } else {
                    matrixData[i]++;
                }
            });

            // Fiziksel Yağmur Efekti
            rCtx.clearRect(0, 0, canvasW, canvasH);
            rCtx.fillStyle = "rgba(255, 255, 255, 0.3)";
            rainDrops.forEach(d => {
                rCtx.beginPath();
                rCtx.arc(d.x, d.y, d.s * 0.5, 0, Math.PI * 2);
                rCtx.fill();
                d.y += d.s * 0.5;
                if(d.y > canvasH) d.y = -10;
            });

            // Şimşek Mekanizması
            if(Math.random() > 0.997) {
                flash.style.opacity = "0.3";
                bg.style.filter = "grayscale(100%) brightness(120%) contrast(300%) blur(0px)";
                setTimeout(() => {
                    flash.style.opacity = "0";
                    bg.style.filter = "grayscale(100%) brightness(5%) contrast(170%) blur(2px)";
                }, 100);
            }

            requestAnimationFrame(draw);
        }

        window.addEventListener('resize', setup);
        setup();
        draw();
    </script>
</body>
</html>
