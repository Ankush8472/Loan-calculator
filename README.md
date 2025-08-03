<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Financial Clarity: Loan Calculator</title>
    <!-- Include Chart.js library for data visualization -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* Modern & Clean CSS Styling */
        :root {
            --primary-color: #005A9C; /* A professional blue */
            --secondary-color: #F0F8FF; /* Light alice blue */
            --text-color: #333;
            --border-color: #ddd;
            --shadow-color: rgba(0, 0, 0, 0.1);
            --success-color: #28a745;
            --danger-color: #dc3545;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
            background-color: #f4f7f9;
            color: var(--text-color);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: flex-start;
        }

        .container {
            width: 100%;
            max-width: 900px;
            background: #fff;
            border-radius: 12px;
            box-shadow: 0 4px 20px var(--shadow-color);
            overflow: hidden;
        }

        header {
            background-color: var(--primary-color);
            color: #fff;
            padding: 20px;
            text-align: center;
        }

        header h1 {
            margin: 0;
            font-size: 1.8em;
        }

        main {
            display: flex;
            flex-wrap: wrap;
        }

        .form-section {
            flex: 1;
            min-width: 300px;
            padding: 25px;
            border-right: 1px solid var(--border-color);
        }

        .results-section {
            flex: 2;
            min-width: 400px;
            padding: 25px;
            background-color: var(--secondary-color);
        }
        
        @media (max-width: 768px) {
            .form-section { border-right: none; border-bottom: 1px solid var(--border-color); }
        }

        .form-group {
            margin-bottom: 20px;
        }

        .form-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
        }

        .form-group input {
            width: 100%;
            padding: 10px;
            border: 1px solid var(--border-color);
            border-radius: 6px;
            font-size: 1em;
            box-sizing: border-box;
        }

        button {
            width: 100%;
            padding: 12px;
            background-color: var(--primary-color);
            color: #fff;
            border: none;
            border-radius: 6px;
            font-size: 1.1em;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        button:hover {
            background-color: #004170;
        }

        .summary {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 25px;
        }

        .summary-box {
            background: #fff;
            padding: 15px;
            border-radius: 8px;
            border: 1px solid var(--border-color);
            text-align: center;
        }

        .summary-box h3 {
            margin: 0 0 5px 0;
            font-size: 1em;
            color: #555;
        }

        .summary-box p {
            margin: 0;
            font-size: 1.5em;
            font-weight: 700;
            color: var(--primary-color);
        }

        .summary-box.total-interest p {
            color: var(--danger-color);
        }
    </style>
</head>
<body>

    <div class="container">
        <header>
            <h1>Financial Clarity: Loan Calculator</h1>
        </header>
        <main>
            <section class="form-section">
                <h2>Loan Details</h2>
                <form id="loan-form">
                    <div class="form-group">
                        <label for="loanAmount">Loan Amount (₹)</label>
                        <input type="number" id="loanAmount" placeholder="e.g., 500000" required>
                    </div>
                    <div class="form-group">
                        <label for="interestRate">Annual Interest Rate (%)</label>
                        <input type="number" id="interestRate" step="0.01" placeholder="e.g., 10.5" required>
                    </div>
                    <div class="form-group">
                        <label for="loanTenure">Loan Tenure (Years)</label>
                        <input type="number" id="loanTenure" placeholder="e.g., 5" required>
                    </div>
                    <button type="submit">Calculate</button>
                </form>
            </section>
            <section class="results-section">
                <h2>Your Financial Breakdown</h2>
                <div id="results-summary" class="summary">
                    <!-- Results will be injected here by JavaScript -->
                </div>
                <div class="chart-container">
                    <canvas id="loanChart"></canvas>
                </div>
            </section>
        </main>
    </div>

    <script>
        // All the interactive logic is here
        const loanForm = document.getElementById('loan-form');
        const resultsSummary = document.getElementById('results-summary');
        const loanChartCanvas = document.getElementById('loanChart');
        let myChart; // To hold the chart instance

        loanForm.addEventListener('submit', function(e) {
            e.preventDefault(); // Prevent form from submitting the traditional way

            // 1. Get user inputs
            const loanAmount = parseFloat(document.getElementById('loanAmount').value);
            const annualInterestRate = parseFloat(document.getElementById('interestRate').value);
            const loanTenureYears = parseFloat(document.getElementById('loanTenure').value);

            // 2. Validate inputs
            if (isNaN(loanAmount) || isNaN(annualInterestRate) || isNaN(loanTenureYears) || loanAmount <= 0 || annualInterestRate <= 0 || loanTenureYears <= 0) {
                alert("Please enter valid positive numbers for all fields.");
                return;
            }

            // 3. Perform Calculations
            const monthlyInterestRate = annualInterestRate / 100 / 12;
            const numberOfPayments = loanTenureYears * 12;

            // Monthly Payment (EMI) Formula: M = P [ i(1 + i)^n ] / [ (1 + i)^n – 1 ]
            const monthlyPayment = loanAmount * (monthlyInterestRate * Math.pow(1 + monthlyInterestRate, numberOfPayments)) / (Math.pow(1 + monthlyInterestRate, numberOfPayments) - 1);

            const totalPayment = monthlyPayment * numberOfPayments;
            const totalInterest = totalPayment - loanAmount;

            // 4. Display Summary Results
            displaySummary(monthlyPayment, loanAmount, totalInterest);
            
            // 5. Generate data for the chart
            generateChartData(loanAmount, monthlyInterestRate, numberOfPayments, monthlyPayment, totalInterest);
        });

        function displaySummary(monthlyPayment, principal, totalInterest) {
            resultsSummary.innerHTML = `
                <div class="summary-box">
                    <h3>Monthly Payment (EMI)</h3>
                    <p>₹ ${monthlyPayment.toLocaleString('en-IN', { maximumFractionDigits: 2 })}</p>
                </div>
                <div class="summary-box">
                    <h3>Principal Amount</h3>
                    <p>₹ ${principal.toLocaleString('en-IN', { maximumFractionDigits: 2 })}</p>
                </div>
                <div class="summary-box total-interest">
                    <h3>Total Interest Payable</h3>
                    <p>₹ ${totalInterest.toLocaleString('en-IN', { maximumFractionDigits: 2 })}</p>
                </div>
                <div class="summary-box">
                    <h3>Total Payment</h3>
                    <p>₹ ${(principal + totalInterest).toLocaleString('en-IN', { maximumFractionDigits: 2 })}</p>
                </div>
            `;
        }

        function generateChartData(principal, monthlyInterestRate, numberOfPayments, monthlyPayment, totalInterest) {
            let remainingBalance = principal;
            const labels = [];
            const principalData = [];
            const interestData = [];
            let cumulativeInterestPaid = 0;

            for (let i = 1; i <= numberOfPayments; i++) {
                const interestForMonth = remainingBalance * monthlyInterestRate;
                cumulativeInterestPaid += interestForMonth;
                const principalForMonth = monthlyPayment - interestForMonth;
                remainingBalance -= principalForMonth;

                // We only add data for each year to keep the chart clean
                if (i % 12 === 0 || i === 1) {
                    labels.push(`Year ${Math.ceil(i / 12)}`);
                    principalData.push(principal - remainingBalance); // Cumulative principal paid
                    interestData.push(cumulativeInterestPaid);
                }
            }
            
            // Add the final state to ensure the chart always ends at the total amount
            if (numberOfPayments % 12 !== 0){
                labels.push(`Year ${Math.ceil(numberOfPayments / 12)}`);
                principalData.push(principal);
                interestData.push(totalInterest);
            }


            // Create or update the chart
            updateChart(labels, principalData, interestData);
        }
        
        function updateChart(labels, principalData, interestData) {
            if (myChart) {
                myChart.destroy(); // Destroy old chart before creating a new one
            }
            const data = {
                labels: labels,
                datasets: [
                    {
                        label: 'Cumulative Principal Paid',
                        data: principalData,
                        backgroundColor: 'rgba(0, 90, 156, 0.7)',
                        borderColor: 'rgba(0, 90, 156, 1)',
                        borderWidth: 1
                    },
                    {
                        label: 'Cumulative Interest Paid',
                        data: interestData,
                        backgroundColor: 'rgba(220, 53, 69, 0.7)',
                        borderColor: 'rgba(220, 53, 69, 1)',
                        borderWidth: 1
                    }
                ]
            };

            myChart = new Chart(loanChartCanvas, {
                type: 'bar',
                data: data,
                options: {
                    responsive: true,
                    scales: {
                        x: {
                            stacked: true,
                        },
                        y: {
                            stacked: true,
                            beginAtZero: true,
                            ticks: {
                                callback: function(value) { return '₹ ' + (value/1000) + 'k' }
                            }
                        }
                    },
                    plugins: {
                        tooltip: {
                            callbacks: {
                                label: function(context) {
                                    let label = context.dataset.label || '';
                                    if (label) {
                                        label += ': ';
                                    }
                                    if (context.parsed.y !== null) {
                                        label += new Intl.NumberFormat('en-IN', { style: 'currency', currency: 'INR' }).format(context.parsed.y);
                                    }
                                    return label;
                                }
                            }
                        }
                    }
                }
            });
        }
    </script>

</body>
</html>
