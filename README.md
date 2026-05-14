# <生日快乐>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>祝福卡片</title>
    <style>
        body {
            background: linear-gradient(145deg, #fff5e6 0%, #ffe0c4 100%);
            font-family: 'Segoe UI', 'Noto Sans CJK SC', system-ui, -apple-system, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .card {
            background: rgba(255, 248, 240, 0.95);
            border-radius: 48px;
            box-shadow: 0 25px 45px rgba(0,0,0,0.2), 0 0 0 1px rgba(255,255,200,0.5);
            padding: 2rem 1.8rem;
            max-width: 600px;
            text-align: center;
            backdrop-filter: blur(2px);
            transition: all 0.2s;
        }
        h1 {
            color: #b45f2b;
            font-size: 2.2rem;
            margin-bottom: 1rem;
            border-left: 6px solid #f7b05e;
            padding-left: 18px;
            display: inline-block;
        }
        .blessing {
            font-size: 1.5rem;
            line-height: 1.45;
            color: #3e2a1f;
            margin: 2rem 0;
            white-space: pre-line;
            background: #fffef7;
            padding: 1.3rem;
            border-radius: 32px;
            box-shadow: inset 0 1px 4px #0001, 0 6px 12px -8px rgba(0,0,0,0.2);
        }
        .footer {
            margin-top: 2rem;
            font-size: 0.9rem;
            color: #b87a4a;
            border-top: 1px solid #ffdfbf;
            padding-top: 1.2rem;
        }
        @media (max-width: 500px) {
            .blessing { font-size: 1.25rem; }
            h1 { font-size: 1.8rem; }
            .card { padding: 1.5rem; }
        }
    </style>
</head>
<body>
<div class="card">
    <h1>✨ 专属祝福 ✨</h1>
    <div class="blessing">
        <!-- 👇 这里改成你想要的任意文字，支持换行和emoji -->
        愿你眼里有光，心中有爱。<br>
        所有的努力都不被辜负，<br>
        每一个明天都比今天更灿烂。<br>
        🌟 健康 · 快乐 · 好运 🌟
    </div>
    <div class="footer">
        —— 来自 LQ 的真心祝愿
    </div>
</div>
</body>
</html>
