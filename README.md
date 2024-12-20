# Simulasi-Arah-Mata-Angin
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulasi Navigasi Kapal Layar</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;700&display=swap');
        
        body {
            margin: 0;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            background-color: #e8eced;
            font-family: 'Roboto', sans-serif;
            overflow: hidden;
        }
        #simulation-container {
            width: 100vw;
            height: 100vh;
            position: relative;
            background-color: #4682B4;
            overflow: hidden;
        }
        .wave {
            position: absolute;
            width: 200%;
            height: 50vh;
            background: rgba(0, 99, 160, 0.735);
            border-radius: 100%;
            bottom: -25vh;
            animation: wave-animation 4s infinite linear;
        }
        .wave:nth-child(2) {
            animation-delay: -2s;
            opacity: 0.8;
            height: 45vh;
        }
        .wave:nth-child(3) {
            animation-delay: -1s;
            opacity: 0.6;
            height: 40vh;
        }
        @keyframes wave-animation {
            0% { transform: translateX(0); }
            100% { transform: translateX(-50%); }
        }

        .wind-line {
            position: absolute;
            width: 1px;
            background: rgba(255, 255, 255, 0.3);
            animation: wind-animation 2s infinite linear;
        }
        @keyframes wind-animation {
            0% { transform: translateY(-100%); }
            100% { transform: translateY(100vh); }
        }

        #title {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 24px;
            font-weight: 700;
            color: rgb(20, 2, 2);
            background-color: rgb(250, 250, 250);
            padding: 10px 20px;
            border-radius: 8px;
            font-family: 'Roboto', sans-serif;
            z-index: 100;
        }

        #sailboat {
            position: absolute;
            width: 50px;
            height: 60px;
            transition: transform 0.3s ease;
            animation: boat-wave 2s infinite ease-in-out;
        }

        .sailboat-shape {
            width: 100%;
            height: 100%;
        }

        .message-bubble {
            position: absolute;
            background-color: white;
            border-radius: 10px;
            padding: 10px;
            max-width: 200px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
            font-size: 14px;
            opacity: 0;
            transition: opacity 0.3s ease;
            pointer-events: none;
            z-index: 100;
        }
        .message-bubble::after {
            content: '';
            position: absolute;
            bottom: -10px;
            left: 50%;
            transform: translateX(-50%);
            border-width: 10px 10px 0;
            border-style: solid;
            border-color: white transparent transparent;
        }
        @keyframes boat-wave {
            0%, 100% { transform: translateY(0) rotate(var(--rotation)); }
            50% { transform: translateY(-5px) rotate(calc(var(--rotation) + 1deg)); }
        }

        #wind-direction {
            position: absolute;
            top: 20px;
            left: 20px;
            font-weight: 700;
            color: rgb(255, 255, 255);
            background-color: rgba(0,0,0,0.5);
            padding: 5px 10px;
            border-radius: 5px;
            z-index: 100;
        }

        #wind-compass {
            position: absolute;
            top: 20px;
            right: 20px;
            width: 100px;
            height: 100px;
        }
        #controls {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            gap: 10px;
            z-index: 100;
        }
        button {
            padding: 10px 20px;
            background-color: #d65f0a;
            color: white;
            border: none;
            cursor: pointer;
            font-family: 'Roboto', sans-serif;
            border-radius: 4px;
            transition: background-color 0.3s ease;
        }
        button:hover {
            background-color: #d0530b;
        }
    </style>
</head>
<body>
    <div id="simulation-container">
        <div id="title">Simulasi Navigasi Kapal Layar</div>
        <div class="wave"></div>
        <div class="wave"></div>
        <div class="wave"></div>
        <div id="wind-direction">Arah Angin: Utara</div>
        <img id="wind-compass" src="Asset/Arah Mata Angin.png" alt="Kompas Angin">
        <div id="sailboat">
            <svg class="sailboat-shape" viewBox="0 0 100 120" xmlns="http://www.w3.org/2000/svg">
                <!-- Hull (deck) -->
                <path d="M35 30 L65 30 L70 90 L30 90 Z" fill="#8B4513"/>
                <!-- Hull shape (bow) -->
                <path d="M30 90 L50 110 L70 90" fill="#8B4513"/>
                <!-- Hull shape (stern) -->
                <path d="M35 30 L50 10 L65 30" fill="#8B4513"/>
                <!-- Main sail -->
                <path d="M50 40 L50 85 L65 85 Z" fill="#FFFFFF" stroke="#444" stroke-width="1"/>
                <!-- Jib sail -->
                <path d="M50 40 L50 85 L35 85 Z" fill="#F0F0F0" stroke="#444" stroke-width="1"/>
                <!-- Mast point -->
                <circle cx="50" cy="40" r="3" fill="#444"/>
                <!-- Deck details -->
                <line x1="40" y1="50" x2="60" y2="50" stroke="#966F33" stroke-width="1"/>
                <line x1="40" y1="70" x2="60" y2="70" stroke="#966F33" stroke-width="1"/>
            </svg>
        </div>
        <div class="message-bubble"></div>
        <div id="controls">
            <button onclick="changeSailDirection('kiri')">Belok Kiri</button>
            <button onclick="changeSailDirection('kanan')">Belok Kanan</button>
        </div>
    </div>

    <script>
        const sailboat = document.getElementById('sailboat');
        const container = document.getElementById('simulation-container');
        const messageBubble = document.querySelector('.message-bubble');

        // Constants and variables
        const MOVEMENT_SPEED = 2;
        const ROTATION_SPEED = 5;
        const WIND_ANGLE = 270; // North wind (up)
        const ACCEPTABLE_WIND_RANGE = 90; // Degrees of acceptable sailing angle

        let boatX = container.clientWidth / 2 - 20;
        let boatY = container.clientHeight / 2;
        let boatRotation = 270;
        let isMoving = true;

        // Create wind lines
        function createWindLines() {
            const numLines = 20;
            for (let i = 0; i < numLines; i++) {
                const line = document.createElement('div');
                line.className = 'wind-line';
                line.style.left = `${Math.random() * 100}%`;
                line.style.height = `${20 + Math.random() * 30}px`;
                line.style.animationDuration = `${1 + Math.random()}s`;
                line.style.animationDelay = `${-Math.random() * 2}s`;
                container.appendChild(line);
            }
        }

        function showMessage(message) {
            messageBubble.textContent = message;
            messageBubble.style.opacity = '1';
            messageBubble.style.left = `${boatX}px`;
            messageBubble.style.top = `${boatY - 80}px`;
            
            setTimeout(() => {
                messageBubble.style.opacity = '0';
            }, 3000);
        }

        function isValidSailingAngle(rotation) {
            let angleDiff = Math.abs(((rotation - WIND_ANGLE + 180 + 360) % 360) - 180);
            return angleDiff <= ACCEPTABLE_WIND_RANGE;
        }

        function updateBoatPosition() {
            boatX = Math.max(0, Math.min(boatX, container.clientWidth - 40));
            boatY = Math.max(0, Math.min(boatY, container.clientHeight - 60));

            sailboat.style.left = `${boatX}px`;
            sailboat.style.top = `${boatY}px`;
            sailboat.style.setProperty('--rotation', `${boatRotation}deg`);
        }

        function moveBoat() {
            if (!isMoving) return;

            const angleRad = boatRotation * (Math.PI / 180);
            
            if (isValidSailingAngle(boatRotation)) {
                const deltaX = MOVEMENT_SPEED * Math.cos(angleRad);
                const deltaY = MOVEMENT_SPEED * Math.sin(angleRad);

                boatX += deltaX;
                boatY += deltaY;

                // Wrap around screen edges
                if (boatX > container.clientWidth) boatX = 0;
                if (boatX < 0) boatX = container.clientWidth - 40;
                if (boatY > container.clientHeight) boatY = 0;
                if (boatY < 0) boatY = container.clientHeight - 60;

                updateBoatPosition();
            }
        }

        function changeSailDirection(direction) {
            const oldRotation = boatRotation;
            
            if (direction === 'kiri') {
                boatRotation = (boatRotation - ROTATION_SPEED + 360) % 360;
                if (isValidSailingAngle(boatRotation)) {
                    showMessage("Kapal berbelok ke kiri untuk menangkap angin dengan lebih baik!");
                } else {
                    boatRotation = oldRotation;
                    showMessage("Tidak bisa berbelok ke kiri, akan melawan arah angin!");
                }
            } else if (direction === 'kanan') {
                boatRotation = (boatRotation + ROTATION_SPEED) % 360;
                if (isValidSailingAngle(boatRotation)) {
                    showMessage("Kapal berbelok ke kanan untuk mengoptimalkan arah layar!");
                } else {
                    boatRotation = oldRotation;
                    showMessage("Tidak bisa berbelok ke kanan, akan melawan arah angin!");
                }
            }

            sailboat.style.setProperty('--rotation', `${boatRotation}deg`);
        }

        function startSimulation() {
            createWindLines();
            setInterval(moveBoat, 30);
        }

        updateBoatPosition();
        startSimulation();
    </script>
</body>
</html>
