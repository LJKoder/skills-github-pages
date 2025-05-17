<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Plotly Graph with Error Bars</title>
  <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 40px;
      text-align: center;
    }
    .input-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 10px;
      max-width: 800px;
      margin: auto;
    }
    input {
      padding: 8px;
      font-size: 14px;
    }
    button {
      margin-top: 20px;
      padding: 10px 20px;
      font-size: 16px;
    }
    #plot {
      margin-top: 40px;
    }
  </style>
</head>
<body>
  <h1>Plotly Graph with Error Bars</h1>
  <div class="input-grid">
    <input type="text" id="xValues" placeholder="X values (space-separated)">
    <input type="text" id="yValues" placeholder="Y values">
    <input type="text" id="xErrors" placeholder="X errors (optional)">
    <input type="text" id="yErrors" placeholder="Y errors">
    <input type="text" id="xLabel" placeholder="X Axis Label">
    <input type="text" id="yLabel" placeholder="Y Axis Label">
  </div>
  <button onclick="plot()">Plot Graph</button>

  <div id="plot"></div>

  <script>
    function parseInput(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(Number);
    }

    function plot() {
      const x = parseInput('xValues');
      const y = parseInput('yValues');
      const xErr = parseInput('xErrors');
      const yErr = parseInput('yErrors');
      const xLabel = document.getElementById('xLabel').value || 'X Axis';
      const yLabel = document.getElementById('yLabel').value || 'Y Axis';

      if (x.length !== y.length || y.length !== yErr.length || (xErr.length && x.length !== xErr.length)) {
        alert('Lengths of X, Y, and error arrays must match.');
        return;
      }

      const trace = {
        x,
        y,
        mode: 'markers',
        type: 'scatter',
        name: 'Data',
        error_y: {
          type: 'data',
          array: yErr,
          visible: true
        },
        marker: { size: 8 }
      };

      if (xErr.length) {
        trace.error_x = {
          type: 'data',
          array: xErr,
          visible: true
        };
      }

      const layout = {
        title: 'Plot with Error Bars',
        xaxis: { title: xLabel },
        yaxis: { title: yLabel }
      };

      Plotly.newPlot('plot', [trace], layout);
    }
  </script>
</body>
</html>
