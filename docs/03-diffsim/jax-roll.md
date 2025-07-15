# JAX Roll

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>JAX Roll Implementation of Finite Differences</title>
<style>
    #finite-diff-container { 
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; 
        margin: 10px; 
        background-color: #f9f9f9; 
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 8px;
    }
    .fd-container { 
        display: grid; 
        grid-template-columns: repeat(auto-fit, minmax(350px, 1fr)); 
        gap: 20px; 
    }
    .fd-grid-container { 
        border: 1px solid #ddd; 
        border-radius: 8px; 
        background-color: #fff; 
        box-shadow: 0 2px 5px rgba(0,0,0,0.1); 
        padding: 10px;
    }
    .fd-controls { 
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
    .fd-button-group { 
        display: flex; 
        gap: 10px;
        align-items: center; 
    }
    .fd-button { 
        padding: 10px 20px; 
        font-size: 16px; 
        font-weight: bold; 
        color: white; 
        background-color: #28a745; 
        border: none; 
        border-radius: 5px; 
        cursor: pointer; 
        transition: background-color 0.2s; 
    }
    .fd-button:hover { 
        background-color: #218838; 
    }
    .fd-button.secondary {
        background-color: #007bff;
    }
    .fd-button.secondary:hover {
        background-color: #0056b3;
    }
    .fd-speed-control {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 5px;
    }
    .fd-speed-control label {
        font-size: 14px;
        color: #666;
    }
    .fd-speed-control input {
        width: 150px;
    }
    .fd-grid-title { 
        text-align: center; 
        font-size: 16px; 
        font-weight: bold; 
        padding-top: 15px; 
        color: #444; 
        margin-bottom: 15px;
    }
    .fd-step-indicator {
        grid-column: 1 / -1;
        display: flex;
        justify-content: center;
        gap: 10px;
        margin-bottom: 20px;
    }
    .fd-step {
        padding: 8px 16px;
        background-color: #f8f9fa;
        border: 1px solid #dee2e6;
        border-radius: 20px;
        font-size: 14px;
        color: #6c757d;
        transition: all 0.3s ease;
    }
    .fd-step.active {
        background-color: #007bff;
        color: white;
        border-color: #007bff;
    }
    
    /* Grid visualization styles */
    .fd-array-grid {
        display: grid;
        grid-template-columns: repeat(8, 1fr);
        gap: 5px;
        margin: 15px 0;
        justify-items: center;
    }
    .fd-grid-cell {
        width: 35px;
        height: 35px;
        border: 2px solid #ddd;
        border-radius: 4px;
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 12px;
        font-weight: bold;
        background-color: #f8f9fa;
        position: relative;
        transition: all 0.3s ease;
    }
    .fd-grid-cell.original {
        background-color: #e3f2fd;
        border-color: #1976d2;
        color: #1976d2;
    }
    .fd-grid-cell.rolled-left {
        background-color: #f3e5f5;
        border-color: #7b1fa2;
        color: #7b1fa2;
    }
    .fd-grid-cell.rolled-right {
        background-color: #fff3e0;
        border-color: #f57c00;
        color: #f57c00;
    }
    .fd-grid-cell.boundary {
        background-color: #ffebee;
        border-color: #d32f2f;
        color: #d32f2f;
        font-weight: bold;
    }
    .fd-grid-cell.highlighted {
        transform: scale(1.1);
        box-shadow: 0 2px 8px rgba(0,0,0,0.2);
        z-index: 10;
    }
    .fd-index-label {
        position: absolute;
        bottom: -18px;
        font-size: 10px;
        color: #666;
    }
    
    /* Code section styles */
    .fd-code-section {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        margin-bottom: 20px;
    }
    .fd-code-header {
        padding: 15px;
        cursor: pointer;
        display: flex;
        align-items: center;
        background-color: #f8f9fa;
        border-radius: 8px 8px 0 0;
        user-select: none;
    }
    .fd-code-toggle {
        width: 0;
        height: 0;
        border-left: 8px solid #495057;
        border-top: 6px solid transparent;
        border-bottom: 6px solid transparent;
        margin-right: 12px;
        transition: transform 0.3s ease;
    }
    .fd-code-toggle.expanded {
        transform: rotate(90deg);
    }
    .fd-code-content {
        padding: 20px;
        display: none;
    }
    .fd-code-content.show {
        display: block;
    }
    .fd-code-block {
        background-color: #f8f9fa;
        border: 1px solid #e9ecef;
        border-radius: 6px;
        padding: 15px;
        margin: 10px 0;
        font-family: 'Courier New', monospace;
        font-size: 14px;
        overflow-x: auto;
    }
    .fd-highlight {
        background-color: #fff3cd;
        padding: 2px 4px;
        border-radius: 3px;
    }
    
    /* Equation display */
    .fd-equation-display {
        text-align: center;
        font-family: 'Times New Roman', serif;
        font-size: 18px;
        margin: 20px 0;
        padding: 15px;
        background-color: #f8f9fa;
        border-radius: 6px;
        border: 1px solid #e9ecef;
    }
    .fd-stencil-visual {
        display: flex;
        justify-content: center;
        align-items: center;
        gap: 10px;
        margin: 15px 0;
    }
    .fd-stencil-term {
        padding: 8px 12px;
        border-radius: 20px;
        font-weight: bold;
        font-size: 14px;
    }
    .fd-stencil-left {
        background-color: #f3e5f5;
        color: #7b1fa2;
        border: 2px solid #7b1fa2;
    }
    .fd-stencil-center {
        background-color: #e3f2fd;
        color: #1976d2;
        border: 2px solid #1976d2;
    }
    .fd-stencil-right {
        background-color: #fff3e0;
        color: #f57c00;
        border: 2px solid #f57c00;
    }
    .fd-operator {
        font-size: 20px;
        font-weight: bold;
        color: #333;
    }
</style>
</head>
<body>

<div id="finite-diff-container">
    <h2>JAX Roll Implementation of Finite Differences</h2>
    <p>See how <code>jnp.roll</code> implements the finite difference stencil for the wave equation. Watch how arrays are shifted and boundary conditions are applied.</p>

    <!-- Code Section -->
    <div class="fd-code-section">
        <div class="fd-code-header" id="fd-codeHeader">
            <div class="fd-code-toggle" id="fd-codeToggle"></div>
            <h3 style="margin: 0; color: #495057;">Wave Equation Implementation</h3>
        </div>
        
        <div class="fd-code-content" id="fd-codeContent">
            <div class="fd-code-block">
# Central difference scheme using jnp.roll<br>
<span class="fd-highlight">u1p = jnp.roll(u1, 1).at[0].set(0)</span>      # u[j-1]<br>
<span class="fd-highlight">u1n = jnp.roll(u1, -1).at[n-1].set(0)</span>   # u[j+1]<br>
<span class="fd-highlight">u2 = 2 * u1 - u0 + C2 * (u1p - 2 * u1 + u1n)</span>
            </div>
            
            <p><strong>Key Insight:</strong> Instead of explicit indexing, we use <code>jnp.roll</code> to shift the entire array, then apply boundary conditions using <code>.at[].set()</code></p>
            
            <div class="fd-equation-display">
                <strong>Finite Difference Stencil:</strong><br>
                ∂²u/∂x² ≈ (u[j-1] - 2u[j] + u[j+1]) / Δx²
            </div>
        </div>
    </div>

    <!-- Controls -->
    <div class="fd-controls">
        <div class="fd-button-group">
            <button id="fd-playButton" class="fd-button">▶️ Play</button>
            <button id="fd-stepButton" class="fd-button secondary">⏭️ Step</button>
            <button id="fd-resetButton" class="fd-button secondary">🔄 Reset</button>
        </div>
        <div class="fd-speed-control">
            <label for="fd-speedSlider">Animation Speed</label>
            <input type="range" id="fd-speedSlider" min="500" max="3000" value="1500" step="100">
        </div>
    </div>

    <!-- Step Indicator -->
    <div class="fd-step-indicator">
        <div class="fd-step active" id="fd-step0">1. Original u1</div>
        <div class="fd-step" id="fd-step1">2. Roll Left (+1)</div>
        <div class="fd-step" id="fd-step2">3. Roll Right (-1)</div>
        <div class="fd-step" id="fd-step3">4. Finite Difference</div>
    </div>

    <div class="fd-container">
        <div class="fd-grid-container">
            <div class="fd-grid-title">Original Array: u1</div>
            <div class="fd-array-grid" id="fd-grid-original"></div>
            <p style="text-align: center; font-size: 12px; color: #666; margin: 5px;">Current wave state</p>
        </div>

        <div class="fd-grid-container">
            <div class="fd-grid-title">u1p = jnp.roll(u1, +1)</div>
            <div class="fd-array-grid" id="fd-grid-left"></div>
            <p style="text-align: center; font-size: 12px; color: #666; margin: 5px;">Left neighbors (boundary at [0] = 0)</p>
        </div>

        <div class="fd-grid-container">
            <div class="fd-grid-title">u1n = jnp.roll(u1, -1)</div>
            <div class="fd-array-grid" id="fd-grid-right"></div>
            <p style="text-align: center; font-size: 12px; color: #666; margin: 5px;">Right neighbors (boundary at [n-1] = 0)</p>
        </div>

        <div class="fd-grid-container">
            <div class="fd-grid-title">Finite Difference Result</div>
            <div class="fd-stencil-visual">
                <div class="fd-stencil-term fd-stencil-left">u1p[j]</div>
                <div class="fd-operator">-</div>
                <div class="fd-stencil-term fd-stencil-center">2×u1[j]</div>
                <div class="fd-operator">+</div>
                <div class="fd-stencil-term fd-stencil-right">u1n[j]</div>
            </div>
            <div class="fd-array-grid" id="fd-grid-result"></div>
            <p style="text-align: center; font-size: 12px; color: #666; margin: 5px;">Second derivative approximation</p>
        </div>
    </div>
</div>

<script>
(function() {
    const n = 8;
    let currentStep = 0;
    let animationInterval;
    let isPlaying = false;
    
    // Sample wave data (sine-like pattern)
    const waveData = [0.0, 0.3, 0.8, 1.2, 1.0, 0.6, 0.2, 0.0];
    
    function createGridCell(value, index, type = 'original') {
        const cell = document.createElement('div');
        cell.className = `fd-grid-cell ${type}`;
        cell.textContent = typeof value === 'number' ? value.toFixed(1) : value;
        
        const indexLabel = document.createElement('div');
        indexLabel.className = 'fd-index-label';
        indexLabel.textContent = index;
        cell.appendChild(indexLabel);
        
        return cell;
    }
    
    function displayGrid(containerId, data, type = 'original', highlightIndex = -1) {
        const container = document.getElementById(containerId);
        container.innerHTML = '';
        
        data.forEach((value, index) => {
            const cell = createGridCell(value, index, type);
            if (index === highlightIndex) {
                cell.classList.add('highlighted');
            }
            if ((type === 'rolled-left' && index === 0) || 
                (type === 'rolled-right' && index === data.length - 1)) {
                cell.classList.add('boundary');
            }
            container.appendChild(cell);
        });
    }
    
    function rollLeft(arr) {
        const result = [...arr];
        const first = result.shift();
        result.push(first);
        result[0] = 0; // Boundary condition: .at[0].set(0)
        return result;
    }
    
    function rollRight(arr) {
        const result = [...arr];
        const last = result.pop();
        result.unshift(last);
        result[result.length - 1] = 0; // Boundary condition: .at[n-1].set(0)
        return result;
    }
    
    function computeFiniteDiff(u1, u1p, u1n) {
        return u1.map((_, i) => u1p[i] - 2 * u1[i] + u1n[i]);
    }
    
    function updateStepIndicator(step) {
        for (let i = 0; i < 4; i++) {
            const stepEl = document.getElementById(`fd-step${i}`);
            stepEl.classList.toggle('active', i === step);
        }
    }
    
    function performStep() {
        const u1 = waveData;
        const u1p = rollLeft(u1);
        const u1n = rollRight(u1);
        const finiteDiff = computeFiniteDiff(u1, u1p, u1n);
        
        // Always show original
        displayGrid('fd-grid-original', u1, 'original');
        
        switch(currentStep) {
            case 0:
                // Show only original
                displayGrid('fd-grid-left', new Array(n).fill(''), 'rolled-left');
                displayGrid('fd-grid-right', new Array(n).fill(''), 'rolled-right');
                displayGrid('fd-grid-result', new Array(n).fill(''), 'original');
                break;
            case 1:
                // Show original + rolled left
                displayGrid('fd-grid-left', u1p, 'rolled-left');
                displayGrid('fd-grid-right', new Array(n).fill(''), 'rolled-right');
                displayGrid('fd-grid-result', new Array(n).fill(''), 'original');
                break;
            case 2:
                // Show original + both rolls
                displayGrid('fd-grid-left', u1p, 'rolled-left');
                displayGrid('fd-grid-right', u1n, 'rolled-right');
                displayGrid('fd-grid-result', new Array(n).fill(''), 'original');
                break;
            case 3:
                // Show everything including result
                displayGrid('fd-grid-left', u1p, 'rolled-left');
                displayGrid('fd-grid-right', u1n, 'rolled-right');
                displayGrid('fd-grid-result', finiteDiff, 'original');
                break;
        }
        
        updateStepIndicator(currentStep);
    }
    
    function nextStep() {
        currentStep = (currentStep + 1) % 4;
        performStep();
    }
    
    function startAnimation() {
        if (isPlaying) return;
        isPlaying = true;
        document.getElementById('fd-playButton').textContent = '⏸️ Pause';
        
        const speed = parseInt(document.getElementById('fd-speedSlider').value);
        animationInterval = setInterval(nextStep, speed);
    }
    
    function stopAnimation() {
        isPlaying = false;
        document.getElementById('fd-playButton').textContent = '▶️ Play';
        clearInterval(animationInterval);
    }
    
    function toggleAnimation() {
        if (isPlaying) {
            stopAnimation();
        } else {
            startAnimation();
        }
    }
    
    function reset() {
        stopAnimation();
        currentStep = 0;
        performStep();
    }
    
    function toggleCode() {
        const content = document.getElementById('fd-codeContent');
        const toggle = document.getElementById('fd-codeToggle');
        
        if (content.classList.contains('show')) {
            content.classList.remove('show');
            toggle.classList.remove('expanded');
        } else {
            content.classList.add('show');
            toggle.classList.add('expanded');
        }
    }
    
    // Event listeners
    document.getElementById('fd-playButton').addEventListener('click', toggleAnimation);
    document.getElementById('fd-stepButton').addEventListener('click', () => {
        stopAnimation();
        nextStep();
    });
    document.getElementById('fd-resetButton').addEventListener('click', reset);
    document.getElementById('fd-codeHeader').addEventListener('click', toggleCode);
    
    document.getElementById('fd-speedSlider').addEventListener('input', () => {
        if (isPlaying) {
            stopAnimation();
            startAnimation();
        }
    });
    
    // Initialize
    reset();
})();
</script>

</body>
</html>