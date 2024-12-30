<script>
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

        // Create weights (equal weights in this case)
        const weights = Array(xValues.length).fill(1);

        // Combine x and y values into data points
        const data = xValues.map((x, i) => [x, yValues[i]]);

        // Compute regression line
        const { beta0, beta1 } = weightedLinearRegressionWithErrors(data, weights);

        // Generate points for the regression line
        const minX = Math.min(...xValues);
        const maxX = Math.max(...xValues);
        const lineXValues = [minX, maxX];
        const lineYValues = lineXValues.map(x => beta1 * x + beta0);

        // Destroy the previous chart instance if it exists
        if (chart) {
            chart.destroy();
        }

        // Create the new chart
        chart = new Chart(ctx, {
            type: 'scatter',
            data: {
                datasets: [
                    {
                        data: xValues.map((x, i) => ({ x: x, y: yValues[i] })),
                        borderColor: 'rgba(54, 162, 235, 1)',
                        backgroundColor: 'rgba(54, 162, 235, 0.2)',
                        borderWidth: 2,
                        showLine: false,
                        pointStyle: 'crossRot',
                        pointRadius: 5,
                    },
                    {
                        label: 'Regression Line',
                        
