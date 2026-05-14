<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>烟花盛宴 · 专属祝福</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }

        body {
            background: linear-gradient(145deg, #0a0f2a 0%, #0a1a3a 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Segoe UI', 'Noto Sans CJK SC', system-ui, -apple-system, sans-serif;
            padding: 20px;
            position: relative;
            overflow-x: hidden;
        }

        #fireworks-canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 1;
        }

        .card {
            position: relative;
            z-index: 2;
            background: rgba(255, 248, 240, 0.92);
            backdrop-filter: blur(8px);
            border-radius: 56px;
            box-shadow: 0 30px 50px rgba(0, 0, 0, 0.3), 0 0 0 1px rgba(255, 245, 220, 0.6);
            padding: 2rem 2rem 2.2rem;
            max-width: 640px;
            width: 100%;
            text-align: center;
            transition: all 0.2s ease;
            cursor: pointer;
            animation: fadeInUp 0.8s ease;
        }

        @keyframes fadeInUp {
            from {
                opacity: 0;
                transform: translateY(40px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        h1 {
            color: #c26a2e;
            font-size: 2.3rem;
            margin-bottom: 1rem;
            border-left: 6px solid #ffb56b;
            padding-left: 20px;
            display: inline-block;
            text-shadow: 2px 2px 8px rgba(0,0,0,0.05);
        }

        .blessing {
            font-size: 1.55rem;
            line-height: 1.5;
            color: #2f241b;
            margin: 2rem 0;
            background: rgba(255, 252, 245, 0.9);
            padding: 1.5rem;
            border-radius: 40px;
            box-shadow: inset 0 1px 3px #0001, 0 12px 20px -10px rgba(0,0,0,0.2);
            white-space: pre-line;
            font-weight: 500;
        }

        .footer {
            margin-top: 1.5rem;
            font-size: 1rem;
            color: #b16e3c;
            border-top: 1px solid #ffdfbf;
            padding-top: 1rem;
            font-style: italic;
        }

        .hint {
            margin-top: 1rem;
            font-size: 0.85rem;
            color: #b87a4a;
            background: rgba(255,235,210,0.6);
            display: inline-block;
            padding: 6px 18px;
            border-radius: 40px;
            backdrop-filter: blur(4px);
        }

        @media (max-width: 560px) {
            .card { padding: 1.5rem; }
            .blessing { font-size: 1.25rem; padding: 1.2rem; }
            h1 { font-size: 1.8rem; }
        }
    </style>
</head>
<body>

<canvas id="fireworks-canvas"></canvas>

<div class="card" id="firework-trigger-card">
    <h1>✨ 专属祝福 ✨</h1>
    <div class="blessing">
        🌟 愿你眼里有光，心中有爱 🌟<br>
        所有的努力都不被辜负，<br>
        每一个明天都比今天更灿烂。<br>
        🎉 健康 · 快乐 · 好运 🎉
    </div>
    <div class="footer">
        —— 来自最真挚的祝愿
    </div>
    <div class="hint">
        💥 点击任意位置 · 绽放更多烟花 💥
    </div>
</div>

<script>
    (function() {
        // ---------- 升级版烟花效果 ----------
        const canvas = document.getElementById('fireworks-canvas');
        let ctx = canvas.getContext('2d');
        let width, height;

        let particles = [];
        let rockets = [];
        
        // 自动烟花定时器
        let autoTimer = null;
        let autoStopTimer = null;
        
        // 配置：烟花持续时长 (毫秒) 默认90秒
        const FIREWORK_DURATION = 90 * 1000;  // 90秒
        
        // 烟花发射间隔 (毫秒) 更密集
        const FIREWORK_INTERVAL = 900;  // 0.9秒一发
        
        const GRAVITY = 0.12;
        const ROCKET_SPEED_BASE = 5.5;
        
        // 颜色池
        const colorPalette = [
            '#ff4d4d', '#ffa64d', '#ffff4d', '#4dff4d', '#4dd2ff', '#ff4da6', '#e6b800', '#ff6a00',
            '#ff3399', '#66ff33', '#ff6600', '#ff33cc', '#33ffcc', '#ff5050'
        ];
        
        function random(min, max) {
            return min + Math.random() * (max - min);
        }
        
        function randomColor() {
            return colorPalette[Math.floor(Math.random() * colorPalette.length)];
        }
        
        function resizeCanvas() {
            width = window.innerWidth;
            height = window.innerHeight;
            canvas.width = width;
            canvas.height = height;
        }
        
        // ---------- 粒子类 (升级: 带大小衰减和旋转可选) ----------
        class Particle {
            constructor(x, y, vx, vy, color, size = 3, life = 1.0, isSpark = false) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.color = color;
                this.size = size;
                this.life = life;
                this.decay = 0.012 + Math.random() * 0.018;
                this.alpha = 1;
                this.isSpark = isSpark; // 小火花衰减更快
                if (isSpark) this.decay *= 1.4;
            }
            update() {
                this.vx *= 0.98;
                this.vy += GRAVITY;
                this.x += this.vx;
                this.y += this.vy;
                this.life -= this.decay;
                this.alpha = this.life * 0.9;
                return this.life > 0;
            }
            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.min(this.alpha, 0.95);
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size * this.life, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                // 小核心高亮
                if (this.size > 2 && this.life > 0.5) {
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.size * this.life * 0.4, 0, Math.PI * 2);
                    ctx.fillStyle = '#fff9c4';
                    ctx.fill();
                }
                ctx.restore();
            }
        }
        
        // ---------- 火箭 ----------
        class Rocket {
            constructor(x, y, targetY, color) {
                this.x = x;
                this.y = y;
                this.targetY = targetY;
                this.color = color;
                this.vx = (Math.random() - 0.5) * 1.5;
                this.vy = -ROCKET_SPEED_BASE - Math.random() * 3;
                this.exploded = false;
                this.trail = [];
            }
            update() {
                this.vx *= 0.99;
                this.vy += 0.09;
                this.x += this.vx;
                this.y += this.vy;
                
                this.trail.push({ x: this.x, y: this.y });
                if (this.trail.length > 8) this.trail.shift();
                
                if (this.y <= this.targetY || this.y < 40) {
                    this.exploded = true;
                    this.explode();
                    return false;
                }
                if (this.y > height + 150) return false;
                return true;
            }
            explode() {
                // 主爆炸粒子数量 60~90 个 (比之前多)
                const mainCount = Math.floor(random(60, 90));
                const particlesThisExplosion = [];
                for (let i = 0; i < mainCount; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const speed = random(2.5, 7);
                    const vx = Math.cos(angle) * speed * (Math.random() * 0.7 + 0.5);
                    const vy = Math.sin(angle) * speed * (Math.random() * 0.7 + 0.5);
                    const color = this.color || randomColor();
                    const size = random(2.5, 5);
                    particles.push(new Particle(this.x, this.y, vx, vy, color, size, 0.95));
                }
                // 二次爆炸: 额外产生20~30个小火花，飞得更远
                const sparkCount = Math.floor(random(20, 35));
                for (let i = 0; i < sparkCount; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const speed = random(3, 8);
                    const vx = Math.cos(angle) * speed;
                    const vy = Math.sin(angle) * speed;
                    particles.push(new Particle(this.x, this.y, vx, vy, randomColor(), 1.8, 0.85, true));
                }
                // 额外加一圈星芒效果
                for (let i = 0; i < 12; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const sp = random(1.5, 4);
                    particles.push(new Particle(this.x, this.y, Math.cos(angle)*sp, Math.sin(angle)*sp, '#ffffaa', 2, 0.7));
                }
            }
            draw(ctx) {
                // 拖尾
                for (let i = 0; i < this.trail.length; i++) {
                    const t = this.trail[i];
                    const alpha = i / this.trail.length * 0.6;
                    ctx.beginPath();
                    ctx.arc(t.x, t.y, 2.8, 0, Math.PI*2);
                    ctx.fillStyle = `rgba(255, 180, 80, ${alpha*0.6})`;
                    ctx.fill();
                }
                ctx.beginPath();
                ctx.arc(this.x, this.y, 3.5, 0, Math.PI*2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.beginPath();
                ctx.arc(this.x, this.y, 1.8, 0, Math.PI*2);
                ctx.fillStyle = '#fff9c4';
                ctx.fill();
            }
        }
        
        // 发射烟花 (可指定点击坐标)
        function launchFirework(clickX = null, clickY = null) {
            let startX, startY, targetY;
            if (clickX !== null && clickY !== null && clickY > 0 && clickY < height) {
                startX = clickX;
                startY = Math.min(height - 20, clickY + 70);
                targetY = Math.max(70, clickY - random(50, 150));
            } else {
                startX = random(50, width - 50);
                startY = height - random(15, 45);
                targetY = random(80, height * 0.7);
            }
            const color = randomColor();
            rockets.push(new Rocket(startX, startY, targetY, color));
        }
        
        // 自动发射 (持续90秒)
        function startAutoFireworks() {
            if (autoTimer) clearInterval(autoTimer);
            autoTimer = setInterval(() => {
                // 限制火箭数量避免过多
                if (rockets.length < 20) {
                    launchFirework();
                }
            }, FIREWORK_INTERVAL);
            
            // 90秒后停止自动发射
            autoStopTimer = setTimeout(() => {
                if (autoTimer) {
                    clearInterval(autoTimer);
                    autoTimer = null;
                    console.log("烟花盛宴结束，不再自动发射");
                }
            }, FIREWORK_DURATION);
        }
        
        // 动画循环
        let animFrame;
        function animate() {
            if (!ctx) return;
            ctx.clearRect(0, 0, width, height);
            // 拖尾效果: 半透明黑色渐变层制造余影
            ctx.fillStyle = 'rgba(10, 15, 42, 0.25)';
            ctx.fillRect(0, 0, width, height);
            
            // 更新火箭
            for (let i = rockets.length-1; i >= 0; i--) {
                const alive = rockets[i].update();
                if (!alive) {
                    rockets.splice(i,1);
                } else {
                    rockets[i].draw(ctx);
                }
            }
            // 更新粒子
            for (let i = particles.length-1; i >= 0; i--) {
                const alive = particles[i].update();
                if (alive) {
                    particles[i].draw(ctx);
                } else {
                    particles.splice(i,1);
                }
            }
            // 性能保护: 粒子太多时裁剪
            if (particles.length > 1600) particles = particles.slice(-1400);
            
            animFrame = requestAnimationFrame(animate);
        }
        
        // 点击任意位置加烟花 (并且不会影响自动烟花)
        function handleClick(e) {
            let clientX = e.clientX;
            let clientY = e.clientY;
            if (clientX && clientY) {
                launchFirework(clientX, clientY);
            }
        }
        
        function initFireworks() {
            resizeCanvas();
            window.addEventListener('resize', () => {
                resizeCanvas();
            });
            startAutoFireworks();
            animate();
            document.body.addEventListener('click', handleClick);
            
            // 进入页面立即放3~5个开场烟花，增强即时感
            for (let i = 0; i < 5; i++) {
                setTimeout(() => launchFirework(), i * 120);
            }
        }
        
        initFireworks();
    })();
</script>
</body>
</html>
