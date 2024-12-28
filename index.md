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
            width: 100%;  /* Full width */
            height: 80vh; /* Height adjusted to viewport */
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

    <button onclick="plotGraph()">Plot</button>

    <div id="chart-container">
        <canvas id="myChart"></canvas>
    </div>

    <script>
        let chart; // To store the Chart.js instance

        function plotGraph() {
            const canvas = document.getElementById('myChart');
            const ctx = canvas.getContext('2d');

            // Set the size dynamically based on window size or other parameters
            canvas.width = window.innerWidth * 0.8;  // 80% of the window width
            canvas.height = window.innerHeight * 0.6; // 60% of the window height
            const xInput = document.getElementById('xValues').value.trim();
            const yInput = document.getElementById('yValues').value.trim();

            if (!xInput || !yInput) {
                alert('Please enter both X and Y values.');
                return;
            }

            // Split by tabs or newlines to handle multiline Excel input
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

            // Destroy the previous chart instance if it exists
            if (chart) {
                chart.destroy();
            }

            // Create the new chart
            chart = new Chart(ctx, {
                type: 'scatter',
                data: {
datasets: [{
    data: xValues.map((x, i) => ({ x: x, y: yValues[i] })),
    borderColor: 'rgba(54, 162, 235, 1)',
    backgroundColor: 'rgba(54, 162, 235, 0.2)',
    borderWidth: 2,
    showLine: true,
    tension: 0.3,
    pointStyle: 'crossRot',  // Change this to 'cross' to make data points appear as Xs
    pointBorderColor: 'black',  // Set border color to black for the cross
    pointBackgroundColor: 'transparent',  // Make the inside of the cross transparent
    pointRadius: 5  // Size of the cross
}]
                },
                options: {
                    responsive: true,
                    aspectRatio: 2, // Maintains a 2:1 width-to-height ratio
                    plugins: {
                        legend: {
                            display: false,
                            position: 'top',
                            labels: {
                                font: {
                                    family: 'Arial',
                                    size: 14,
                                    weight: 'bold'
                                }
                            }
                        }
                    },
                    scales: {
                        x: {
                            title: {
                                display: true,
                                text: 'X Values',
                                font: {
                                    family: 'Arial',
                                    size: 16,
                                    weight: 'bold'
                                },
                            color: 'black'  // Set the color of the X axis title to black
                            },
                            grid: {
                                color: 'rgba(200, 200, 200, 0.5)',
                                lineWidth: 0.5
                            },
                            ticks: {
                                font: {
                                    family: 'Arial',
                                    size: 12
                                },
                                color: 'black'
                            }
                        },
                        y: {
                            title: {
                                display: true,
                                text: 'Y Values',
                                font: {
                                    family: 'Arial',
                                    size: 16,
                                    weight: 'bold'
                                },
                            color: 'black'  // Set the color of the y axis title to black
                            },
                            ticks: {
                                font: {
                                    family: 'Arial',
                                    size: 12
                                },
                                color: 'black'
                            }
                        }
                    },
                    layout: {
                        padding: 20
                        
                    },
                    elements: {
                        point: {
                            radius: 5,
                            hoverRadius: 7,
                            borderWidth: 1
                        }
                    },
                    backgroundColor: 'white'
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
