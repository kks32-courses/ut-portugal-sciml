# DeepONet Derivative

<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>DeepONet: Polynomial Derivative Decomposition</title>
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>

<style>
    #deeponet-container { 
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif; 
        margin: 10px; 
        background-color: #f9f9f9; 
        padding: 15px;
        border: 1px solid #ccc;
        border-radius: 8px;
    }
    .don-container { 
        display: grid; 
        grid-template-columns: repeat(auto-fit, minmax(400px, 1fr)); 
        gap: 20px; 
    }
    .don-plot-container { 
        border: 1px solid #ddd; 
        border-radius: 8px; 
        background-color: #fff; 
        box-shadow: 0 2px 5px rgba(0,0,0,0.1); 
        padding: 10px;
    }
    .don-controls { 
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
    .don-input-group {
        display: flex;
        flex-direction: column;
        align-items: center;
        min-width: 120px;
    }
    .don-input-group label { 
        font-weight: bold; 
        margin-bottom: 8px; 
        color: #333; 
        text-align: center;
        font-size: 14px;
    }
    .don-input-group input[type=number] { 
        width: 80px;
        padding: 8px;
        border: 2px solid #ddd;
        border-radius: 4px;
        text-align: center;
        font-size: 16px;
        font-weight: bold;
    }
    .don-input-group input[type=number]:focus {
        border-color: #007bff;
        outline: none;
    }
    .don-button { 
        padding: 12px 24px; 
        font-size: 16px; 
        font-weight: bold; 
        color: white; 
        background-color: #28a745; 
        border: none; 
        border-radius: 5px; 
        cursor: pointer; 
        transition: background-color 0.2s; 
    }
    .don-button:hover { 
        background-color: #218838; 
    }
    .don-button.secondary {
        background-color: #6c757d;
    }
    .don-button.secondary:hover {
        background-color: #545b62;
    }
    .don-plot-title { 
        text-align: center; 
        font-size: 16px; 
        font-weight: bold; 
        padding-top: 15px; 
        color: #444; 
    }
    
    .don-equations-section {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        margin-bottom: 20px;
    }
    
    .don-equations-header {
        padding: 15px;
        cursor: pointer;
        display: flex;
        align-items: center;
        background-color: #f8f9fa;
        border-radius: 8px 8px 0 0;
        transition: background-color 0.2s;
        user-select: none;
    }
    
    .don-equations-header:hover {
        background-color: #e9ecef;
    }
    
    .don-equations-toggle {
        width: 0;
        height: 0;
        border-left: 8px solid #495057;
        border-top: 6px solid transparent;
        border-bottom: 6px solid transparent;
        margin-right: 12px;
        transition: transform 0.3s ease;
    }
    
    .don-equations-toggle.expanded {
        transform: rotate(90deg);
    }
    
    .don-equations-content {
        padding: 20px;
        display: none;
    }
    
    .don-equations-content.show {
        display: block;
    }
    
    .don-equation {
        background-color: #f8f9fa;
        border: 1px solid #e9ecef;
        border-radius: 6px;
        padding: 15px;
        margin: 10px 0;
        font-family: 'Courier New', monospace;
        font-size: 16px;
    }
    
    .don-equation-title {
        font-weight: bold;
        color: #495057;
        margin-bottom: 8px;
        font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
    }

    .don-network-diagram {
        background-color: #f8f9fa;
        border: 1px solid #e9ecef;
        border-radius: 6px;
        padding: 20px;
        margin: 10px 0;
        text-align: center;
        font-family: 'Courier New', monospace;
        font-size: 14px;
    }

    .don-results-section {
        grid-column: 1 / -1;
        background-color: #fff;
        border: 1px solid #ddd;
        border-radius: 8px;
        margin-bottom: 20px;
        padding: 20px;
    }

    .don-decomposition {
        background-color: #e7f3ff;
        border: 1px solid #b3d9ff;
        border-radius: 6px;
        padding: 15px;
        margin: 10px 0;
        font-family: 'Courier New', monospace;
        font-size: 16px;
        text-align: center;
    }

    .don-weight-display {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
        gap: 15px;
        margin: 15px 0;
    }

    .don-weight-item {
        background-color: #f8f9fa;
        border: 1px solid #e9ecef;
        border-radius: 4px;
        padding: 12px;
        text-align: center;
    }

    .don-weight-label {
        font-weight: bold;
        color: #495057;
        margin-bottom: 5px;
        font-size: 14px;
    }

    .don-weight-value {
        font-family: 'Courier New', monospace;
        font-size: 18px;
        color: #007bff;
        font-weight: bold;
    }

    .polynomial-display {
        background-color: #d4edda;
        border: 1px solid #c3e6cb;
        border-radius: 4px;
        padding: 15px;
        margin: 10px 0;
        text-align: center;
        font-family: 'Courier New', monospace;
        font-size: 18px;
        font-weight: bold;
        color: #155724;
    }

    .derivative-display {
        background-color: #cce5ff;
        border: 1px solid #99ccff;
        border-radius: 4px;
        padding: 15px;
        margin: 10px 0;
        text-align: center;
        font-family: 'Courier New', monospace;
        font-size: 18px;
        font-weight: bold;
        color: #004085;
    }

    .don-branch { color: #28a745; font-weight: bold; }
    .don-trunk { color: #dc3545; font-weight: bold; }
    .don-output { color: #007bff; font-weight: bold; }
</style>
</head>
<body>

<div id="deeponet-container">
    <h2>DeepONet Concept: Derivative as Linear Combination of Basis Functions</h2>
    <p>Enter a cubic polynomial and see how its derivative (always quadratic) can be expressed as a linear combination of simple basis functions: constant, linear, and quadratic terms.</p>

    <!-- Equations Section -->
    <div class="don-equations-section">
        <div class="don-equations-header" id="don-equationsHeader">
            <div class="don-equations-toggle" id="don-equationsToggle"></div>
            <h3 style="margin: 0; color: #495057;">DeepONet Concept: Basis Function Decomposition</h3>
        </div>
        
        <div class="don-equations-content" id="don-equationsContent">
            <div class="don-network-diagram">
                <div style="margin-bottom: 15px; font-size: 16px; font-weight: bold;">Operator Learning: Polynomial → Derivative</div>
                <div>Input: f(x) = ax³ + bx² + cx + d</div>
                <div style="margin: 5px 0;">↓ (Differentiation Operator)</div>
                <div>Output: f'(x) = 3ax² + 2bx + c</div>
                <div style="margin: 10px 0; color: #007bff; font-weight: bold;">DeepONet learns this operator!</div>
            </div>
            
            <div class="don-equation">
                <div class="don-equation-title">1. Analytical Derivative:</div>
                <div><strong>f'(x) = 3ax² + 2bx + c</strong></div>
                <div style="margin-top: 8px; font-size: 14px; color: #6c757d;">
                    Direct differentiation of cubic polynomial → quadratic result
                </div>
            </div>
            
            <div class="don-equation">
                <div class="don-equation-title">2. Basis Function Decomposition:</div>
                <div><strong>f'(x) = w₁ × 1 + w₂ × x + w₃ × x²</strong></div>
                <div style="margin-top: 8px; font-size: 14px; color: #6c757d;">
                    Any quadratic can be written as combination of: constant, linear, quadratic basis
                </div>
            </div>
            
            <div class="don-equation">
                <div class="don-equation-title">3. DeepONet Mapping:</div>
                <div><strong>Branch Network: [a,b,c,d] → [w₁, w₂, w₃]</strong></div>
                <div><strong>Trunk Network: x → [1, x, x²]</strong></div>
                <div style="margin-top: 8px; font-size: 14px; color: #6c757d;">
                    Branch learns coefficients, Trunk learns basis functions
                </div>
            </div>
            
            <div class="don-equation">
                <div class="don-equation-title">4. Perfect Match:</div>
                <div><strong>w₁ = c, w₂ = 2b, w₃ = 3a</strong></div>
                <div style="margin-top: 8px; font-size: 14px; color: #6c757d;">
                    For this operator, the mapping is analytical and exact!
                </div>
            </div>
        </div>
    </div>

    <!-- Controls -->
    <div class="don-controls">
        <div class="don-input-group">
            <label for="don-aInput">Coefficient a<br>(x³ term)</label>
            <input type="number" id="don-aInput" value="1.0" step="0.1" min="-5" max="5">
        </div>
        <div class="don-input-group">
            <label for="don-bInput">Coefficient b<br>(x² term)</label>
            <input type="number" id="don-bInput" value="0.5" step="0.1" min="-5" max="5">
        </div>
        <div class="don-input-group">
            <label for="don-cInput">Coefficient c<br>(x term)</label>
            <input type="number" id="don-cInput" value="-0.3" step="0.1" min="-5" max="5">
        </div>
        <div class="don-input-group">
            <label for="don-dInput">Coefficient d<br>(constant)</label>
            <input type="number" id="don-dInput" value="0.2" step="0.1" min="-5" max="5">
        </div>
        <button id="don-randomButton" class="don-button secondary">Random</button>
        <button id="don-updateButton" class="don-button">Analyze</button>
    </div>

    <!-- Results Section -->
    <div class="don-results-section">
        <div class="polynomial-display" id="don-polynomialDisplay">
            f(x) = 1.0x³ + 0.5x² - 0.3x + 0.2
        </div>
        
        <div class="derivative-display" id="don-derivativeDisplay">
            f'(x) = 3.0x² + 1.0x - 0.3
        </div>
        
        <div class="don-decomposition" id="don-decompositionDisplay">
            f'(x) = (-0.3) × 1 + (1.0) × x + (3.0) × x²
        </div>
        
        <div style="font-weight: bold; margin-bottom: 10px; color: #495057; text-align: center;">
            DeepONet Basis Weights (Branch Network Output):
        </div>
        <div class="don-weight-display">
            <div class="don-weight-item">
                <div class="don-weight-label">w₁ (Constant)</div>
                <div class="don-weight-value" id="don-w1Value">-0.3</div>
            </div>
            <div class="don-weight-item">
                <div class="don-weight-label">w₂ (Linear)</div>
                <div class="don-weight-value" id="don-w2Value">1.0</div>
            </div>
            <div class="don-weight-item">
                <div class="don-weight-label">w₃ (Quadratic)</div>
                <div class="don-weight-value" id="don-w3Value">3.0</div>
            </div>
        </div>
    </div>

    <div class="don-container">
        <div class="don-plot-container">
            <div class="don-plot-title">Input Polynomial f(x)</div>
            <div id="don-plotInput"></div>
        </div>
        <div class="don-plot-container">
            <div class="don-plot-title">Basis Functions: 1, x, x²</div>
            <div id="don-plotBasis"></div>
        </div>
        <div class="don-plot-container">
            <div class="don-plot-title">Weighted Basis Components</div>
            <div id="don-plotComponents"></div>
        </div>
        <div class="don-plot-container">
            <div class="don-plot-title">Final Result: f'(x) = Σ wᵢ × φᵢ(x)</div>
            <div id="don-plotResult"></div>
        </div>
    </div>
</div>

<script>
    (function() {
        // --- SIMPLE BASIS FUNCTIONS ---
        const basisFunctions = [
            x => 1,        // φ₁(x) = 1 (constant)
            x => x,        // φ₂(x) = x (linear)
            x => x * x     // φ₃(x) = x² (quadratic)
        ];

        const basisNames = ['1', 'x', 'x²'];
        const basisColors = ['#dc3545', '#28a745', '#007bff'];

        // --- CONFIGURATION ---
        const x_range = [-2, 2];
        const num_points = 100;
        let x_values = [];
        for (let i = 0; i <= num_points; i++) {
            x_values.push(x_range[0] + (i / num_points) * (x_range[1] - x_range[0]));
        }

        // --- PLOTTING SETUP ---
        const plotLayout = {
            margin: { l: 50, r: 20, t: 20, b: 40 },
            xaxis: { 
                range: x_range, 
                title: 'x',
                zeroline: true, 
                zerolinewidth: 1, 
                zerolinecolor: '#ddd' 
            },
            yaxis: { 
                title: 'f(x)',
                zeroline: true, 
                zerolinewidth: 1, 
                zerolinecolor: '#ddd' 
            },
            showlegend: true,
            autosize: true,
            legend: { x: 0.02, y: 0.98 }
        };

        // --- UTILITY FUNCTIONS ---
        function getPolynomialCoeffs() {
            const a = parseFloat(document.getElementById('don-aInput').value) || 0;
            const b = parseFloat(document.getElementById('don-bInput').value) || 0;
            const c = parseFloat(document.getElementById('don-cInput').value) || 0;
            const d = parseFloat(document.getElementById('don-dInput').value) || 0;
            return { a, b, c, d };
        }

        function polynomialFunction(x, a, b, c, d) {
            return a * Math.pow(x, 3) + b * Math.pow(x, 2) + c * x + d;
        }

        function polynomialDerivative(x, a, b, c, d) {
            return 3 * a * Math.pow(x, 2) + 2 * b * x + c;
        }

        function formatPolynomial(a, b, c, d, isDerivative = false) {
            const formatCoeff = (coeff, power, isFirst = false) => {
                if (coeff === 0) return '';
                const sign = coeff >= 0 ? (isFirst ? '' : ' + ') : ' - ';
                const absCoeff = Math.abs(coeff);
                const coeffStr = (absCoeff === 1 && power > 0) ? '' : absCoeff.toFixed(1);
                
                if (power === 0) return sign + absCoeff.toFixed(1);
                if (power === 1) return sign + coeffStr + 'x';
                return sign + coeffStr + 'x^' + power;
            };

            let result = isDerivative ? "f'(x) = " : "f(x) = ";
            let terms = [];
            
            if (isDerivative) {
                // For derivative: 3ax² + 2bx + c
                if (3 * a !== 0) terms.push(formatCoeff(3 * a, 2, terms.length === 0));
                if (2 * b !== 0) terms.push(formatCoeff(2 * b, 1, terms.length === 0));
                if (c !== 0) terms.push(formatCoeff(c, 0, terms.length === 0));
            } else {
                // For original: ax³ + bx² + cx + d
                if (a !== 0) terms.push(formatCoeff(a, 3, terms.length === 0));
                if (b !== 0) terms.push(formatCoeff(b, 2, terms.length === 0));
                if (c !== 0) terms.push(formatCoeff(c, 1, terms.length === 0));
                if (d !== 0) terms.push(formatCoeff(d, 0, terms.length === 0));
            }
            
            if (terms.length === 0) return result + '0';
            return result + terms.join('');
        }

        function formatDecomposition(a, b, c) {
            const w1 = c;      // constant term
            const w2 = 2 * b;  // linear term coefficient
            const w3 = 3 * a;  // quadratic term coefficient
            
            const formatTerm = (coeff, basis, isFirst = false) => {
                if (coeff === 0) return '';
                const sign = coeff >= 0 ? (isFirst ? '' : ' + ') : ' - ';
                const absCoeff = Math.abs(coeff);
                return `${sign}(${absCoeff.toFixed(1)}) × ${basis}`;
            };

            let result = "f'(x) = ";
            let terms = [];
            
            if (w1 !== 0) terms.push(formatTerm(w1, '1', terms.length === 0));
            if (w2 !== 0) terms.push(formatTerm(w2, 'x', terms.length === 0));
            if (w3 !== 0) terms.push(formatTerm(w3, 'x²', terms.length === 0));
            
            if (terms.length === 0) return result + '0';
            return result + terms.join('');
        }

        function updateDisplays() {
            const { a, b, c, d } = getPolynomialCoeffs();
            
            // Update polynomial and derivative displays
            document.getElementById('don-polynomialDisplay').textContent = formatPolynomial(a, b, c, d, false);
            document.getElementById('don-derivativeDisplay').textContent = formatPolynomial(a, b, c, d, true);
            document.getElementById('don-decompositionDisplay').textContent = formatDecomposition(a, b, c);
            
            // Update weight displays
            document.getElementById('don-w1Value').textContent = c.toFixed(1);     // constant term
            document.getElementById('don-w2Value').textContent = (2 * b).toFixed(1); // linear term
            document.getElementById('don-w3Value').textContent = (3 * a).toFixed(1); // quadratic term
        }

        function toggleEquations() {
            const content = document.getElementById('don-equationsContent');
            const toggle = document.getElementById('don-equationsToggle');
            
            if (content.classList.contains('show')) {
                content.classList.remove('show');
                toggle.classList.remove('expanded');
            } else {
                content.classList.add('show');
                toggle.classList.add('expanded');
            }
        }

        function randomPolynomial() {
            document.getElementById('don-aInput').value = ((Math.random() - 0.5) * 4).toFixed(1);
            document.getElementById('don-bInput').value = ((Math.random() - 0.5) * 4).toFixed(1);
            document.getElementById('don-cInput').value = ((Math.random() - 0.5) * 4).toFixed(1);
            document.getElementById('don-dInput').value = ((Math.random() - 0.5) * 4).toFixed(1);
            analyzePolynomial();
        }

        // --- PLOTTING FUNCTIONS ---
        function plotInputPolynomial() {
            const { a, b, c, d } = getPolynomialCoeffs();
            const y_values = x_values.map(x => polynomialFunction(x, a, b, c, d));
            
            const trace = {
                x: x_values,
                y: y_values,
                mode: 'lines',
                type: 'scatter',
                line: { color: '#6610f2', width: 3 },
                name: 'f(x)'
            };
            
            const layout = { ...plotLayout };
            const maxY = Math.max(...y_values.map(Math.abs));
            layout.yaxis = { ...plotLayout.yaxis, range: [-maxY * 1.1, maxY * 1.1] };
            
            Plotly.react('don-plotInput', [trace], layout);
        }

        function plotBasisFunctions() {
            const traces = basisFunctions.map((func, i) => {
                const y_values = x_values.map(x => func(x));
                return {
                    x: x_values,
                    y: y_values,
                    mode: 'lines',
                    type: 'scatter',
                    line: { 
                        color: basisColors[i], 
                        width: 3
                    },
                    name: `φ₁${i+1}(x) = ${basisNames[i]}`
                };
            });
            
            const layout = { ...plotLayout };
            layout.yaxis = { ...plotLayout.yaxis, title: 'φᵢ(x)', range: [-4, 4] };
            
            Plotly.react('don-plotBasis', traces, layout);
        }

        function plotWeightedComponents() {
            const { a, b, c } = getPolynomialCoeffs();
            const weights = [c, 2 * b, 3 * a]; // w1, w2, w3
            
            const traces = basisFunctions.map((func, i) => {
                const y_values = x_values.map(x => weights[i] * func(x));
                return {
                    x: x_values,
                    y: y_values,
                    mode: 'lines',
                    type: 'scatter',
                    line: { 
                        color: basisColors[i], 
                        width: 2,
                        dash: 'dash'
                    },
                    name: `w₁${i+1}φ₁${i+1}(x) = ${weights[i].toFixed(1)} × ${basisNames[i]}`
                };
            });
            
            const layout = { ...plotLayout };
            layout.yaxis = { ...plotLayout.yaxis, title: 'wᵢφᵢ(x)' };
            
            Plotly.react('don-plotComponents', traces, layout);
        }

        function plotFinalResult() {
            const { a, b, c, d } = getPolynomialCoeffs();
            const weights = [c, 2 * b, 3 * a]; // w1, w2, w3
            
            // Analytical derivative
            const analyticalDerivative = x_values.map(x => polynomialDerivative(x, a, b, c, d));
            const analyticalTrace = {
                x: x_values,
                y: analyticalDerivative,
                mode: 'lines',
                type: 'scatter',
                line: { color: '#fd7e14', width: 4 },
                name: "Analytical f'(x)"
            };
            
            // DeepONet reconstruction (should be identical)
            const deeponetReconstruction = x_values.map(x => {
                return weights.reduce((sum, weight, i) => {
                    return sum + weight * basisFunctions[i](x);
                }, 0);
            });
            
            const deeponetTrace = {
                x: x_values,
                y: deeponetReconstruction,
                mode: 'lines',
                type: 'scatter',
                line: { color: '#007bff', width: 3, dash: 'dot' },
                name: 'DeepONet: Σ wᵢφᵢ(x)'
            };
            
            // Individual components (faded)
            const componentTraces = basisFunctions.map((func, i) => {
                const y_values = x_values.map(x => weights[i] * func(x));
                return {
                    x: x_values,
                    y: y_values,
                    mode: 'lines',
                    type: 'scatter',
                    line: { 
                        color: basisColors[i], 
                        width: 1,
                        dash: 'dot'
                    },
                    opacity: 0.3,
                    showlegend: false
                };
            });
            
            const allTraces = [...componentTraces, analyticalTrace, deeponetTrace];
            
            const layout = { ...plotLayout };
            layout.yaxis = { ...plotLayout.yaxis, title: "f'(x)" };
            
            Plotly.react('don-plotResult', allTraces, layout);
        }

        function analyzePolynomial() {
            updateDisplays();
            plotInputPolynomial();
            plotBasisFunctions();
            plotWeightedComponents();
            plotFinalResult();
        }

        // --- EVENT LISTENERS ---
        function setupEventListeners() {
            ['don-aInput', 'don-bInput', 'don-cInput', 'don-dInput'].forEach(id => {
                document.getElementById(id).addEventListener('input', analyzePolynomial);
            });
            
            document.getElementById('don-randomButton').addEventListener('click', randomPolynomial);
            document.getElementById('don-updateButton').addEventListener('click', analyzePolynomial);
            document.getElementById('don-equationsHeader').addEventListener('click', toggleEquations);
            
            window.addEventListener('resize', () => {
                const plots = ['don-plotInput', 'don-plotBasis', 'don-plotComponents', 'don-plotResult'];
                plots.forEach(plotId => {
                    Plotly.relayout(plotId, { 
                        'width': document.getElementById(plotId).parentElement.clientWidth - 20 
                    });
                });
            });
        }

        // --- INITIALIZATION ---
        function init() {
            setupEventListeners();
            analyzePolynomial();
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