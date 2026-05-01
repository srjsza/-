<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>光屿 V16 - 滤镜工坊</title>
    <script src="https://cdn.tailwindcss.com">
    </script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Serif+SC:wght@400;600;700&family=Inter:wght@300;400;500;700&display=swap" rel="stylesheet">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: #0C0B0F;
            font-family: 'Inter', 'Noto Sans SC', sans-serif;
            color: #e4e4e7;
            -webkit-tap-highlight-color: transparent;
            overflow-x: hidden;
            display: flex;
            justify-content: center;
            align-items: flex-start;
            min-height: 100vh;
            padding: 20px 12px 40px;
            position: relative;
        }

        /* 暗房噪点纹理 */
        body::before {
            content: '';
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 0;
            background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.03'/%3E%3C/svg%3E");
            background-size: 200px 200px;
        }

        /* 主容器 */
        .app-container {
            position: relative;
            z-index: 1;
            width: 100%;
            max-width: 440px;
            display: flex;
            flex-direction: column;
            gap: 18px;
        }

        /* 玻璃卡片 */
        .glass-card {
            background: rgba(255, 255, 255, 0.03);
            backdrop-filter: blur(24px);
            -webkit-backdrop-filter: blur(24px);
            border: 1px solid rgba(255, 255, 255, 0.06);
            border-radius: 18px;
            padding: 20px;
        }

        /* 预览区双层装裱 */
        .canvas-outer {
            background: rgba(255, 255, 255, 0.04);
            border-radius: 16px;
            padding: 2px;
            position: relative;
            aspect-ratio: 4/5;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        .canvas-inner {
            background: #000;
            border-radius: 8px;
            border: 1px solid rgba(255, 255, 255, 0.08);
            overflow: hidden;
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        canvas {
            max-width: 100%;
            max-height: 100%;
            object-fit: contain;
        }

        /* 装饰分割线 */
        .divider-line {
            width: 100%;
            height: 1px;
            background: linear-gradient(to right, rgba(192, 180, 252, 0.3), transparent 70%);
            margin: 4px 0 8px;
        }

        /* 分类Tab */
        .tab-scroll {
            display: flex;
            gap: 0;
            overflow-x: auto;
            padding-bottom: 6px;
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        .tab-scroll::-webkit-scrollbar {
            display: none;
        }
        .tab-btn {
            font-family: 'Noto Serif SC', 'Inter', serif;
            font-size: 12px;
            font-weight: 500;
            padding: 8px 18px;
            border: none;
            background: transparent;
            color: #7c7a85;
            cursor: pointer;
            position: relative;
            white-space: nowrap;
            transition: color 0.3s;
            letter-spacing: 0.5px;
        }
        .tab-btn::after {
            content: '';
            position: absolute;
            bottom: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 2px;
            background: linear-gradient(to right, #C0B4FC, #A28FFA);
            border-radius: 1px;
            transition: width 0.35s cubic-bezier(0.25, 0.8, 0.25, 1.2);
        }
        .tab-btn.active {
            color: #f0edff;
        }
        .tab-btn.active::after {
            width: 60%;
        }

        /* 滤镜网格 - 幻灯片卡片 */
        .filter-grid {
            display: none;
            grid-template-columns: repeat(4, 1fr);
            gap: 8px;
        }
        .filter-grid.active {
            display: grid;
        }
        .filter-card {
            background: rgba(255, 255, 255, 0.03);
            border: 1px solid rgba(255, 255, 255, 0.05);
            border-radius: 10px;
            padding: 10px 4px 8px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 5px;
            cursor: pointer;
            transition: all 0.25s cubic-bezier(0.25, 0.8, 0.25, 1.2);
            position: relative;
            box-shadow: inset 0 1px 0 rgba(255, 255, 255, 0.02), 0 2px 6px rgba(0, 0, 0, 0.3);
            font-family: 'Noto Serif SC', 'Inter', serif;
            font-size: 10px;
            color: #a1a1aa;
            letter-spacing: 0.3px;
            white-space: nowrap;
            overflow: hidden;
            text-overflow: ellipsis;
        }
        .filter-card:hover {
            background: rgba(255, 255, 255, 0.06);
            border-color: rgba(255, 255, 255, 0.1);
            transform: translateY(-2px);
            box-shadow: 0 6px 16px rgba(0, 0, 0, 0.4);
        }
        .filter-card.active-card {
            border-color: rgba(192, 180, 252, 0.5);
            background: rgba(160, 140, 250, 0.08);
            color: #fff;
            transform: translateY(-3px);
            box-shadow: 0 8px 24px rgba(160, 140, 250, 0.15), 0 0 0 3px rgba(192, 180, 252, 0.08);
        }
        .filter-card.active-card::after {
            content: '';
            position: absolute;
            bottom: -5px;
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 0;
            border-left: 5px solid transparent;
            border-right: 5px solid transparent;
            border-top: 5px solid rgba(192, 180, 252, 0.5);
        }
        .filter-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            flex-shrink: 0;
        }

        /* 参数区域 */
        .param-section-title {
            font-family: 'Inter', sans-serif;
            font-size: 9px;
            font-weight: 700;
            letter-spacing: 2px;
            color: #7c7a85;
            text-transform: uppercase;
            margin-bottom: 6px;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        .param-section-title::after {
            content: '';
            flex: 1;
            height: 1px;
            background: rgba(255, 255, 255, 0.06);
        }
        .param-card {
            background: rgba(255, 255, 255, 0.02);
            border-radius: 8px;
            padding: 8px 12px;
            margin-bottom: 4px;
            display: flex;
            flex-direction: column;
            gap: 4px;
        }
        .param-label-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .param-name {
            font-family: 'Noto Serif SC', 'Inter', serif;
            font-size: 10px;
            color: #a1a1aa;
            letter-spacing: 0.3px;
        }
        .param-value {
            font-family: 'Inter', monospace;
            font-size: 10px;
            font-weight: 700;
            color: #f0edff;
        }
        input[type=range] {
            -webkit-appearance: none;
            width: 100%;
            height: 3px;
            background: #2A2A30;
            border-radius: 2px;
            outline: none;
        }
        input[type=range]::-webkit-slider-thumb {
            -webkit-appearance: none;
            height: 16px;
            width: 16px;
            border-radius: 50%;
            background: #fff;
            border: 2px solid #A28FFA;
            cursor: pointer;
            box-shadow: 0 0 12px rgba(162, 143, 250, 0.4);
            transition: box-shadow 0.2s;
        }
        input[type=range]::-webkit-slider-thumb:active {
            box-shadow: 0 0 20px rgba(162, 143, 250, 0.7);
        }

        /* 底部按钮 */
        .bottom-buttons {
            display: grid;
            grid-template-columns: 1fr 1.5fr 1fr;
            gap: 10px;
        }
        .btn-action {
            padding: 12px 8px;
            border-radius: 10px;
            font-family: 'Noto Serif SC', 'Inter', serif;
            font-size: 11px;
            font-weight: 700;
            cursor: pointer;
            border: none;
            transition: all 0.25s;
            letter-spacing: 0.5px;
            text-align: center;
        }
        .btn-reset {
            background: transparent;
            border: 1px solid rgba(160, 140, 250, 0.3);
            color: #a1a1aa;
        }
        .btn-reset:hover {
            border-color: rgba(160, 140, 250, 0.6);
            color: #fff;
        }
        .btn-save {
            background: linear-gradient(135deg, #C0B4FC, #A28FFA);
            color: #fff;
            box-shadow: 0 4px 16px rgba(162, 143, 250, 0.25);
        }
        .btn-save:hover {
            box-shadow: 0 6px 22px rgba(162, 143, 250, 0.4);
            transform: translateY(-1px);
        }
        .btn-save:active {
            transform: scale(0.97);
        }
        .btn-export {
            background: transparent;
            border: 1px solid rgba(160, 140, 250, 0.2);
            color: #a1a1aa;
        }
        .btn-export:hover {
            border-color: rgba(160, 140, 250, 0.5);
            color: #fff;
        }

        /* 上传按钮样式 */
        .upload-label {
            font-family: 'Inter', 'Noto Sans SC', sans-serif;
            font-size: 9px;
            font-weight: 600;
            padding: 8px 14px;
            border-radius: 24px;
            cursor: pointer;
            letter-spacing: 0.5px;
            transition: all 0.25s;
            display: inline-block;
            text-align: center;
        }
        .upload-image-label {
            background: linear-gradient(135deg, #C0B4FC, #A28FFA);
            color: #fff;
            box-shadow: 0 2px 10px rgba(162, 143, 250, 0.3);
        }
        .upload-cube-label {
            background: transparent;
            border: 1px solid rgba(160, 140, 250, 0.25);
            color: #a1a1aa;
        }
        .upload-cube-label:hover {
            border-color: rgba(160, 140, 250, 0.5);
            color: #fff;
        }
    </style>
</head>
<body>

    <div class="app-container">
        <!-- 顶部标题栏 -->
        <div style="display:flex;justify-content:space-between;align-items:flex-start;">
            <div>
                <h1 style="font-family:'Noto Serif SC',serif;font-size:28px;font-weight:700;letter-spacing:0.08em;color:#f0edff;line-height:1.2;">
                    光屿
                    <span style="display:inline-block;font-family:'Inter',sans-serif;font-size:9px;font-weight:500;background:rgba(160,140,250,0.15);color:#B8A8F8;padding:2px 8px;border-radius:10px;vertical-align:middle;margin-left:6px;letter-spacing:0.05em;">V16</span>
                </h1>
                <p style="font-family:'Inter','Noto Sans SC',sans-serif;font-size:9px;font-style:italic;color:#7c7a85;letter-spacing:0.5px;margin-top:2px;">Pure Nature · Unified Glow Logic</p>
            </div>
            <div style="display:flex;gap:8px;">
                <label class="upload-label upload-image-label">
                    IMAGE
                    <input type="file" id="upload" hidden accept="image/*">
                </label>
                <label class="upload-label upload-cube-label">
                    .CUBE
                    <input type="file" id="uploadLut" hidden accept=".cube">
                </label>
            </div>
        </div>

        <!-- 预览区 -->
        <div class="canvas-outer">
            <div class="canvas-inner">
                <canvas id="mainCanvas"></canvas>
            </div>
        </div>
        <div class="divider-line"></div>

        <!-- 分类Tab -->
        <div class="tab-scroll">
            <button onclick="switchTab('classic')" class="tab-btn active" id="tab-classic">经典</button>
            <button onclick="switchTab('nature')" class="tab-btn" id="tab-nature">自然</button>
            <button onclick="switchTab('glow')" class="tab-btn" id="tab-glow">朦胧</button>
            <button onclick="switchTab('healing')" class="tab-btn" id="tab-healing">治愈</button>
            <button onclick="switchTab('cool')" class="tab-btn" id="tab-cool">破碎</button>
        </div>

        <!-- 滤镜网格 -->
        <div id="grid-classic" class="filter-grid active">
            <div class="filter-card" onclick="setPreset('dreamcore')"><span class="filter-dot" style="background:#e8b4f8;"></span>千禧</div>
            <div class="filter-card" onclick="setPreset('korean')"><span class="filter-dot" style="background:#f8c8dc;"></span>韩系</div>
            <div class="filter-card" onclick="setPreset('leica')"><span class="filter-dot" style="background:#d4a574;"></span>徕卡</div>
            <div class="filter-card" onclick="setPreset('ditto')"><span class="filter-dot" style="background:#b8d4f8;"></span>Ditto</div>
            <div class="filter-card" onclick="setPreset('fuji')"><span class="filter-dot" style="background:#c8e0c8;"></span>富士胶片</div>
            <div class="filter-card" onclick="setPreset('berlin')"><span class="filter-dot" style="background:#a8b8c8;"></span>柏林雾</div>
            <div class="filter-card" onclick="setPreset('dream90')"><span class="filter-dot" style="background:#f8d8b8;"></span>90梦</div>
            <div class="filter-card" onclick="setPreset('childhood')"><span class="filter-dot" style="background:#f8e8f0;"></span>童梦</div>
            <div class="filter-card" onclick="setPreset('ethereal')"><span class="filter-dot" style="background:#d8d8f0;"></span>空灵</div>
            <div class="filter-card" onclick="setPreset('cyanDawn')"><span class="filter-dot" style="background:#b8d8f0;"></span>蓝调晨曦</div>
        </div>

        <div id="grid-nature" class="filter-grid">
            <div class="filter-card" onclick="setPreset('musu')"><span class="filter-dot" style="background:#f0d8c8;"></span>暮苏</div>
            <div class="filter-card" onclick="setPreset('wuqi')"><span class="filter-dot" style="background:#c8d8d8;"></span>雾起</div>
            <div class="filter-card" onclick="setPreset('judao')"><span class="filter-dot" style="background:#f0c898;"></span>橘岛</div>
            <div class="filter-card" onclick="setPreset('qingxi')"><span class="filter-dot" style="background:#f8f0d0;"></span>晴曦</div>
            <div class="filter-card" onclick="setPreset('xiuyin')"><span class="filter-dot" style="background:#c8c0d8;"></span>岫隐</div>
            <div class="filter-card" onclick="setPreset('yuanwei')"><span class="filter-dot" style="background:#c8c0e8;"></span>鸢尾</div>
            <div class="filter-card" onclick="setPreset('misen')"><span class="filter-dot" style="background:#c8d8b8;"></span>弥森</div>
        </div>

        <div id="grid-glow" class="filter-grid">
            <div class="filter-card" onclick="setPreset('phantom')"><span class="filter-dot" style="background:#e0d0f0;"></span>幻昼</div>
            <div class="filter-card" onclick="setPreset('is梦')"><span class="filter-dot" style="background:#d8c8e8;"></span>屿梦</div>
            <div class="filter-card" onclick="setPreset('starHabit')"><span class="filter-dot" style="background:#f0e8d8;"></span>星栖</div>
            <div class="filter-card" onclick="setPreset('cloudTip')"><span class="filter-dot" style="background:#e0e8f0;"></span>云杪</div>
            <div class="filter-card" onclick="setPreset('moonRoam')"><span class="filter-dot" style="background:#d8d8f0;"></span>月漫</div>
        </div>

        <div id="grid-healing" class="filter-grid">
            <div class="filter-card" onclick="setPreset('begonia')"><span class="filter-dot" style="background:#f0c8c8;"></span>棠眠</div>
            <div class="filter-card" onclick="setPreset('waterListen')"><span class="filter-dot" style="background:#c8e0e8;"></span>汀见</div>
            <div class="filter-card" onclick="setPreset('gardenNight')"><span class="filter-dot" style="background:#e8e0c8;"></span>栀晚</div>
            <div class="filter-card" onclick="setPreset('misty')"><span class="filter-dot" style="background:#d8e0e0;"></span>雾眠</div>
            <div class="filter-card" onclick="setPreset('starTrace')"><span class="filter-dot" style="background:#e0d8c8;"></span>星溯</div>
        </div>

        <div id="grid-cool" class="filter-grid">
            <div class="filter-card" onclick="setPreset('frostSeq')"><span class="filter-dot" style="background:#d0d8e8;"></span>霜序</div>
            <div class="filter-card" onclick="setPreset('deepTide')"><span class="filter-dot" style="background:#8898a8;"></span>沉汐</div>
            <div class="filter-card" onclick="setPreset('ashIsle')"><span class="filter-dot" style="background:#b8a090;"></span>烬屿</div>
            <div class="filter-card" onclick="setPreset('fogWild')"><span class="filter-dot" style="background:#b8c8b8;"></span>雾野</div>
            <div class="filter-card" onclick="setPreset('pineMist')"><span class="filter-dot" style="background:#98a888;"></span>松岚</div>
            <div class="filter-card" onclick="setPreset('riverFog')"><span class="filter-dot" style="background:#a0b8c8;"></span>川雾</div>
        </div>

        <!-- 参数区 -->
        <div class="glass-card" style="padding:16px 18px;">
            <div class="param-section-title">光影与柔化</div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">光晕强度</span><span id="v-bloom" class="param-value">0.00</span></div><input type="range" id="bloom" min="0" max="100" value="0"></div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">柔焦氛围</span><span id="v-soft" class="param-value">0.00</span></div><input type="range" id="soft" min="0" max="100" value="0"></div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">高光柔化</span><span id="v-highlight" class="param-value">0.00</span></div><input type="range" id="highlight" min="0" max="100" value="0"></div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">曝光</span><span id="v-exp" class="param-value">0.00</span></div><input type="range" id="exp" min="-100" max="100" value="0"></div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">暗角/边缘</span><span id="v-vignette" class="param-value">0.00</span></div><input type="range" id="vignette" min="0" max="100" value="0"></div>

            <div class="param-section-title" style="margin-top:12px;">色彩与质感</div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">色温</span><span id="v-temp" class="param-value">0.00</span></div><input type="range" id="temp" min="-50" max="50" value="0"></div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">色彩偏移</span><span id="v-tintShift" class="param-value">0.00</span></div><input type="range" id="tintShift" min="-50" max="50" value="0"></div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">胶片颗粒</span><span id="v-grain" class="param-value">0.00</span></div><input type="range" id="grain" min="0" max="100" value="0"></div>
            <div class="param-card"><div class="param-label-row"><span class="param-name">边缘模糊</span><span id="v-edge" class="param-value">0.00</span></div><input type="range" id="edge" min="0" max="100" value="0"></div>
        </div>

        <!-- 底部按钮 -->
        <div class="bottom-buttons">
            <button id="resetBtn" class="btn-action btn-reset">重置</button>
            <button id="downloadBtn" class="btn-action btn-save">保存图片</button>
            <button class="btn-action btn-export">导出预设</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('mainCanvas');
        const ctx = canvas.getContext('2d');
        const upload = document.getElementById('upload');
        const uploadLut = document.getElementById('uploadLut');
        let imgObj = null;
        let currentMode = 'none';
        let customLut = null;

        const configs = {
            // 经典10款
            dreamcore: { b:82, g:68, px:5, soft:10, vig:0, high:0, temp:0, tint:10, exp:5, edge:0, c:1.1, br:1.0, s:1.2, t:[1,1,1.1], l:0 },
            korean: { b:48, g:12, px:1, soft:20, vig:0, high:15, temp:-5, tint:5, exp:10, edge:0, c:0.95, br:1.15, s:0.85, t:[1.05,1.02,1.05], l:0 },
            leica: { b:20, g:20, px:1, soft:15, vig:25, high:0, temp:-10, tint:0, exp:-5, edge:5, c:1.3, br:0.9, s:1.05, t:[0.98,1,1.05], l:5 },
            ditto: { b:60, g:90, px:2, soft:30, vig:15, high:10, temp:5, tint:-5, exp:0, edge:10, c:1.05, br:1, s:0.9, t:[1.05,1.02,0.95], l:30 },
            fuji: { b:25, g:35, px:1, soft:20, vig:10, high:0, temp:12, tint:8, exp:-2, edge:0, c:1.2, br:0.95, s:0.8, t:[1,1.05,1], l:10 },
            berlin: { b:95, g:45, px:1, soft:15, vig:40, high:20, temp:-15, tint:0, exp:5, edge:15, c:0.85, br:1.05, s:0.7, t:[0.9,1,1.1], l:40 },
            dream90: { b:90, g:85, px:4, soft:25, vig:20, high:30, temp:10, tint:15, exp:8, edge:12, c:1.15, br:1.08, s:1.35, t:[1.12,1.08,0.92], st:[12,8,0], l:25 },
            childhood: { b:95, g:20, px:1, soft:40, vig:0, high:45, temp:15, tint:10, exp:12, edge:0, c:0.8, br:1.15, s:1.1, t:[1.1,1.05,0.95], st:[10,5,0], l:55 },
            ethereal: { b:88, g:15, px:1, soft:35, vig:30, high:50, temp:-20, tint:-10, exp:5, edge:20, c:1.1, br:1.05, s:0.9, t:[0.95,1.05,1.12], st:[-5,5,15], l:15 },
            cyanDawn: { b:92, g:10, px:1, soft:45, vig:25, high:40, temp:-25, tint:-15, exp:10, edge:15, c:0.88, br:1.1, s:0.95, t:[0.92,1.08,1.15], st:[-10,8,20], l:35 },

            // 自然7款
            musu: { b:40, g:5, px:1, soft:25, vig:0, high:20, temp:15, tint:10, exp:5, edge:0, c:0.85, br:1.1, s:1.0, t:[1.12, 1.08, 1.05], st:[15, 8, 5], l:0 },
            wuqi: { b:60, g:2, px:1, soft:35, vig:10, high:15, temp:-20, tint:-5, exp:0, edge:20, c:0.9, br:1.05, s:0.7, t:[0.9, 1.1, 1.15], st:[0, 10, 15], l:0 },
            judao: { b:25, g:15, px:1, soft:20, vig:20, high:10, temp:25, tint:10, exp:-5, edge:10, c:1.1, br:0.95, s:1.1, t:[1.2, 1.05, 0.95], l:20 },
            qingxi: { b:30, g:1, px:1, soft:20, vig:0, high:35, temp:10, tint:0, exp:15, edge:0, c:1.05, br:1.15, s:1.2, t:[1.05, 1.08, 1.0], l:0 },
            xiuyin: { b:50, g:8, px:1, soft:45, vig:40, high:20, temp:-15, tint:5, exp:-10, edge:40, c:0.95, br:0.9, s:0.85, t:[0.9, 1.0, 1.1], st:[-5, 10, 15], l:15 },
            yuanwei: { b:45, g:3, px:1, soft:30, vig:15, high:25, temp:-10, tint:20, exp:5, edge:15, c:0.9, br:1.08, s:0.8, t:[1.05, 1.0, 1.2], st:[12, 0, 10], l:0 },
            misen: { b:40, g:5, px:1, soft:35, vig:20, high:20, temp:5, tint:-10, exp:0, edge:25, c:1.0, br:1.0, s:1.1, t:[1.0, 1.15, 0.9], st:[15, 10, -5], l:0 },

            // 朦胧5款
            phantom: { b:70, g:0, px:1, soft:60, vig:0, high:65, temp:8, tint:-8, exp:15, edge:25, c:0.82, br:1.15, s:0.9, t:[1.08, 1.05, 1.05], l:35 },
            is梦: { b:40, g:5, px:1, soft:40, vig:60, high:25, temp:-18, tint:8, exp:-12, edge:45, c:0.85, br:0.95, s:0.75, t:[0.9, 0.95, 1.15], st:[-5, 0, 15], l:45 },
            starHabit: { b:85, g:3, px:1, soft:65, vig:10, high:75, temp:5, tint:-2, exp:8, edge:35, c:0.92, br:1.05, s:1.1, t:[1.05, 1, 1.1], st:[10,0,15], l:20 },
            cloudTip: { b:45, g:0, px:1, soft:35, vig:0, high:55, temp:-15, tint:-10, exp:18, edge:15, c:0.8, br:1.2, s:0.95, t:[0.92, 1.1, 1.15], l:40 },
            moonRoam: { b:55, g:8, px:1, soft:45, vig:40, high:40, temp:-25, tint:5, exp:-5, edge:40, c:0.88, br:1.0, s:0.85, t:[0.85, 0.95, 1.2], st:[-5, -5, 20], l:30 },

            // 治愈5款
            begonia: { b:65, g:3, px:1, soft:40, vig:0, high:45, temp:10, tint:15, exp:12, edge:20, c:0.85, br:1.1, s:1.2, t:[1.12, 1.05, 1.02], l:45 },
            waterListen: { b:45, g:0, px:1, soft:25, vig:0, high:50, temp:-8, tint:-15, exp:20, edge:15, c:0.8, br:1.2, s:1.0, t:[0.95, 1.15, 1.05], l:50 },
            gardenNight: { b:55, g:5, px:1, soft:45, vig:15, high:45, temp:15, tint:5, exp:10, edge:25, c:0.9, br:1.05, s:1.1, t:[1.1, 1.08, 0.95], l:35 },
            misty: { b:75, g:2, px:1, soft:30, vig:15, high:50, temp:-10, tint:5, exp:-5, edge:10, c:0.8, br:1.1, s:0.6, t:[1,1.05,1.1], st:[0,5,10], l:75 },
            starTrace: { b:35, g:45, px:3, soft:25, vig:35, high:25, temp:20, tint:10, exp:0, edge:12, c:1.2, br:1.0, s:1.1, t:[1.1,1,0.9], st:[15,5,-5], l:15 },

            // 破碎6款
            frostSeq: { b:25, g:10, px:1, soft:20, vig:25, high:15, temp:-10, tint:0, exp:-5, edge:30, c:0.75, br:1.0, s:0.1, t:[1,1,1], l:30 },
            deepTide: { b:35, g:12, px:1, soft:35, vig:55, high:20, temp:-20, tint:5, exp:-15, edge:50, c:0.85, br:0.9, s:0.7, t:[0.8, 0.9, 1.1], st:[-10, -5, 25], l:40 },
            ashIsle: { b:45, g:15, px:1, soft:25, vig:45, high:30, temp:20, tint:15, exp:-10, edge:55, c:0.82, br:1.0, s:0.85, t:[1.1, 0.95, 0.9], st:[15, 5, -10], l:35 },
            fogWild: { b:65, g:8, px:1, soft:55, vig:30, high:35, temp:-5, tint:10, exp:-5, edge:45, c:0.8, br:1.0, s:0.8, t:[0.95, 1.08, 1], st:[-5, 15, -5], l:45 },
            pineMist: { b:45, g:10, px:1, soft:35, vig:55, high:20, temp:-15, tint:5, exp:-12, edge:50, c:0.9, br:0.9, s:0.6, t:[0.85, 0.98, 0.9], st:[-15, 10, -10], l:30 },
            riverFog: { b:75, g:0, px:1, soft:45, vig:10, high:55, temp:-10, tint:-5, exp:25, edge:20, c:0.75, br:1.2, s:0.9, t:[0.9, 1.05, 1.15], l:55 }
        };

        function switchTab(tab) {
            document.querySelectorAll('.filter-grid').forEach(g => g.classList.remove('active'));
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('grid-' + tab).classList.add('active');
            document.getElementById('tab-' + tab).classList.add('active');
        }

        upload.onchange = (e) => {
            const reader = new FileReader();
            reader.onload = (ev) => {
                const img = new Image();
                img.onload = () => { imgObj = img;
                    canvas.width = img.width;
                    canvas.height = img.height;
                    render(); };
                img.src = ev.target.result;
            };
            reader.readAsDataURL(e.target.files[0]);
        };

        uploadLut.onchange = (e) => {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = (ev) => { parseCube(ev.target.result);
                currentMode = 'custom';
                render(); };
            reader.readAsText(file);
        };

        function parseCube(text) {
            const lines = text.split('\n');
            let size = 32,
                data = [];
            for (let line of lines) {
                line = line.trim();
                if (!line || line.startsWith('#')) continue;
                if (line.startsWith('LUT_3D_SIZE')) { size = parseInt(line.split(' ')[1]); continue; }
                const parts = line.split(/\s+/).map(parseFloat);
                if (parts.length === 3) data.push(parts);
            }
            customLut = { size, data };
        }

        function setPreset(m) {
            currentMode = m;
            customLut = null;
            const conf = configs[m];
            const mapping = { 'bloom': conf.b, 'soft': conf.soft, 'grain': conf.g, 'vignette': conf.vig, 'highlight': conf
                    .high, 'temp': conf.temp, 'tintShift': conf.tint, 'exp': conf.exp, 'edge': conf.edge };
            for (let id in mapping) { document.getElementById(id).value = mapping[id]; }

            // 更新选中态
            document.querySelectorAll('.filter-card').forEach(c => c.classList.remove('active-card'));
            const targetCard = Array.from(document.querySelectorAll('.filter-card')).find(c => {
                const onclickAttr = c.getAttribute('onclick');
                return onclickAttr && onclickAttr.includes("'" + m + "'");
            });
            if (targetCard) targetCard.classList.add('active-card');

            render();
        }

        function render() {
            if (!imgObj) return;
            const p = {
                b: parseInt(document.getElementById('bloom').value),
                soft: parseInt(document.getElementById('soft').value),
                g: parseInt(document.getElementById('grain').value),
                vig: parseInt(document.getElementById('vignette').value),
                high: parseInt(document.getElementById('highlight').value),
                temp: parseInt(document.getElementById('temp').value),
                tint: parseInt(document.getElementById('tintShift').value),
                exp: parseInt(document.getElementById('exp').value),
                edge: parseInt(document.getElementById('edge').value)
            };

            for (let key in p) {
                let id = key === 'vig' ? 'vignette' : key === 'high' ? 'highlight' : key === 'tint' ? 'tintShift' : key ===
                    'b' ? 'bloom' : key === 'g' ? 'grain' : key;
                const el = document.getElementById('v-' + id);
                if (el) el.innerText = p[key].toFixed