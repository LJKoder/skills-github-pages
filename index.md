
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title> Weighted Regression with Error Bars</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 40px;
      text-align: center;
    }
    .input-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 15px;
  max-width: 900px;
  margin: 0 auto 30px;
}
.input-grid strong {
  font-size: 16px;
  grid-column: span 3;
  text-align: left;
  padding-top: 10px;
}
    input[type="text"] {
      padding: 8px;
      font-size: 14px;
      width: 100%;
    }
    button {
      padding: 10px 20px;
      margin: 10px;
      font-size: 16px;
      background-color: #4CAF50;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
    }
    button:hover {
      background-color: #45a049;
    }
    #plot {
      width: 100%;
      height: 75vh;
    }
  </style>
</head>
<body>
  <h1> Graph with Error Bars and Weighted Regression </h1>
  <p>This tool lets you input data with optional error bars and calculates a weighted linear regression. <p></p> You should simply copy-paste data from your excel columns, then press the plot button.</p>
<div class="input-grid">
  <!-- Axis labels -->
  <div><strong>Axis Labels</strong></div><div></div><div></div>

  <!-- Description text across all columns -->
  <div style="grid-column: span 3;">
    <p>
      Note that you can use simple HTML formatting for the axes labels.
      Use the <code>&lt;sub&gt;</code> and <code>&lt;sup&gt;</code> tags.
      For example: <code>H&lt;sub&gt;2&lt;/sub&gt;O</code> will render as H<sub>2</sub>O.
    </p>
  </div>

  <!-- Axis label inputs -->
  <input type="text" id="xLabel" placeholder="X Axis Label">
  <input type="text" id="yLabel" placeholder="Y Axis Label">
  <div></div>

  <!-- X values -->
  <div><strong>X Values</strong></div><div></div><div></div>
  <input type="text" id="xValues" placeholder="X Values (space-separated)">
  <input type="text" id="xErrors" placeholder="X Errors (optional)">
  <div></div>

  <!-- Y values -->
  <div><strong>Y Values</strong></div><div></div><div></div>
  <input type="text" id="yValues" placeholder="Y Values">
  <input type="text" id="yErrors" placeholder="Y Errors (optional)">
  <div></div>
</div>


  <button onclick="plotData()">Plot</button>
  <button onclick="downloadPlot()">Download Chart</button>
  <div id="plot"></div>

  <script>
    function parseValues(id) {
      const val = document.getElementById(id).value.trim();
      return val === "" ? [] : val.split(/\s+/).map(Number);
    }

    function weightedLinearRegression(x, y, weights) {
      const sum = arr => arr.reduce((a, b) => a + b, 0);
      const w = weights;
      const wx = x.map((xi, i) => w[i] * xi);
      const wy = y.map((yi, i) => w[i] * yi);
      const wxy = x.map((xi, i) => w[i] * xi * y[i]);
      const wx2 = x.map((xi, i) => w[i] * xi * xi);

      const sumw = sum(w);
      const sumwx = sum(wx);
      const sumwy = sum(wy);
      const sumwxy = sum(wxy);
      const sumwx2 = sum(wx2);

      const xbar = sumwx / sumw;
      const ybar = sumwy / sumw;

      const slope = (sumwxy - sumwx * ybar) / (sumwx2 - sumwx * xbar);
      const intercept = ybar - slope * xbar;

      const n = x.length;
      const residuals = y.map((yi, i) => yi - (slope * x[i] + intercept));
      const variance = sum(residuals.map((r, i) => w[i] * r * r)) / (n - 2);

      const slopeUncertainty = Math.sqrt(variance / (sumwx2 - sumwx * xbar));
      const interceptUncertainty = Math.sqrt(variance * (1 / sumw + xbar * xbar / (sumwx2 - sumwx * xbar)));

      return { slope, intercept, slopeUncertainty, interceptUncertainty };
    }

    function plotData() {
      const x = parseValues('xValues');
      const y = parseValues('yValues');
      const xErr = parseValues('xErrors');
      const yErr = parseValues('yErrors');
      const xLabel = document.getElementById('xLabel').value || 'X Axis';
      const yLabel = document.getElementById('yLabel').value || 'Y Axis';

      if (x.length !== y.length) {
        alert('X and Y values must be the same length.');
        return;
      }

      const weights = yErr.length === y.length ? yErr.map(e => 1 / (e * e)) : Array(x.length).fill(1);

      const { slope, intercept, slopeUncertainty, interceptUncertainty } = weightedLinearRegression(x, y, weights);

      const lineX = [Math.min(...x), Math.max(...x)];
      const lineY = lineX.map(xi => slope * xi + intercept);

      const data = [];

      data.push({
        x: x,
        y: y,
        mode: 'markers',
        type: 'scatter',
        name: '',
      marker: {
        color: 'black',
        size: 7,
        symbol: 'x-thin-open'
      },
        error_x: xErr.length === x.length ? {
          type: 'data',
          array: xErr,
          visible: true
        } : undefined,
        error_y: yErr.length === y.length ? {
          type: 'data',
          array: yErr,
          visible: true
        } : undefined
      });

      data.push({
        x: lineX,
        y: lineY,
        mode: 'lines',
        type: 'scatter',
        name: '',
        line: { color: 'red', width: 2 }
      });

const annotationText = `y = (${slope.toExponential(3)} ± ${slopeUncertainty.toExponential(3)})x + (${intercept.toExponential(3)} ± ${interceptUncertainty.toExponential(3)})`;


      const layout = {
        title: '',
        xaxis: { title: xLabel },
        yaxis: { title: yLabel },
        showlegend: false,
        annotations: [{
          x: 0.05,
          y: 0.95,
          xref: 'paper',
          yref: 'paper',
          text: annotationText,
          showarrow: false,
          font: { color: 'black', size: 14 }
        }]
      };

      Plotly.newPlot('plot', data, layout);
    }

    function downloadPlot() {
      Plotly.downloadImage('plot', { format: 'png', filename: 'plot_with_regression' });
    }
  </script>
</body>
</html>
