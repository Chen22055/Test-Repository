# 高斯波包量子演化模拟器实现计划

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** 创建单HTML文件，在浏览器中使用PyScript模拟高斯波包在谐振子势阱中的时间演化

**Architecture:** 使用PyScript加载Python + NumPy，在Canvas上实时绘制波函数演化动画

**Tech Stack:** PyScript, NumPy, HTML5 Canvas

---

### Task 1: 创建HTML框架文件

**Files:**
- Create: `index.html`

**Step 1: 创建基础HTML结构**

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>高斯波包在谐振子势阱中的演化</title>
    <link rel="stylesheet" href="https://pyscript.net/releases/2024.1.1/core.css">
    <script type="module" src="https://pyscript.net/releases/2024.1.1/core.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            max-width: 900px;
            margin: 0 auto;
            padding: 20px;
            background: #f5f5f5;
        }
        h1 { text-align: center; color: #333; }
        #canvas-container {
            text-align: center;
            margin: 20px 0;
        }
        canvas {
            background: white;
            border: 1px solid #ccc;
        }
        #controls {
            background: white;
            padding: 20px;
            border-radius: 8px;
            margin: 20px 0;
        }
        .control-group {
            margin: 10px 0;
        }
        label { display: inline-block; width: 150px; }
        button {
            padding: 10px 20px;
            margin: 5px;
            font-size: 16px;
            cursor: pointer;
        }
        #loading {
            text-align: center;
            font-size: 18px;
            color: #666;
        }
    </style>
</head>
<body>
    <h1>高斯波包在谐振子势阱中的演化</h1>
    <div id="loading">正在加载Python环境...</div>
    <div id="canvas-container">
        <canvas id="waveCanvas" width="800" height="400"></canvas>
    </div>
    <div id="controls" style="display:none;">
        <div class="control-group">
            <label>势阱频率 ω:</label>
            <input type="range" id="omega" min="0.5" max="3" step="0.1" value="1">
            <span id="omega-val">1.0</span>
        </div>
        <div class="control-group">
            <label>初始位置 x₀:</label>
            <input type="range" id="x0" min="-3" max="3" step="0.1" value="2">
            <span id="x0-val">2.0</span>
        </div>
        <div class="control-group">
            <label>初始动量 p₀:</label>
            <input type="range" id="p0" min="-3" max="3" step="0.1" value="0">
            <span id="p0-val">0.0</span>
        </div>
        <div class="control-group">
            <button id="playBtn">播放</button>
            <button id="resetBtn">重置</button>
        </div>
    </div>
    <script type="py">
import numpy as np
from pyscript import window, document
import asyncio

# 物理常数 (使用原子单位)
m = 1.0  # 质量
hbar = 1.0

# 网格参数
N = 500
x_min, x_max = -10, 10
x = np.linspace(x_min, x_max, N)
dx = x[1] - x[0]

# 全局状态
state = {
    'omega': 1.0,
    'x0': 2.0,
    'p0': 0.0,
    't': 0.0,
    'running': False,
    'psi': None
}

def gaussian_wavepacket(x, x0, p0, sigma=0.5):
    return np.exp(-((x - x0) ** 2) / (2 * sigma ** 2)) * np.exp(1j * p0 * x)

def harmonic_potential(x, omega):
    return 0.5 * m * omega ** 2 * x ** 2

def hamiltonian(omega):
    V = harmonic_potential(x, omega)
    # 动能算符 (有限差分)
    T = -hbar ** 2 / (2 * m) * np.eye(N, k=1) * (-1/dx**2) + \
        2 * hbar ** 2 / (2 * m) * np.eye(N) * (1/dx**2) + \
        -hbar ** 2 / (2 * m) * np.eye(N, k=-1) * (-1/dx**2)
    H = T + np.diag(V)
    return H

def propagate_psi(psi, omega, dt):
    H = hamiltonian(omega)
    # 使用简单的时间演化算符
    U = np.exp(-1j * H * dt / hbar)
    return U @ psi

def normalize(psi):
    norm = np.sqrt(np.sum(np.abs(psi)**2) * dx)
    return psi / norm

def init_wavefunction():
    state['psi'] = normalize(gaussian_wavepacket(x, state['x0'], state['p0']))
    state['t'] = 0.0

def draw():
    canvas = document.getElementById('waveCanvas')
    ctx = canvas.getContext('2d')
    width = canvas.width
    height = canvas.height
    
    # 清空画布
    ctx.fillStyle = 'white'
    ctx.fillRect(0, 0, width, height)
    
    if state['psi'] is None:
        return
    
    # 绘制势阱
    V = harmonic_potential(x, state['omega'])
    V_normalized = V / np.max(V) * height * 0.8
    ctx.beginPath()
    ctx.strokeStyle = '#888'
    ctx.lineWidth = 2
    for i, (xi, vi) in enumerate(zip(x, V_normalized)):
        px = (i / N) * width
        py = height - vi
        if i == 0:
            ctx.moveTo(px, py)
        else:
            ctx.lineTo(px, py)
    ctx.stroke()
    
    # 绘制概率密度 |ψ|²
    prob_density = np.abs(state['psi']) ** 2
    prob_norm = prob_density / np.max(prob_density) * height * 0.8
    ctx.beginPath()
    ctx.strokeStyle = 'blue'
    ctx.lineWidth = 2
    for i, p in enumerate(prob_norm):
        px = (i / N) * width
        py = height - p
        if i == 0:
            ctx.moveTo(px, py)
        else:
            ctx.lineTo(px, py)
    ctx.stroke()
    
    # 绘制波函数实部
    psi_real = np.real(state['psi'])
    real_norm = psi_real / np.max(np.abs(psi_real)) * height * 0.4
    ctx.beginPath()
    ctx.strokeStyle = 'red'
    ctx.lineWidth = 1.5
    for i, r in enumerate(real_norm):
        px = (i / N) * width
        py = height/2 - r
        if i == 0:
            ctx.moveTo(px, py)
        else:
            ctx.lineTo(px, py)
    ctx.stroke()
    
    # 显示时间
    ctx.fillStyle = 'black'
    ctx.font = '16px Arial'
    ctx.fillText(f't = {state["t"]:.2f}', 10, 25)
    
    # 图例
    ctx.fillStyle = 'blue'
    ctx.fillText('|ψ|²', 10, height - 10)
    ctx.fillStyle = 'red'
    ctx.fillText('Re(ψ)', 60, height - 10)
    ctx.fillStyle = '#888'
    ctx.fillText('V(x)', 120, height - 10)

async def update():
    dt = 0.05
    while True:
        if state['running'] and state['psi'] is not None:
            state['psi'] = propagate_psi(state['psi'], state['omega'], dt)
            state['psi'] = normalize(state['psi'])
            state['t'] += dt
            draw()
        await asyncio.sleep(30/1000)

def setup():
    document.getElementById('loading').style.display = 'none'
    document.getElementById('controls').style.display = 'block'
    init_wavefunction()
    draw()
    asyncio.create_task(update())
    
    # 绑定事件
    def on_play(e):
        state['running'] = not state['running']
        document.getElementById('playBtn').textContent = '暂停' if state['running'] else '播放'
    
    def on_reset(e):
        state['running'] = False
        document.getElementById('playBtn').textContent = '播放'
        init_wavefunction()
        draw()
    
    def on_omega_change(e):
        state['omega'] = float(e.target.value)
        document.getElementById('omega-val').textContent = f'{state["omega"]:.1f}'
    
    def on_x0_change(e):
        state['x0'] = float(e.target.value)
        document.getElementById('x0-val').textContent = f'{state["x0"]:.1f}'
        init_wavefunction()
        if not state['running']:
            draw()
    
    def on_p0_change(e):
        state['p0'] = float(e.target.value)
        document.getElementById('p0-val').textContent = f'{state["p0"]:.1f}'
        init_wavefunction()
        if not state['running']:
            draw()
    
    document.getElementById('playBtn').addEventListener('click', on_play)
    document.getElementById('resetBtn').addEventListener('click', on_reset)
    document.getElementById('omega').addEventListener('input', on_omega_change)
    document.getElementById('x0').addEventListener('input', on_x0_change)
    document.getElementById('p0').addEventListener('input', on_p0_change)

# 等待PyScript加载完成后初始化
window.onload = setup
    </script>
</body>
</html>
```

**Step 2: 验证文件创建成功**

运行: `ls -la index.html`
Expected: 文件存在

---

### Task 2: 测试HTML文件

**Step 1: 检查文件语法**

运行: `python3 -c "from html.parser import HTMLParser; HTMLParser().feed(open('index.html').read())"`
Expected: 无错误输出

---

### Task 3: 部署到GitHub

**Files:**
- Create: `.nojekyll`
- Modify: README.md

**Step 1: 创建README**

```markdown
# 高斯波包量子演化模拟器

在浏览器中模拟高斯波包在谐振子势阱中的时间演化。

## 在线运行

访问: https://你的用户名.github.io/仓库名/

## 功能

- 实时动画显示波函数演化
- 可调节势阱频率 ω
- 可调节初始位置 x₀
- 可调节初始动量 p₀

## 技术

- PyScript (Python in browser)
- NumPy (数值计算)
- HTML5 Canvas (可视化)
```

**Step 2: 验证完成**

运行: `ls -la`
Expected: 包含 index.html, README.md

---

**Plan complete! Two execution options:**

1. **Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration

2. **Parallel Session (separate)** - Open new session with executing-plans, batch execution with checkpoints

Which approach?
