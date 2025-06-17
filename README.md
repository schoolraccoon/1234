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
        .point-sharptip { border-left: 5px solid #e74c3c; }
        .point-middle { border-left: 5px solid #f39c12; }
        .point-flatside { border-left: 5px solid #3498db; }
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
        class EllipticalConductorSimulation {
            constructor() {
                this.epsilon0 = 8.854e-12; // F/m (진공 유전율)
                this.updateParametersFromUI(); // 초기 UI 값으로 파라미터 업데이트
                this.setupEventListeners();
                this.updateSimulation(); // 초기 시뮬레이션 실행
            }
            
            setupEventListeners() {
                const controls = ['charge', 'semiMajor', 'semiMinor', 'resolution'];
                controls.forEach(id => {
                    const element = document.getElementById(id);
                    element.addEventListener('input', () => {
                        this.updateDisplayValues();
                        this.updateParametersFromUI();
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
            
            updateParametersFromUI() {
                this.totalCharge = parseFloat(document.getElementById('charge').value) * 1e-9; // nC -> C
                this.a = parseFloat(document.getElementById('semiMajor').value) * 0.01;      // cm -> m
                this.b = parseFloat(document.getElementById('semiMinor').value) * 0.01;      // cm -> m
                this.resolution = parseInt(document.getElementById('resolution').value);
            }
            
            // 타원 상의 특정 각도(theta)에서의 곡률 반지름을 계산
            getCurvatureRadius(theta) {
                // 타원의 곡률 반지름 공식: R = (a^2 sin^2(theta) + b^2 cos^2(theta))^(3/2) / (ab)
                const numerator = Math.pow(Math.pow(this.a * Math.sin(theta), 2) + Math.pow(this.b * Math.cos(theta), 2), 1.5);
                const denominator = this.a * this.b;
                return numerator / denominator;
            }
            
            // 표면 전하 밀도 계산 (곡률에 반비례)
            getSurfaceChargeDensity(theta) {
                const curvatureRadius = this.getCurvatureRadius(theta);

                // 타원형 도체의 전하 밀도 공식은 곡률에 반비례합니다.
                // sigma = (Q / (2 * PI * c)) * (1 / sqrt(x^2/a^2 + y^2/b^2)) 형태와 유사하게,
                // 곡률 반지름의 역수에 비례하도록 설정하고, 총 전하량에 맞게 정규화합니다.
                
                // 타원의 둘레 (Ramanujan's approximation)를 사용하여 총 전하량 분배에 대한 상수를 구합니다.
                const h = Math.pow((this.a - this.b) / (this.a + this.b), 2);
                const circumference = Math.PI * (this.a + this.b) * (1 + (3 * h) / (10 + Math.sqrt(4 - 3 * h)));

                // 곡률에 반비례하는 인자 (뾰족할수록 커집니다)
                const chargeDistributionFactor = 1 / curvatureRadius;

                // 정규화 상수 (이 상수는 전하 밀도 분포를 적분했을 때 총 전하량이 되도록 조절됩니다)
                // 시뮬레이션 편의상 이 값을 조정하여 뾰족한 곳이 더 부각되도록 할 수 있습니다.
                // 여기서는 대략적으로 평균적인 1/R 값에 비례하여 스케일링합니다.
                const avgInverseCurvature = (1 / (this.b * this.b / this.a) + 1 / (this.a * this.a / this.b)) / 2; // (1/R_sharp + 1/R_flat) / 2
                const normalizationConstant = this.totalCharge / (circumference * avgInverseCurvature); 

                // 최종 전하 밀도
                return normalizationConstant * chargeDistributionFactor;
            }
            
            // 타원 외부 점에서 전기장 계산
            calculateElectricField(x, y) {
                const r = Math.sqrt(x*x + y*y);
                
                // 도체 내부 (또는 매우 가까운 지점)에서는 전기장이 0으로 가정합니다.
                // 타원 내부인지 외부인지 판별
                const isInside = (x * x / (this.a * this.a) + y * y / (this.b * this.b)) <= 1;

                if (isInside) {
                    return {Ex: 0, Ey: 0, magnitude: 0};
                }
                
                // 각도 계산
                let theta = Math.atan2(y, x);
                // 각도를 0 ~ 2PI 범위로 정규화 (선택 사항이지만 일관성을 위해)
                if (theta < 0) theta += 2 * Math.PI;

                // 타원 표면까지의 거리 (해당 각도에서)
                const rSurface = (this.a * this.b) / Math.sqrt(
                    Math.pow(this.b * Math.cos(theta), 2) + Math.pow(this.a * Math.sin(theta), 2)
                );
                
                // 측정 지점의 실제 위치가 타원 표면으로부터 5mm 떨어져 있으므로
                // 해당 지점에서 전기장 강도를 계산
                // 전기장은 표면 전하 밀도에 비례합니다 (E = sigma / epsilon0)
                const chargeDensity = this.getSurfaceChargeDensity(theta);
                const surfaceFieldMagnitude = Math.abs(chargeDensity) / this.epsilon0;

                // 측정 지점이 표면에서 멀어질수록 전기장은 감소합니다.
                // 1/r^2 법칙을 근사적으로 적용하되, 표면 가까이에서는 표면 전기장 값을 유지.
                // 여기서는 표면에서의 전기장 값을 직접적으로 반환하여
                // 뾰족한 곳에서 전기장이 높다는 것을 강조합니다.
                // 실제 거리에 따른 감소는 시뮬레이션의 스케일과 목적에 따라 조정될 수 있습니다.
                const effectiveDistanceFactor = Math.pow(rSurface / r, 1.5); // 거리가 멀어질수록 감소
                const fieldMagnitude = surfaceFieldMagnitude * effectiveDistanceFactor;

                // 전기장 벡터의 방향은 도체 표면에 수직한 방향입니다 (전하가 양수일 경우 바깥쪽).
                // 타원 표면의 법선 벡터 방향으로 전기장 벡터를 구성합니다.
                // 타원의 법선 벡터는 (x/a^2, y/b^2)에 비례합니다.
                const normalX = x / (this.a * this.a);
                const normalY = y / (this.b * this.b);
                const normalMag = Math.sqrt(normalX * normalX + normalY * normalY);

                let Ex = fieldMagnitude * normalX / normalMag;
                let Ey = fieldMagnitude * normalY / normalMag;

                // 도체 내부에서 0으로 설정했으므로, 밖에서는 항상 양의 방향으로 나오도록.
                // r = 0 인 경우를 방지
                if (r === 0) return {Ex: 0, Ey: 0, magnitude: 0};

                return {
                    Ex: Ex,
                    Ey: Ey,
                    magnitude: fieldMagnitude
                };
            }
            
            // 측정 지점 정의 (타원 표면으로부터 5mm 떨어진 지점)
            getMeasurementPoints() {
                const offset = 0.005; // 5mm
                return {
                    sharp_tip: {
                        x: this.a + offset, // 뾰족한 끝 (장축 양 끝)
                        y: 0,
                        angle: 0, // 해당 지점에 가장 가까운 타원 표면의 각도
                        color: '#e74c3c',
                        name: 'Sharp Tip'
                    },
                    middle: {
                        // 중간 지점을 뾰족한 끝과 평평한 면 사이의 대략적인 위치로 설정
                        // 예를 들어, x = a * cos(PI/4), y = b * sin(PI/4) 에서 5mm 떨어진 지점
                        x: this.a * Math.cos(Math.PI/4) + offset * Math.cos(Math.PI/4),
                        y: this.b * Math.sin(Math.PI/4) + offset * Math.sin(Math.PI/4),
                        angle: Math.PI/4,
                        color: '#f39c12',
                        name: 'Middle'
                    },
                    flat_side: {
                        x: 0,
                        y: this.b + offset, // 평평한 면 (단축 양 끝)
                        angle: Math.PI/2, // 해당 지점에 가장 가까운 타원 표면의 각도
                        color: '#3498db',
                        name: 'Flat Side'
                    }
                };
            }
            
            updateSimulation() {
                this.updateParametersFromUI(); // 항상 최신 UI 값 반영
                const measurementPoints = this.getMeasurementPoints();
                
                this.plotVectorField(measurementPoints);
                this.plotContours(measurementPoints);
                this.plotChargeDensityDistribution();
                this.plotComparison(measurementPoints);
                this.updateResults(measurementPoints);
            }
            
            plotVectorField(measurementPoints) {
                const {a, b, resolution} = this;
                
                const xRange = [-a * 1.5, a * 1.5]; // plot 범위 조정
                const yRange = [-a * 1.0, a * 1.0]; // plot 범위 조정
                const step = (xRange[1] - xRange[0]) / resolution;
                
                const vectors = [];
                for (let i = 0; i <= resolution; i++) {
                    const x = xRange[0] + i * step;
                    for (let j = 0; j <= resolution; j++) {
                        const y = yRange[0] + j * step;
                        
                        const field = this.calculateElectricField(x, y);
                        // 전기장 크기가 0이 아닌 유효한 값일 경우만 벡터 추가
                        if (field.magnitude > 1e3) { // 너무 작은 필드는 그리지 않음 (노이즈 방지)
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
                    const field = this.calculateElectricField(point.x, point.y);
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
                        text: [`${point.name}`], // 텍스트는 이름만 표시, 값은 hovertemplate 또는 결과 패널에서
                        textposition: 'top center',
                        textfont: {size: 10},
                        name: point.name,
                        hovertemplate: `<b>${point.name}</b><br>E = ${field.magnitude.toExponential(2)} V/m<extra></extra>`
                    });
                });
                
                const layout = {
                    title: 'Electric Field Vectors',
                    xaxis: {title: 'X (m)', scaleanchor: 'y', range: xRange},
                    yaxis: {title: 'Y (m)', range: yRange},
                    showlegend: true,
                    margin: {t: 40, b: 40, l: 40, r: 40},
                    annotations: []
                };
                
                // 벡터 화살표 추가 (간격을 두어 너무 많지 않게)
                const arrowScale = 0.01; // 화살표 길이 스케일
                const maxArrows = 100; // 최대 화살표 개수 제한
                const arrowStride = Math.max(1, Math.floor(vectors.length / maxArrows));

                for (let k = 0; k < vectors.length; k += arrowStride) {
                    const v = vectors[k];
                    const norm = v.mag;
                    if (norm > 0) {
                        layout.annotations.push({
                            x: v.x,
                            y: v.y,
                            ax: v.x + v.Ex / norm * arrowScale,
                            ay: v.y + v.Ey / norm * arrowScale,
                            xref: 'x',
                            yref: 'y',
                            axref: 'x',
                            ayref: 'y',
                            arrowhead: 2,
                            arrowsize: 1,
                            arrowwidth: 2,
                            arrowcolor: 'blue'
                        });
                    }
                }
                
                Plotly.newPlot('fieldPlot', traces, layout);
            }
            
            plotContours(measurementPoints) {
                const {a, b, resolution} = this;
                
                const x = [];
                const y = [];
                const z = [];
                
                const xRange = [-a * 1.5, a * 1.5];
                const yRange = [-a * 1.0, a * 1.0];
                
                for (let i = 0; i < resolution; i++) {
                    x.push(xRange[0] + (xRange[1] - xRange[0]) * i / (resolution - 1));
                }
                
                for (let j = 0; j < resolution; j++) {
                    y.push(yRange[0] + (yRange[1] - yRange[0]) * j / (resolution - 1));
                }
                
                for (let j = 0; j < resolution; j++) {
                    const row = [];
                    for (let i = 0; i < resolution; i++) {
                        const field = this.calculateElectricField(x[i], y[j]);
                        row.push(Math.log10(Math.max(1e3, field.magnitude))); // log 스케일로 표시, 최소값 설정
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
                            coloring: 'fill',
                            showlines: true // 등고선 라인 표시
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
                        line: {color: 'white', width: 4}, // 도체 경계를 더 잘 보이게
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
                    xaxis: {title: 'X (m)', scaleanchor: 'y', range: xRange},
                    yaxis: {title: 'Y (m)', range: yRange},
                    showlegend: false,
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('contourPlot', traces, layout);
            }
            
            plotChargeDensityDistribution() {
                const {a, b} = this;
                const angles = [];
                const chargeDensities = [];
                const curvatureRadii = [];
                
                for (let i = 0; i <= 360; i += 5) {
                    const theta = i * Math.PI / 180;
                    angles.push(i);
                    
                    const density = this.getSurfaceChargeDensity(theta);
                    chargeDensities.push(density * 1e9); // C/m² -> nC/m²
                    
                    const curvature = this.getCurvatureRadius(theta);
                    curvatureRadii.push(curvature * 100); // m -> cm
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
            
            plotComparison(measurementPoints) {
                const names = [];
                const fieldStrengths = [];
                const colors = [];
                
                Object.values(measurementPoints).forEach(point => {
                    const field = this.calculateElectricField(point.x, point.y);
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
                    yaxis: {title: 'Electric Field (V/m)', type: 'log'}, // 로그 스케일
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('comparisonPlot', [trace], layout);
            }
            
            updateResults(measurementPoints) {
                let html = '';
                
                Object.values(measurementPoints).forEach(point => {
                    const field = this.calculateElectricField(point.x, point.y);
                    const curvature = this.getCurvatureRadius(point.angle);
                    const chargeDensity = this.getSurfaceChargeDensity(point.angle);
                    
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
            new EllipticalConductorSimulation();
        });
    </script>
</body>
</html>
