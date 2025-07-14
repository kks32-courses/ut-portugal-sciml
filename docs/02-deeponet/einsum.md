# DeepONet: Einstein Summation

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Interactive DeepONet Einsum Demo</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/plotly.js/2.18.0/plotly.min.js"></script>
<style>
    #einsum-interactive-container { 
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; 
        margin: 10px; 
        background-color: #f9f9f9; 
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 8px;
    }
    
    .einsum-container { 
        display: grid; 
        grid-template-columns: repeat(auto-fit, minmax(350px, 1fr)); 
        gap: 20px; 
    }
    
    .einsum-plot-container { 
        border: 1px solid #ddd; 
        border-radius: 8px; 
        background-color: #fff; 
        box-shadow: 0 2px 5px rgba(0,0,0,0.1); 
        padding: 10px;
    }
    
    .einsum-controls { 
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
    
    .einsum-slider-group { 
        display: flex; 
        flex-direction: column; 
        align-items: center; 
    }
    
    .einsum-slider-group label { 
        font-weight: bold; 
        margin-bottom: 10px; 
        color: #333; 
    }
    
    .einsum-slider-group input[type=range] { 
        width: 220px; 
    }
    
    .einsum-demo-button { 
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
    
    .einsum-demo-button:hover { 
        background-color: #218838; 
    }
    
    .einsum-plot-title { 
        text-align: center; 
        font-size: 16px; 
        font-weight: bold; 
        padding-top: 15px; 
        color: #444; 
    }
    
    #einsum-statusMessage { 
        grid-column: 1 / -1; 
        text-align: center; 
        font-size: 18px; 
        color: #007bff; 
        font-weight: bold; 
        min-height: 25px; 
    }
    
    .einsum-equations-section {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        margin-bottom: 20px;
    }
    
    .einsum-equations-header {
        padding: 15px 1em;
        cursor: pointer;
        display: flex;
        align-items: center;
        background-color: #f8f9fa;
        border-radius: 8px 8px 0 0;
        transition: background-color 0.2s;
        user-select: none;
    }
    
    .einsum-equations-header:hover {
        background-color: #e9ecef;
    }
    
    .einsum-equations-toggle {
        width: 0;
        height: 0;
        border-left: 8px solid #495057;
        border-top: 6px solid transparent;
        border-bottom: 6px solid transparent;
        margin-right: 12px;
        transition: transform 0.3s ease;
    }
    
    .einsum-equations-toggle.expanded {
        transform: rotate(90deg);
    }
    
    .einsum-equations-content {
        padding: 20px;
        display: none;
    }
    
    .einsum-equations-content.show {
        display: block;
    }
    
    .einsum-equation {
        background-color: #f8f9fa;
        border: 1px solid #e9ecef;
        border-radius: 6px;
        padding: 15px;
        margin: 10px 0;
        font-family: 'Courier New', monospace;
        font-size: 16px;
    }
    
    .einsum-equation-title {
        font-weight: bold;
        color: #495057;
        margin-bottom: 8px;
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
    }
    
    .einsum-highlight {
        background-color: #fff3cd;
        border: 1px solid #ffeaa7;
        padding: 15px;
        border-radius: 6px;
        margin: 10px 0;
        font-size: 14px;
        color: #856404;
    }
    
    .einsum-operation-visual {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        padding: 20px;
        margin-bottom: 20px;
    }
    
    .einsum-flow-container {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 15px;
        margin: 20px 0;
        flex-wrap: wrap;
    }
    
    .einsum-tensor-box {
        border: 3px solid;
        border-radius: 12px;
        padding: 15px;
        background-color: #f8f9fa;
        font-family: 'Courier New', monospace;
        font-size: 12px;
        text-align: center;
        min-width: 120px;
        position: relative;
    }
    
    .einsum-tensor-box.branch {
        border-color: #28a745;
        background-color: #d4edda;
    }
    
    .einsum-tensor-box.trunk {
        border-color: #dc3545;
        background-color: #f8d7da;
    }
    
    .einsum-tensor-box.result {
        border-color: #ffc107;
        background-color: #fff3cd;
    }
    
    .einsum-arrow {
        font-size: 24px;
        font-weight: bold;
        color: #6f42c1;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    
    .einsum-tensor-label {
        font-weight: bold;
        margin-bottom: 8px;
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
        font-size: 14px;
    }
    
    .einsum-detailed-computation {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        padding: 20px;
        margin-bottom: 20px;
    }
    
    .einsum-computation-title {
        font-size: 18px;
        font-weight: bold;
        color: #495057;
        margin-bottom: 15px;
        text-align: center;
    }
    
    .einsum-computation-steps {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
        gap: 15px;
    }
    
    .einsum-step-box {
        border: 2px solid #e9ecef;
        border-radius: 8px;
        padding: 15px;
        background-color: #f8f9fa;
    }
    
    .einsum-step-box.active {
        border-color: #007bff;
        background-color: #e3f2fd;
    }
    
    .einsum-step-title {
        font-weight: bold;
        color: #495057;
        margin-bottom: 8px;
        font-size: 14px;
    }
    
    .einsum-step-content {
        font-family: 'Courier New', monospace;
        font-size: 12px;
        color: #666;
    }
    
    .einsum-wide-container {
        grid-column: 1 / -1;
    }
</style>
</head>
<body>

<div id="einsum-interactive-container">
    <h2>Interactive DeepONet Einsum Demo</h2>
    <p>Explore how torch.einsum('bp,bnp->bn', branch_out, trunk_out) combines branch and trunk network outputs step by step.</p>

    <!-- Controls -->
    <div class="einsum-controls">
        <div class="einsum-slider-group">
            <label for="einsum-batchSlider">Batch Size: <span id="einsum-batchValue">2</span></label>
            <input type="range" id="einsum-batchSlider" min="1" max="3" value="2" step="1">
        </div>
        <div class="einsum-slider-group">
            <label for="einsum-pSlider">Feature Dim (p): <span id="einsum-pValue">3</span></label>
            <input type="range" id="einsum-pSlider" min="2" max="4" value="3" step="1">
        </div>
        <div class="einsum-slider-group">
            <label for="einsum-nSlider">Query Points (n): <span id="einsum-nValue">3</span></label>
            <input type="range" id="einsum-nSlider" min="2" max="4" value="3" step="1">
        </div>
        <button id="einsum-randomizeButton" class="einsum-demo-button">Randomize Values</button>
        <button id="einsum-animateButton" class="einsum-demo-button">Animate Computation</button>
    </div>

    <!-- Equations Section -->
    <div class="einsum-equations-section">
        <div class="einsum-equations-header" id="einsum-equationsHeader">
            <div class="einsum-equations-toggle" id="einsum-equationsToggle"></div>
            <h3 style="margin: 0; color: #495057;">DeepONet Einsum Operation</h3>
        </div>
        
        <div class="einsum-equations-content" id="einsum-equationsContent">
            <div class="einsum-equation">
                <div class="einsum-equation-title">DeepONet Forward Pass:</div>
                <div><strong>result = torch.einsum('bp,bnp->bn', branch_out, trunk_out)</strong></div>
                <div style="margin-top: 8px; font-size: 14px; color: #6c757d;">
                    This computes dot products between branch features and trunk features for each query point
                </div>
            </div>
            
            <div class="einsum-equation">
                <div class="einsum-equation-title">Mathematical Operation:</div>
                <div><strong>result[b,n] = Σ(p=0 to P-1) branch[b,p] × trunk[b,n,p]</strong></div>
                <div style="margin-top: 8px; font-size: 14px; color: #6c757d;">
                    For each batch b and query point n, sum over all feature dimensions p
                </div>
            </div>
            
            <div class="einsum-highlight">
                <strong>In DeepONet:</strong> Branch learns function representations, Trunk learns spatial/temporal coordinates. 
                Einsum combines them to predict function values at any query location.
            </div>
        </div>
    </div>

    <!-- Operation Flow -->
    <div class="einsum-operation-visual">
        <h3 style="text-align: center; margin-bottom: 20px;">Tensor Flow Visualization</h3>
        <div class="einsum-flow-container" id="einsum-flowContainer">
            <!-- Dynamic content filled by JavaScript -->
        </div>
    </div>

    <!-- Detailed Computation -->
    <div class="einsum-detailed-computation">
        <h3 class="einsum-computation-title">Step-by-Step Computation</h3>
        <div class="einsum-computation-steps" id="einsum-computationSteps">
            <!-- Dynamic content filled by JavaScript -->
        </div>
    </div>

    <div id="einsum-statusMessage"></div>

    <div class="einsum-container">
        <div class="einsum-plot-container">
            <div class="einsum-plot-title">Branch Output Vectors [batch, p]</div>
            <div id="einsum-plotBranch"></div>
        </div>
        <div class="einsum-plot-container">
            <div class="einsum-plot-title">Trunk Feature Matrix [batch, n, p]</div>
            <div id="einsum-plotTrunk"></div>
        </div>
        <div class="einsum-plot-container einsum-wide-container">
            <div class="einsum-plot-title">Dot Product Computation Visualization</div>
            <div id="einsum-plotComputation"></div>
        </div>
        <div class="einsum-plot-container">
            <div class="einsum-plot-title">Final Result [batch, n]</div>
            <div id="einsum-plotResult"></div>
        </div>
    </div>
</div>

<script>
(function() {
    let branchData, trunkData, resultData;
    let animationActive = false;
    let currentComputationStep = -1;
    
    // Generate random data with cleaner values
    function generateBranchData(batch, p) {
        const data = [];
        for (let b = 0; b < batch; b++) {
            const row = [];
            for (let i = 0; i < p; i++) {
                row.push(Math.round((Math.random() - 0.5) * 6 * 10) / 10);
            }
            data.push(row);
        }
        return data;
    }
    
    function generateTrunkData(batch, n, p) {
        const data = [];
        for (let b = 0; b < batch; b++) {
            const batchData = [];
            for (let i = 0; i < n; i++) {
                const point = [];
                for (let j = 0; j < p; j++) {
                    point.push(Math.round((Math.random() - 0.5) * 6 * 10) / 10);
                }
                batchData.push(point);
            }
            data.push(batchData);
        }
        return data;
    }
    
    function computeEinsum(branchOut, trunkOut) {
        const batch = branchOut.length;
        const n = trunkOut[0].length;
        const result = [];
        
        for (let b = 0; b < batch; b++) {
            const batchResult = [];
            for (let i = 0; i < n; i++) {
                let dotProduct = 0;
                for (let j = 0; j < branchOut[b].length; j++) {
                    dotProduct += branchOut[b][j] * trunkOut[b][i][j];
                }
                batchResult.push(Math.round(dotProduct * 100) / 100);
            }
            result.push(batchResult);
        }
        return result;
    }
    
    // Create improved heatmap with proper integer labels
    function createBranchHeatmap(data, containerId) {
        const batch = data.length;
        const p = data[0].length;
        
        const trace = {
            z: data,
            type: 'heatmap',
            colorscale: 'Viridis',
            showscale: true,
            colorbar: { title: 'Value' },
            text: data.map(row => row.map(val => val.toFixed(1))),
            texttemplate: '%{text}',
            textfont: { size: 12, color: 'white' }
        };
        
        const layout = {
            margin: { l: 60, r: 20, t: 20, b: 40 },
            xaxis: { 
                title: 'Feature Dimension',
                tickmode: 'array',
                tickvals: Array.from({length: p}, (_, i) => i),
                ticktext: Array.from({length: p}, (_, i) => `p${i}`)
            },
            yaxis: { 
                title: 'Batch',
                tickmode: 'array',
                tickvals: Array.from({length: batch}, (_, i) => i),
                ticktext: Array.from({length: batch}, (_, i) => `B${i}`)
            },
            height: 250
        };
        
        Plotly.react(containerId, [trace], layout);
    }
    
    // Create trunk visualization showing all batch items in separate subplots
    function createTrunkVisualization(data, containerId) {
        const batch = data.length;
        const n = data[0].length;
        const p = data[0][0].length;
        
        const traces = [];
        
        for (let b = 0; b < batch; b++) {
            const trace = {
                z: data[b],
                type: 'heatmap',
                colorscale: 'RdBu',
                showscale: b === 0,
                colorbar: b === 0 ? { title: 'Value', x: 1.02 } : undefined,
                text: data[b].map(row => row.map(val => val.toFixed(1))),
                texttemplate: '%{text}',
                textfont: { size: 10 },
                xaxis: `x${b + 1}`,
                yaxis: `y${b + 1}`
            };
            traces.push(trace);
        }
        
        // Create subplot layout
        const layout = {
            margin: { l: 40, r: 50, t: 40, b: 40 },
            height: 300,
            grid: { rows: 1, columns: batch, pattern: 'independent' }
        };
        
        // Add axis configurations for each subplot
        for (let b = 0; b < batch; b++) {
            const xkey = b === 0 ? 'xaxis' : `xaxis${b + 1}`;
            const ykey = b === 0 ? 'yaxis' : `yaxis${b + 1}`;
            
            layout[xkey] = {
                title: b === Math.floor(batch/2) ? 'Feature Dim' : '',
                tickmode: 'array',
                tickvals: Array.from({length: p}, (_, i) => i),
                ticktext: Array.from({length: p}, (_, i) => `p${i}`)
            };
            
            layout[ykey] = {
                title: `B${b} Query Points`,
                tickmode: 'array',
                tickvals: Array.from({length: n}, (_, i) => i),
                ticktext: Array.from({length: n}, (_, i) => `n${i}`)
            };
        }
        
        Plotly.react(containerId, traces, layout);
    }
    
    // Create computation visualization
    function createComputationVisualization(branchData, trunkData, resultData, containerId) {
        const batch = branchData.length;
        const n = trunkData[0].length;
        const p = branchData[0].length;
        
        const traces = [];
        
        // Create a visualization showing the dot product computation
        for (let b = 0; b < batch; b++) {
            for (let i = 0; i < n; i++) {
                const x = Array.from({length: p}, (_, j) => j);
                const y_branch = branchData[b];
                const y_trunk = trunkData[b][i];
                
                traces.push({
                    x: x,
                    y: y_branch,
                    mode: 'lines+markers',
                    name: `B${b} Branch`,
                    line: { color: '#28a745', width: 3 },
                    marker: { size: 8 },
                    xaxis: `x${b * n + i + 1}`,
                    yaxis: `y${b * n + i + 1}`,
                    showlegend: b === 0 && i === 0
                });
                
                traces.push({
                    x: x,
                    y: y_trunk,
                    mode: 'lines+markers',
                    name: `B${b} Trunk n${i}`,
                    line: { color: '#dc3545', width: 3, dash: 'dash' },
                    marker: { size: 8 },
                    xaxis: `x${b * n + i + 1}`,
                    yaxis: `y${b * n + i + 1}`,
                    showlegend: b === 0 && i === 0
                });
            }
        }
        
        const layout = {
            margin: { l: 40, r: 20, t: 60, b: 40 },
            height: 200 * batch,
            grid: { rows: batch, columns: n, pattern: 'independent' },
            annotations: []
        };
        
        // Add subplot titles and axes
        for (let b = 0; b < batch; b++) {
            for (let i = 0; i < n; i++) {
                const subplotIndex = b * n + i + 1;
                const xkey = subplotIndex === 1 ? 'xaxis' : `xaxis${subplotIndex}`;
                const ykey = subplotIndex === 1 ? 'yaxis' : `yaxis${subplotIndex}`;
                
                layout[xkey] = {
                    title: b === batch - 1 ? 'Feature Index' : '',
                    tickmode: 'array',
                    tickvals: Array.from({length: p}, (_, j) => j),
                    ticktext: Array.from({length: p}, (_, j) => `p${j}`)
                };
                
                layout[ykey] = {
                    title: i === 0 ? `B${b}` : '',
                    range: [-4, 4]
                };
                
                // Add result annotation
                layout.annotations.push({
                    x: 0.5,
                    y: 1,
                    xref: `x${subplotIndex} domain`,
                    yref: `y${subplotIndex} domain`,
                    text: `Result: ${resultData[b][i].toFixed(2)}`,
                    showarrow: false,
                    font: { size: 12, color: '#ffc107' },
                    bgcolor: 'rgba(255, 193, 7, 0.2)',
                    bordercolor: '#ffc107',
                    borderwidth: 1
                });
            }
        }
        
        Plotly.react(containerId, traces, layout);
    }
    
    // Create result heatmap
    function createResultHeatmap(data, containerId) {
        const batch = data.length;
        const n = data[0].length;
        
        const trace = {
            z: data,
            type: 'heatmap',
            colorscale: 'Plasma',
            showscale: true,
            colorbar: { title: 'Result Value' },
            text: data.map(row => row.map(val => val.toFixed(2))),
            texttemplate: '%{text}',
            textfont: { size: 12, color: 'white' }
        };
        
        const layout = {
            margin: { l: 60, r: 20, t: 20, b: 40 },
            xaxis: { 
                title: 'Query Points',
                tickmode: 'array',
                tickvals: Array.from({length: n}, (_, i) => i),
                ticktext: Array.from({length: n}, (_, i) => `n${i}`)
            },
            yaxis: { 
                title: 'Batch',
                tickmode: 'array',
                tickvals: Array.from({length: batch}, (_, i) => i),
                ticktext: Array.from({length: batch}, (_, i) => `B${i}`)
            },
            height: 250
        };
        
        Plotly.react(containerId, [trace], layout);
    }
    
    // Update flow visualization
    function updateFlowVisualization() {
        const container = document.getElementById('einsum-flowContainer');
        const batch = parseInt(document.getElementById('einsum-batchSlider').value);
        const p = parseInt(document.getElementById('einsum-pSlider').value);
        const n = parseInt(document.getElementById('einsum-nSlider').value);
        
        function formatMatrix(matrix, precision = 1) {
            return matrix.map(row => 
                '[' + row.map(val => val.toFixed(precision)).join(', ') + ']'
            ).join('\n');
        }
        
        container.innerHTML = `
            <div class="einsum-tensor-box branch">
                <div class="einsum-tensor-label">Branch [${batch}, ${p}]</div>
                <pre>${formatMatrix(branchData)}</pre>
            </div>
            
            <div class="einsum-arrow">⊗</div>
            
            <div class="einsum-tensor-box trunk">
                <div class="einsum-tensor-label">Trunk [${batch}, ${n}, ${p}]</div>
                <div style="font-size: 10px;">Batch 0:<br>${formatMatrix(trunkData[0])}</div>
                ${batch > 1 ? `<div style="font-size: 10px; margin-top: 5px;">Batch 1:<br>${formatMatrix(trunkData[1])}</div>` : ''}
                ${batch > 2 ? `<div style="font-size: 10px; margin-top: 5px;">Batch 2:<br>${formatMatrix(trunkData[2])}</div>` : ''}
            </div>
            
            <div class="einsum-arrow">→</div>
            
            <div class="einsum-tensor-box result">
                <div class="einsum-tensor-label">Result [${batch}, ${n}]</div>
                <pre>${formatMatrix(resultData)}</pre>
            </div>
        `;
    }
    
    // Update computation steps
    function updateComputationSteps() {
        const container = document.getElementById('einsum-computationSteps');
        const batch = parseInt(document.getElementById('einsum-batchSlider').value);
        const n = parseInt(document.getElementById('einsum-nSlider').value);
        const p = parseInt(document.getElementById('einsum-pSlider').value);
        
        let stepsHTML = '';
        let stepIndex = 0;
        
        for (let b = 0; b < batch; b++) {
            for (let i = 0; i < n; i++) {
                const dotProduct = branchData[b].map((val, j) => 
                    `${val.toFixed(1)}×${trunkData[b][i][j].toFixed(1)}`
                ).join(' + ');
                
                const isActive = currentComputationStep === stepIndex;
                
                stepsHTML += `
                    <div class="einsum-step-box ${isActive ? 'active' : ''}">
                        <div class="einsum-step-title">Batch ${b}, Query Point ${i}</div>
                        <div class="einsum-step-content">
                            ${dotProduct}<br>
                            = ${resultData[b][i].toFixed(2)}
                        </div>
                    </div>
                `;
                stepIndex++;
            }
        }
        
        container.innerHTML = stepsHTML;
    }
    
    // Animate computation
    function animateComputation() {
        if (animationActive) return;
        
        animationActive = true;
        const button = document.getElementById('einsum-animateButton');
        button.textContent = 'Animating...';
        button.disabled = true;
        
        const batch = parseInt(document.getElementById('einsum-batchSlider').value);
        const n = parseInt(document.getElementById('einsum-nSlider').value);
        const totalSteps = batch * n;
        
        currentComputationStep = 0;
        
        const animateStep = () => {
            updateComputationSteps();
            
            const b = Math.floor(currentComputationStep / n);
            const i = currentComputationStep % n;
            
            document.getElementById('einsum-statusMessage').textContent = 
                `Computing dot product for Batch ${b}, Query Point ${i}: ${resultData[b][i].toFixed(2)}`;
            
            currentComputationStep++;
            
            if (currentComputationStep < totalSteps) {
                setTimeout(animateStep, 1000);
            } else {
                currentComputationStep = -1;
                document.getElementById('einsum-statusMessage').textContent = 'Animation complete!';
                updateComputationSteps();
                
                setTimeout(() => {
                    animationActive = false;
                    button.textContent = 'Animate Computation';
                    button.disabled = false;
                    document.getElementById('einsum-statusMessage').textContent = '';
                }, 2000);
            }
        };
        
        animateStep();
    }
    
    // Main update function
    function updateVisualization() {
        const batch = parseInt(document.getElementById('einsum-batchSlider').value);
        const p = parseInt(document.getElementById('einsum-pSlider').value);
        const n = parseInt(document.getElementById('einsum-nSlider').value);
        
        // Update value displays
        document.getElementById('einsum-batchValue').textContent = batch;
        document.getElementById('einsum-pValue').textContent = p;
        document.getElementById('einsum-nValue').textContent = n;
        
        // Generate data
        branchData = generateBranchData(batch, p);
        trunkData = generateTrunkData(batch, n, p);
        resultData = computeEinsum(branchData, trunkData);
        
        // Update all visualizations
        createBranchHeatmap(branchData, 'einsum-plotBranch');
        createTrunkVisualization(trunkData, 'einsum-plotTrunk');
        createComputationVisualization(branchData, trunkData, resultData, 'einsum-plotComputation');
        createResultHeatmap(resultData, 'einsum-plotResult');
        
        // Update flow and computation steps
        updateFlowVisualization();
        updateComputationSteps();
    }
    
    // Toggle equations
    function toggleEquations() {
        const content = document.getElementById('einsum-equationsContent');
        const toggle = document.getElementById('einsum-equationsToggle');
        
        if (content.classList.contains('show')) {
            content.classList.remove('show');
            toggle.classList.remove('expanded');
        } else {
            content.classList.add('show');
            toggle.classList.add('expanded');
        }
    }
    
    // Event listeners
    document.getElementById('einsum-batchSlider').addEventListener('input', updateVisualization);
    document.getElementById('einsum-pSlider').addEventListener('input', updateVisualization);
    document.getElementById('einsum-nSlider').addEventListener('input', updateVisualization);
    document.getElementById('einsum-randomizeButton').addEventListener('click', updateVisualization);
    document.getElementById('einsum-animateButton').addEventListener('click', animateComputation);
    document.getElementById('einsum-equationsHeader').addEventListener('click', toggleEquations);
    
    // Handle window resize
    window.addEventListener('resize', () => {
        setTimeout(() => {
            const plots = ['einsum-plotBranch', 'einsum-plotTrunk', 'einsum-plotComputation', 'einsum-plotResult'];
            plots.forEach(plotId => {
                const element = document.getElementById(plotId);
                if (element && element.parentElement) {
                    Plotly.relayout(plotId, { 'width': element.parentElement.clientWidth - 20 });
                }
            });
        }, 100);
    });
    
    // Initialize
    updateVisualization();
})();
</script>

</body>
</html>