<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Electric Field Concentration Around Elliptical Conductor</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/plotly.js/2.26.0/plotly.min.js"></script>
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
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
        }
        h1 {
            text-align: center;
            color: #2c3e50;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
        }
        .controls {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
            padding: 20px;
            background: rgba(52, 152, 219, 0.1);
            border-radius: 15px;
        }
        .control-group {
            display: flex;
            flex-direction: column;
        }
        label {
            font-weight: 600;
            margin-bottom: 8px;
            color: #34495e;
        }
        input[type="range"] {
            width: 100%;
            margin: 10px 0;
            appearance: none;
            height: 8px;
            border-radius: 4px;
            background: #ddd;
            outline: none;
        }
        input[type="range"]::-webkit-slider-thumb {
            appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #3498db;
            cursor: pointer;
            box-shadow: 0 2px 6px rgba(0,0,0,0.2);
        }
        .value-display {
            font-weight: bold;
            color: #2980b9;
            text-align: center;
            margin-top: 5px;
        }
        .plot-container {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 20px;
            margin-bottom: 30px;
        }
        .plot-box {
            background: white;
            border-radius: 15px;
            padding: 15px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }
        .results-panel {
            background: rgba(46, 204, 113, 0.1);
            border-radius: 15px;
            padding: 20px;
            margin-top: 20px;
        }
        .measurement-point {
            display: inline-block;
            margin: 10px;
            padding: 15px;
            background: white;
            border-radius: 10px;
            box-shadow: 0 3px 10px rgba(0,0,0,0.1);
            min-width: 200px;
        }
        .point-sharp { border-left: 5px solid #e74c3c; }
        .point-middle { border-left: 5px solid #f39c12; }
        .point-flat { border-left: 5px solid #3498db; }
        .field-value {
            font-size: 1.2em;
            font-weight: bold;
            color: #2c3e50;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>⚡ Electric Field Concentration Simulation</h1>
        
        <div class="controls">
            <div class="control-group">
                <label for="charge">Total Charge (nC):</label>
                <input type="range" id="charge" min="100" max="1000" value="400" step="50">
                <div class="value-display" id="chargeValue">400 nC</div>
            </div>
            
            <div class="control-group">
                <label for="semiMajor">Semi-major Axis (cm):</label>
                <input type="range" id="semiMajor" min="8" max="15" value="10" step="0.5">
                <div class="value-display" id="semiMajorValue">10.0 cm</div>
            </div>
            
            <div class="control-group">
                <label for="semiMinor">Semi-minor Axis (cm):</label>
                <input type="range" id="semiMinor" min="1" max="4" value="2" step="0.1">
                <div class="value-display" id="semiMinorValue">2.0 cm</div>
            </div>
            
            <div class="control-group">
                <label for="resolution">Field Resolution:</label>
                <input type="range" id="resolution" min="20" max="60" value="40" step="5">
                <div class="value-display" id="resolutionValue">40 points</div>
            </div>
        </div>
        
        <div class="plot-container">
            <div class="plot-box">
                <div id="fieldPlot"></div>
            </div>
            <div class="plot-box">
                <div id="contourPlot"></div>
            </div>
        </div>
        
        <div class="plot-container">
            <div class="plot-box">
                <div id="comparisonPlot"></div>
            </div>
            <div class="plot-box">
                <div id="distancePlot"></div>
            </div>
        </div>
        
        <div class="results-panel">
            <h3>🔬 Measurement Results</h3>
            <div id="measurementResults"></div>
        </div>
    </div>

    <script>
        class ElectricFieldSimulation {
            constructor() {
                this.epsilon0 = 8.854e-12; // F/m
                this.updateSimulation();
                this.setupEventListeners();
            }
            
            setupEventListeners() {
                const controls = ['charge', 'semiMajor', 'semiMinor', 'resolution'];
                controls.forEach(id => {
                    const element = document.getElementById(id);
                    element.addEventListener('input', () => {
                        this.updateDisplayValues();
                        this.updateSimulation();
                    });
                });
                this.updateDisplayValues();
            }
            
            updateDisplayValues() {
                document.getElementById('chargeValue').textContent = 
                    document.getElementById('charge').value + ' nC';
                document.getElementById('semiMajorValue').textContent = 
                    document.getElementById('semiMajor').value + ' cm';
                document.getElementById('semiMinorValue').textContent = 
                    document.getElementById('semiMinor').value + ' cm';
                document.getElementById('resolutionValue').textContent = 
                    document.getElementById('resolution').value + ' points';
            }
            
            getParameters() {
                return {
                    totalCharge: parseFloat(document.getElementById('charge').value) * 1e-9,
                    a: parseFloat(document.getElementById('semiMajor').value) * 0.01,
                    b: parseFloat(document.getElementById('semiMinor').value) * 0.01,
                    resolution: parseInt(document.getElementById('resolution').value)
                };
            }
            
            calculateElectricField(x, y, params) {
                const {totalCharge, a, b} = params;
                
                const r = Math.sqrt(x*x + y*y);
                if (r === 0) return {Ex: 0, Ey: 0, magnitude: 0};
                
                const theta = Math.atan2(y, x);
                
                // Distance to ellipse surface
                const rSurface = (a * b) / Math.sqrt(
                    Math.pow(b * Math.cos(theta), 2) + Math.pow(a * Math.sin(theta), 2)
                );
                
                const distanceFromSurface = r - rSurface;
                
                if (distanceFromSurface <= 0) {
                    return {Ex: 0, Ey: 0, magnitude: 0};
                }
                
                // Curvature radius calculation
                const curvatureRadius = Math.pow(a * b, 2) / 
                    Math.pow(Math.pow(a * Math.sin(theta), 2) + Math.pow(b * Math.cos(theta), 2), 1.5);
                
                // Surface charge density (enhanced at sharp points)
                const baseDensity = totalCharge / (Math.PI * a * b * 4);
                const enhancementFactor = 0.02 / curvatureRadius;
                const localChargeDensity = baseDensity * enhancementFactor;
                
                // Electric field magnitude
                const EMagnitude = localChargeDensity / (this.epsilon0 * Math.pow(distanceFromSurface, 2));
                
                // Field components
                const Ex = EMagnitude * x / r;
                const Ey = EMagnitude * y / r;
                
                return {
                    Ex: Ex,
                    Ey: Ey,
                    magnitude: EMagnitude
                };
            }
            
            generateFieldGrid(params) {
                const {resolution, a} = params;
                const xRange = [-a * 1.5, a * 2];
                const yRange = [-a, a];
                
                const x = [];
                const y = [];
                const Ex = [];
                const Ey = [];
                const magnitude = [];
                
                for (let i = 0; i < resolution; i++) {
                    const xi = xRange[0] + (xRange[1] - xRange[0]) * i / (resolution - 1);
                    const xRow = [];
                    const yRow = [];
                    const ExRow = [];
                    const EyRow = [];
                    const magRow = [];
                    
                    for (let j = 0; j < resolution; j++) {
                        const yj = yRange[0] + (yRange[1] - yRange[0]) * j / (resolution - 1);
                        const field = this.calculateElectricField(xi, yj, params);
                        
                        xRow.push(xi);
                        yRow.push(yj);
                        ExRow.push(field.Ex);
                        EyRow.push(field.Ey);
                        magRow.push(field.magnitude);
                    }
                    
                    x.push(xRow);
                    y.push(yRow);
                    Ex.push(ExRow);
                    Ey.push(EyRow);
                    magnitude.push(magRow);
                }
                
                return {x, y, Ex, Ey, magnitude};
            }
            
            getMeasurementPoints(params) {
                const {a, b} = params;
                return {
                    sharp_tip: {x: a * 1.2, y: 0, curvatureRadius: 0.005, color: '#e74c3c'},
                    middle: {x: a * 0.8, y: b * 0.75, curvatureRadius: 0.02, color: '#f39c12'},
                    flat_side: {x: 0, y: b * 1.25, curvatureRadius: 0.1, color: '#3498db'}
                };
            }
            
            updateSimulation() {
                const params = this.getParameters();
                const fieldGrid = this.generateFieldGrid(params);
                const measurementPoints = this.getMeasurementPoints(params);
                
                this.plotVectorField(fieldGrid, params, measurementPoints);
                this.plotContours(fieldGrid, params, measurementPoints);
                this.plotComparison(measurementPoints, params);
                this.plotDistanceEffect(measurementPoints, params);
                this.updateResults(measurementPoints, params);
            }
            
            plotVectorField(fieldGrid, params, measurementPoints) {
                const {x, y, Ex, Ey, magnitude} = fieldGrid;
                const {a, b} = params;
                
                // Sample every nth point for vectors
                const skip = Math.max(1, Math.floor(params.resolution / 15));
                const xSample = [];
                const ySample = [];
                const ExSample = [];
                const EySample = [];
                
                for (let i = 0; i < x.length; i += skip) {
                    for (let j = 0; j < x[i].length; j += skip) {
                        xSample.push(x[i][j]);
                        ySample.push(y[i][j]);
                        ExSample.push(Ex[i][j]);
                        EySample.push(Ey[i][j]);
                    }
                }
                
                // Create ellipse boundary
                const theta = [];
                const ellipseX = [];
                const ellipseY = [];
                for (let i = 0; i <= 100; i++) {
                    const t = 2 * Math.PI * i / 100;
                    theta.push(t);
                    ellipseX.push(a * Math.cos(t));
                    ellipseY.push(b * Math.sin(t));
                }
                
                const traces = [
                    {
                        type: 'scatter',
                        mode: 'lines',
                        x: ellipseX,
                        y: ellipseY,
                        fill: 'toself',
                        fillcolor: 'rgba(128,128,128,0.7)',
                        line: {color: 'gray', width: 2},
                        name: 'Conductor',
                        hoverinfo: 'none'
                    }
                ];
                
                // Add measurement points
                Object.entries(measurementPoints).forEach(([name, point]) => {
                    const field = this.calculateElectricField(point.x, point.y, params);
                    traces.push({
                        type: 'scatter',
                        mode: 'markers',
                        x: [point.x],
                        y: [point.y],
                        marker: {
                            size: 12,
                            color: point.color,
                            line: {color: 'white', width: 2}
                        },
                        name: `${name}: ${field.magnitude.toExponential(2)} V/m`,
                        hovertemplate: `<b>${name}</b><br>E = ${field.magnitude.toExponential(2)} V/m<extra></extra>`
                    });
                });
                
                const layout = {
                    title: 'Electric Field Vectors',
                    xaxis: {title: 'Distance (m)', scaleanchor: 'y'},
                    yaxis: {title: 'Distance (m)'},
                    showlegend: true,
                    margin: {t: 40, b: 40, l: 40, r: 40},
                    annotations: []
                };
                
                // Add vector annotations
                for (let i = 0; i < Math.min(xSample.length, 100); i++) {
                    if (Math.sqrt(ExSample[i]*ExSample[i] + EySample[i]*EySample[i]) > 1e6) {
                        const norm = Math.sqrt(ExSample[i]*ExSample[i] + EySample[i]*EySample[i]);
                        const scale = 0.02;
                        layout.annotations.push({
                            x: xSample[i],
                            y: ySample[i],
                            ax: xSample[i] + ExSample[i] / norm * scale,
                            ay: ySample[i] + EySample[i] / norm * scale,
                            arrowhead: 2,
                            arrowsize: 1,
                            arrowwidth: 1,
                            arrowcolor: 'blue',
                            axref: 'x',
                            ayref: 'y'
                        });
                    }
                }
                
                Plotly.newPlot('fieldPlot', traces, layout);
            }
            
            plotContours(fieldGrid, params, measurementPoints) {
                const {x, y, magnitude} = fieldGrid;
                const {a, b} = params;
                
                // Create ellipse boundary
                const ellipseX = [];
                const ellipseY = [];
                for (let i = 0; i <= 100; i++) {
                    const t = 2 * Math.PI * i / 100;
                    ellipseX.push(a * Math.cos(t));
                    ellipseY.push(b * Math.sin(t));
                }
                
                const traces = [
                    {
                        type: 'contour',
                        x: x[0],
                        y: y.map(row => row[0]),
                        z: magnitude,
                        colorscale: 'Plasma',
                        contours: {
                            coloring: 'fill'
                        },
                        colorbar: {
                            title: 'E (V/m)'
                        },
                        hovertemplate: 'E = %{z:.2e} V/m<extra></extra>'
                    },
                    {
                        type: 'scatter',
                        mode: 'lines',
                        x: ellipseX,
                        y: ellipseY,
                        line: {color: 'white', width: 3},
                        name: 'Conductor',
                        hoverinfo: 'none'
                    }
                ];
                
                // Add measurement points
                Object.entries(measurementPoints).forEach(([name, point]) => {
                    traces.push({
                        type: 'scatter',
                        mode: 'markers+text',
                        x: [point.x],
                        y: [point.y],
                        marker: {
                            size: 10,
                            color: 'white',
                            line: {color: point.color, width: 3}
                        },
                        text: [name],
                        textposition: 'top center',
                        textfont: {color: 'white', size: 10},
                        name: name,
                        hoverinfo: 'none'
                    });
                });
                
                const layout = {
                    title: 'Electric Field Strength Contours',
                    xaxis: {title: 'Distance (m)', scaleanchor: 'y'},
                    yaxis: {title: 'Distance (m)'},
                    showlegend: false,
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('contourPlot', traces, layout);
            }
            
            plotComparison(measurementPoints, params) {
                const names = Object.keys(measurementPoints);
                const fieldStrengths = [];
                const colors = [];
                
                names.forEach(name => {
                    const point = measurementPoints[name];
                    const field = this.calculateElectricField(point.x, point.y, params);
                    fieldStrengths.push(field.magnitude);
                    colors.push(point.color);
                });
                
                const trace = {
                    type: 'bar',
                    x: names,
                    y: fieldStrengths,
                    marker: {color: colors, opacity: 0.8},
                    text: fieldStrengths.map(f => f.toExponential(2)),
                    textposition: 'outside',
                    hovertemplate: '%{x}: %{y:.2e} V/m<extra></extra>'
                };
                
                const layout = {
                    title: 'Field Strength Comparison',
                    xaxis: {title: 'Measurement Point'},
                    yaxis: {title: 'Electric Field (V/m)', type: 'log'},
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('comparisonPlot', [trace], layout);
            }
            
            plotDistanceEffect(measurementPoints, params) {
                const distances = [];
                for (let i = 0.01; i <= 0.2; i += 0.01) {
                    distances.push(i);
                }
                
                const traces = [];
                
                Object.entries(measurementPoints).forEach(([name, basePoint]) => {
                    const fields = [];
                    distances.forEach(d => {
                        const testX = basePoint.x + d;
                        const testY = basePoint.y;
                        const field = this.calculateElectricField(testX, testY, params);
                        fields.push(field.magnitude);
                    });
                    
                    traces.push({
                        type: 'scatter',
                        mode: 'lines+markers',
                        x: distances.map(d => d * 100), // Convert to cm
                        y: fields,
                        name: name,
                        line: {color: basePoint.color, width: 3},
                        marker: {color: basePoint.color, size: 6},
                        hovertemplate: '%{x} cm: %{y:.2e} V/m<extra></extra>'
                    });
                });
                
                // Add 1/r² reference line
                if (distances.length > 0) {
                    const sharpPoint = measurementPoints.sharp_tip;
                    const refField = this.calculateElectricField(
                        sharpPoint.x + distances[0], sharpPoint.y, params
                    );
                    const reference = distances.map(d => 
                        refField.magnitude * Math.pow(distances[0] / d, 2)
                    );
                    
                    traces.push({
                        type: 'scatter',
                        mode: 'lines',
                        x: distances.map(d => d * 100),
                        y: reference,
                        name: '1/r² reference',
                        line: {color: 'black', dash: 'dash', width: 2},
                        hovertemplate: '1/r² ref: %{y:.2e} V/m<extra></extra>'
                    });
                }
                
                const layout = {
                    title: 'Electric Field vs Distance',
                    xaxis: {title: 'Distance from surface (cm)'},
                    yaxis: {title: 'Electric Field (V/m)', type: 'log'},
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('distancePlot', traces, layout);
            }
            
            updateResults(measurementPoints, params) {
                let html = '';
                
                Object.entries(measurementPoints).forEach(([name, point]) => {
                    const field = this.calculateElectricField(point.x, point.y, params);
                    const className = `measurement-point point-${name.split('_')[0]}`;
                    
                    html += `
                        <div class="${className}">
                            <h4>${name.replace('_', ' ').toUpperCase()}</h4>
                            <p><strong>Position:</strong> (${(point.x*100).toFixed(1)}, ${(point.y*100).toFixed(1)}) cm</p>
                            <p><strong>Curvature Radius:</strong> ${(point.curvatureRadius*100).toFixed(1)} cm</p>
                            <p><strong>Electric Field:</strong> <span class="field-value">${field.magnitude.toExponential(2)} V/m</span></p>
                        </div>
                    `;
                });
                
                document.getElementById('measurementResults').innerHTML = html;
            }
        }
        
        // Initialize simulation when page loads
        window.addEventListener('load', () => {
            new ElectricFieldSimulation();
        });
    </script>
</body>
</html>
