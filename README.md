
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Genesis Expense Tracker</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Poppins', sans-serif;
        }
        body {
            background: #0f172a;
            color: white;
            text-align: center;
            padding: 20px;
        }
        .container {
            max-width: 500px;
            background: rgba(255, 255, 255, 0.1);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0px 8px 16px rgba(0, 0, 0, 0.4);
            backdrop-filter: blur(15px);
            margin: auto;
        }
        h1 {
            color: #38bdf8;
            margin-bottom: 15px;
            text-shadow: 0 0 10px #38bdf8;
        }
        input, select, button {
            width: 100%;
            padding: 12px;
            margin: 10px 0;
            border-radius: 8px;
            border: none;
            font-size: 16px;
            outline: none;
            transition: 0.3s;
        }
        input, select {
            background: rgba(255, 255, 255, 0.2);
            color: white;
        }
        input::placeholder, select {
            color: #ddd;
        }
        button {
            background: #38bdf8;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: 0.3s;
            box-shadow: 0 0 10px #38bdf8;
        }
        button:hover {
            background: #0ea5e9;
            transform: scale(1.05);
        }
        .transactions {
            margin-top: 15px;
            text-align: left;
            padding: 0;
        }
        .transactions li {
            list-style: none;
            padding: 10px;
            background: rgba(255, 255, 255, 0.2);
            margin: 5px 0;
            border-radius: 5px;
            backdrop-filter: blur(5px);
            color: white;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .transactions .date {
            font-size: 12px;
            color: #aaa;
        }
        .transactions .delete-btn {
            background: red;
            border: none;
            padding: 5px 10px;
            cursor: pointer;
            border-radius: 5px;
            color: white;
            font-size: 14px;
        }
        .balance {
            font-size: 20px;
            margin-bottom: 10px;
            font-weight: bold;
        }
        .balance-bar {
            width: 100%;
            height: 15px;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 10px;
            overflow: hidden;
            margin-bottom: 10px;
        }
        .balance-fill {
            height: 100%;
            background: #38bdf8;
            width: 100%;
            transition: width 0.5s;
        }
        .popup {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(255, 0, 0, 0.9);
            color: white;
            padding: 20px;
            border-radius: 10px;
            display: none;
            box-shadow: 0 0 20px red;
        }
        .popup button {
            background: white;
            color: red;
            font-weight: bold;
            border: none;
            padding: 8px 15px;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
        }
    </style>
</head>
<body>

    <h1>Genesis Expense Tracker 💰</h1>
    <div class="container">
        
        <!-- Reset Button -->
        <button onclick="resetExpenses()">🔄 Reset Expenses</button>

        <!-- Salary Input -->
        <input type="number" id="salary" placeholder="Enter Your Salary (₹)" oninput="updateSalary()" required>
        
        <!-- Spending Limit Input -->
        <input type="number" id="spendingLimit" placeholder="Set Your Spending Limit (₹)" required>
        
        <p class="balance">💰 Salary Left: <span id="totalBalance">₹0</span></p>
        
        <!-- Live Balance Bar -->
        <div class="balance-bar"><div class="balance-fill" id="balanceFill"></div></div>
        
        <!-- Expense Input -->
        <input type="number" id="amount" placeholder="Enter Expense Amount (₹)" required>
        
        <!-- Category Selection -->
        <select id="categorySelect" onchange="toggleCustomCategory()">
            <option value="Food">Food</option>
            <option value="Transport">Transport</option>
            <option value="Shopping">Shopping</option>
            <option value="Entertainment">Entertainment</option>
            <option value="custom">Custom...</option>
        </select>
        
        <!-- Custom Category Input -->
        <input type="text" id="customCategory" placeholder="Enter Custom Category" style="display:none;">
        
        <!-- Add Expense Button -->
        <button onclick="addExpense()">➕ Add Expense</button>
        
        <!-- Transactions List -->
        <ul class="transactions" id="transactionList"></ul>
    </div>

    <!-- Popup Alert -->
    <div class="popup" id="popup">
        ⚠️ You have exceeded your spending limit! Please control your expenses.
        <br>
        <button onclick="closePopup()">OK</button>
    </div>

    <script>
        let salary = 0;
        let remainingBalance = 0;
        let spendingLimit = 0;
        let transactions = [];

        function updateSalary() {
            salary = parseFloat(document.getElementById("salary").value) || 0;
            spendingLimit = parseFloat(document.getElementById("spendingLimit").value) || 0;
            remainingBalance = salary;
            updateUI();
        }

        function toggleCustomCategory() {
            let categorySelect = document.getElementById("categorySelect").value;
            let customInput = document.getElementById("customCategory");

            if (categorySelect === "custom") {
                customInput.style.display = "block";
                customInput.focus();
            } else {
                customInput.style.display = "none";
                customInput.value = "";
            }
        }

        function addExpense() {
            let amount = parseFloat(document.getElementById("amount").value);
            let category = document.getElementById("categorySelect").value;
            let customCategory = document.getElementById("customCategory").value.trim();
            let date = new Date().toLocaleString();

            if (category === "custom" && customCategory !== "") {
                category = customCategory;
            }

            transactions.push({ amount, category, date });
            remainingBalance -= amount;
            updateUI();
        }

        function updateUI() {
            document.getElementById("totalBalance").innerText = `₹${remainingBalance.toFixed(2)}`;
            let transactionList = document.getElementById("transactionList");
            transactionList.innerHTML = "";

            transactions.forEach((t, index) => {
                transactionList.innerHTML += `
                    <li>${t.category} - ₹${t.amount} <span class="date">${t.date}</span> 
                        <button class="delete-btn" onclick="removeExpense(${index})">❌</button>
                    </li>`;
            });

            if (remainingBalance < spendingLimit) {
                document.getElementById("popup").style.display = "block";
            }
        }

        function removeExpense(index) {
            transactions.splice(index, 1);
            updateUI();
        }

        function resetExpenses() {
            transactions = [];
            updateUI();
        }

        function closePopup() {
            document.getElementById("popup").style.display = "none";
        }
    </script>

</body>
</html>
