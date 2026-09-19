
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Happy Birthday!</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            background-color: #0b0c10;
            overflow: hidden;
            font-family: 'Courier New', Courier, monospace;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        canvas {
            display: block;
            background: #0b0c10;
        }
        #overlay {
            position: absolute;
            color: #fff;
            text-align: center;
            cursor: pointer;
            z-index: 10;
            background: rgba(0, 0, 0, 0.7);
            padding: 20px 40px;
            border-radius: 10px;
            border: 1px solid #ff69b4;
            box-shadow: 0 0 15px rgba(255, 105, 180, 0.5);
        }
    </style>
</head>
<body>

    <div id="overlay">Click anywhere to start 🎉</div>
    <canvas id="canvas"></canvas>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');

        let width = canvas.width = window.innerWidth;
        let height = canvas.height = window.innerHeight;

        window.addEventListener('resize', () => {
            width = canvas.width = window.innerWidth;
            height = canvas.height = window.innerHeight;
            calculateTargetPositions();
        });

        // Message setup
        const lines = ["HAPPY", "INTAL'S", "BIRTHDAY"];
        let letters = [];

        const opts = {
            charSize: 32,
            charSpacing: 35,
            lineHeight: 50,
            fireworkSpawnTime: 30,
            gravity: 0.1
        };

        function calculateTargetPositions() {
            letters = [];
            const startY = height / 2 - (lines.length * opts.lineHeight) / 2;

            lines.forEach((line, lineIdx) => {
                const lineWidth = line.length * opts.charSpacing;
                const startX = width / 2 - lineWidth / 2 + opts.charSpacing / 2;
                const y = startY + lineIdx * opts.lineHeight;

                for (let i = 0; i < line.length; i++) {
                    const char = line[i];
                    if (char !== ' ') {
                        const x = startX + i * opts.charSpacing;
                        letters.push(new Letter(char, x, y));
                    }
                }
            });
        }

        // Letter Class
        function Letter(char, x, y) {
            this.char = char;
            this.x = x;
            this.y = y;

            this.fireworkY = height;
            this.fireworkX = x;

            const hue = (x / width) * 360;
            this.color = `hsl(${hue}, 80%, 60%)`;
            this.lightColor = `hsl(${hue}, 100%, 80%)`;

            this.reset();
        }

        Letter.prototype.reset = function () {
            this.phase = 'firework';
            this.tick = 0;
            this.spawned = false;
            this.spawningTime = Math.floor(Math.random() * 150) + 10;
            this.currentY = height;
            this.vy = -(Math.random() * 3 + 7);
            this.particles = [];
        };

        Letter.prototype.step = function () {
            if (this.phase === 'firework') {
                this.tick++;
                if (this.tick < this.spawningTime) return;

                this.currentY += this.vy;

                // Create rocket tail trail
                if (Math.random() < 0.5) {
                    this.particles.push({
                        x: this.x + (Math.random() - 0.5) * 4,
                        y: this.currentY,
                        vx: (Math.random() - 0.5) * 1,
                        vy: Math.random() * 2,
                        alpha: 1,
                        color: this.lightColor
                    });
                }

                // Check explosion peak near target position
                if (this.currentY <= this.y) {
                    this.phase = 'letter';
                    // Create explosion particles
                    for (let i = 0; i < 20; i++) {
                        const angle = Math.random() * Math.PI * 2;
                        const speed = Math.random() * 4 + 1;
                        this.particles.push({
                            x: this.x,
                            y: this.y,
                            vx: Math.cos(angle) * speed,
                            vy: Math.sin(angle) * speed,
                            alpha: 1,
                            color: this.color
                        });
                    }
                }
            }
        };

        Letter.prototype.draw = function () {
            // Draw explosion & rocket particles
            for (let i = this.particles.length - 1; i >= 0; i--) {
                const p = this.particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.alpha -= 0.02;

                if (p.alpha <= 0) {
                    this.particles.splice(i, 1);
                    continue;
                }

                ctx.save();
                ctx.globalAlpha = p.alpha;
                ctx.fillStyle = p.color;
                ctx.beginPath();
                ctx.arc(p.x, p.y, 2, 0, Math.PI * 2);
                ctx.fill();
                ctx.restore();
            }

            if (this.phase === 'firework' && this.tick >= this.spawningTime) {
                // Rocket head
                ctx.fillStyle = this.lightColor;
                ctx.beginPath();
                ctx.arc(this.x, this.currentY, 3, 0, Math.PI * 2);
                ctx.fill();
            } else if (this.phase === 'letter') {
                // Glow effect
                ctx.save();
                ctx.shadowColor = this.color;
                ctx.shadowBlur = 15;
                ctx.fillStyle = this.color;
                ctx.font = `bold ${opts.charSize}px 'Courier New', monospace`;
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText(this.char, this.x, this.y);
                ctx.restore();
            }
        };

        // Synthesize Happy Birthday Melody using Web Audio API
        function playAudio() {
            const AudioContext = window.AudioContext || window.webkitAudioContext;
            const audioCtx = new AudioContext();

            const notes = [
                { note: 261.63, duration: 0.4 }, { note: 261.63, duration: 0.4 },
                { note: 293.66, duration: 0.8 }, { note: 261.63, duration: 0.8 },
                { note: 349.23, duration: 0.8 }, { note: 329.63, duration: 1.2 },

                { note: 261.63, duration: 0.4 }, { note: 261.63, duration: 0.4 },
                { note: 293.66, duration: 0.8 }, { note: 261.63, duration: 0.8 },
                { note: 392.00, duration: 0.8 }, { note: 349.23, duration: 1.2 },

                { note: 261.63, duration: 0.4 }, { note: 261.63, duration: 0.4 },
                { note: 523.25, duration: 0.8 }, { note: 440.00, duration: 0.8 },
                { note: 349.23, duration: 0.8 }, { note: 329.63, duration: 0.8 },
                { note: 293.66, duration: 0.8 },

                { note: 466.16, duration: 0.4 }, { note: 466.16, duration: 0.4 },
                { note: 440.00, duration: 0.8 }, { note: 349.23, duration: 0.8 },
                { note: 392.00, duration: 0.8 }, { note: 349.23, duration: 1.2 }
            ];

            let currentTime = audioCtx.currentTime + 0.1;

            function playMelody() {
                notes.forEach(item => {
                    const osc = audioCtx.createOscillator();
                    const gain = audioCtx.createGain();

                    osc.type = 'triangle';
                    osc.frequency.value = item.note;

                    gain.gain.setValueAtTime(0.3, currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.001, currentTime + item.duration - 0.05);

                    osc.connect(gain);
                    gain.connect(audioCtx.destination);

                    osc.start(currentTime);
                    osc.stop(currentTime + item.duration);

                    currentTime += item.duration;
                });

                // Loop melody
                setTimeout(playMelody, (currentTime - audioCtx.currentTime) * 1000);
            }

            playMelody();
        }

        // Main Animation Loop
        function animate() {
            ctx.fillStyle = 'rgba(11, 12, 16, 0.2)';
            ctx.fillRect(0, 0, width, height);

            let allExploded = true;
            letters.forEach(letter => {
                letter.step();
                letter.draw();
                if (letter.phase !== 'letter') allExploded = false;
            });

            // Loop animation back after holding for a bit
            if (allExploded) {
                setTimeout(() => {
                    letters.forEach(letter => letter.reset());
                }, 3000);
            }

            requestAnimationFrame(animate);
        }

        // Setup & Start on user interaction (browser audio requirement)
        calculateTargetPositions();

        document.body.addEventListener('click', () => {
            const overlay = document.getElementById('overlay');
            if (overlay) overlay.style.display = 'none';
            playAudio();
            animate();
        }, { once: true });

    </script>
</body>
</html>
