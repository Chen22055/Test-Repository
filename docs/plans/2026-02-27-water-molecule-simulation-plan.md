# Water Molecule Simulation Implementation Plan

> **For Claude:** Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Single HTML file with classical MD water molecule simulation

**Architecture:** Canvas-based 2D simulation with ball-and-stick molecules, soft collision physics, rotational dynamics

**Tech Stack:** Pure HTML5 Canvas + JavaScript

---

### Task 1: Create water molecule simulation HTML file

**Files:**
- Create: `water-simulation.html`

**Step 1: Write the simulation code**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Water Molecule Simulation</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #0a1628;
            color: white;
            display: flex;
            flex-direction: column;
            align-items: center;
            margin: 0;
            padding: 20px;
        }
        h1 { margin-bottom: 10px; }
        canvas {
            border: 2px solid #1e3a5f;
            cursor: crosshair;
        }
        #controls {
            display: flex;
            gap: 20px;
            margin-top: 15px;
            flex-wrap: wrap;
            justify-content: center;
        }
        .control-group {
            display: flex;
            align-items: center;
            gap: 10px;
        }
        button {
            padding: 8px 20px;
            font-size: 14px;
            cursor: pointer;
            background: #3498db;
            color: white;
            border: none;
            border-radius: 4px;
        }
        button:hover { background: #2980b9; }
    </style>
</head>
<body>
    <h1>💧 Water Molecule Simulation</h1>
    <canvas id="canvas" width="800" height="500"></canvas>
    <div id="controls">
        <div class="control-group">
            <label>Temperature:</label>
            <input type="range" id="temp" min="0.1" max="5" step="0.1" value="1">
            <span id="tempVal">1.0</span>
        </div>
        <div class="control-group">
            <label>Molecules:</label>
            <input type="range" id="count" min="5" max="50" step="1" value="20">
            <span id="countVal">20</span>
        </div>
        <button id="playBtn">Pause</button>
        <button id="resetBtn">Reset</button>
    </div>

    <script>
        const canvas = document.getElementById('canvas');
        const ctx = canvas.getContext('2d');
        
        const OXYGEN_RADIUS = 12;
        const HYDROGEN_RADIUS = 8;
        const BOND_LENGTH = 20;
        const BOND_ANGLE = 104.5 * Math.PI / 180;
        
        let molecules = [];
        let running = true;
        let temperature = 1.0;
        
        class Molecule {
            constructor(x, y) {
                this.x = x;
                this.y = y;
                this.vx = (Math.random() - 0.5) * temperature * 2;
                this.vy = (Math.random() - 0.5) * temperature * 2;
                this.angle = Math.random() * Math.PI * 2;
                this.omega = (Math.random() - 0.5) * temperature * 0.5;
            }
            
            update() {
                this.x += this.vx;
                this.y += this.vy;
                this.angle += this.omega;
                
                if (this.x < 30 || this.x > canvas.width - 30) {
                    this.vx *= -0.95;
                    this.x = Math.max(30, Math.min(canvas.width - 30, this.x));
                }
                if (this.y < 30 || this.y > canvas.height - 30) {
                    this.vy *= -0.95;
                    this.y = Math.max(30, Math.min(canvas.height - 30, this.y));
                }
            }
            
            draw() {
                ctx.save();
                ctx.translate(this.x, this.y);
                ctx.rotate(this.angle);
                
                ctx.strokeStyle = 'white';
                ctx.lineWidth = 3;
                ctx.beginPath();
                ctx.moveTo(0, 0);
                ctx.lineTo(BOND_LENGTH * Math.cos(BOND_ANGLE/2), BOND_LENGTH * Math.sin(BOND_ANGLE/2));
                ctx.moveTo(0, 0);
                ctx.lineTo(BOND_LENGTH * Math.cos(-BOND_ANGLE/2), BOND_LENGTH * Math.sin(-BOND_ANGLE/2));
                ctx.stroke();
                
                ctx.fillStyle = '#e74c3c';
                ctx.beginPath();
                ctx.arc(0, 0, OXYGEN_RADIUS, 0, Math.PI * 2);
                ctx.fill();
                
                ctx.fillStyle = '#ecf0f1';
                ctx.beginPath();
                ctx.arc(BOND_LENGTH * Math.cos(BOND_ANGLE/2), BOND_LENGTH * Math.sin(BOND_ANGLE/2), HYDROGEN_RADIUS, 0, Math.PI * 2);
                ctx.fill();
                ctx.beginPath();
                ctx.arc(BOND_LENGTH * Math.cos(-BOND_ANGLE/2), BOND_LENGTH * Math.sin(-BOND_ANGLE/2), HYDROGEN_RADIUS, 0, Math.PI * 2);
                ctx.fill();
                
                ctx.restore();
            }
        }
        
        function init(count) {
            molecules = [];
            for (let i = 0; i < count; i++) {
                molecules.push(new Molecule(
                    50 + Math.random() * (canvas.width - 100),
                    50 + Math.random() * (canvas.height - 100)
                ));
            }
        }
        
        function checkCollisions() {
            for (let i = 0; i < molecules.length; i++) {
                for (let j = i + 1; j < molecules.length; j++) {
                    const dx = molecules[j].x - molecules[i].x;
                    const dy = molecules[j].y - molecules[i].y;
                    const dist = Math.sqrt(dx * dx + dy * dy);
                    const minDist = OXYGEN_RADIUS * 2 + 5;
                    
                    if (dist < minDist && dist > 0) {
                        const nx = dx / dist;
                        const ny = dy / dist;
                        const overlap = minDist - dist;
                        
                        molecules[i].x -= nx * overlap * 0.5;
                        molecules[i].y -= ny * overlap * 0.5;
                        molecules[j].x += nx * overlap * 0.5;
                        molecules[j].y += ny * overlap * 0.5;
                        
                        const dvx = molecules[j].vx - molecules[i].vx;
                        const dvy = molecules[j].vy - molecules[i].vy;
                        const dvn = dvx * nx + dvy * ny;
                        
                        if (dvn < 0) {
                            molecules[i].vx += dvn * nx * 0.8;
                            molecules[i].vy += dvn * ny * 0.8;
                            molecules[j].vx -= dvn * nx * 0.8;
                            molecules[j].vy -= dvn * ny * 0.8;
                            
                            molecules[i].omega += (Math.random() - 0.5) * 0.2;
                            molecules[j].omega += (Math.random() - 0.5) * 0.2;
                        }
                    }
                }
            }
        }
        
        function draw() {
            ctx.fillStyle = '#0a1628';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            for (const m of molecules) {
                m.draw();
            }
        }
        
        function update() {
            if (running) {
                for (const m of molecules) {
                    m.update();
                }
                checkCollisions();
            }
            draw();
            requestAnimationFrame(update);
        }
        
        canvas.addEventListener('click', (e) => {
            const rect = canvas.getBoundingClientRect();
            const x = e.clientX - rect.left;
            const y = e.clientY - rect.top;
            molecules.push(new Molecule(x, y));
            document.getElementById('count').value = molecules.length;
            document.getElementById('countVal').textContent = molecules.length;
        });
        
        document.getElementById('playBtn').addEventListener('click', () => {
            running = !running;
            document.getElementById('playBtn').textContent = running ? 'Pause' : 'Play';
        });
        
        document.getElementById('resetBtn').addEventListener('click', () => {
            init(parseInt(document.getElementById('count').value));
        });
        
        document.getElementById('temp').addEventListener('input', (e) => {
            temperature = parseFloat(e.target.value);
            document.getElementById('tempVal').textContent = temperature.toFixed(1);
        });
        
        document.getElementById('count').addEventListener('input', (e) => {
            const count = parseInt(e.target.value);
            document.getElementById('countVal').textContent = count;
            while (molecules.length < count) {
                molecules.push(new Molecule(
                    50 + Math.random() * (canvas.width - 100),
                    50 + Math.random() * (canvas.height - 100)
                ));
            }
            while (molecules.length > count) {
                molecules.pop();
            }
        });
        
        init(20);
        update();
    </script>
</body>
</html>
```

**Step 2: Verify file exists**

Run: `ls water-simulation.html`

**Step 3: Test in browser** (conceptually - verify no syntax errors)

---

### Task 2: Deploy to GitHub Pages

**Files:**
- Modify: `index.html` - Replace with water simulation (or rename water-simulation.html)

**Step 1: Check git status**

Run: `git status`

**Step 2: Stage and commit**

```bash
git add water-simulation.html
git commit -m "feat: add water molecule simulation"
```

**Step 3: Push to remote**

```bash
git push origin main
```

**Step 4: Enable GitHub Pages**

In GitHub: Settings → Pages → Source: main branch → Save

---

**Plan complete and saved to `docs/plans/2026-02-27-water-molecule-simulation-plan.md`. Two execution options:**

**1. Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration

**2. Parallel Session (separate)** - Open new session with executing-plans, batch execution with checkpoints

**Which approach?**
