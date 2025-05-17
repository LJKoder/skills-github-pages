<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Plot with Error Bars 1 </title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 40px;
    }
    .input-grid {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 15px;
      max-width: 900px;
      margin: 0 auto 30px;
    }
    input[type="text"] {
      width: 100%;
      padding: 8px;
      font-size: 14px;
    }
    button {
      padding: 10px 20px;
      margin: 10px;
      font-size: 16px;
      cursor: pointer;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 4px;
    }
    button:hover {
      background-color: #45a049;
    }
    #chart-container {
      width: 100%;
      height: 70vh;
    }
    canvas {
      width: 100% !important;
      height: 100% !important;
    }
  </style>

  <!-- Load Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>

  <!-- Load Error Bars plugin -->
  <script src="https://cdn.jsdelivr.net/npm/@sgratzl/chartjs-chart-errorbars@4.0.0/build/index.umd.min.js"></script>
</head>
<body>
  <h1>Plot Data with Error Bars v1</h1>
  <div class="input-grid">
    <input type="text" id="xLabel" placeholder="X Axis Label">
    <input type="text" id="yLabel" placeholder="Y Axis Label">
    <div></div>

    <input type="text" id="xValues" placeholder="X Values (space/tab/newline-separated)">
    <input type="text" id="yValues" placeholder="Y Values">
    <div></div>

    <input type="text" id="xErrors" placeholder="X Errors">
    <input type="text" id="yErrors" placeholder="Y Errors">
    <div></div>
  </div>

  <button onclick="plotGraph()">Plot Data</button>
  <button onclick="plotRegressionLine()">Plot Regression Line</button>
  <button onclick="downloadChart()">Download Chart</button>

  <div id="chart-container">
    <canvas id="myChart"></canvas>
  </div>

  <script>
    let chart;

    // Register Chart.js and plugin
    const { Chart } = window.Chart;
    const { ErrorBarController, ErrorBar } = window['chartjs-chart-errorbars'];
    Chart.register(ErrorBarController, ErrorBar);

    function parseInput(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(parseFloat);
    }

    function plotGraph() {
      const xValues = parseInput('xValues');
      const yValues = parseInput('yValues');
      const xErrors = parseInput('xErrors');
      const yErrors = parseInput('yErrors');
      const xLabel = document.getElementById('xLabel').value || 'X Axis';
      const yLabel = document.getElementById('yLabel').value || 'Y Axis';

      if (xValues.length !== yValues.length ||
          (xErrors.length && xErrors.length !== xValues.length) ||
          (yErrors.length && yErrors.length !== yValues.length)) {
        alert('Length of values and errors must match.');
        return;
      }

      const data = xValues.map((x, i) => ({
        x,
        y: yValues[i],
        xError: xErrors[i] || 0,
        yError: yErrors[i] || 0
      }));

      if (chart) chart.destroy();

      try {
        chart = new Chart(document.getElementById('myChart'), {
          type: 'errorBar',
          data: {
            datasets: [{
              label: 'Measured Data',
              data,
              backgroundColor: 'rgba(54, 162, 235, 0.6)',
              borderColor: 'rgba(54, 162, 235, 1)',
              pointRadius: 5
            }]
          },
          options: {
            responsive: true,
            plugins: {
              legend: { display: true },
              tooltip: { mode: 'nearest', intersect: false }
            },
            scales: {
              x: {
                title: {
                  display: true,
                  text: xLabel
                },
                type: 'linear',
                position: 'bottom'
              },
              y: {
                title: {
                  display: true,
                  text: yLabel
                }
              }
            }
          }
        });
        console.log("✅ Chart rendered with error bars.");
      } catch (err) {
        console.error("❌ Chart rendering failed:", err);
      }
    }

    function plotRegressionLine() {
      const xValues = parseInput('xValues');
      const yValues = parseInput('yValues');
      const yErrors = parseInput('yErrors');

      if (xValues.length !== yValues.length || (yErrors.length && yErrors.length !== yValues.length)) {
        alert('X, Y, and Y error lengths must match.');
        return;
      }

      const weights = yErrors.length ? yErrors.map(e => 1 / (e * e)) : Array(xValues.length).fill(1);
      const data = xValues.map((x, i) => [x, yValues[i]]);

      const { beta0, beta1 } = weightedLinearRegressionWithErrors(data, weights);

      const minX = Math.min(...xValues);
      const maxX = Math.max(...xValues);
      const lineData = [
        { x: minX, y: beta1 * minX + beta0 },
        { x: maxX, y: beta1 * maxX + beta0 }
      ];

      chart.data.datasets.push({
        label: 'Regression Line',
        data: lineData,
        type: 'line',
        borderColor: 'rgba(255, 99, 132, 1)',
        backgroundColor: 'rgba(255, 99, 132, 0.2)',
        fill: false,
        tension: 0,
        borderWidth: 2,
        pointRadius: 0,
      });

      chart.update();
    }

    function weightedLinearRegressionWithErrors(data, weights) {
      const x = data.map(d => d[0]);
      const y = data.map(d => d[1]);
      const sum = arr => arr.reduce((a, b) => a + b, 0);

      const sumw = sum(weights);
      const sumwx = sum(x.map((xi, i) => weights[i] * xi));
      const sumwy = sum(y.map((yi, i) => weights[i] * yi));
      const sumwxy = sum(x.map((xi, i) => weights[i] * xi * y[i]));
      const sumwx2 = sum(x.map((xi, i) => weights[i] * xi * xi));

      const xbar = sumwx / sumw;
      const ybar = sumwy / sumw;

      const beta1 = (sumwxy - sumwx * ybar) / (sumwx2 - sumwx * xbar);
      const beta0 = ybar - beta1 * xbar;

      return { beta0, beta1 };
    }

    function downloadChart() {
      const link = document.createElement('a');
      link.download = 'chart.png';
      link.href = document.getElementById('myChart').toDataURL();
      link.click();
    }
  </script>
</body>
</html>
