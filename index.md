<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Chart with Error Bars</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-error-bars"></script>
  <style>
    body { font-family: Arial; text-align: center; margin: 40px; }
    input, button { margin: 8px; padding: 6px; }
    #chart-container { height: 500px; }
  </style>
</head>
<body>
  <h1>Chart with Error Bars</h1>

  <input type="text" id="xValues" placeholder="X values (e.g. 1 2 3)">
  <input type="text" id="yValues" placeholder="Y values (e.g. 4 5 6)">
  <input type="text" id="yErrors" placeholder="Y errors (e.g. 0.5 0.4 0.6)">
  <button onclick="plotGraph()">Plot</button>

  <div id="chart-container">
    <canvas id="myChart"></canvas>
  </div>

  <script>
    let chart;

    function parse(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(Number);
    }

    function plotGraph() {
      const x = parse("xValues");
      const y = parse("yValues");
      const yerr = parse("yErrors");

      if (x.length !== y.length || y.length !== yerr.length) {
        alert("All inputs must have the same length");
        return;
      }

      const data = x.map((xv, i) => ({
        x: xv,
        y: y[i],
        yMin: y[i] - yerr[i],
        yMax: y[i] + yerr[i],
      }));

      if (chart) chart.destroy();

      chart = new Chart(document.getElementById("myChart"), {
        type: 'scatter',
        data: {
          datasets: [{
            label: "Data with Y error bars",
            data,
            backgroundColor: 'rgba(54, 162, 235, 0.7)',
            borderColor: 'rgba(54, 162, 235, 1)',
            pointRadius: 5,
          }]
        },
        options: {
          plugins: {
            legend: { display: true }
          },
          scales: {
            x: { title: { display: true, text: "X Axis" } },
            y: { title: { display: true, text: "Y Axis" } }
          }
        }
      });
    }
  </script>
</body>
</html>
