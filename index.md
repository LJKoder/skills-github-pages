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
        }

        canvas {
            max-width: 600px;
            margin: 0 auto;
        }
    </style>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
    <h1>Plot X and Y Data</h1>
    <p>Enter X and Y values (comma-separated) to plot on the graph.</p>

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

    <button onclick="plotGraph()">Plot</button>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        function plotGraph() {
            const xInput = document.getElementById('xValues').value.trim();
            const yInput = document.getElementById('yValues').value.trim();

            if (!xInput || !yInput) {
                alert('Please enter both X and Y values.');
                return;
            }

            const xValues = xInput.split(',').map(val => parseFloat(val.trim()));
            const yValues = yInput.split(',').map(val => parseFloat(val.trim()));

            if (xValues.some(isNaN) || yValues.some(isNaN)) {
                alert('Please ensure all inputs are valid numbers.');
                return;
            }

            if (xValues.length !== yValues.length) {
                alert('X and Y values must have the same length.');
                return;
            }

            // Destroy the previous chart instance if it exists
            if (chart) {
                chart.destroy();
            }

            // Create the chart
            const ctx = document.getElementById('myChart').getContext('2d');
            chart = new Chart(ctx, {
                type: 'scatter',
                data: {
                    datasets: [{
                        label: 'X vs Y Plot',
                        data: xValues.map((x, i) => ({ x: x, y: yValues[i] })),
                        borderColor: 'rgba(75, 192, 192, 1)',
                        backgroundColor: 'rgba(75, 192, 192, 0.2)',
                        borderWidth: 2,
                        showLine: true, // Connect points with a line
                        tension: 0.4 // Smooth the line
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: true
                        }
                    },
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: 'X Values'
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Y Values'
                            }
                        }
                    }
                }
            });
        }

        function clearInput(inputId) {
            const inputBox = document.getElementById(inputId);
            inputBox.value = '';
        }
    </script>
</body>
</html>
