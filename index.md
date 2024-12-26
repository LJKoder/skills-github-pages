---
title: Welcome to my blog
---
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Number Input Website</title>
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
    </style>
</head>
<body>
    <h1>Enter Numbers</h1>
    <p>Paste numbers in the boxes below and click "Submit" to process.</p>

    <div class="input-section">
        <div>
            <input type="text" id="numberInput1" placeholder="Input 1">
            <br>
            <button onclick="processNumbers('numberInput1', 'output1')">Submit</button>
            <button class="clear-button" onclick="clearInput('numberInput1', 'output1')">Clear</button>
        </div>
        <div>
            <input type="text" id="numberInput2" placeholder="Input 2">
            <br>
            <button onclick="processNumbers('numberInput2', 'output2')">Submit</button>
            <button class="clear-button" onclick="clearInput('numberInput2', 'output2')">Clear</button>
        </div>
    </div>

    <div id="output1" style="margin-top: 20px; font-size: 18px;"></div>
    <div id="output2" style="margin-top: 20px; font-size: 18px;"></div>

    <script>
        function processNumbers(inputId, outputId) {
            const inputBox = document.getElementById(inputId);
            const outputDiv = document.getElementById(outputId);

            // Get the input value
            const numbers = inputBox.value.trim();

            if (numbers) {
                outputDiv.textContent = `You entered: ${numbers}`;
            } else {
                outputDiv.textContent = 'Please enter some numbers.';
            }
        }

        function clearInput(inputId, outputId) {
            const inputBox = document.getElementById(inputId);
            const outputDiv = document.getElementById(outputId);

            // Clear the input box and output div
            inputBox.value = '';
            outputDiv.textContent = '';
        }
    </script>
</body>
</html>
