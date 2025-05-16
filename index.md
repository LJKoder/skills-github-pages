<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Plot X and Y Data</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 50px;
    }

    .input-section {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 20px;
      margin: 20px 0;
    }

    .input-group {
      display: flex;
      flex-direction: column;
      align-items: center;
    }

    input[type="text"] {
      width: 200px;
      padding: 10px;
      font-size: 16px;
      margin-bottom: 5px;
    }

    button {
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 5px;
    }

    button:hover {
      background-color: #45a049;
    }

    .clear-button {
      background-color: #f44336;
      margin-top: 5px;
    }

    .clear-button:hover {
      background-color: #d32f2f;
    }

    #chart-container {
      margin-top: 50px;
      width: 100%;
      height: 80vh;
    }

    canvas {
      width: 100% !important;
      height: 100% !important;
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
  <h1>Plot X and Y Data</h1>
  <p>Enter values below to plot the data and regression line.</p>

  <div class="input-section">
    <div class="input-group">
      <input type="text" id="xLabel" placeholder="X Axis Label" />
      <input type="text" id="xValues" placeholder="X Values" />
      <button class="clear-button" onclick="clearInput('xValues')">Clear</button>
    </div>

    <div class="input-group">
      <input type="text" id="yLabel" placeholder="Y Axis Label" />
      <input type="text" id="yValues" placeholder="Y Values" />
      <button class="clear-button" onclick="clearInput('yValues')">Clear</button>
    </div>

    <div class="input-group">
      <input type="text" id="yErrors" placeholder="Y Error Values" />
      <button class="clear-button" onclick="clearInput('yErrors')">Clear</button>
    </div>
  </div>

  <button onclick="plotGraph()">Plot Data</button>
  <button onclick="plotRegressionLine()">Plot Regression Line</button>

  <div id="chart-container">
    <canvas id="myChart"></canvas>
  </div>

  <script>
    let chart;

    function clearInput(id) {
      document.getElementById(id).value = '';
    }

    function plotGraph() {
      const canvas = document.getElementById('myChart');
      const ctx = canvas.getContext('2d');

      const xInput = document.getElementById('xValues').value.trim();
      const yInput = document.getElementById('yValues').value.trim();
      const xLabel = document.getElementById('xLabel').value || 'X Values';
      const yLabel = document.getElementById('yLabel').value || 'Y Values';

      const xValues = xInput.split(/\s+/).map(Number);
      const yValues = yInput.split(/\s+/).map(Number);

      if (xValues.length !== yValues.length || xValues.some(isNaN) || yValues.some(isNaN)) {
        alert('Please ensure X and Y values are valid and of equal length.');
        return;
      }

      if (chart) chart.destroy();

      chart = new Chart(ctx, {
        type: 'scatter',
        data: {
          datasets: [{
            data: xValues.map((x, i) => ({ x: x, y: yValues[i] })),
            borderColor: 'rgba(54, 162, 235, 1)',
            backgroundColor: 'rgba(54, 162, 235, 0.2)',
            borderWidth: 2,
            showLine: false,
            pointStyle: 'crossRot',
            pointBorderColor: 'black',
            pointBackgroundColor: 'transparent',
            pointRadius: 5
          }]
        },
        options: {
          responsive: true,
          scales: {
            x: {
              title: {
                display: true,
                text: xLabel,
              }
            },
            y: {
              title: {
                display: true,
                text: yLabel,
              }
            }
          }
        }
      });
    }

    function plotRegressionLine() {
      const xInput = document.getElementById('xValues').value.trim();
      const yInput = document.getElementById('yValues').value.trim();
      const yErrorsInput = document.getElementById('yErrors').value.trim();

      const xValues = xInput.split(/\s+/).map(Number);
      const yValues = yInput.split(/\s+/).map(Number);

      if (xValues.length !== yValues.length || xValues.some(isNaN) || yValues.some(isNaN)) {
        alert('Invalid or mismatched X and Y values.');
        return;
      }

      let weights = [];
      if (yErrorsInput) {
        const yErrors = yErrorsInput.split(/\s+/).map(Number);
        if (yErrors.length !== yValues.length || yErrors.some(e => isNaN(e) || e <= 0)) {
          alert('Y error values must be positive numbers matching Y values length.');
          return;
        }
        weights = yErrors.map(e => 1 / (e * e));
      } else {
        weights = Array(yValues.length).fill(1);
      }

      const data = xValues.map((x, i) => [x, yValues[i]]);
      const { beta0, beta1, seBeta0, seBeta1 } = weightedLinearRegressionWithErrors(data, weights);

      const minX = Math.min(...xValues);
      const maxX = Math.max(...xValues);
      const lineX = [minX, maxX];
      const lineY = lineX.map(x => beta1 * x + beta0);

      Chart.register({
        id: 'regressionEquation',
        beforeDraw(chart) {
          const ctx = chart.ctx;
          const chartArea = chart.chartArea;
          const x = chartArea.left + (chartArea.right - chartArea.left) * 0.4;
          const y = chartArea.top + (chartArea.bottom - chartArea.top) * 0.75;

          ctx.save();
          ctx.fillStyle = 'black';
          ctx.font = '16px Arial';
          ctx.fillText(`Y = ${formatWithError(beta1, seBeta1)} X + ${formatWithError(beta0, seBeta0)}`, x, y);
          ctx.restore();
        }
      });

      chart.data.datasets.push({
        label: 'Regression Line',
        data: lineX.map((x, i) => ({ x: x, y: lineY[i] })),
        borderColor: 'rgba(255, 99, 132, 1)',
        borderWidth: 2,
        showLine: true,
        fill: false,
        tension: 0,
      });

      chart.update();
    }

    function weightedLinearRegressionWithErrors(data, weights) {
      const n = data.length;
      const sum = arr => arr.reduce((a, b) => a + b, 0);
      const x = data.map(d => d[0]);
      const y = data.map(d => d[1]);

      const sumw = sum(weights);
      const sumwx = sum(x.map((v, i) => weights[i] * v));
      const sumwy = sum(y.map((v, i) => weights[i] * v));
      const sumwxy = sum(x.map((v, i) => weights[i] * v * y[i]));
      const sumwx2 = sum(x.map((v, i) => weights[i] * v * v));

      const xbar = sumwx / sumw;
      const ybar = sumwy / sumw;

      const beta1 = (sumwxy - sumwx * ybar) / (sumwx2 - sumwx * xbar);
      const beta0 = ybar - beta1 * xbar;

      let rss = 0;
      for (let i = 0; i < n; i++) {
        const fit = beta1 * x[i] + beta0;
        rss += weights[i] * Math.pow(y[i] - fit, 2);
      }

      const dfw = sumw - 2;
      const svar = rss / dfw;
      const xxbar = sum(x.map((v, i) => weights[i] * Math.pow(v - xbar, 2)));
      const svar1 = svar / xxbar;
      const svar0 = svar * sumwx2 / (sumw * xxbar);

      return {
        beta0,
        beta1,
        seBeta0: Math.sqrt(svar0),
        seBeta1: Math.sqrt(svar1)
      };
    }

    function formatWithError(value, error) {
      const valueParts = value.toExponential().split('e');
      const errorParts = error.toExponential().split('e');

      const valueMantissa = parseFloat(valueParts[0]);
      const valueExponent = parseInt(valueParts[1], 10);
      const errorMantissa = parseFloat(errorParts[0]);
      const errorExponent = parseInt(errorParts[1], 10);

      const adjustedError = errorMantissa * Math.pow(10, errorExponent - valueExponent);

      const superscripts = { '0': '⁰', '1': '¹', '2': '²', '3': '³', '4': '⁴', '5': '⁵', '6': '⁶', '7': '⁷', '8': '⁸', '9': '⁹', '-': '⁻' };
      const superscriptExponent = valueExponent.toString().split('').map(c => superscripts[c]).join('');

      return `(${valueMantissa.toFixed(3)} ± ${adjustedError.toFixed(3)})×10${superscriptExponent}`;
    }
  </script>
</body>
</html>
