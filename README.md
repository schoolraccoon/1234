<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Surface Charge Distribution on Elliptical Conductor</title>
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
        <h1>⚡ Surface Charge Distribution on Elliptical Conductor</h1>
        
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
                <div id="chargeDensityPlot"></div>
            </div>
            <div class="plot-box">
                <div id="comparisonPlot"></div>
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
            
            // 주어진 각도에서 타원의 곡률 반지름 계산
            getCurvatureRadius(theta, a, b) {
                const numerator = Math.pow(a * b, 2);
                const denominator = Math.pow(
                    Math.pow(a * Math.sin(theta), 2) + Math.pow(b * Math.cos(theta), 2), 
                    1.5
                );
                return numerator / denominator;
            }
            
            // 표면 전하 밀도 계산 (곡률에 반비례)
            getSurfaceChargeDensity(theta, params) {
                const {totalCharge, a, b} = params;
                const curvatureRadius = this.getCurvatureRadius(theta, a, b);
                
                // 기본 전하 밀도
                const baseDensity = totalCharge / (2 * Math.PI * Math.sqrt(a * b));
                
                // 곡률 효과: 곡률 반지름에 반비례
                const curvatureFactor = 0.05 / curvatureRadius;
                
                return baseDensity * curvatureFactor;
            }
            
            // 타원 표면의 점에서 전기장 계산
            getElectricFieldAtSurface(theta, params) {
                const chargeDensity = this.getSurfaceChargeDensity(theta, params);
                // 도체 표면에서 전기장: E = σ/ε₀
                return Math.abs(chargeDensity) / this.epsilon0;
            }
            
            // 타원 외부 점에서 전기장 계산
            calculateElectricField(x, y, params) {
                const {a, b} = params;
                const r = Math.sqrt(x*x + y*y);
                
                if (r === 0) return {Ex: 0, Ey: 0, magnitude: 0};
                
                const theta = Math.atan2(y, x);
                
                // 타원 표면까지의 거리
                const rSurface = (a * b) / Math.sqrt(
                    Math.pow(b * Math.cos(theta), 2) + Math.pow(a * Math.sin(theta), 2)
                );
                
                const distanceFromSurface = Math.max(0.001, r - rSurface);
                
                if (r <= rSurface) {
                    return {Ex: 0, Ey: 0, magnitude: 0};
                }
                
                // 표면에서의 전기장 강도
                const surfaceField = this.getElectricFieldAtSurface(theta, params);
                
                // 거리에 따른 감소 (대략적으로 1/r² 법칙)
                const fieldMagnitude = surfaceField * Math.pow(0.01 / distanceFromSurface, 1.5);
                
                // 전기장 벡터 성분
                const Ex = fieldMagnitude * x / r;
                const Ey = fieldMagnitude * y / r;
                
                return {
                    Ex: Ex,
                    Ey: Ey,
                    magnitude: fieldMagnitude
                };
            }
            
            // 측정 지점 정의
            getMeasurementPoints(params) {
                const {a, b} = params;
                return {
                    sharp_tip: {
                        x: a + 0.005, // 뾰족한 끝에서 5mm 떨어진 지점
                        y: 0,
                        angle: 0,
                        color: '#e74c3c',
                        name: 'Sharp Tip'
                    },
                    middle: {
                        x: a * 0.7 + 0.005,
                        y: b * 0.7 + 0.005,
                        angle: Math.PI/4,
                        color: '#f39c12',
                        name: 'Middle'
                    },
                    flat_side: {
                        x: 0,
                        y: b + 0.005, // 평평한 면에서 5mm 떨어진 지점
                        angle: Math.PI/2,
                        color: '#3498db',
                        name: 'Flat Side'
                    }
                };
            }
            
            updateSimulation() {
                const params = this.getParameters();
                const measurementPoints = this.getMeasurementPoints(params);
                
                this.plotVectorField(params, measurementPoints);
                this.plotContours(params, measurementPoints);
                this.plotChargeDensityDistribution(params);
                this.plotComparison(measurementPoints, params);
                this.updateResults(measurementPoints, params);
            }
            
            plotVectorField(params, measurementPoints) {
                const {a, b, resolution} = params;
                
                // 벡터 필드 그리드 생성
                const xRange = [-a * 0.5, a * 1.5];
                const yRange = [-a * 0.8, a * 0.8];
                const skip = Math.max(1, Math.floor(resolution / 15));
                
                const vectors = [];
                for (let i = 0; i < resolution; i += skip) {
                    for (let j = 0; j < resolution; j += skip) {
                        const x = xRange[0] + (xRange[1] - xRange[0]) * i / (resolution - 1);
                        const y = yRange[0] + (yRange[1] - yRange[0]) * j / (resolution - 1);
                        
                        const field = this.calculateElectricField(x, y, params);
                        if (field.magnitude > 1e5) {
                            vectors.push({x, y, Ex: field.Ex, Ey: field.Ey, mag: field.magnitude});
                        }
                    }
                }
                
                // 타원 그리기
                const ellipseX = [];
                const ellipseY = [];
                for (let i = 0; i <= 100; i++) {
                    const theta = 2 * Math.PI * i / 100;
                    ellipseX.push(a * Math.cos(theta));
                    ellipseY.push(b * Math.sin(theta));
                }
                
                const traces = [
                    {
                        type: 'scatter',
                        mode: 'lines',
                        x: ellipseX,
                        y: ellipseY,
                        fill: 'toself',
                        fillcolor: 'rgba(128,128,128,0.8)',
                        line: {color: 'black', width: 3},
                        name: 'Conductor',
                        hoverinfo: 'none'
                    }
                ];
                
                // 측정 지점 추가
                Object.values(measurementPoints).forEach(point => {
                    const field = this.calculateElectricField(point.x, point.y, params);
                    traces.push({
                        type: 'scatter',
                        mode: 'markers+text',
                        x: [point.x],
                        y: [point.y],
                        marker: {
                            size: 15,
                            color: point.color,
                            line: {color: 'white', width: 2}
                        },
                        text: [`${point.name}: ${field.magnitude.toExponential(1)} V/m`],
                        textposition: 'top center',
                        textfont: {size: 10},
                        name: point.name,
                        hovertemplate: `<b>${point.name}</b><br>E = ${field.magnitude.toExponential(2)} V/m<extra></extra>`
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
                
                // 벡터 화살표 추가
                vectors.slice(0, 50).forEach(v => {
                    const norm = Math.sqrt(v.Ex*v.Ex + v.Ey*v.Ey);
                    if (norm > 0) {
                        const scale = 0.02;
                        layout.annotations.push({
                            x: v.x,
                            y: v.y,
                            ax: v.x + v.Ex / norm * scale,
                            ay: v.y + v.Ey / norm * scale,
                            arrowhead: 2,
                            arrowsize: 1,
                            arrowwidth: 2,
                            arrowcolor: 'blue',
                            axref: 'x',
                            ayref: 'y'
                        });
                    }
                });
                
                Plotly.newPlot('fieldPlot', traces, layout);
            }
            
            plotContours(params, measurementPoints) {
                const {a, b, resolution} = params;
                
                // 등고선용 그리드 생성
                const x = [];
                const y = [];
                const z = [];
                
                const xRange = [-a * 0.5, a * 1.5];
                const yRange = [-a * 0.8, a * 0.8];
                
                for (let i = 0; i < resolution; i++) {
                    const xi = xRange[0] + (xRange[1] - xRange[0]) * i / (resolution - 1);
                    x.push(xi);
                }
                
                for (let j = 0; j < resolution; j++) {
                    const yj = yRange[0] + (yRange[1] - yRange[0]) * j / (resolution - 1);
                    y.push(yj);
                }
                
                for (let j = 0; j < resolution; j++) {
                    const row = [];
                    for (let i = 0; i < resolution; i++) {
                        const field = this.calculateElectricField(x[i], y[j], params);
                        row.push(Math.log10(Math.max(1e4, field.magnitude)));
                    }
                    z.push(row);
                }
                
                // 타원 경계
                const ellipseX = [];
                const ellipseY = [];
                for (let i = 0; i <= 100; i++) {
                    const theta = 2 * Math.PI * i / 100;
                    ellipseX.push(a * Math.cos(theta));
                    ellipseY.push(b * Math.sin(theta));
                }
                
                const traces = [
                    {
                        type: 'contour',
                        x: x,
                        y: y,
                        z: z,
                        colorscale: 'Hot',
                        contours: {
                            coloring: 'fill'
                        },
                        colorbar: {
                            title: 'log₁₀(E) [V/m]'
                        },
                        hovertemplate: 'E = 10^%{z:.1f} V/m<extra></extra>'
                    },
                    {
                        type: 'scatter',
                        mode: 'lines',
                        x: ellipseX,
                        y: ellipseY,
                        line: {color: 'white', width: 4},
                        name: 'Conductor',
                        hoverinfo: 'none'
                    }
                ];
                
                // 측정 지점 추가
                Object.values(measurementPoints).forEach(point => {
                    traces.push({
                        type: 'scatter',
                        mode: 'markers',
                        x: [point.x],
                        y: [point.y],
                        marker: {
                            size: 12,
                            color: 'white',
                            line: {color: point.color, width: 3}
                        },
                        name: point.name,
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
            
            plotChargeDensityDistribution(params) {
                const {a, b} = params;
                const angles = [];
                const chargeDensities = [];
                const curvatureRadii = [];
                
                for (let i = 0; i <= 360; i += 5) {
                    const theta = i * Math.PI / 180;
                    angles.push(i);
                    
                    const density = this.getSurfaceChargeDensity(theta, params);
                    chargeDensities.push(density * 1e9); // nC/m²로 변환
                    
                    const curvature = this.getCurvatureRadius(theta, a, b);
                    curvatureRadii.push(curvature * 100); // cm로 변환
                }
                
                const trace1 = {
                    type: 'scatter',
                    mode: 'lines+markers',
                    x: angles,
                    y: chargeDensities,
                    name: 'Charge Density',
                    line: {color: '#e74c3c', width: 3},
                    marker: {size: 6},
                    yaxis: 'y',
                    hovertemplate: 'Angle: %{x}°<br>σ = %{y:.2f} nC/m²<extra></extra>'
                };
                
                const trace2 = {
                    type: 'scatter',
                    mode: 'lines',
                    x: angles,
                    y: curvatureRadii,
                    name: 'Curvature Radius',
                    line: {color: '#3498db', width: 2, dash: 'dash'},
                    yaxis: 'y2',
                    hovertemplate: 'Angle: %{x}°<br>R = %{y:.2f} cm<extra></extra>'
                };
                
                const layout = {
                    title: 'Surface Charge Density vs Curvature',
                    xaxis: {title: 'Angle (degrees)'},
                    yaxis: {
                        title: 'Charge Density (nC/m²)',
                        titlefont: {color: '#e74c3c'},
                        tickfont: {color: '#e74c3c'}
                    },
                    yaxis2: {
                        title: 'Curvature Radius (cm)',
                        titlefont: {color: '#3498db'},
                        tickfont: {color: '#3498db'},
                        overlaying: 'y',
                        side: 'right'
                    },
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('chargeDensityPlot', [trace1, trace2], layout);
            }
            
            plotComparison(measurementPoints, params) {
                const names = [];
                const fieldStrengths = [];
                const colors = [];
                
                Object.values(measurementPoints).forEach(point => {
                    const field = this.calculateElectricField(point.x, point.y, params);
                    names.push(point.name);
                    fieldStrengths.push(field.magnitude);
                    colors.push(point.color);
                });
                
                const trace = {
                    type: 'bar',
                    x: names,
                    y: fieldStrengths,
                    marker: {color: colors, opacity: 0.8},
                    text: fieldStrengths.map(f => f.toExponential(1)),
                    textposition: 'outside',
                    hovertemplate: '%{x}: %{y:.2e} V/m<extra></extra>'
                };
                
                const layout = {
                    title: 'Electric Field Comparison',
                    xaxis: {title: 'Measurement Point'},
                    yaxis: {title: 'Electric Field (V/m)', type: 'log'},
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('comparisonPlot', [trace], layout);
            }
            
            updateResults(measurementPoints, params) {
                let html = '';
                
                Object.values(measurementPoints).forEach(point => {
                    const field = this.calculateElectricField(point.x, point.y, params);
                    const curvature = this.getCurvatureRadius(point.angle, params.a, params.b);
                    const chargeDensity = this.getSurfaceChargeDensity(point.angle, params);
                    
                    const className = `measurement-point point-${point.name.toLowerCase().replace(' ', '')}`;
                    
                    html += `
                        <div class="${className}">
                            <h4>${point.name.toUpperCase()}</h4>
                            <p><strong>Position:</strong> (${(point.x*100).toFixed(1)}, ${(point.y*100).toFixed(1)}) cm</p>
                            <p><strong>Curvature Radius:</strong> ${(curvature*100).toFixed(2)} cm</p>
                            <p><strong>Charge Density:</strong> ${(chargeDensity*1e9).toFixed(2)} nC/m²</p>
                            <p><strong>Electric Field:</strong> <span class="field-value">${field.magnitude.toExponential(2)} V/m</span></p>
                        </div>
                    `;
                });
                
                document.getElementById('measurementResults').innerHTML = html;
            }
        }
        
        // 페이지 로드 시 시뮬레이션 초기화
        window.addEventListener('load', () => {
            new ElectricFieldSimulation();
        });
    </script>
</body>
</html>
