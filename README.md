<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stock Ticker</title>
    <!-- Use Tailwind CSS for easy styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f0f4f8;
        }
        .stock-card {
            background-color: #ffffff;
            border: 1px solid #e2e8f0;
            border-radius: 12px;
            padding: 1rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            display: flex;
            align-items: center;
            justify-content: space-between;
            flex-wrap: wrap;
            width: 100%;
            box-sizing: border-box;
        }
    </style>
</head>
<body class="p-4">
    <!-- The main container for the stock ticker -->
    <div id="stock-ticker" class="stock-card">
        <!-- Loading state -->
        <div id="loading" class="text-gray-500 font-semibold animate-pulse">Loading stock data...</div>
    </div>

    <script>
        // Wait for the window to load before running the script
        window.onload = function() {
            const stockTickerEl = document.getElementById('stock-ticker');
            const loadingEl = document.getElementById('loading');

            // Function to fetch and display stock data
            async function getStockData() {
                try {
                    // --- IMPORTANT: This is a mock API call for demonstration. ---
                    // In a real application, you would replace this with a fetch to a real stock API
                    // like Alpha Vantage, Finnhub, or IEX Cloud.
                    // Example: await fetch('https://api.example.com/stocks?symbol=GOOGL');
                    // You would need to sign up for a service and use an API key.

                    // Simulate fetching data with a delay
                    const mockData = {
                        symbol: 'GOOGL',
                        price: 155.75,
                        change: 1.23,
                        changePercent: 0.79,
                    };
                    
                    // Simulate a network delay
                    await new Promise(resolve => setTimeout(resolve, 1500));

                    // Hide the loading message
                    loadingEl.classList.add('hidden');

                    const isPositive = mockData.change >= 0;
                    const changeColor = isPositive ? 'text-green-500' : 'text-red-500';
                    const changeIcon = isPositive ? '▲' : '▼';

                    // Update the HTML content with the fetched data
                    stockTickerEl.innerHTML = `
                        <div class="flex flex-col items-start mr-4">
                            <span class="text-xl md:text-2xl font-bold text-gray-800">${mockData.symbol}</span>
                            <span class="text-sm md:text-base font-medium text-gray-500">Google</span>
                        </div>
                        <div class="flex flex-col items-end">
                            <span class="text-xl md:text-2xl font-bold text-gray-800">$${mockData.price.toFixed(2)}</span>
                            <div class="flex items-center text-sm md:text-base ${changeColor}">
                                <span>${changeIcon}</span>
                                <span class="ml-1">${mockData.change.toFixed(2)} (${mockData.changePercent.toFixed(2)}%)</span>
                            </div>
                        </div>
                    `;

                } catch (error) {
                    // Handle errors gracefully
                    console.error('Error fetching stock data:', error);
                    stockTickerEl.innerHTML = `<p class="text-red-500 font-semibold">Failed to load stock data.</p>`;
                }
            }
            
            // Call the function to display data on load
            getStockData();
            
            // Set an interval to refresh the data (e.g., every 5 minutes)
            // Note: Be mindful of API rate limits when using a real API
            setInterval(getStockData, 300000); // 300000ms = 5 minutes
        };
    </script>
</body>
</html>
