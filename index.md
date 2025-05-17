<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Chart with Error Bars</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-error-bars@3.0.0/dist/chartjs-plugin-error-bars.min.js"></script>
  <style>
    body { font-family: Arial, sans-serif; text-align: center; padding: 20px; }
    input, button { margin: 5px; padding: 8px; width: 250px; }
    canvas { max-width: 800px; margin-top: 20px; }
  </style>
</head>
<body>

  <h2>Scatter Plot with Y Error Bars 1</h2>

  <div>
    <input id="xValues" type="text" placeholder="X values (e.g. 1 2 3 4)">
    <input id="yValues" type="text" placeholder="Y values (e.g. 5 6 7 8)">
    <input id="yErrors" type="text" placeholder="Y errors (e.g. 0.2 0.3 0.2 0.4)">
    <br>
    <button onclick="plotGraph()">Plot with Error Bars</button>
  </div>

  <canvas id="myChart"></canvas>

  <script>
    let chart;

    function parseInput(id) {
      return document.getElementById(id).value.trim().split(/\s+/).map(Number);
    }

    function plotGraph() {
      const x = parseInput("xValues");
      const y = parseInput("yValues");
      const yerr = parseInput("yErrors");

      if (x.length !== y.length || y.length !== yerr.length) {
        alert("Lengths of all arrays must match.");
        return;
      }

      const errorBars = {};
      y.forEach((_, i) => {
        errorBars[i] = { plus: yerr[i], minus: yerr[i] };
      });

      const data = x.map((xv, i) => ({ x: xv, y: y[i] }));

      if (chart) chart.destroy();

      chart = new Chart(document.getElementById("myChart"), {
        type: "scatter",
        data: {
          datasets: [{
            label: "Data with Y Error Bars",
            data,
            errorBarData: {
              y: errorBars
            },
            backgroundColor: "rgba(75,192,192,0.6)",
            borderColor: "rgba(75,192,192,1)",
            pointRadius: 5
          }]
        },
        options: {
          responsive: true,
          plugins: {
            legend: { display: true },
            tooltip: { enabled: true }
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
