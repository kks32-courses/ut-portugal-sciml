# Interactive Projectile Motion Demo

This interactive demo shows how differentiable simulation can be used to solve inverse problems in projectile motion. Click on targets to see how the optimal launch parameters are calculated!

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Differentiable Projectile Motion</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: white;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            overflow: hidden;
        }
        
        .header {
            background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
            color: white;
            padding: 20px;
            text-align: center;
        }
        
        .header h1 {
            margin: 0;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }
        
        .content {
            padding: 20px;
        }
        
        .demo-section {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            margin-bottom: 30px;
        }
        
        .canvas-container {
            position: relative;
            border: 3px solid #ddd;
            border-radius: 10px;
            background: linear-gradient(to bottom, #87CEEB 0%, #98FB98 100%);
            box-shadow: inset 0 0 20px rgba(0,0,0,0.1);
        }
        
        #gameCanvas {
            width: 100%;
            height: 400px;
            cursor: crosshair;
            border-radius: 7px;
        }
        
        .controls {
            background: #f8f9fa;
            padding: 20px;
            border-radius: 10px;
            border: 2px solid #e9ecef;
        }
        
        .control-group {
            margin-bottom: 15px;
        }
        
        .control-group label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
            color: #333;
        }
        
        .control-group input, .control-group select {
            width: 100%;
            padding: 8px;
            border: 2px solid #ddd;
            border-radius: 5px;
            font-size: 14px;
        }
        
        .control-group input:focus, .control-group select:focus {
            outline: none;
            border-color: #4CAF50;
        }
        
        .button {
            background: linear-gradient(135deg, #4CAF50 0%, #45a049 100%);
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            font-weight: bold;
            transition: all 0.3s ease;
            margin: 5px;
        }
        
        .button:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }
        
        .button:active {
            transform: translateY(0);
        }
        
        .info-panel {
            background: #e8f5e8;
            padding: 15px;
            border-radius: 10px;
            margin-top: 15px;
            border: 2px solid #4CAF50;
        }
        
        .info-panel h3 {
            margin-top: 0;
            color: #2e7d32;
        }
        
        .results {
            font-family: 'Courier New', monospace;
            background: #f5f5f5;
            padding: 10px;
            border-radius: 5px;
            margin-top: 10px;
            border: 1px solid #ddd;
        }
        
        .theory-section {
            background: #fff3e0;
            padding: 20px;
            border-radius: 10px;
            margin-top: 20px;
            border: 2px solid #ff9800;
        }
        
        .theory-section h2 {
            color: #e65100;
            margin-top: 0;
        }
        
        .math {
            font-family: 'Times New Roman', serif;
            font-style: italic;
            background: #f9f9f9;
            padding: 10px;
            border-radius: 5px;
            margin: 10px 0;
            border-left: 4px solid #ff9800;
        }
        
        .success {
            color: #4CAF50;
            font-weight: bold;
        }
        
        .warning {
            color: #ff9800;
            font-weight: bold;
        }
        
        .error {
            color: #f44336;
            font-weight: bold;
        }
        
        .legend {
            display: flex;
            justify-content: space-around;
            margin-top: 10px;
            font-size: 14px;
        }
        
        .legend-item {
            display: flex;
            align-items: center;
            gap: 5px;
        }
        
        .legend-color {
            width: 15px;
            height: 15px;
            border-radius: 3px;
        }
        
        @media (max-width: 768px) {
            .demo-section {
                grid-template-columns: 1fr;
            }
            
            .header h1 {
                font-size: 2em;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🎯 Differentiable Projectile Motion</h1>
            <p>Click on targets to see how gradient-based optimization finds the optimal launch parameters!</p>
        </div>
        
        <div class="content">
            <div class="demo-section">
                <div class="canvas-container">
                    <canvas id="gameCanvas" width="600" height="400"></canvas>
                    <div class="legend">
                        <div class="legend-item">
                            <div class="legend-color" style="background: #4CAF50;"></div>
                            <span>Trajectory</span>
                        </div>
                        <div class="legend-item">
                            <div class="legend-color" style="background: #f44336;"></div>
                            <span>Target</span>
                        </div>
                        <div class="legend-item">
                            <div class="legend-color" style="background: #2196F3;"></div>
                            <span>Launch Point</span>
                        </div>
                        <div class="legend-item">
                            <div class="legend-color" style="background: #ff9800;"></div>
                            <span>Optimization Path</span>
                        </div>
                    </div>
                </div>
                
                <div class="controls">
                    <div class="control-group">
                        <label>Gravity (m/s²):</label>
                        <input type="range" id="gravitySlider" min="1" max="20" value="9.81" step="0.1">
                        <span id="gravityValue">9.81</span>
                    </div>
                    
                    <div class="control-group">
                        <label>Optimization Method:</label>
                        <select id="optimizationMethod">
                            <option value="gradient">Gradient Descent</option>
                            <option value="analytical">Analytical Solution</option>
                            <option value="newton">Newton's Method</option>
                        </select>
                    </div>
                    
                    <div class="control-group">
                        <label>Learning Rate:</label>
                        <input type="range" id="learningRateSlider" min="0.01" max="1" value="0.1" step="0.01">
                        <span id="learningRateValue">0.1</span>
                    </div>
                    
                    <div class="control-group">
                        <label>Max Iterations:</label>
                        <input type="range" id="maxIterationsSlider" min="10" max="1000" value="100" step="10">
                        <span id="maxIterationsValue">100</span>
                    </div>
                    
                    <button class="button" onclick="clearCanvas()">Clear Canvas</button>
                    <button class="button" onclick="addRandomTarget()">Add Random Target</button>
                    <button class="button" onclick="animateTrajectory()">Animate Last Shot</button>
                    
                    <div class="info-panel">
                        <h3>Current Results</h3>
                        <div id="results" class="results">
                            Click on a target to see optimization results!
                        </div>
                    </div>
                </div>
            </div>
            
            <div class="theory-section">
                <h2>🧮 Mathematical Background</h2>
                
                <h3>Physics Model</h3>
                <div class="math">
                    <strong>Position equations:</strong><br>
                    x(t) = v₀ cos(θ) t<br>
                    y(t) = v₀ sin(θ) t - ½ g t²
                </div>
                
                <h3>Differentiable Simulation Approach</h3>
                <p>Instead of using the analytical solution, we can solve this as an optimization problem:</p>
                
                <div class="math">
                    <strong>Loss function:</strong><br>
                    L(v₀, θ) = (x_target - x_final)² + (y_target - y_final)²
                </div>
                
                <div class="math">
                    <strong>Gradient descent update:</strong><br>
                    v₀ ← v₀ - α ∂L/∂v₀<br>
                    θ ← θ - α ∂L/∂θ
                </div>
                
                <h3>Analytical Solution (for comparison)</h3>
                <div class="math">
                    <strong>For a target at (x_t, y_t):</strong><br>
                    θ = arctan((v₀² ± √(v₀⁴ - g(gx_t² + 2y_t v₀²))) / (gx_t))<br>
                    v₀ = √(gx_t² / (2 cos²(θ)(x_t tan(θ) - y_t)))
                </div>
                
                <p><strong>Key Insight:</strong> Differentiable simulation allows us to solve complex inverse problems where analytical solutions might not exist or be computationally expensive.</p>
            </div>
        </div>
    </div>
    
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        // Game state
        let targets = [];
        let trajectories = [];
        let launchPoint = {x: 50, y: canvas.height - 50};
        let currentTrajectory = null;
        let animationId = null;
        let optimizationPaths = [];
        
        // Physics constants
        let gravity = 9.81;
        let learningRate = 0.1;
        let maxIterations = 100;
        
        // Initialize
        function init() {
            updateSliderValues();
            drawScene();
            addRandomTarget();
        }
        
        function updateSliderValues() {
            const gravitySlider = document.getElementById('gravitySlider');
            const learningRateSlider = document.getElementById('learningRateSlider');
            const maxIterationsSlider = document.getElementById('maxIterationsSlider');
            
            gravity = parseFloat(gravitySlider.value);
            learningRate = parseFloat(learningRateSlider.value);
            maxIterations = parseInt(maxIterationsSlider.value);
            
            document.getElementById('gravityValue').textContent = gravity.toFixed(2);
            document.getElementById('learningRateValue').textContent = learningRate.toFixed(2);
            document.getElementById('maxIterationsValue').textContent = maxIterations;
        }
        
        // Event listeners
        document.getElementById('gravitySlider').addEventListener('input', updateSliderValues);
        document.getElementById('learningRateSlider').addEventListener('input', updateSliderValues);
        document.getElementById('maxIterationsSlider').addEventListener('input', updateSliderValues);
        
        canvas.addEventListener('click', (e) => {
            const rect = canvas.getBoundingClientRect();
            const x = (e.clientX - rect.left) * (canvas.width / rect.width);
            const y = (e.clientY - rect.top) * (canvas.height / rect.height);
            
            // Check if clicking on existing target
            const clickedTarget = targets.find(target => 
                Math.sqrt((target.x - x)**2 + (target.y - y)**2) < 20
            );
            
            if (clickedTarget) {
                solveForTarget(clickedTarget);
            } else {
                // Add new target
                targets.push({x, y, hit: false});
                drawScene();
            }
        });
        
        // Physics simulation
        function simulateProjectile(v0, angle, dt = 0.01) {
            const vx = v0 * Math.cos(angle);
            const vy = v0 * Math.sin(angle);
            
            let x = launchPoint.x;
            let y = launchPoint.y;
            let vx_current = vx;
            let vy_current = vy;
            
            const path = [{x, y}];
            
            while (y >= canvas.height - 50 && path.length < 10000) {
                x += vx_current * dt;
                y += vy_current * dt;
                vy_current -= gravity * dt;
                
                if (path.length % 10 === 0) {  // Sample every 10th point
                    path.push({x, y});
                }
            }
            
            return {
                path: path,
                finalX: x,
                finalY: y,
                timeOfFlight: path.length * dt
            };
        }
        
        function calculateLoss(v0, angle, target) {
            const result = simulateProjectile(v0, angle);
            const dx = result.finalX - target.x;
            const dy = result.finalY - target.y;
            return dx * dx + dy * dy;
        }
        
        function calculateGradient(v0, angle, target, epsilon = 1e-6) {
            const loss = calculateLoss(v0, angle, target);
            
            const lossV0Plus = calculateLoss(v0 + epsilon, angle, target);
            const lossAnglePlus = calculateLoss(v0, angle + epsilon, target);
            
            const gradV0 = (lossV0Plus - loss) / epsilon;
            const gradAngle = (lossAnglePlus - loss) / epsilon;
            
            return {gradV0, gradAngle};
        }
        
        function solveForTarget(target) {
            const method = document.getElementById('optimizationMethod').value;
            
            if (method === 'analytical') {
                solveAnalytical(target);
            } else if (method === 'gradient') {
                solveGradientDescent(target);
            } else if (method === 'newton') {
                solveNewton(target);
            }
        }
        
        function solveAnalytical(target) {
            const dx = target.x - launchPoint.x;
            const dy = target.y - launchPoint.y;
            
            // Try different initial velocities
            let bestV0 = null;
            let bestAngle = null;
            let bestLoss = Infinity;
            
            for (let v0 = 10; v0 <= 100; v0 += 1) {
                const discriminant = v0**4 - gravity * (gravity * dx**2 + 2 * dy * v0**2);
                
                if (discriminant >= 0) {
                    const sqrtDisc = Math.sqrt(discriminant);
                    const angle1 = Math.atan((v0**2 + sqrtDisc) / (gravity * dx));
                    const angle2 = Math.atan((v0**2 - sqrtDisc) / (gravity * dx));
                    
                    for (let angle of [angle1, angle2]) {
                        if (!isNaN(angle) && angle > 0 && angle < Math.PI/2) {
                            const loss = calculateLoss(v0, angle, target);
                            if (loss < bestLoss) {
                                bestLoss = loss;
                                bestV0 = v0;
                                bestAngle = angle;
                            }
                        }
                    }
                }
            }
            
            if (bestV0 !== null) {
                const result = simulateProjectile(bestV0, bestAngle);
                trajectories.push({
                    path: result.path,
                    color: '#4CAF50',
                    target: target,
                    v0: bestV0,
                    angle: bestAngle,
                    method: 'analytical'
                });
                
                target.hit = true;
                updateResults({
                    method: 'Analytical',
                    v0: bestV0,
                    angle: bestAngle * 180 / Math.PI,
                    iterations: 'N/A',
                    finalLoss: bestLoss,
                    success: bestLoss < 100
                });
                
                drawScene();
            }
        }
        
        function solveGradientDescent(target) {
            let v0 = 30;  // Initial guess
            let angle = Math.PI / 4;  // 45 degrees
            
            const path = [{v0, angle, loss: calculateLoss(v0, angle, target)}];
            
            let bestLoss = Infinity;
            let bestV0 = v0;
            let bestAngle = angle;
            
            for (let i = 0; i < maxIterations; i++) {
                const {gradV0, gradAngle} = calculateGradient(v0, angle, target);
                
                v0 -= learningRate * gradV0;
                angle -= learningRate * gradAngle;
                
                // Clamp values to reasonable ranges\n                v0 = Math.max(1, Math.min(150, v0));\n                angle = Math.max(0.01, Math.min(Math.PI/2 - 0.01, angle));\n                \n                const loss = calculateLoss(v0, angle, target);\n                path.push({v0, angle, loss});\n                \n                if (loss < bestLoss) {\n                    bestLoss = loss;\n                    bestV0 = v0;\n                    bestAngle = angle;\n                }\n                \n                if (loss < 10) break;  // Good enough\n            }\n            \n            optimizationPaths.push(path);\n            \n            const result = simulateProjectile(bestV0, bestAngle);\n            trajectories.push({\n                path: result.path,\n                color: '#4CAF50',\n                target: target,\n                v0: bestV0,\n                angle: bestAngle,\n                method: 'gradient'\n            });\n            \n            target.hit = bestLoss < 100;\n            updateResults({\n                method: 'Gradient Descent',\n                v0: bestV0,\n                angle: bestAngle * 180 / Math.PI,\n                iterations: path.length,\n                finalLoss: bestLoss,\n                success: bestLoss < 100\n            });\n            \n            drawScene();\n        }\n        \n        function solveNewton(target) {\n            let v0 = 30;\n            let angle = Math.PI / 4;\n            \n            const path = [{v0, angle, loss: calculateLoss(v0, angle, target)}];\n            \n            for (let i = 0; i < maxIterations; i++) {\n                const {gradV0, gradAngle} = calculateGradient(v0, angle, target);\n                \n                // Approximate Hessian with finite differences\n                const hessian = calculateHessian(v0, angle, target);\n                \n                // Newton's method update (simplified)\n                const det = hessian.h11 * hessian.h22 - hessian.h12 * hessian.h21;\n                if (Math.abs(det) > 1e-10) {\n                    const inv_h11 = hessian.h22 / det;\n                    const inv_h12 = -hessian.h12 / det;\n                    const inv_h21 = -hessian.h21 / det;\n                    const inv_h22 = hessian.h11 / det;\n                    \n                    const dv0 = inv_h11 * gradV0 + inv_h12 * gradAngle;\n                    const dangle = inv_h21 * gradV0 + inv_h22 * gradAngle;\n                    \n                    v0 -= 0.1 * dv0;  // Damped Newton\n                    angle -= 0.1 * dangle;\n                } else {\n                    // Fall back to gradient descent\n                    v0 -= learningRate * gradV0;\n                    angle -= learningRate * gradAngle;\n                }\n                \n                v0 = Math.max(1, Math.min(150, v0));\n                angle = Math.max(0.01, Math.min(Math.PI/2 - 0.01, angle));\n                \n                const loss = calculateLoss(v0, angle, target);\n                path.push({v0, angle, loss});\n                \n                if (loss < 10) break;\n            }\n            \n            optimizationPaths.push(path);\n            \n            const result = simulateProjectile(v0, angle);\n            trajectories.push({\n                path: result.path,\n                color: '#4CAF50',\n                target: target,\n                v0: v0,\n                angle: angle,\n                method: 'newton'\n            });\n            \n            const finalLoss = calculateLoss(v0, angle, target);\n            target.hit = finalLoss < 100;\n            updateResults({\n                method: 'Newton\\'s Method',\n                v0: v0,\n                angle: angle * 180 / Math.PI,\n                iterations: path.length,\n                finalLoss: finalLoss,\n                success: finalLoss < 100\n            });\n            \n            drawScene();\n        }\n        \n        function calculateHessian(v0, angle, target, epsilon = 1e-6) {\n            const {gradV0, gradAngle} = calculateGradient(v0, angle, target, epsilon);\n            \n            const {gradV0: gradV0_v0, gradAngle: gradAngle_v0} = calculateGradient(v0 + epsilon, angle, target, epsilon);\n            const {gradV0: gradV0_angle, gradAngle: gradAngle_angle} = calculateGradient(v0, angle + epsilon, target, epsilon);\n            \n            return {\n                h11: (gradV0_v0 - gradV0) / epsilon,\n                h12: (gradV0_angle - gradV0) / epsilon,\n                h21: (gradAngle_v0 - gradAngle) / epsilon,\n                h22: (gradAngle_angle - gradAngle) / epsilon\n            };\n        }\n        \n        function updateResults(result) {\n            const resultsDiv = document.getElementById('results');\n            const statusClass = result.success ? 'success' : 'warning';\n            \n            resultsDiv.innerHTML = `\n                <div class=\"${statusClass}\">Method: ${result.method}</div>\n                <div>Initial Velocity: ${result.v0.toFixed(2)} m/s</div>\n                <div>Launch Angle: ${result.angle.toFixed(2)}°</div>\n                <div>Iterations: ${result.iterations}</div>\n                <div>Final Loss: ${result.finalLoss.toFixed(4)}</div>\n                <div class=\"${statusClass}\">Status: ${result.success ? 'HIT!' : 'MISS'}</div>\n            `;\n        }\n        \n        function drawScene() {\n            ctx.clearRect(0, 0, canvas.width, canvas.height);\n            \n            // Draw ground\n            ctx.fillStyle = '#8B4513';\n            ctx.fillRect(0, canvas.height - 50, canvas.width, 50);\n            \n            // Draw launch point\n            ctx.fillStyle = '#2196F3';\n            ctx.beginPath();\n            ctx.arc(launchPoint.x, launchPoint.y, 8, 0, 2 * Math.PI);\n            ctx.fill();\n            \n            // Draw targets\n            targets.forEach(target => {\n                ctx.fillStyle = target.hit ? '#4CAF50' : '#f44336';\n                ctx.beginPath();\n                ctx.arc(target.x, target.y, 15, 0, 2 * Math.PI);\n                ctx.fill();\n                \n                // Draw target center\n                ctx.fillStyle = 'white';\n                ctx.beginPath();\n                ctx.arc(target.x, target.y, 5, 0, 2 * Math.PI);\n                ctx.fill();\n            });\n            \n            // Draw trajectories\n            trajectories.forEach(trajectory => {\n                ctx.strokeStyle = trajectory.color;\n                ctx.lineWidth = 3;\n                ctx.beginPath();\n                \n                trajectory.path.forEach((point, index) => {\n                    if (index === 0) {\n                        ctx.moveTo(point.x, point.y);\n                    } else {\n                        ctx.lineTo(point.x, point.y);\n                    }\n                });\n                \n                ctx.stroke();\n            });\n            \n            // Draw optimization paths\n            optimizationPaths.forEach(path => {\n                ctx.strokeStyle = '#ff9800';\n                ctx.lineWidth = 2;\n                ctx.setLineDash([5, 5]);\n                \n                // Draw loss over iterations (simplified visualization)\n                const maxLoss = Math.max(...path.map(p => p.loss));\n                const minLoss = Math.min(...path.map(p => p.loss));\n                \n                ctx.beginPath();\n                path.forEach((point, index) => {\n                    const x = 10 + (index / path.length) * 200;\n                    const y = 50 + ((point.loss - minLoss) / (maxLoss - minLoss)) * 100;\n                    \n                    if (index === 0) {\n                        ctx.moveTo(x, y);\n                    } else {\n                        ctx.lineTo(x, y);\n                    }\n                });\n                ctx.stroke();\n                ctx.setLineDash([]);\n            });\n        }\n        \n        function clearCanvas() {\n            targets = [];\n            trajectories = [];\n            optimizationPaths = [];\n            document.getElementById('results').innerHTML = 'Click on a target to see optimization results!';\n            drawScene();\n        }\n        \n        function addRandomTarget() {\n            const x = Math.random() * (canvas.width - 200) + 100;\n            const y = Math.random() * (canvas.height - 200) + 50;\n            targets.push({x, y, hit: false});\n            drawScene();\n        }\n        \n        function animateTrajectory() {\n            if (trajectories.length === 0) return;\n            \n            const lastTrajectory = trajectories[trajectories.length - 1];\n            let frameIndex = 0;\n            \n            function animate() {\n                drawScene();\n                \n                // Draw partial trajectory\n                if (frameIndex < lastTrajectory.path.length) {\n                    ctx.strokeStyle = '#ff5722';\n                    ctx.lineWidth = 5;\n                    ctx.beginPath();\n                    \n                    for (let i = 0; i <= frameIndex; i++) {\n                        const point = lastTrajectory.path[i];\n                        if (i === 0) {\n                            ctx.moveTo(point.x, point.y);\n                        } else {\n                            ctx.lineTo(point.x, point.y);\n                        }\n                    }\n                    ctx.stroke();\n                    \n                    // Draw projectile\n                    const currentPoint = lastTrajectory.path[frameIndex];\n                    ctx.fillStyle = '#ff5722';\n                    ctx.beginPath();\n                    ctx.arc(currentPoint.x, currentPoint.y, 5, 0, 2 * Math.PI);\n                    ctx.fill();\n                    \n                    frameIndex++;\n                    animationId = requestAnimationFrame(animate);\n                } else {\n                    drawScene();\n                }\n            }\n            \n            if (animationId) {\n                cancelAnimationFrame(animationId);\n            }\n            animate();\n        }\n        \n        // Initialize the demo\n        init();\n    </script>\n</body>\n</html>"