<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plot X and Y Data</title>
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
</head>
<body>
    <h1>Plot X and Y Data</h1>
    <p>Enter X and Y values (tab or newline-separated) to plot on the graph.</p>

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
    </div>

    <button onclick="plotGraph()">Plot Data</button>
    <button onclick="plotRegressionLine()">Plot Regression Line</button>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        function plotGraph() {
            const canvas = document.getElementById('myChart');
            const ctx = canvas.getContext('2d');

            const xInput = document.getElementById('xValues').value.trim();
            const yInput = document.getElementById('yValues').value.trim();

            if (!xInput || !yInput) {
                alert('Please enter both X and Y values.');
                return;
            }

            const xValues = xInput.split(/\s+/).map(val => parseFloat(val.trim()));
            const yValues = yInput.split(/\s+/).map(val => parseFloat(val.trim()));

            if (xValues.some(isNaN) || yValues.some(isNaN)) {
                alert('Please ensure all inputs are valid numbers.');
                return;
            }

            if (xValues.length !== yValues.length) {
                alert('X and Y values must have the same length.');
                return;
            }

            if (chart) {
                chart.destroy();
            }

            chart = new Chart(ctx, {
                type: 'scatter',
                data: {
                    datasets: [{
                        data: xValues.map((x, i) => ({ x: x, y: yValues[i] })),
                        borderColor: 'rgba(54, 162, 235, 1)',
                        backgroundColor: 'rgba(54, 162, 235, 0.2)',
                        borderWidth: 2,
                        showLine: false,
                        tension: 0.3,
                        pointStyle: 'crossRot',  // Change this to 'cross' to make data points appear as Xs
                        pointBorderColor: 'black',  // Set border color to black for the cross
                        pointBackgroundColor: 'transparent',  // Make the inside of the cross transparent
                        pointRadius: 5  // Size of the cross
                    }]
                },
                options: {
                    responsive: true,
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: 'X Values',
                            },
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Y Values',
                            },
                        },
                    },
                },
            });
        }

        function plotRegressionLine() {
            const xInput = document.getElementById('xValues').value.trim();
            const yInput = document.getElementById('yValues').value.trim();

            if (!xInput || !yInput) {
                alert('Please enter both X and Y values.');
                return;
            }

            const xValues = xInput.split(/\s+/).map(val => parseFloat(val.trim()));
            const yValues = yInput.split(/\s+/).map(val => parseFloat(val.trim()));

            if (xValues.some(isNaN) || yValues.some(isNaN)) {
                alert('Please ensure all inputs are valid numbers.');
                return;
            }

            if (xValues.length !== yValues.length) {
                alert('X and Y values must have the same length.');
                return;
            }

            const weights = Array(xValues.length).fill(1);
            const data = xValues.map((x, i) => [x, yValues[i]]);

            const { beta0, beta1 } = weightedLinearRegressionWithErrors(data, weights);

            const minX = Math.min(...xValues);
            const maxX = Math.max(...xValues);
            const lineXValues = [minX, maxX];
            const lineYValues = lineXValues.map(x => beta1 * x + beta0);

            chart.data.datasets.push({
                label: 'Regression Line',
                data: lineXValues.map((x, i) => ({ x: x, y: lineYValues[i] })),
                borderColor: 'rgba(255, 99, 132, 1)',
                borderWidth: 2,
                showLine: true,
                fill: false,
                tension: 0,
            });
// Add a custom plugin to display the regression equation
Chart.register({
    id: 'regressionEquation',
    beforeDraw(chart) {
        const ctx = chart.ctx;
        const chartArea = chart.chartArea;

        // Calculate position for the text
        const x = chartArea.right - 150; // Adjust the x position
        const y = chartArea.top + 30;   // Adjust the y position

        ctx.save();
        ctx.fillStyle = 'black'; // Text color
        ctx.font = '16px Arial';
        ctx.fillText(`y = ${beta1.toFixed(2)}x + ${beta0.toFixed(2)}`, x, y);
        ctx.restore();
    }
});

// Add regression line dataset
chart.data.datasets.push({
    label: 'Regression Line',
    data: lineXValues.map((x, i) => ({ x: x, y: lineYValues[i] })),
    borderColor: 'rgba(255, 99, 132, 1)',
    borderWidth: 2,
    showLine: true,
    fill: false,
    tension: 0,
});

// Update the chart
chart.update();
        }

        function clearInput(inputId) {
            document.getElementById(inputId).value = '';
        }

        function weightedLinearRegressionWithErrors(data, weights) {
            const n = data.length;
            const x = data.map(d => d[0]);
            const y = data.map(d => d[1]);

            const sum = arr => arr.reduce((a, b) => a + b, 0);
            const sumw = sum(weights);
            const sumwx = sum(x.map((val, i) => weights[i] * val));
            const sumwy = sum(y.map((val, i) => weights[i] * val));
            const sumwxy = sum(x.map((val, i) => weights[i] * val * y[i]));
            const sumwx2 = sum(x.map((val, i) => weights[i] * val * val));

            const xbar = sumwx / sumw;
            const ybar = sumwy / sumw;

            const beta1 = (sumwxy - sumwx * ybar) / (sumwx2 - sumwx * xbar);
            const beta0 = ybar - beta1 * xbar;

            return { beta0, beta1 };
        }
    </script>
</body>
</html>
