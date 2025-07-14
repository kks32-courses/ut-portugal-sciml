# DiffSim: Projectile control demo

This interactive demo shows how differentiable simulation can be used to solve inverse problems in projectile motion. Click on targets to see how the optimal launch parameters are calculated!

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Interactive Projectile Motion with Differential Programming</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/plotly.js/2.27.0/plotly.min.js"></script>

<style>
    #projectile-container { 
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; 
        margin: 10px; 
        background-color: #f9f9f9; 
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 8px;
    }
    .proj-container { 
        display: grid; 
        grid-template-columns: repeat(auto-fit, minmax(350px, 1fr)); 
        gap: 20px; 
    }
    .proj-plot-container { 
        border: 1px solid #ddd; 
        border-radius: 8px; 
        background-color: #fff; 
        box-shadow: 0 2px 5px rgba(0,0,0,0.1); 
        padding: 10px;
    }
    .proj-controls { 
        grid-column: 1 / -1; 
        padding: 20px; 
        background-color: #fff; 
        border-radius: 8px; 
        border: 1px solid #ddd; 
        display: flex; 
        flex-wrap: wrap; 
        justify-content: space-around; 
        align-items: center; 
        gap: 20px; 
        margin-bottom: 20px;
    }
    .proj-slider-group { 
        display: flex; 
        flex-direction: column; 
        align-items: center; 
    }
    .proj-slider-group label { 
        font-weight: bold; 
        margin-bottom: 10px; 
        color: #333; 
    }
    .proj-slider-group input[type=range] { 
        width: 180px; 
    }
    .proj-button { 
        padding: 10px 20px; 
        font-size: 16px; 
        font-weight: bold; 
        color: white; 
        border: none; 
        border-radius: 5px; 
        cursor: pointer; 
        transition: background-color 0.2s; 
        margin: 5px;
    }
    .proj-fire-button { 
        background-color: #28a745; 
    }
    .proj-fire-button:hover { 
        background-color: #218838; 
    }
    .proj-optimize-button { 
        background-color: #007bff; 
    }
    .proj-optimize-button:hover { 
        background-color: #0056b3; 
    }
    .proj-reset-button { 
        background-color: #6c757d; 
    }
    .proj-reset-button:hover { 
        background-color: #545b62; 
    }
    .proj-stop-button { 
        background-color: #dc3545; 
    }
    .proj-stop-button:hover { 
        background-color: #c82333; 
    }
    .proj-plot-title { 
        text-align: center; 
        font-size: 16px; 
        font-weight: bold; 
        padding-top: 15px; 
        color: #444; 
    }
    #proj-statusMessage { 
        grid-column: 1 / -1; 
        text-align: center; 
        font-size: 16px; 
        color: #007bff; 
        font-weight: bold; 
        min-height: 20px; 
        margin-bottom: 10px;
    }
    
    /* Info panel styles */
    .proj-info-section {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        margin-bottom: 20px;
        padding: 20px;
    }
    
    .proj-info-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
        gap: 15px;
        margin-bottom: 15px;
    }
    
    .proj-info-item {
        text-align: center;
        padding: 10px;
        background-color: #f8f9fa;
        border-radius: 6px;
        border: 1px solid #e9ecef;
    }
    
    .proj-info-label {
        font-size: 12px;
        color: #6c757d;
        margin-bottom: 5px;
    }
    
    .proj-info-value {
        font-size: 16px;
        font-weight: bold;
        color: #495057;
    }
    
    /* Equations section */
    .proj-equations-section {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        margin-bottom: 20px;
    }
    
    .proj-equations-header {
        padding: 15px 20px;
        cursor: pointer;
        display: flex;
        align-items: center;
        background-color: #f8f9fa;
        border-radius: 8px 8px 0 0;
        transition: background-color 0.2s;
        user-select: none;
    }
    
    .proj-equations-header:hover {
        background-color: #e9ecef;
    }
    
    .proj-equations-toggle {
        width: 0;
        height: 0;
        border-left: 8px solid #495057;
        border-top: 6px solid transparent;
        border-bottom: 6px solid transparent;
        margin-right: 12px;
        transition: transform 0.3s ease;
    }
    
    .proj-equations-toggle.expanded {
        transform: rotate(90deg);
    }
    
    .proj-equations-content {
        padding: 20px;
        display: none;
    }
    
    .proj-equations-content.show {
        display: block;
    }
    
    .proj-equation {
        background-color: #f8f9fa;
        border: 1px solid #e9ecef;
        border-radius: 6px;
        padding: 15px;
        margin: 10px 0;
        font-family: 'Courier New', monospace;
        font-size: 14px;
    }
    
    .proj-equation-title {
        font-weight: bold;
        color: #495057;
        margin-bottom: 8px;
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
    }
    
    .proj-optimization-info {
        background-color: #fff3cd;
        border: 1px solid #ffeaa7;
        border-radius: 6px;
        padding: 15px;
        margin: 10px 0;
        font-size: 14px;
    }
</style>
</head>
<body>

<div id="projectile-container">
    <h2>Interactive Projectile Motion with Differential Programming</h2>
    <p>Adjust parameters to hit the target, or use automatic differentiation to optimize the trajectory. The AI uses Adam optimizer with proper convergence criteria to prevent oscillation.</p>

    <!-- Controls -->
    <div class="proj-controls">
        <div class="proj-slider-group">
            <label for="proj-angleSlider">Launch Angle: <span id="proj-angleValue">45</span>°</label>
            <input type="range" id="proj-angleSlider" min="5" max="85" value="45" step="1">
        </div>
        <div class="proj-slider-group">
            <label for="proj-velocitySlider">Initial Velocity: <span id="proj-velocityValue">25</span> m/s</label>
            <input type="range" id="proj-velocitySlider" min="10" max="50" value="25" step="1">
        </div>
        <div class="proj-slider-group">
            <label for="proj-learningRateSlider">Learning Rate: <span id="proj-learningRateValue">0.01</span></label>
            <input type="range" id="proj-learningRateSlider" min="0.001" max="0.1" value="0.01" step="0.001">
        </div>
        <div>
            <button id="proj-fireButton" class="proj-button proj-fire-button">Fire!</button>
            <button id="proj-optimizeButton" class="proj-button proj-optimize-button">Auto-Optimize</button>
            <button id="proj-stopButton" class="proj-button proj-stop-button">Stop</button>
            <button id="proj-resetButton" class="proj-button proj-reset-button">New Target</button>
        </div>
    </div>

    <!-- Info Panel -->
    <div class="proj-info-section">
        <div class="proj-info-grid">
            <div class="proj-info-item">
                <div class="proj-info-label">Target Distance</div>
                <div class="proj-info-value" id="proj-targetDistance">40.0 m</div>
            </div>
            <div class="proj-info-item">
                <div class="proj-info-label">Landing Distance</div>
                <div class="proj-info-value" id="proj-landingDistance">-- m</div>
            </div>
            <div class="proj-info-item">
                <div class="proj-info-label">Error Distance</div>
                <div class="proj-info-value" id="proj-errorDistance">-- m</div>
            </div>
            <div class="proj-info-item">
                <div class="proj-info-label">Loss Function</div>
                <div class="proj-info-value" id="proj-lossValue">--</div>
            </div>
            <div class="proj-info-item">
                <div class="proj-info-label">Optimization Step</div>
                <div class="proj-info-value" id="proj-stepCount">0</div>
            </div>
            <div class="proj-info-item">
                <div class="proj-info-label">Gradient Norm</div>
                <div class="proj-info-value" id="proj-gradientNorm">--</div>
            </div>
        </div>
    </div>

    <!-- Equations Section -->
    <div class="proj-equations-section">
        <div class="proj-equations-header" id="proj-equationsHeader">
            <div class="proj-equations-toggle" id="proj-equationsToggle"></div>
            <h3 style="margin: 0; color: #495057;">Mathematical Framework & Optimization</h3>
        </div>
        
        <div class="proj-equations-content" id="proj-equationsContent">
            <div class="proj-equation">
                <div class="proj-equation-title">1. Analytical Projectile Motion:</div>
                <div><strong>x(t) = v₀ cos(θ) × t</strong></div>
                <div><strong>y(t) = v₀ sin(θ) × t - ½gt²</strong></div>
                <div style="margin-top: 8px; font-size: 12px; color: #6c757d;">
                    Time of flight: T = 2v₀sin(θ)/g, Range: R = v₀²sin(2θ)/g
                </div>
            </div>
            
            <div class="proj-equation">
                <div class="proj-equation-title">2. Loss Function & Gradients:</div>
                <div><strong>L = (x_final - x_target)²</strong></div>
                <div><strong>∇L = [∂L/∂θ, ∂L/∂v₀]</strong></div>
                <div style="margin-top: 8px; font-size: 12px; color: #6c757d;">
                    Computed via finite differences: ∂L/∂p ≈ [L(p+ε) - L(p)]/ε
                </div>
            </div>
            
            <div class="proj-optimization-info">
                <div class="proj-equation-title">3. Adam Optimizer with Convergence Criteria:</div>
                <div><strong>m_t = β₁m_{t-1} + (1-β₁)∇L</strong> (momentum)</div>
                <div><strong>v_t = β₂v_{t-1} + (1-β₂)(∇L)²</strong> (adaptive learning)</div>
                <div><strong>θ_new = θ - α × m̂_t / (√v̂_t + ε)</strong></div>
                <div style="margin-top: 8px; font-size: 12px; color: #6c757d;">
                    Stops when: |∇L| < 0.01 OR loss < 0.1 OR loss increase detected
                </div>
            </div>
        </div>
    </div>

    <div id="proj-statusMessage"></div>

    <div class="proj-container">
        <div class="proj-plot-container">
            <div class="proj-plot-title">Projectile Trajectory</div>
            <div id="proj-plot"></div>
        </div>
        <div class="proj-plot-container">
            <div class="proj-plot-title">Optimization Progress</div>
            <div id="proj-lossPlot"></div>
        </div>
    </div>
</div>

<script>
    (function() {
        // Global state
        let targetX = 40.0;
        let isOptimizing = false;
        let optimizationHistory = [];
        let currentTrajectory = null;
        let stepCount = 0;
        
        // Adam optimizer state
        let adamState = {
            m_angle: 0,
            v_angle: 0,
            m_velocity: 0,
            v_velocity: 0,
            beta1: 0.9,
            beta2: 0.999,
            epsilon: 1e-8
        }
        
        // Physics constants
        const g = 9.81;
        const epsilon = 0.001; // for numerical gradients
        
        // Analytical projectile motion
        function calculateRange(angle, velocity) {
            const angleRad = angle * Math.PI / 180;
            const range = (velocity * velocity * Math.sin(2 * angleRad)) / g;
            return range;
        }
        
        function calculateTrajectory(angle, velocity) {
            const angleRad = angle * Math.PI / 180;
            const vx = velocity * Math.cos(angleRad);
            const vy = velocity * Math.sin(angleRad);
            
            const timeOfFlight = 2 * vy / g;
            const points = [];
            const steps = 50;
            
            for (let i = 0; i <= steps; i++) {
                const t = (i / steps) * timeOfFlight;
                const x = vx * t;
                const y = vy * t - 0.5 * g * t * t;
                if (y >= 0) points.push([x, y]);
            }
            
            return points;
        }
        
        // Loss function
        function calculateLoss(angle, velocity) {
            const range = calculateRange(angle, velocity);
            return Math.pow(range - targetX, 2);
        }
        
        // Numerical gradients
        function calculateGradients(angle, velocity) {
            const baseLoss = calculateLoss(angle, velocity);
            
            const anglePlus = calculateLoss(angle + epsilon, velocity);
            const velocityPlus = calculateLoss(angle, velocity + epsilon);
            
            const gradAngle = (anglePlus - baseLoss) / epsilon;
            const gradVelocity = (velocityPlus - baseLoss) / epsilon;
            
            return [gradAngle, gradVelocity];
        }
        
        // Adam optimizer step
        function adamStep(params, gradients, learningRate, step) {
            const [angle, velocity] = params;
            const [gradAngle, gradVelocity] = gradients;
            
            // Update biased first moment estimate
            adamState.m_angle = adamState.beta1 * adamState.m_angle + (1 - adamState.beta1) * gradAngle;
            adamState.m_velocity = adamState.beta1 * adamState.m_velocity + (1 - adamState.beta1) * gradVelocity;
            
            // Update biased second raw moment estimate
            adamState.v_angle = adamState.beta2 * adamState.v_angle + (1 - adamState.beta2) * gradAngle * gradAngle;
            adamState.v_velocity = adamState.beta2 * adamState.v_velocity + (1 - adamState.beta2) * gradVelocity * gradVelocity;
            
            // Compute bias-corrected first moment estimate
            const m_angle_hat = adamState.m_angle / (1 - Math.pow(adamState.beta1, step));
            const m_velocity_hat = adamState.m_velocity / (1 - Math.pow(adamState.beta1, step));
            
            // Compute bias-corrected second raw moment estimate
            const v_angle_hat = adamState.v_angle / (1 - Math.pow(adamState.beta2, step));
            const v_velocity_hat = adamState.v_velocity / (1 - Math.pow(adamState.beta2, step));
            
            // Update parameters
            const newAngle = angle - learningRate * m_angle_hat / (Math.sqrt(v_angle_hat) + adamState.epsilon);
            const newVelocity = velocity - learningRate * m_velocity_hat / (Math.sqrt(v_velocity_hat) + adamState.epsilon);
            
            // Clamp to valid ranges
            return [
                Math.max(5, Math.min(85, newAngle)),
                Math.max(10, Math.min(50, newVelocity))
            ];
        }
        
        // Check convergence
        function checkConvergence(gradients, loss, lossHistory) {
            const gradientNorm = Math.sqrt(gradients[0] * gradients[0] + gradients[1] * gradients[1]);
            
            // Gradient-based convergence
            if (gradientNorm < 0.01) return { converged: true, reason: "Gradient norm < 0.01" };
            
            // Loss-based convergence
            if (loss < 0.1) return { converged: true, reason: "Loss < 0.1 (close enough to target)" };
            
            // Loss increase detection (oscillation)
            if (lossHistory.length > 5) {
                const recent = lossHistory.slice(-5);
                const increasing = recent.every((val, i) => i === 0 || val >= recent[i-1]);
                if (increasing) return { converged: true, reason: "Failed to converge - oscillation detected" };
            }
            
            // Max iterations - only success if loss is very low
            if (lossHistory.length > 200) {
                if (loss < 0.01) {
                    return { converged: true, reason: "✅ Converged: Max iterations reached" };
                } else {
                    return { converged: true, reason: "Failed to converge - max iterations reached" };
                }
            }
            
            return { converged: false, reason: "" };
        }
        
        // Update displays
        function updateInfo(angle, velocity) {
            const range = calculateRange(angle, velocity);
            const loss = calculateLoss(angle, velocity);
            const error = Math.abs(range - targetX);
            
            document.getElementById('proj-angleValue').textContent = angle.toFixed(1);
            document.getElementById('proj-velocityValue').textContent = velocity.toFixed(1);
            document.getElementById('proj-landingDistance').textContent = range.toFixed(2) + ' m';
            document.getElementById('proj-errorDistance').textContent = error.toFixed(2) + ' m';
            document.getElementById('proj-lossValue').textContent = loss.toFixed(2);
            
            if (error < 1.0) {
                document.getElementById('proj-statusMessage').textContent = '🎯 Target Hit! Excellent shot!';
                document.getElementById('proj-statusMessage').style.color = '#28a745';
            } else if (isOptimizing) {
                document.getElementById('proj-statusMessage').textContent = '🧠 Optimizing trajectory...';
                document.getElementById('proj-statusMessage').style.color = '#007bff';
            } else {
                document.getElementById('proj-statusMessage').textContent = `Miss by ${error.toFixed(2)}m`;
                document.getElementById('proj-statusMessage').style.color = '#dc3545';
            }
        }
        
        // Update plots
        function updatePlots(angle, velocity, showTrajectory = false, isOptimizing = false) {
            const range = calculateRange(angle, velocity);
            const traces = [];
            
            // Always show target
            const targetTrace = {
                x: [targetX],
                y: [0],
                mode: 'markers',
                marker: { color: 'red', size: 12, symbol: 'star' },
                name: 'Target'
            };
            traces.push(targetTrace);
            
            // Show trajectory only if requested
            if (showTrajectory) {
                currentTrajectory = calculateTrajectory(angle, velocity);
                const trajectoryTrace = {
                    x: currentTrajectory.map(p => p[0]),
                    y: currentTrajectory.map(p => p[1]),
                    mode: 'lines',
                    line: { 
                        color: isOptimizing ? 'rgba(128, 128, 128, 0.4)' : '#007bff', 
                        width: isOptimizing ? 2 : 3 
                    },
                    name: 'Trajectory'
                };
                traces.push(trajectoryTrace);
                
                const landingTrace = {
                    x: [range],
                    y: [0],
                    mode: 'markers',
                    marker: { 
                        color: isOptimizing ? 'rgba(128, 128, 128, 0.6)' : '#28a745', 
                        size: isOptimizing ? 6 : 10, 
                        symbol: 'circle' 
                    },
                    name: 'Landing'
                };
                traces.push(landingTrace);
            }
            
            const plotLayout = {
                margin: { l: 40, r: 20, t: 20, b: 40 },
                xaxis: { title: 'Distance (m)', range: [0, 80] },
                yaxis: { title: 'Height (m)', range: [0, 30] },
                showlegend: false,
                autosize: true
            };
            
            Plotly.newPlot('proj-plot', traces, plotLayout);
            
            // Update loss plot
            if (optimizationHistory.length > 1) {
                const lossTrace = {
                    x: Array.from({length: optimizationHistory.length}, (_, i) => i + 1),
                    y: optimizationHistory,
                    mode: 'lines+markers',
                    line: { color: '#dc3545', width: 2 },
                    marker: { size: 4 },
                    name: 'Loss'
                };
                
                const lossLayout = {
                    margin: { l: 40, r: 20, t: 20, b: 40 },
                    xaxis: { title: 'Iteration' },
                    yaxis: { title: 'Loss', type: 'log' },
                    showlegend: false,
                    autosize: true
                };
                
                Plotly.newPlot('proj-lossPlot', [lossTrace], lossLayout);
            }
        }
        
        // Animate projectile motion with improved timing
        function animateProjectileMotion(angle, velocity) {
            if (!currentTrajectory || currentTrajectory.length === 0) return;
            
            // Calculate realistic animation duration based on trajectory
            const angleRad = angle * Math.PI / 180;
            const vy = velocity * Math.sin(angleRad);
            const actualFlightTime = 2 * vy / g; // seconds
            
            // Scale animation to be fast but visible (0.8-2.5 seconds based on flight time)
            const animationDuration = Math.min(2500, Math.max(800, actualFlightTime * 400));
            const frameRate = 60; // fps
            const frameCount = Math.floor(animationDuration / (1000/frameRate));
            const frameDuration = 1000 / frameRate;
            
            let currentFrame = 0;
            
            // Start with only target visible
            updatePlots(angle, velocity, false, false);
            
            function animateFrame() {
                if (currentFrame >= frameCount) {
                    // Animation complete - show the final blue trajectory
                    setTimeout(() => {
                        updatePlots(angle, velocity, true, false);
                        // Remove projectile marker
                        const currentTraces = document.getElementById('proj-plot').data;
                        if (currentTraces.length > 3) {
                            Plotly.deleteTraces('proj-plot', [3]);
                        }
                    }, 100);
                    return;
                }
                
                const progress = currentFrame / (frameCount - 1);
                const trajectoryIndex = Math.floor(progress * (currentTrajectory.length - 1));
                const projectilePos = currentTrajectory[trajectoryIndex];
                
                // Create gray trajectory up to current position
                const partialTrajectory = currentTrajectory.slice(0, trajectoryIndex + 1);
                
                // Update plot with partial gray trajectory and projectile
                const traces = [];
                
                // Target
                traces.push({
                    x: [targetX],
                    y: [0],
                    mode: 'markers',
                    marker: { color: 'red', size: 12, symbol: 'star' },
                    name: 'Target'
                });
                
                // Partial gray trajectory
                if (partialTrajectory.length > 1) {
                    traces.push({
                        x: partialTrajectory.map(p => p[0]),
                        y: partialTrajectory.map(p => p[1]),
                        mode: 'lines',
                        line: { color: 'rgba(128, 128, 128, 0.6)', width: 2 },
                        name: 'Flight Path'
                    });
                }
                
                // Projectile
                traces.push({
                    x: [projectilePos[0]],
                    y: [projectilePos[1]],
                    mode: 'markers',
                    marker: { 
                        color: '#ff6b35', 
                        size: 15, 
                        symbol: 'circle',
                        line: { color: '#d63031', width: 2 }
                    },
                    name: 'Projectile'
                });
                
                const plotLayout = {
                    margin: { l: 40, r: 20, t: 20, b: 40 },
                    xaxis: { title: 'Distance (m)', range: [0, 80] },
                    yaxis: { title: 'Height (m)', range: [0, 30] },
                    showlegend: false,
                    autosize: true
                };
                
                Plotly.newPlot('proj-plot', traces, plotLayout);
                
                currentFrame++;
                
                if (currentFrame < frameCount) {
                    setTimeout(animateFrame, frameDuration);
                }
            }
            
            // Start animation
            setTimeout(animateFrame, 100);
        }
        
        // Event handlers
        function fire() {
            const angle = parseFloat(document.getElementById('proj-angleSlider').value);
            const velocity = parseFloat(document.getElementById('proj-velocitySlider').value);
            
            // Calculate trajectory first
            currentTrajectory = calculateTrajectory(angle, velocity);
            
            updateInfo(angle, velocity);
            animateProjectileMotion(angle, velocity);
        }
        
        function optimize() {
            if (isOptimizing) return;
            
            isOptimizing = true;
            stepCount = 0;
            optimizationHistory = [];
            
            // Reset Adam state
            adamState.m_angle = 0;
            adamState.v_angle = 0;
            adamState.m_velocity = 0;
            adamState.v_velocity = 0;
            
            let angle = parseFloat(document.getElementById('proj-angleSlider').value);
            let velocity = parseFloat(document.getElementById('proj-velocitySlider').value);
            const learningRate = parseFloat(document.getElementById('proj-learningRateSlider').value);
            
            function optimizeStep() {
                if (!isOptimizing) return;
                
                stepCount++;
                const loss = calculateLoss(angle, velocity);
                optimizationHistory.push(loss);
                
                const gradients = calculateGradients(angle, velocity);
                const gradientNorm = Math.sqrt(gradients[0] * gradients[0] + gradients[1] * gradients[1]);
                
                // Check convergence
                const convergence = checkConvergence(gradients, loss, optimizationHistory);
                
                // Update displays
                document.getElementById('proj-stepCount').textContent = stepCount;
                document.getElementById('proj-gradientNorm').textContent = gradientNorm.toFixed(4);
                
                if (convergence.converged) {
                    isOptimizing = false;
                    // Show final trajectory with solid blue line
                    updatePlots(angle, velocity, true, false);
                    
                    if (convergence.reason.includes("✅")) {
                        document.getElementById('proj-statusMessage').textContent = convergence.reason;
                        document.getElementById('proj-statusMessage').style.color = '#28a745';
                    } else {
                        document.getElementById('proj-statusMessage').textContent = convergence.reason;
                        document.getElementById('proj-statusMessage').style.color = '#dc3545';
                    }
                    return;
                }
                
                // Adam optimization step
                [angle, velocity] = adamStep([angle, velocity], gradients, learningRate, stepCount);
                
                // Update UI
                document.getElementById('proj-angleSlider').value = angle;
                document.getElementById('proj-velocitySlider').value = velocity;
                
                updateInfo(angle, velocity);
                updatePlots(angle, velocity, true, true); // Show trajectory, is optimizing (gray line)
                
                setTimeout(optimizeStep, 100);
            }
            
            optimizeStep();
        }
        
        function stopOptimization() {
            isOptimizing = false;
            document.getElementById('proj-statusMessage').textContent = '⏹️ Optimization stopped';
            document.getElementById('proj-statusMessage').style.color = '#6c757d';
        }
        
        function resetTarget() {
            targetX = 20 + Math.random() * 40;
            document.getElementById('proj-targetDistance').textContent = targetX.toFixed(1) + ' m';
            document.getElementById('proj-landingDistance').textContent = '-- m';
            document.getElementById('proj-errorDistance').textContent = '-- m';
            document.getElementById('proj-lossValue').textContent = '--';
            document.getElementById('proj-stepCount').textContent = '0';
            document.getElementById('proj-gradientNorm').textContent = '--';
            document.getElementById('proj-statusMessage').textContent = '';
            
            optimizationHistory = [];
            stepCount = 0;
            isOptimizing = false;
            
            const angle = parseFloat(document.getElementById('proj-angleSlider').value);
            const velocity = parseFloat(document.getElementById('proj-velocitySlider').value);
            updatePlots(angle, velocity, false, false); // Don't show trajectory after reset
        }
        
        function toggleEquations() {
            const content = document.getElementById('proj-equationsContent');
            const toggle = document.getElementById('proj-equationsToggle');
            
            if (content.classList.contains('show')) {
                content.classList.remove('show');
                toggle.classList.remove('expanded');
            } else {
                content.classList.add('show');
                toggle.classList.add('expanded');
            }
        }
        
        // Initialize
        function init() {
            // Event listeners
            document.getElementById('proj-angleSlider').addEventListener('input', () => {
                if (!isOptimizing) {
                    // Only update info, don't show trajectory until fire is pressed
                    const angle = parseFloat(document.getElementById('proj-angleSlider').value);
                    const velocity = parseFloat(document.getElementById('proj-velocitySlider').value);
                    document.getElementById('proj-angleValue').textContent = angle.toFixed(1);
                    updatePlots(angle, velocity, false, false); // Don't show trajectory
                }
            });
            document.getElementById('proj-velocitySlider').addEventListener('input', () => {
                if (!isOptimizing) {
                    // Only update info, don't show trajectory until fire is pressed
                    const angle = parseFloat(document.getElementById('proj-angleSlider').value);
                    const velocity = parseFloat(document.getElementById('proj-velocitySlider').value);
                    document.getElementById('proj-velocityValue').textContent = velocity.toFixed(1);
                    updatePlots(angle, velocity, false, false); // Don't show trajectory
                }
            });
            document.getElementById('proj-learningRateSlider').addEventListener('input', function() {
                document.getElementById('proj-learningRateValue').textContent = this.value;
            });
            
            document.getElementById('proj-fireButton').addEventListener('click', fire);
            document.getElementById('proj-optimizeButton').addEventListener('click', optimize);
            document.getElementById('proj-stopButton').addEventListener('click', stopOptimization);
            document.getElementById('proj-resetButton').addEventListener('click', resetTarget);
            document.getElementById('proj-equationsHeader').addEventListener('click', toggleEquations);
            
            // Initial setup
            resetTarget();
            // Don't show trajectory initially
            updatePlots(45, 25, false, false);
        }
        
        if (document.readyState === 'loading') {
            document.addEventListener('DOMContentLoaded', init);
        } else {
            init();
        }
    })();
</script>

</body>
</html>