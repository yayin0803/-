<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>烟花祝福 · 专属贺卡</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none; /* 避免拖动干扰点击烟花 */
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

        /* 烟花 Canvas 背景层 */
        #fireworks-canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;  /* 让点击穿透到卡片 */
            z-index: 1;
        }

        /* 祝福卡片层 */
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
            cursor: pointer;  /* 提示点击可放烟花 */
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
        // ---------- 烟花效果 (纯 Canvas 粒子系统) ----------
        const canvas = document.getElementById('fireworks-canvas');
        let ctx = canvas.getContext('2d');
        let width, height;

        // 粒子数组
        let particles = [];
        let rockets = [];       // 升空中的火箭
        let autoTimer = null;

        // 参数配置
        const GRAVITY = 0.15;
        const FIREWORK_INTERVAL = 2000;   // 自动烟花间隔(ms)
        const ROCKET_SPEED_BASE = 5;
        
        // 随机颜色 (暖色烟花)
        const colorPalette = [
            '#ff4d4d', '#ffa64d', '#ffff4d', '#4dff4d', '#4dd2ff', '#ff4da6', '#e6b800', '#ff6a00'
        ];

        function random(min, max) {
            return min + Math.random() * (max - min);
        }

        function randomColor() {
            return colorPalette[Math.floor(Math.random() * colorPalette.length)];
        }

        // 调整画布尺寸
        function resizeCanvas() {
            width = window.innerWidth;
            height = window.innerHeight;
            canvas.width = width;
            canvas.height = height;
        }

        // ----- 粒子（爆炸后的火星）-----
        class Particle {
            constructor(x, y, vx, vy, color, size = 3, life = 1.0) {
                this.x = x;
                this.y = y;
                this.vx = vx;
                this.vy = vy;
                this.color = color;
                this.size = size;
                this.life = life;      // 1 → 0
                this.decay = 0.015 + Math.random() * 0.02;
                this.alpha = 1;
            }
            update() {
                this.vx *= 0.98;
                this.vy += GRAVITY;
                this.x += this.vx;
                this.y += this.vy;
                this.life -= this.decay;
                this.alpha = this.life * 0.8;
                return this.life > 0;
            }
            draw(ctx) {
                ctx.save();
                ctx.globalAlpha = Math.min(this.alpha, 0.9);
                ctx.beginPath();
                ctx.arc(this.x, this.y, this.size * this.life, 0, Math.PI * 2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.restore();
            }
        }

        // ----- 火箭（向上飞，然后爆炸）-----
        class Rocket {
            constructor(x, y, targetY, color) {
                this.x = x;
                this.y = y;
                this.targetY = targetY;
                this.color = color;
                this.vx = (Math.random() - 0.5) * 1.2;
                this.vy = -ROCKET_SPEED_BASE - Math.random() * 3;
                this.exploded = false;
                this.trail = [];
            }
            update() {
                this.vx *= 0.99;
                this.vy += 0.08;
                this.x += this.vx;
                this.y += this.vy;
                
                // 记录轨迹
                this.trail.push({ x: this.x, y: this.y });
                if (this.trail.length > 6) this.trail.shift();
                
                // 到达目标高度 或者 超出屏幕顶部 则爆炸
                if (this.y <= this.targetY || this.y < 50) {
                    this.exploded = true;
                    this.explode();
                    return false;
                }
                // 超出屏幕下方（极少情况）
                if (this.y > height + 100) return false;
                return true;
            }
            explode() {
                const particleCount = 45 + Math.floor(Math.random() * 30);
                for (let i = 0; i < particleCount; i++) {
                    const angle = Math.random() * Math.PI * 2;
                    const speed = random(2, 6);
                    const vx = Math.cos(angle) * speed * (Math.random() * 0.8 + 0.6);
                    const vy = Math.sin(angle) * speed * (Math.random() * 0.8 + 0.6);
                    const color = this.color || randomColor();
                    const size = random(2, 4.5);
                    particles.push(new Particle(this.x, this.y, vx, vy, color, size, 0.95));
                }
                // 额外加一圈小星芒
                for (let i=0;i<12;i++) {
                    const angle = Math.random() * Math.PI*2;
                    const sp = random(1.2, 3);
                    particles.push(new Particle(this.x, this.y, Math.cos(angle)*sp, Math.sin(angle)*sp, this.color, 1.5, 0.7));
                }
            }
            draw(ctx) {
                // 画轨迹
                for (let i=0; i<this.trail.length; i++) {
                    const t = this.trail[i];
                    const alpha = i / this.trail.length * 0.6;
                    ctx.beginPath();
                    ctx.arc(t.x, t.y, 2.5, 0, Math.PI*2);
                    ctx.fillStyle = `rgba(255, 180, 80, ${alpha*0.5})`;
                    ctx.fill();
                }
                ctx.beginPath();
                ctx.arc(this.x, this.y, 3, 0, Math.PI*2);
                ctx.fillStyle = this.color;
                ctx.fill();
                ctx.beginPath();
                ctx.arc(this.x, this.y, 1.5, 0, Math.PI*2);
                ctx.fillStyle = '#fff9c4';
                ctx.fill();
            }
        }

        // 发射一枚随机位置的烟花
        function launchFirework(clickX = null, clickY = null) {
            let startX, startY, targetY;
            if (clickX !== null && clickY !== null && clickY > 0 && clickY < height) {
                // 点击位置发射：从点击位置底部下方开始，飞向点击高度附近
                startX = clickX;
                startY = Math.min(height - 20, clickY + 60);
                targetY = Math.max(80, clickY - random(40, 120));
            } else {
                // 自动烟花：从屏幕底部随机水平位置升起
                startX = random(40, width - 40);
                startY = height - random(10, 40);
                targetY = random(90, height * 0.65);
            }
            const color = randomColor();
            rockets.push(new Rocket(startX, startY, targetY, color));
        }

        // 定期自动发射
        function startAutoFireworks() {
            if (autoTimer) clearInterval(autoTimer);
            autoTimer = setInterval(() => {
                // 自动烟花每秒最多不超过限制，且不超过火箭数量太多
                if (rockets.length < 12) {
                    launchFirework();
                }
            }, FIREWORK_INTERVAL);
        }

        // 动画循环
        let animFrame;
        function animate() {
            if (!ctx) return;
            ctx.clearRect(0, 0, width, height);
            ctx.globalCompositeOperation = 'source-over';
            // 绘制半透明拖尾效果 (制造余影)
            ctx.fillStyle = 'rgba(10, 15, 42, 0.25)';
            ctx.fillRect(0, 0, width, height);
            
            // 更新火箭
            for (let i=rockets.length-1; i>=0; i--) {
                const alive = rockets[i].update();
                if (!alive) {
                    rockets.splice(i,1);
                } else {
                    rockets[i].draw(ctx);
                }
            }
            // 更新粒子
            for (let i=particles.length-1; i>=0; i--) {
                const alive = particles[i].update();
                if (alive) {
                    particles[i].draw(ctx);
                } else {
                    particles.splice(i,1);
                }
            }
            
            // 限制粒子总数，避免过多卡顿
            if (particles.length > 1200) particles = particles.slice(-1000);
            
            animFrame = requestAnimationFrame(animate);
        }

        // 窗口点击额外烟花 (仅在卡片区域或任意处触发)
        function handleClick(e) {
            // 获取点击坐标
            let clientX = e.clientX;
            let clientY = e.clientY;
            if (clientX && clientY) {
                launchFirework(clientX, clientY);
                // 额外加一个小闪烁反馈
                setTimeout(() => {}, 10);
            }
        }

        // 初始化
        function initFireworks() {
            resizeCanvas();
            window.addEventListener('resize', () => {
                resizeCanvas();
                // 不清空粒子但重置尺寸会让画布适配，之前的坐标会偏移，但影响不大
            });
            // 自动烟花
            startAutoFireworks();
            // 动画开始
            animate();
            // 绑定点击任意位置触发烟花
            document.body.addEventListener('click', handleClick);
            // 首次进入立刻放几朵欢迎烟花
            setTimeout(() => {
                for (let i = 0; i < 3; i++) {
                    setTimeout(() => launchFirework(), i * 180);
                }
            }, 200);
        }

        initFireworks();
    })();
</script>
</body>
</html>
