<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>타원형 도체의 표면 전하 분포</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/plotly.js/2.26.0/plotly.min.js"></script>
    <style>
        body {
            font-family: 'Malgun Gothic', 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
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
        .info-panel {
            background: rgba(52, 152, 219, 0.1);
            border-radius: 15px;
            padding: 20px;
            margin-bottom: 30px;
            border-left: 5px solid #3498db;
        }
        .info-panel h3 {
            color: #2c3e50;
            margin-top: 0;
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
        .physics-explanation {
            background: rgba(241, 196, 15, 0.1);
            border-radius: 15px;
            padding: 20px;
            margin-bottom: 30px;
            border-left: 5px solid #f1c40f;
        }
        .formula {
            background: #f8f9fa;
            padding: 10px;
            border-radius: 5px;
            font-family: 'Courier New', monospace;
            margin: 10px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>⚡ 타원형 도체의 표면 전하 분포</h1>
        
        <div class="info-panel">
            <h3>🔬 시뮬레이션 개요</h3>
            <p>이 시뮬레이션은 타원형 도체 표면에서의 전하 분포와 전기장을 시각화합니다. 
            도체 표면의 곡률이 전하 밀도에 미치는 영향과 그에 따른 전기장의 변화를 관찰할 수 있습니다.</p>
        </div>

        <div class="physics-explanation">
            <h3>📚 물리학적 원리</h3>
            <p><strong>가우스 법칙과 경계 조건:</strong> 도체 내부의 전기장은 0이고, 표면에서 전기장은 수직입니다.</p>
            <div class="formula">E = σ/ε₀ (표면에서의 전기장)</div>
            <p><strong>곡률 효과:</strong> 뾰족한 부분일수록 전하 밀도가 높아집니다. 이는 곡률 반지름에 반비례합니다.</p>
            <div class="formula">σ ∝ 1/R (σ: 전하밀도, R: 곡률반지름)</div>
            <p><strong>주요 특징:</strong> 타원의 끝부분(뾰족한 곳)에서 전기장이 가장 강하고, 옆면(곡률이 작은 곳)에서 전기장이 가장 약합니다.</p>
        </div>
        
        <div class="controls">
            <div class="control-group">
                <label for="charge">총 전하량 (nC):</label>
                <input type="range" id="charge" min="100" max="1000" value="400" step="50">
                <div class="value-display" id="chargeValue">400 nC</div>
            </div>
            
            <div class="control-group">
                <label for="semiMajor">장축 반지름 (cm):</label>
                <input type="range" id="semiMajor" min="8" max="15" value="10" step="0.5">
                <div class="value-display" id="semiMajorValue">10.0 cm</div>
            </div>
            
            <div class="control-group">
                <label for="semiMinor">단축 반지름 (cm):</label>
                <input type="range" id="semiMinor" min="1" max="4" value="2" step="0.1">
                <div class="value-display" id="semiMinorValue">2.0 cm</div>
            </div>
            
            <div class="control-group">
                <label for="resolution">해상도:</label>
                <input type="range" id="resolution" min="20" max="60" value="40" step="5">
                <div class="value-display" id="resolutionValue">40 포인트</div>
            </div>
        </div>
        
        <div class="plot-container">
            <div class="plot-box">
                <div id="fieldPlot"></div>
            </div>
            <div class="plot-box">
                <div id="comparisonPlot"></div>
            </div>
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
                    document.getElementById('resolution').value + ' 포인트';
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
                
                // 타원의 둘레 근사값
                const perimeter = Math.PI * (3 * (a + b) - Math.sqrt((3 * a + b) * (a + 3 * b)));
                
                // 기본 전하 밀도 (균등 분포라면)
                const baseDensity = totalCharge / perimeter;
                
                // 곡률 효과: 곡률 반지름에 반비례
                // 정규화 인수를 사용하여 물리적으로 타당한 분포 생성
                const avgRadius = (a + b) / 2;
                const curvatureFactor = avgRadius / curvatureRadius;
                
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
                
                // 측정점에서 가장 가까운 타원 표면점의 각도 찾기
                let minDistance = Infinity;
                let closestTheta = 0;
                
                for (let i = 0; i < 360; i++) {
                    const theta = i * Math.PI / 180;
                    const surfaceX = a * Math.cos(theta);
                    const surfaceY = b * Math.sin(theta);
                    const distance = Math.sqrt((x - surfaceX) ** 2 + (y - surfaceY) ** 2);
                    
                    if (distance < minDistance) {
                        minDistance = distance;
                        closestTheta = theta;
                    }
                }
                
                const surfaceX = a * Math.cos(closestTheta);
                const surfaceY = b * Math.sin(closestTheta);
                const distanceFromSurface = Math.max(0.001, minDistance);
                
                // 타원 내부 점이면 전기장 0
                if (minDistance < 0.001) {
                    return {Ex: 0, Ey: 0, magnitude: 0};
                }
                
                // 해당 각도에서의 표면 전기장 강도
                const surfaceField = this.getElectricFieldAtSurface(closestTheta, params);
                
                // 거리에 따른 감소
                const fieldMagnitude = surfaceField * Math.pow(0.005 / distanceFromSurface, 1.5);
                
                // 전기장 벡터는 표면에서 측정점 방향
                const directionX = (x - surfaceX) / distanceFromSurface;
                const directionY = (y - surfaceY) / distanceFromSurface;
                
                return {
                    Ex: fieldMagnitude * directionX,
                    Ey: fieldMagnitude * directionY,
                    magnitude: fieldMagnitude
                };
            }
            
            // 측정 지점 정의
            getMeasurementPoints(params) {
                const {a, b} = params;
                const offset = 0.005; // 5mm 오프셋
                
                // 각 각도에서 타원 표면의 점 계산
                const getEllipsePoint = (theta) => {
                    return {
                        x: a * Math.cos(theta),
                        y: b * Math.sin(theta)
                    };
                };
                
                // 표면에서 법선 방향으로 오프셋된 측정점 계산
                const getMeasurementPoint = (theta) => {
                    const surface = getEllipsePoint(theta);
                    // 타원에서 법선 벡터 계산
                    const nx = surface.x / (a * a);
                    const ny = surface.y / (b * b);
                    const norm = Math.sqrt(nx * nx + ny * ny);
                    
                    return {
                        x: surface.x + (nx / norm) * offset,
                        y: surface.y + (ny / norm) * offset
                    };
                };
                
                return {
                    sharp_tip: {
                        ...getMeasurementPoint(0), // 뾰족한 끝 (θ=0)
                        angle: 0,
                        color: '#e74c3c',
                        name: '뾰족한 끝'
                    },
                    middle: {
                        ...getMeasurementPoint(Math.PI/4), // 중간 부분 (θ=π/4)
                        angle: Math.PI/4,
                        color: '#f39c12',
                        name: '중간 부분'
                    },
                    flat_side: {
                        ...getMeasurementPoint(Math.PI/2), // 평평한 면 (θ=π/2)
                        angle: Math.PI/2,
                        color: '#3498db',
                        name: '평평한 면'
                    }
                };
            }
            
            updateSimulation() {
                const params = this.getParameters();
                const measurementPoints = this.getMeasurementPoints(params);
                
                this.plotVectorField(params, measurementPoints);
                this.plotComparison(measurementPoints, params);
                this.plotChargeDensityDistribution(params);
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
                
                // 타원 그리기 - 전하 밀도에 따른 색상 변화
                const ellipsePoints = [];
                const chargeDensities = [];
                for (let i = 0; i <= 100; i++) {
                    const theta = 2 * Math.PI * i / 100;
                    const x = a * Math.cos(theta);
                    const y = b * Math.sin(theta);
                    ellipsePoints.push({x, y});
                    const density = this.getSurfaceChargeDensity(theta, params);
                    chargeDensities.push(density);
                }
                
                const maxDensity = Math.max(...chargeDensities);
                const minDensity = Math.min(...chargeDensities);
                
                const traces = [
                    {
                        type: 'scatter',
                        mode: 'markers',
                        x: ellipsePoints.map(p => p.x),
                        y: ellipsePoints.map(p => p.y),
                        marker: {
                            size: 8,
                            color: chargeDensities,
                            colorscale: 'Hot',
                            cmin: minDensity,
                            cmax: maxDensity,
                            colorbar: {
                                title: '전하 밀도<br>(C/m)',
                                titleside: 'right'
                            }
                        },
                        name: '도체 표면',
                        hovertemplate: '전하밀도: %{marker.color:.2e} C/m<extra></extra>'
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
                    title: '전기장 벡터와 표면 전하밀도',
                    xaxis: {title: '거리 (m)', scaleanchor: 'y'},
                    yaxis: {title: '거리 (m)'},
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
                    title: '위치별 전기장 비교',
                    xaxis: {title: '측정 지점'},
                    yaxis: {title: '전기장 (V/m)', type: 'log'},
                    margin: {t: 40, b: 40, l: 40, r: 40}
                };
                
                Plotly.newPlot('comparisonPlot', [trace], layout);
            }
        }
        
        // 페이지 로드 시 시뮬레이션 초기화
        window.addEventListener('load', () => {
            new ElectricFieldSimulation();
        });
    </script>
</body>
</html>
