# 高斯波包量子演化模拟器

在浏览器中模拟高斯波包在谐振子势阱中的时间演化。

## 在线运行

1. 将此仓库部署到 GitHub Pages
2. 访问: `https://你的用户名.github.io/仓库名/`

## 功能

- 实时动画显示波函数演化
- 蓝色曲线：概率密度 |ψ|²
- 红色曲线：波函数实部 Re(ψ)
- 灰色曲线：谐振子势阱 V(x) = ½mω²x²

### 可调参数

- 势阱频率 ω
- 初始位置 x₀
- 初始动量 p₀

## 技术

- PyScript (Python in browser)
- NumPy (数值计算)
- HTML5 Canvas (可视化)

## 本地运行

直接在浏览器中打开 `index.html` 文件即可（需要联网加载 PyScript）。

## 部署到 GitHub Pages

```bash
# 1. 创建 GitHub 仓库并推送代码
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/你的用户名/仓库名.git
git push -u origin main

# 2. 启用 GitHub Pages
# 在仓库设置中: Settings → Pages → Source: main branch → Save
```
