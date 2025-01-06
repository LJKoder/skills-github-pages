<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plot X, Y Data with Error Bars</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin: 50px;
        }

        .input-section {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin: 20px 0;
        }

        input[type="text"] {
            width: 200px;
            padding: 10px;
            font-size: 16px;
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
            margin-left: 10px;
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
    <script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-errorbars"></script>
</head>
<body>
    <h1>Plot X, Y Data with Error Bars</h1>
    <p>Enter X, Y values and their uncertainties (tab or newline-separated) to plot with error bars.</p>

    <div class="input-section">
        <div>
            <input type="text" id="xValues" placeholder="X Values">
            <br>
            <button class="clear-button" onclick="clearInput('xValues')">Clear</button>
        </div>
        <div>
            <input type="text" id="yValues" placeholder="Y Values">
            <br>
            <button class="clear-button" onclick="clearInput('yValues')">Clear</button>
        </div>
        <div>
            <input type="text" id="xUncertainty" placeholder="X Uncertainty">
            <br>
            <button class="clear-button" onclick="clearInput('xUncertainty')">Clear</button>
        </div>
        <div>
            <input type="text" id="yUncertainty" placeholder="Y Uncertainty">
            <br>
            <button class="clear-button" onclick="clearInput('yUncertainty')">Clear</button>
        </div>
    </div>

    <button onclick="plotGraph()">Plot Data</button>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        // Register the error bar plugin
        Chart.register(ChartErrorBars);

        function plotGraph() {
            const canvas = document.getElementById('myChart');
            const ctx = canvas.getContext('2d');

            const xInput = document.getElementById('xValues').value.trim();
            const yInput = document.getElementById('yValues').value.trim();
            const xUncertaintyInput = document.getElementById('xUncertainty').value.trim();
            const yUncertaintyInput = document.getElementById('yUncertainty').value.trim();

            if (!xInput || !yInput || !xUncertaintyInput || !yUncertaintyInput) {
                alert('Please enter X, Y values, and their uncertainties.');
                return;
            }

            const xValues = xInput.split(/\s+/).map(val => parseFloat(val.trim()));
            const yValues = yInput.split(/\s+/).map(val => parseFloat(val.trim()));
            const xUncertainties = xUncertaintyInput.split(/\s+/).map(val => parseFloat(val.trim()));
            const yUncertainties = yUncertaintyInput.split(/\s+/).map(val => parseFloat(val.trim()));

            if (
                xValues.some(isNaN) || yValues.some(isNaN) ||
                xUncertainties.some(isNaN) || yUncertainties.some(isNaN)
            ) {
                alert('Ensure all inputs are valid numbers.');
                return;
            }

            if (
                xValues.length !== yValues.length ||
                xUncertainties.length !== xValues.length ||
                yUncertainties.length !== yValues.length
            ) {
                alert('Ensure all input arrays have the same length.');
                return;
            }

            if (chart) {
                chart.destroy();
            }

            chart = new Chart(ctx, {
                type: 'scatter',
                data: {
                    datasets: [{
                        label: 'Data with Error Bars',
                        data: xValues.map((x, i) => ({
                            x: x,
                            y: yValues[i],
                            xMin: x - xUncertainties[i],
                            xMax: x + xUncertainties[i],
                            yMin: yValues[i] - yUncertainties[i],
                            yMax: yValues[i] + yUncertainties[i],
                        })),
                        borderColor: 'rgba(54, 162, 235, 1)',
                        backgroundColor: 'rgba(54, 162, 235, 0.2)',
                        errorBarColor: 'rgba(0, 0, 0, 0.8)',
                        pointRadius: 5,
                        showLine: false,
                    }]
                },
                options: {
                    responsive: true,
                    scales: {
                        x: { title: { display: true, text: 'X Values' } },
                        y: { title: { display: true, text: 'Y Values' } }
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(tooltipItem) {
                                    const data = tooltipItem.raw;
                                    return `X: ${data.x} (±${data.xMax - data.x})\nY: ${data.y} (±${data.yMax - data.y})`;
                                }
                            }
                        }
                    }
                }
            });
        }

        function clearInput(inputId) {
            document.getElementById(inputId).value = '';
        }
    </script>
</body>
</html>
