<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Electric Field Simulation on Elliptical Conductor</title>
    <script src="https://cdn.plot.ly/plotly-2.26.0.min.js"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 20px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .container {
            max-width: 1000px; /* 컨테이너 너비 조정 */
            margin: 0 auto;
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            padding: 30px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            gap: 20px; /* 요소 간 간격 추가 */
        }
        h1 {
            text-align: center;
            color: #2c3e50;
            margin-bottom: 20px; /* 마진 조정 */
            font-size: 2.2em; /* 폰트 크기 조정 */
            text-shadow: 2px 2px 4px rgba(0,0,0,0.1);
        }
        .controls {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); /* 컨트롤 너비 조정 */
            gap: 15px; /* 간격 조정 */
            margin-bottom: 20px;
            padding: 15px; /* 패딩 조정 */
            background: rgba(52, 152, 219, 0.1);
            border-radius: 15px;
        }
        .control-group {
            display: flex;
            flex-direction: column;
        }
        label {
            font-weight: 600;
            margin-bottom: 5px; /* 마진 조정 */
            color: #34495e;
            font-size: 0.9em; /* 폰트 크기 조정 */
        }
        input[type="range"] {
            width: 100%;
            margin: 5px 0; /* 마진 조정 */
            appearance: none;
            height: 6px; /* 높이 조정 */
            border-radius: 3px; /* 둥근 정도 조정 */
            background: #ddd;
            outline: none;
        }
        input[type="range"]::-webkit-slider-thumb {
            appearance: none;
            width: 18px; /* 크기 조정 */
            height: 18px; /* 크기 조정 */
            border-radius: 50%;
            background: #3498db;
            cursor: pointer;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2); /* 그림자 조정 */
        }
        .value-display {
            font-weight: bold;
            color: #2980b9;
            text-align: center;
            margin-top: 3px; /* 마진 조정 */
            font-size: 0.85em; /* 폰트 크기 조정 */
        }
        .plot-container {
            display: grid;
            grid-template-columns: 1fr 1fr; /* 2개 컬럼 */
            gap: 20px; /* 간격 조정 */
            flex-grow: 1; /* 컨테이너 내에서 플롯이 공간을 차지하도록 */
        }
        .plot-box {
            background: white;
            border-radius: 15px;
            padding: 15px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
            min-height: 400px; /* 최소 높이 설정 */
            display: flex;
            justify-content: center;
            align-items: center;
        }
        .results-panel {
            background: rgba(46, 204, 113, 0.1);
            border-radius: 15px;
            padding: 15px;
            margin-top: 10px; /* 마진 조정 */
            text-align: center;
        }
        .measurement-point {
            display: inline-block;
            margin: 8px; /* 마진 조정 */
            padding: 12px; /* 패딩 조정 */
            background: white;
            border-radius: 10px;
            box-shadow: 0 3px 8px rgba(0,0,0,0.1); /* 그림자 조정 */
            min-width: 180px; /* 최소 너비 조정 */
        }
        .point-sharptip { border-left: 5px solid #e74c3c; }
        .point-middle { border-left: 5px solid #f39c12; }
        .point-flatside { border-left: 5px solid #3498db; }
        .field-value {
            font-size: 1.1em; /* 폰트 크기 조정 */
            font-weight: bold;
            color: #2c3e50;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>⚡ 전기장 시뮬레이션: 타원형 도체</h1>
        
        <div class="controls">
            <div class="control-group">
                <label for="charge">총 전하량 (nC):</label>
                <input type="range" id="charge" min="100" max="1000" value="400" step="50">
                <div class="value-display" id="chargeValue">400 nC</div>
            </div>
            
            <div class="control-group">
                <label for="semiMajor">장반경 (cm):</label>
                <input type="range" id="semiMajor" min="8" max="15" value="10" step="0.5">
                <div class="value-display" id="semiMajorValue">10.0 cm</div>
            </div>
            
            <div class="control-group">
                <label for="semiMinor">단반경 (cm):</label>
                <input type="range" id="semiMinor" min="1" max="4" value="2" step="0.1">
                <div class="value-display" id="semiMinorValue">2.0 cm</div>
            </div>
            
            <div class="control-group">
                <label for="resolution">그리드 해상도:</label>
                <input type="range" id="resolution" min="20" max="60" value="40" step="5">
                <div class="value-display" id="resolutionValue">40 points</div>
            </div>
        </div>
        
        <div class="plot-container">
            <div class="plot-box">
                <div id="fieldPlot" style="width:100%; height:100%;"></div>
            </div>
            <div class="plot-box">
                <div id="comparisonPlot" style="width:100%; height:100%;"></div>
            </div>
        </div>
        
        <div class="results-panel">
            <h3>🔬 측정 결과</h3>
            <div id="measurementResults"></div>
        </div>
    </div>

    <script>
        class EllipticalConductorSimulation {
            constructor() {
                this.epsilon0 = 8.854e-12; // F/m (진공 유전율)
                this.updateParametersFromUI(); 
                this.setupEventListeners();
                this.updateSimulation(); 
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
                this.updateDisplayValues(); // 초기 값 표시
            }
            
            updateDisplayValues() {
                document.getElementById('chargeValue').textContent = 
                    document.getElementById('charge').value + ' nC';
                document.getElementById('semiMajorValue').textContent = 
                    parseFloat(document.getElementById('semiMajor').value).toFixed(1) + ' cm';
                document.getElementById('semiMinorValue').textContent = 
                    parseFloat(document.getElementById('semiMinor').value).toFixed(1) + ' cm';
                document.getElementById('resolutionValue').textContent = 
                    document.getElementById('resolution').value + ' points';
            }
            
            updateParametersFromUI() {
                this.totalCharge = parseFloat(document.getElementById('charge').value) * 1e-9; // nC -> C
                this.a = parseFloat(document.getElementById('semiMajor').value) * 0.01;      // cm -> m (장반경)
                this.b = parseFloat(document.getElementById('semiMinor').value) * 0.01;      // cm -> m (단반경)
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
                
                // 타원 둘레 근사 (Ramanujan's approximation)
                const h = Math.pow((this.a - this.b) / (this.a + this.b), 2);
                const circumference = Math.PI * (this.a + this.b) * (1 + (3 * h) / (10 + Math.sqrt(4 - 3 * h)));

                // 곡률에 반비례하는 인자 (뾰족할수록 커짐)
                const chargeDistributionFactor = 1 / curvatureRadius;

                // 정규화 상수 계산 (전하 밀도 분포를 적분했을 때 총 전하량이 되도록)
                // 뾰족한 곳 (theta = 0)과 평평한 곳 (theta = PI/2)의 곡률 반지름을 이용한 평균 역 곡률
                const minCurvatureRadius = this.getCurvatureRadius(0);  
                const maxCurvatureRadius = this.getCurvatureRadius(Math.PI / 2);  
                const avgInverseCurvature = (1 / minCurvatureRadius + 1 / maxCurvatureRadius) / 2;
                
                const normalizationConstant = this.totalCharge / (circumference * avgInverseCurvature); 

                return normalizationConstant * chargeDistributionFactor;
            }
            
            // 타원 외부 점에서 전기장 계산
            calculateElectricField(x, y) {
                const r = Math.sqrt(x*x + y*y);
                
                // 타원 내부인지 외부인지 판별: (x/a)^2 + (y/b)^2 <= 1
                // 작은 오차를 허용하여 경계면 처리
                const isInside = (x * x / (this.a * this.a) + y * y / (this.b * this.b)) <= 1.001; 
                
                if (isInside) {
                    return {Ex: 0, Ey: 0, magnitude: 0};
                }
                
                // 각도 계산 (theta = 0~2PI)
                let theta = Math.atan2(y, x);
                if (theta < 0) theta += 2 * Math.PI;

                // 해당 각도에서 타원 표면까지의 거리
                const rSurface = (this.a * this.b) / Math.sqrt(
                    Math.pow(this.b * Math.cos(theta), 2) + Math.pow(this.a * Math.sin(theta), 2)
                );
                
                const chargeDensity = this.getSurfaceChargeDensity(theta);
                const surfaceFieldMagnitude = Math.abs(chargeDensity) / this.epsilon0;

                // 측정 지점의 실제 거리를 고려한 전기장 감소
                // E ~ (sigma / epsilon0) * (r_surface / r_point)^n 형태로 감쇠
                // r_point는 원점으로부터 측정 지점까지의 거리
                // n=1.5는 시각적인 효과를 위해 조절된 값입니다.
                const effectiveDistanceFactor = Math.pow(rSurface / r, 1.5); 
                const fieldMagnitude = surfaceFieldMagnitude * effectiveDistanceFactor;

                // 전기장 벡터의 방향은 도체 표면에 수직한 방향입니다 (전하가 양수일 경우 바깥쪽).
                // 타원 표면의 법선 벡터 (x/a^2, y/b^2)에 비례
                const normalX = x / (this.a * this.a);
                const normalY = y / (this.b * this.b);
                const normalMag = Math.sqrt(normalX * normalX + normalY * normalY);

                let Ex = fieldMagnitude * normalX / normalMag;
                let Ey = fieldMagnitude * normalY / normalMag;
                
                // 너무 작은 값은 0으로 처리하여 시각적 노이즈 감소
                if (fieldMagnitude < 1e3) { // 1000 V/m 미만은 무시
                    return {Ex: 0, Ey: 0, magnitude: 0};
                }

                return {
                    Ex: Ex,
                    Ey: Ey,
                    magnitude: fieldMagnitude
                };
            }
            
            // 측정 지점 정의 (타원 표면으로부터 5mm 떨어진 지점)
            getMeasurementPoints() {
                const offset = 0.005; // 5mm

                // Sharp Tip (theta = 0)
                const sharpTipX = this.a;
                const sharpTipY = 0;
                const sharpTipNormalX = sharpTipX / (this.a * this.a);
                const sharpTipNormalY = sharpTipY / (this.b * this.b);
                const sharpTipNormalMag = Math.sqrt(sharpTipNormalX * sharpTipNormalX + sharpTipNormalY * sharpTipNormalY);
                const sharpTipPointX = sharpTipX + offset * (sharpTipNormalX / sharpTipNormalMag);
                const sharpTipPointY = sharpTipY + offset * (sharpTipNormalY / sharpTipNormalMag);

                // Flat Side (theta = PI/2)
                const flatSideX = 0;
                const flatSideY = this.b;
                const flatSideNormalX = flatSideX / (this.a * this.a);
                const flatSideNormalY = flatSideY / (this.b * this.b);
                const flatSideNormalMag = Math.sqrt(flatSideNormalX * flatSideNormalX + flatSideNormalY * flatSideNormalY);
                const flatSidePointX = flatSideX + offset * (flatSideNormalX / flatSideNormalMag);
                const flatSidePointY = flatSideY + offset * (flatSideNormalY / flatSideNormalMag);

                // Middle (theta = PI/4)
                const middleTheta = Math.PI / 4;
                const middleEllipseX = this.a * Math.cos(middleTheta);
                const middleEllipseY = this.b * Math.sin(middleTheta);
                const middleNormalX = middleEllipseX / (this.a * this.a);
                const middleNormalY = middleEllipseY / (this.b * this.b);
                const middleNormalMag = Math.sqrt(middleNormalX * middleNormalX + middleNormalY * middleNormalY);
                const middlePointX = middleEllipseX + offset * (middleNormalX / middleNormalMag);
                const middlePointY = middleEllipseY + offset * (middleNormalY / middleNormalMag);

                return {
                    sharp_tip: {
                        x: sharpTipPointX, 
                        y: sharpTipPointY,
                        angle: 0, 
                        color: '#e74c3c',
                        name: 'Sharp Tip'
                    },
                    middle: {
                        x: middlePointX,
                        y: middlePointY,
                        angle: middleTheta,
                        color: '#f39c12',
                        name: 'Middle'
                    },
                    flat_side: {
                        x: flatSidePointX,
                        y: flatSidePointY, 
                        angle: Math.PI/2, 
                        color: '#3498db',
                        name: 'Flat Side'
                    }
                };
            }
            
            updateSimulation() {
                this.updateParametersFromUI();
                const measurementPoints = this.getMeasurementPoints();
                
                this.plotVectorField(measurementPoints);
                this.plotComparison(measurementPoints);
                this.updateResults(measurementPoints);
            }
            
            plotVectorField(measurementPoints) {
                const {a, b, resolution} = this;
                
                // 플롯 범위를 고정된 값으로 설정 (장반경 최대값에 기반하여 넉넉하게)
                // Assuming max semiMajor is 15cm = 0.15m. Let's make the plot range fixed, e.g., +/- 0.25m
                const fixedPlotRange = 0.25; // meters
                const xRange = [-fixedPlotRange, fixedPlotRange]; 
                const yRange = [-fixedPlotRange, fixedPlotRange]; 
                const stepX = (xRange[1] - xRange[0]) / resolution;
                const stepY = (yRange[1] - yRange[0]) / resolution;
                
                const vectors = [];
                for (let i = 0; i <= resolution; i++) {
                    const x = xRange[0] + i * stepX;
                    for (let j = 0; j <= resolution; j++) {
                        const y = yRange[0] + j * stepY;
                        
                        const field = this.calculateElectricField(x, y);
                        // 전기장 크기가 0이 아닌 유효한 값일 경우만 벡터 추가
                        if (field.magnitude > 0) { 
                            vectors.push({x, y, Ex: field.Ex, Ey: field.Ey, mag: field.magnitude});
                        }
                    }
                }
                
                // 타원 그리기 (도체)
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
                        name: '도체',
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
                        text: [`${point.name}`], 
                        textposition: 'top center',
                        textfont: {size: 10},
                        name: point.name,
                        hovertemplate: `<b>${point.name}</b><br>E = ${field.magnitude.toExponential(2)} V/m<extra></extra>`
                    });
                });
                
                const layout = {
                    title: '전기장 벡터 분포',
                    xaxis: {title: 'X (m)', scaleanchor: 'y', range: xRange},
                    yaxis: {title: 'Y (m)', range: yRange},
                    showlegend: true,
                    margin: {t: 40, b: 40, l: 40, r: 40},
                    annotations: []
                };
                
                // 벡터 화살표 추가 (간격을 두어 너무 많지 않게)
                // Arrow scale should also be relative to the fixed plot range, not ellipse size
                const arrowScale = fixedPlotRange * 0.05; // Adjust as needed for good visualization
                const maxArrows = 200; // 최대 화살표 개수 제한
                const arrowStride = Math.max(1, Math.floor(vectors.length / maxArrows));

                for (let k = 0; k < vectors.length; k += arrowStride) {
                    const v = vectors[k];
                    const norm = v.mag;
                    if (norm > 0) {
                        // 화살표 머리가 v.x, v.y에 위치하도록 조정
                        layout.annotations.push({
                            x: v.x, 
                            y: v.y,
                            ax: v.x - (v.Ex / norm * arrowScale), // 꼬리 위치 조정
                            ay: v.y - (v.Ey / norm * arrowScale),
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
                    // 막대 위에 과학적 표기법으로 값 표시
                    text: fieldStrengths.map(f => f.toExponential(1)), 
                    textposition: 'outside',
                    hovertemplate: '%{x}: %{y:.2e} V/m<extra></extra>'
                };
                
                // Y축 눈금 값과 텍스트를 과학적 표기법으로 생성하는 헬퍼 함수
                const generateLogTicks = (minVal, maxVal) => {
                    const tickVals = [];
                    const tickTexts = [];
                    let currentPower = Math.floor(Math.log10(minVal));
                    let currentVal = Math.pow(10, currentPower);

                    while (currentVal <= maxVal * 1.1) { // 최대값보다 조금 더 크게 커버
                        for (let i = 1; i < 10; i++) {
                            const val = i * currentVal;
                            if (val >= minVal && val <= maxVal * 1.1) {
                                tickVals.push(val);
                                // LaTeX 스타일의 과학적 표기법 텍스트 생성
                                tickTexts.push(`${i === 1 ? '10' : i}e${currentPower}`);
                            }
                        }
                        currentPower++;
                        currentVal = Math.pow(10, currentPower);
                    }
                    return { tickVals, tickTexts };
                };

                const { tickVals, tickTexts } = generateLogTicks(
                    Math.max(1e3, Math.min(...fieldStrengths) / 2), // 최소값은 1e3 (1000) 또는 실제 데이터의 절반 중 큰 값
                    Math.max(...fieldStrengths) * 2 // 최대값의 2배까지 표시
                );

                const layout = {
                    title: '측정 지점별 전기장 비교',
                    xaxis: {title: '측정 지점'},
                    yaxis: {
                        title: '전기장 (V/m)', 
                        type: 'log', 
                        automargin: true,
                        tickmode: 'array',
                        tickvals: tickVals,
                        ticktext: tickTexts,
                        // Y축 범위 조정: 최소값은 1000V/m 미만이면 1000V/m로 설정, 최대값은 가장 큰 데이터의 1.5배 정도로 설정
                        range: [Math.log10(Math.max(1e3, Math.min(...fieldStrengths) / 2)), Math.log10(Math.max(...fieldStrengths) * 1.5)]
                    }, 
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
                            <p><strong>위치:</strong> (${(point.x*100).toFixed(1)}, ${(point.y*100).toFixed(1)}) cm</p>
                            <p><strong>곡률 반경:</strong> ${(curvature*100).toFixed(2)} cm</p>
                            <p><strong>전하 밀도:</strong> ${(chargeDensity*1e9).toFixed(2)} nC/m²</p>
                            <p><strong>전기장:</strong> <span class="field-value">${field.magnitude.toExponential(2)} V/m</span></p>
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
