# 高斯波包在谐振子势阱中演化的模拟器

## Project Overview
- **Project type**: Single-page web application (physics simulation)
- **Core functionality**: 在浏览器中使用PyScript模拟并可视化高斯波包在谐振子势阱中的量子力学时间演化
- **Target users**: 物理学习者和量子力学爱好者

## Functionality Specification

### Core Features
1. **Python计算引擎**
   - 使用PyScript/Pyodide加载Python环境
   - 使用NumPy进行数值计算
   - 使用scipy.integrate.odeint求解含时薛定谔方程

2. **高斯波包初始状态**
   - 可配置：初始位置x₀、动量p₀、宽度σ
   - 谐振子势阱：V(x) = ½mω²x²

3. **实时动画**
   - Canvas绘制波函数
   - 显示：|ψ(x,t)|² (概率密度)、Re(ψ)、Im(ψ)、V(x)
   - 可调播放速度

### User Interactions
- 播放/暂停按钮
- 参数滑块：ω (势阱频率)、初始位置、初始动量
- 重置按钮

## Technical Implementation
- Single HTML file包含所有代码
- PyScript加载Python环境
- requestAnimationFrame驱动动画

## Acceptance Criteria
- [x] 页面加载后自动初始化PyScript
- [x] 高斯波包在谐振子势阱中正确演化
- [x] 动画流畅显示概率密度
- [x] 参数可调节并实时反映变化

## Deployment
- 部署到GitHub仓库
- 启用GitHub Pages
