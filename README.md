# Hi there 👋

I Like Money :)

---

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
            margin-bottom: 1rem;
        }
    </style>
</head>
<body class="p-4">
    <!-- The main container for the stock ticker -->
    <div id="stock-ticker" class="stock-card">
        <!-- Loading state -->
        <div id="loading" class="text-gray-500 font-semibold animate-pulse">Loading stock data...</div>
    </div>

    <!-- Container for the LLM-powered chat -->
    <div id="llm-container" class="stock-card flex-col items-start space-y-4">
        <h2 class="text-lg md:text-xl font-bold text-gray-800">Ask the AI about Stocks</h2>
        <input type="text" id="llm-input" class="w-full p-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500" placeholder="e.g., Explain what affects stock prices?">
        <button id="llm-button" class="w-full px-4 py-2 bg-blue-600 text-white font-semibold rounded-md hover:bg-blue-700 transition-colors duration-200">Get Insight</button>
        <div id="llm-response" class="w-full p-4 mt-2 bg-gray-100 rounded-md text-gray-700 text-sm md:text-base min-h-[50px]">
            <!-- Response from the LLM will appear here -->
        </div>
    </div>

    <script>
        // Wait for the window to load before running the script
        window.onload = function() {
            const stockTickerEl = document.getElementById('stock-ticker');
            const loadingEl = document.getElementById('loading');
            const llmInput = document.getElementById('llm-input');
            const llmButton = document.getElementById('llm-button');
            const llmResponse = document.getElementById('llm-response');

            // Function to fetch and display mock stock data
            async function getStockData() {
                try {
                    // --- IMPORTANT: This is a mock API call for demonstration. ---
                    // In a real application, you would replace this with a fetch to a real stock API
                    const mockData = {
                        symbol: 'GOOGL',
                        price: (Math.random() * 10 + 150).toFixed(2),
                        change: (Math.random() * 5 - 2.5).toFixed(2),
                        changePercent: (Math.random() * 2 - 1).toFixed(2),
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
                            <span class="text-xl md:text-2xl font-bold text-gray-800">$${mockData.price}</span>
                            <div class="flex items-center text-sm md:text-base ${changeColor}">
                                <span>${changeIcon}</span>
                                <span class="ml-1">${mockData.change} (${mockData.changePercent}%)</span>
                            </div>
                        </div>
                    `;

                } catch (error) {
                    // Handle errors gracefully
                    console.error('Error fetching stock data:', error);
                    stockTickerEl.innerHTML = `<p class="text-red-500 font-semibold">Failed to load stock data.</p>`;
                }
            }
            
            // Function to call the Gemini API and get an LLM response
            async function getLLMResponse(prompt) {
                // Exponential backoff logic for retries
                const MAX_RETRIES = 3;
                let retryCount = 0;
                let delay = 1000;

                while (retryCount < MAX_RETRIES) {
                    try {
                        const apiKey = "";
                        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;
                        
                        const payload = {
                            contents: [{
                                role: "user",
                                parts: [{ text: prompt }]
                            }]
                        };

                        const response = await fetch(apiUrl, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });

                        if (!response.ok) {
                            throw new Error(`HTTP error! status: ${response.status}`);
                        }

                        const result = await response.json();
                        
                        if (result.candidates && result.candidates.length > 0 &&
                            result.candidates[0].content && result.candidates[0].content.parts &&
                            result.candidates[0].content.parts.length > 0) {
                            return result.candidates[0].content.parts[0].text;
                        } else {
                            throw new Error("Invalid API response format");
                        }

                    } catch (error) {
                        retryCount++;
                        console.error(`Attempt ${retryCount} failed:`, error);
                        if (retryCount < MAX_RETRIES) {
                            await new Promise(res => setTimeout(res, delay));
                            delay *= 2; // Exponentially increase the delay
                        } else {
                            throw error;
                        }
                    }
                }
            }
            
            // Event listener for the LLM button
            llmButton.addEventListener('click', async () => {
                const prompt = llmInput.value.trim();
                if (!prompt) {
                    llmResponse.textContent = "Please enter a question to get an insight.";
                    return;
                }
                
                // Show a loading state
                llmButton.disabled = true;
                llmButton.textContent = "Generating...";
                llmResponse.textContent = "";

                try {
                    // Prepend a stock-related context to the user's prompt
                    const fullPrompt = `You are an AI assistant specialized in providing brief, easy-to-understand insights about finance and stock markets. Respond to the user's question '${prompt}' concisely and helpfully. Do not provide financial advice.`;
                    
                    const text = await getLLMResponse(fullPrompt);
                    llmResponse.textContent = text;
                } catch (error) {
                    console.error('Failed to get LLM response:', error);
                    llmResponse.textContent = "Sorry, an error occurred while getting the insight. Please try again later.";
                } finally {
                    llmButton.disabled = false;
                    llmButton.textContent = "Get Insight";
                }
            });

            // Call the function to display data on load
            getStockData();
            
            // Set an interval to refresh the data (e.g., every 5 minutes)
            setInterval(getStockData, 300000); // 300000ms = 5 minutes
        };
    </script>
</body>
</html>


---

### My Goofy GitHub Stats

<div align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=kimmyjay726&show_icons=true&theme=radical" alt="Your GitHub Stats" />
  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=kimmyjay726&layout=compact&theme=radical" alt="Top Languages" />
</div>

---

### My Skills & Tech Stack

| Language | Framework | Tools |
| :--- | :--- | :--- |
| ![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white) | ![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB) | ![Git](https://img.shields.io/badge/Git-F05032?style=for-the-badge&logo=git&logoColor=white) |
| ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black) | ![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white) | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white) |
