<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pappagallo nella Foresta Pluviale</title>
    <meta name="description" content="Gioco HTML5 del pappagallo che vola nella foresta pluviale">
    <meta name="author" content="Creato con Claude AI">
    
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }
        
        body {
            font-family: 'Arial', sans-serif;
            background: linear-gradient(180deg, #2E8B57 0%, #228B22 50%, #006400 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            overflow: hidden;
        }
        
        #gameContainer {
            position: relative;
            background: rgba(255,255,255,0.1);
            border-radius: 15px;
            padding: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0,0,0,0.3);
        }
        
        #gameCanvas {
            border: 3px solid #654321;
            border-radius: 10px;
            background: linear-gradient(180deg, #87CEEB 0%, #98FB98 30%, #228B22 70%, #006400 100%);
            cursor: pointer;
            display: block;
        }
        
        #ui {
            background: rgba(139, 69, 19, 0.9);
            padding: 15px 20px;
            border-radius: 10px;
            margin-bottom: 15px;
            display: flex;
            justify-content: flex-start;
            align-items: center;
            color: white;
            gap: 20px;
        }
        
        .score {
            font-size: 20px;
            font-weight: bold;
            color: #ffffff;
        }
        
        button {
            background: linear-gradient(45deg, #FF4500, #FF6347);
            border: none;
            color: white;
            padding: 10px 20px;
            border-radius: 25px;
            cursor: pointer;
            font-weight: bold;
            font-size: 16px;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
        }
        
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(0,0,0,0.3);
        }
        
        button:active {
            transform: translateY(0);
        }
        
        #instructions {
            background: rgba(139, 69, 19, 0.9);
            padding: 15px;
            border-radius: 10px;
            margin-top: 15px;
            color: white;
            text-align: center;
            font-size: 14px;
            line-height: 1.5;
        }
        
        #gameOver {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(139, 69, 19, 0.95);
            padding: 30px;
            border-radius: 15px;
            display: none;
            box-shadow: 0 0 30px rgba(0,0,0,0.5);
            color: white;
            text-align: center;
            z-index: 1000;
        }
        
        #gameOver h2 {
            margin-bottom: 15px;
            color: #FFD700;
        }
        
        #gameOver p {
            margin-bottom: 20px;
            font-size: 18px;
        }
        
        /* Responsive design */
        @media (max-width: 860px) {
            #gameCanvas {
                width: 100%;
                max-width: 600px;
                height: auto;
            }
            
            #gameContainer {
                margin: 10px;
                padding: 15px;
            }
            
            #ui {
                flex-direction: column;
                gap: 10px;
                text-align: center;
            }
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <div id="ui">
            <button onclick="startGame()" aria-label="Inizia il gioco">🦜 Inizia Gioco!</button>
            <div class="score">Punteggio: <span id="score">0</span></div>
        </div>
        
        <canvas id="gameCanvas" width="800" height="400" aria-label="Area di gioco del pappagallo"></canvas>
        
        <div id="instructions">
            🦜 <strong>Come giocare:</strong><br>
            Clicca ovunque o premi la <strong>BARRA SPAZIATRICE</strong> per far volare il pappagallo!<br>
            Attraversa i varchi nelle scale di tronchi e accumula punti!
        </div>
        
        <div id="gameOver" role="dialog" aria-labelledby="gameOverTitle">
            <h2 id="gameOverTitle">🦜 Game Over!</h2>
            <p>Punteggio finale: <span id="finalScore">0</span></p>
            <button onclick="startGame()" aria-label="Riprova il gioco">🔄 Riprova!</button>
        </div>
    </div>

    <script>
        // Variabili globali del gioco
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const gameOverDiv = document.getElementById('gameOver');
        const finalScoreElement = document.getElementById('finalScore');
        
        // Stato del gioco
        let gameState = {
            player: { 
                x: 100, 
                y: 100, 
                width: 35, 
                height: 35, 
                velocityY: 0, 
                flying: false 
            },
            obstacles: [],
            score: 0,
            gameSpeed: 2,
            running: false,
            ground: 0
        };
        
        // Costanti di gioco
        const GAME_CONFIG = {
            FLAP_FORCE: -6,
            GRAVITY: 0.25,
            GAP_SIZE: 120,
            OBSTACLE_WIDTH: 40,
            OBSTACLE_COLORS: ['#8B4513', '#A0522D', '#D2691E', '#CD853F', '#DEB887'],
            GROUND_HEIGHT: 30
        };
        
        // Inizializzazione del gioco
        function initGame() {
            gameState.ground = canvas.height - GAME_CONFIG.GROUND_HEIGHT;
            drawWelcomeScreen();
        }
        
        // Schermata di benvenuto
        function drawWelcomeScreen() {
            ctx.fillStyle = '#FFFFFF';
            ctx.font = 'bold 24px Arial';
            ctx.textAlign = 'center';
            ctx.fillText('🦜 Clicca "Inizia Gioco!" per volare! 🌲', canvas.width / 2, canvas.height / 2);
        }
        
        // Avvia il gioco
        function startGame() {
            try {
                // Reset dello stato del gioco
                gameState = {
                    player: { 
                        x: 100, 
                        y: 100, 
                        width: 35, 
                        height: 35, 
                        velocityY: 0, 
                        flying: false 
                    },
                    obstacles: [],
                    score: 0,
                    gameSpeed: 2,
                    running: true,
                    ground: canvas.height - GAME_CONFIG.GROUND_HEIGHT
                };
                
                // Reset UI
                gameOverDiv.style.display = 'none';
                scoreElement.textContent = '0';
                
                // Avvia il loop di gioco
                requestAnimationFrame(gameLoop);
            } catch (error) {
                console.error('Errore nell\'avvio del gioco:', error);
            }
        }
        
        // Funzione di volo
        function flap() {
            if (!gameState.running) return;
            gameState.player.velocityY = GAME_CONFIG.FLAP_FORCE;
            gameState.player.flying = true;
        }
        
        // Crea ostacoli
        function createObstacle() {
            const gapY = 80 + Math.random() * (canvas.height - 200);
            const color = GAME_CONFIG.OBSTACLE_COLORS[Math.floor(Math.random() * GAME_CONFIG.OBSTACLE_COLORS.length)];
            
            // Ostacolo superiore
            gameState.obstacles.push({
                x: canvas.width,
                y: 0,
                width: GAME_CONFIG.OBSTACLE_WIDTH,
                height: gapY - GAME_CONFIG.GAP_SIZE / 2,
                color: color,
                passed: false,
                type: 'top'
            });
            
            // Ostacolo inferiore
            gameState.obstacles.push({
                x: canvas.width,
                y: gapY + GAME_CONFIG.GAP_SIZE / 2,
                width: GAME_CONFIG.OBSTACLE_WIDTH,
                height: gameState.ground - (gapY + GAME_CONFIG.GAP_SIZE / 2),
                color: color,
                passed: false,
                type: 'bottom'
            });
        }
        
        // Aggiorna il giocatore
        function updatePlayer() {
            gameState.player.velocityY += GAME_CONFIG.GRAVITY;
            gameState.player.y += gameState.player.velocityY;
            
            // Limiti dello schermo
            if (gameState.player.y < 0) {
                gameState.player.y = 0;
                gameState.player.velocityY = 0;
            }
            if (gameState.player.y >= gameState.ground - gameState.player.height) {
                endGame();
            }
        }
        
        // Aggiorna gli ostacoli
        function updateObstacles() {
            for (let i = gameState.obstacles.length - 1; i >= 0; i--) {
                const obstacle = gameState.obstacles[i];
                obstacle.x -= gameState.gameSpeed;
                
                // Controllo punteggio
                if (!obstacle.passed && obstacle.x + obstacle.width < gameState.player.x && obstacle.type === 'top') {
                    obstacle.passed = true;
                    gameState.score += 10;
                    scoreElement.textContent = gameState.score;
                    gameState.gameSpeed += 0.05;
                }
                
                // Rimozione ostacoli fuori schermo
                if (obstacle.x + obstacle.width < 0) {
                    gameState.obstacles.splice(i, 1);
                }
            }
            
            // Creazione nuovi ostacoli
            if (gameState.obstacles.length === 0 || 
                gameState.obstacles[gameState.obstacles.length - 1].x < canvas.width - 250) {
                createObstacle();
            }
        }
        
        // Controllo collisioni
        function checkCollisions() {
            const player = gameState.player;
            
            for (const obstacle of gameState.obstacles) {
                if (player.x + 5 < obstacle.x + obstacle.width - 5 &&
                    player.x + player.width - 5 > obstacle.x + 5 &&
                    player.y + 5 < obstacle.y + obstacle.height - 5 &&
                    player.y + player.height - 5 > obstacle.y + 5) {
                    
                    const centerX = obstacle.x + obstacle.width / 2;
                    const playerCenterX = player.x + player.width / 2;
                    
                    if (Math.abs(playerCenterX - centerX) < 30) {
                        const topObstacle = gameState.obstacles.find(obs => 
                            obs.type === 'top' && Math.abs(obs.x - obstacle.x) < 5);
                        
                        if (topObstacle) {
                            const gapStart = topObstacle.height;
                            const gapEnd = obstacle.y;
                            
                            if (player.y < gapStart || player.y + player.height > gapEnd) {
                                endGame();
                                return;
                            }
                        }
                    }
                }
            }
        }
        
        // Fine del gioco
        function endGame() {
            gameState.running = false;
            finalScoreElement.textContent = gameState.score;
            gameOverDiv.style.display = 'block';
        }
        
        // Disegna il giocatore (pappagallo)
        function drawPlayer() {
            const { x, y } = gameState.player;
            const size = 35;
            
            ctx.save();
            
            // Corpo del pappagallo
            ctx.fillStyle = '#00FF00';
            ctx.beginPath();
            ctx.ellipse(x + size/2, y + size*0.6, size*0.35, size*0.25, 0, 0, Math.PI * 2);
            ctx.fill();
            
            // Testa
            ctx.fillStyle = '#32CD32';
            ctx.beginPath();
            ctx.arc(x + size*0.7, y + size*0.35, size*0.2, 0, Math.PI * 2);
            ctx.fill();
            
            // Becco
            ctx.fillStyle = '#FF8C00';
            ctx.beginPath();
            ctx.moveTo(x + size*0.85, y + size*0.35);
            ctx.lineTo(x + size*0.95, y + size*0.4);
            ctx.lineTo(x + size*0.85, y + size*0.45);
            ctx.fill();
            
            // Ala
            ctx.fillStyle = '#0080FF';
            ctx.beginPath();
            ctx.ellipse(x + size*0.4, y + size*0.5, size*0.15, size*0.3, -0.3, 0, Math.PI * 2);
            ctx.fill();
            
            // Coda
            ctx.fillStyle = '#FF0000';
            ctx.beginPath();
            ctx.ellipse(x + size*0.1, y + size*0.6, size*0.15, size*0.1, 0.5, 0, Math.PI * 2);
            ctx.fill();
            
            // Occhio
            ctx.fillStyle = '#000000';
            ctx.beginPath();
            ctx.arc(x + size*0.75, y + size*0.3, size*0.04, 0, Math.PI * 2);
            ctx.fill();
            
            // Pupilla
            ctx.fillStyle = '#FFFFFF';
            ctx.beginPath();
            ctx.arc(x + size*0.76, y + size*0.29, size*0.02, 0, Math.PI * 2);
            ctx.fill();
            
            ctx.restore();
        }
        
        // Disegna gli ostacoli
        function drawObstacles() {
            gameState.obstacles.forEach(obstacle => {
                const gradient = ctx.createLinearGradient(obstacle.x, 0, obstacle.x + obstacle.width, 0);
                gradient.addColorStop(0, obstacle.color);
                gradient.addColorStop(0.5, '#8B4513');
                gradient.addColorStop(1, obstacle.color);
                
                ctx.fillStyle = gradient;
                ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
                
                // Texture del legno
                ctx.strokeStyle = '#654321';
                ctx.lineWidth = 2;
                for (let i = 0; i < obstacle.height; i += 20) {
                    ctx.beginPath();
                    ctx.moveTo(obstacle.x + 5, obstacle.y + i);
                    ctx.lineTo(obstacle.x + obstacle.width - 5, obstacle.y + i);
                    ctx.stroke();
                }
                
                // Ombra
                ctx.fillStyle = 'rgba(0,0,0,0.2)';
                ctx.fillRect(obstacle.x, obstacle.y, 3, obstacle.height);
                ctx.fillRect(obstacle.x + obstacle.width - 3, obstacle.y, 3, obstacle.height);
            });
        }
        
        // Disegna lo sfondo
        function drawBackground() {
            // Suolo
            ctx.fillStyle = '#654321';
            ctx.fillRect(0, gameState.ground, canvas.width, canvas.height - gameState.ground);
            
            // Erba
            ctx.fillStyle = '#228B22';
            for (let i = 0; i < canvas.width; i += 15) {
                ctx.fillRect(i, gameState.ground, 3, 8);
                ctx.fillRect(i + 7, gameState.ground, 2, 6);
            }
        }
        
        // Loop principale del gioco
        function gameLoop() {
            if (!gameState.running) return;
            
            try {
                // Pulisci il canvas
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                
                // Aggiorna lo stato del gioco
                updatePlayer();
                updateObstacles();
                checkCollisions();
                
                // Disegna tutto
                drawBackground();
                drawObstacles();
                drawPlayer();
                
                // Continua il loop
                requestAnimationFrame(gameLoop);
            } catch (error) {
                console.error('Errore nel loop di gioco:', error);
                endGame();
            }
        }
        
        // Event listeners
        function setupEventListeners() {
            // Click sul canvas
            canvas.addEventListener('click', flap, { passive: true });
            
            // Tastiera
            document.addEventListener('keydown', (event) => {
                if (event.code === 'Space') {
                    event.preventDefault();
                    flap();
                }
            });
            
            // Prevenzione del menu contestuale
            canvas.addEventListener('contextmenu', (event) => {
                event.preventDefault();
            });
        }
        
        // Inizializzazione quando la pagina è caricata
        document.addEventListener('DOMContentLoaded', () => {
            try {
                initGame();
                setupEventListeners();
            } catch (error) {
                console.error('Errore nell\'inizializzazione:', error);
                alert('Si è verificato un errore nel caricamento del gioco. Ricarica la pagina.');
            }
        });
        
        // Gestione degli errori globali
        window.addEventListener('error', (event) => {
            console.error('Errore globale:', event.error);
        });
    </script>
</body>
</html>
