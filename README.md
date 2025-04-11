<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Smart Expense Tracker</title>
  <style>
    :root {
      --bg-color: #f0f2f5;
      --text-color: #1a1a1a;
      --card-color: #ffffff;
      --accent-color: #00bfa6;
      --input-bg: #e0e0e0;
    }

    [data-theme="dark"] {
      --bg-color: #1e2a38;
      --text-color: #f1f1f1;
      --card-color: #2e3d51;
      --accent-color: #00ffc3;
      --input-bg: #3b4d64;
    }

    * {
      margin: 0;
      padding: 0;
      box-sizing: border-box;
    }

    body {
      background: var(--bg-color);
      font-family: 'Segoe UI', sans-serif;
      color: var(--text-color);
      display: flex;
      justify-content: center;
      padding: 40px 20px;
    }

    .tracker {
      background: var(--card-color);
      padding: 30px;
      border-radius: 15px;
      max-width: 500px;
      width: 100%;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
    }

    .header {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    h1 {
      font-size: 24px;
      color: var(--accent-color);
    }

    .Theme-btn {
      background: var(--accent-color);
      border: none;
      color: #000;
      padding: 5px 10px;
      border-radius: 8px;
      cursor: pointer;
      font-weight: bold;
    }

    .balance {
      text-align: center;
      font-size: 20px;
      margin: 20px 0;
    }

    .limit-warning {
      text-align: center;
      color: red;
      font-weight: bold;
      margin-bottom: 10px;
    }

    form, .filter {
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    input, select, button {
      padding: 10px;
      font-size: 16px;
      border: none;
      border-radius: 10px;
      background: var(--input-bg);
      color: var(--text-color);
    }

    button[type="submit"] {
      background: var(--accent-color);
      color: #000;
      font-weight: bold;
      cursor: pointer;
    }

    .expenses {
      margin-top: 20px;
      max-height: 250px;
      overflow-y: auto;
    }

    .expense {
      display: flex;
      flex-direction: column;
      background: #dfe6e9;
      padding: 10px;
      border-radius: 8px;
      margin-bottom: 10px;
    }

    [data-theme="dark"] .expense {
      background: #3b4d64;
    }

    .expense-top {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .expense span {
      margin: 2px 0;
    }

    .actions {
      display: flex;
      gap: 10px;
    }

    .actions button {
      padding: 5px 8px;
      font-size: 12px;
      border-radius: 6px;
      cursor: pointer;
    }

    .edit-btn {
      background: #ffd32a;
      color: #000;
    }

    .delete-btn {
      background: #ff6b6b;
      color: white;
    }

    .category-summary {
      margin-top: 15px;
      font-size: 14px;
      font-weight: bold;
    }
  </style>
</head>
<body data-theme="light">

  <div class="tracker">
    <div class="header">
      <h1>Expense Tracker</h1>
      <button class="Theme-btn" onclick="ThemeTheme()">🌗 Theme</button>
    </div>

    <div class="balance">Balance: ₹<span id="balance">0.00</span></div>
    <div id="limit-warning" class="limit-warning" style="display: none;">
      🚨 You have exceeded your ₹6000 spending limit!
    </div>

    <form id="expense-form">
      <input type="text" id="description" placeholder="Expense Description" required />
      <select id="category" required>
        <option value="" disabled selected>Select Category</option>
        <option value="Food">🍔 Food</option>
        <option value="Transport">🚌 Transport</option>
        <option value="Shopping">🛍️ Shopping</option>
        <option value="Entertainment">🎬 Entertainment</option>
        <option value="Utilities">💡 Utilities</option>
        <option value="Other">📦 Other</option>
      </select>
      <input type="number" id="amount" placeholder="Amount (₹)" required min="0.01" step="0.01" />
      <button type="submit">Add Expense</button>
    </form>

    <div class="filter">
      <select id="filter-category">
        <option value="All">🔎 Filter by Category</option>
        <option value="Food">🍔 Food</option>
        <option value="Transport">🚌 Transport</option>
        <option value="Shopping">🛍️ Shopping</option>
        <option value="Entertainment">🎬 Entertainment</option>
        <option value="Utilities">💡 Utilities</option>
        <option value="Other">📦 Other</option>
      </select>
    </div>

    <div class="expenses" id="expenses-list"></div>
    <div class="category-summary" id="category-summary"></div>
  </div>

  <audio id="alert-sound" src="https://www.soundjay.com/button/beep-07.wav" preload="auto"></audio>

  <script>
    const form = document.getElementById('expense-form');
    const descriptionInput = document.getElementById('description');
    const categoryInput = document.getElementById('category');
    const amountInput = document.getElementById('amount');
    const balanceDisplay = document.getElementById('balance');
    const expensesList = document.getElementById('expenses-list');
    const warning = document.getElementById('limit-warning');
    const filterCategory = document.getElementById('filter-category');
    const categorySummary = document.getElementById('category-summary');
    const alertSound = document.getElementById('alert-sound');
    const body = document.body;

    const SPENDING_LIMIT = 6000;

    let expenses = JSON.parse(localStorage.getItem("expenses")) || [];
    let editIndex = -1;

    function updateBalance() {
      const total = expenses.reduce((sum, exp) => sum + exp.amount, 0);
      balanceDisplay.textContent = total.toFixed(2);

      if (total > SPENDING_LIMIT) {
        warning.style.display = 'block';
        alertSound.play();
      } else {
        warning.style.display = 'none';
      }
    }

    function renderExpenses() {
      expensesList.innerHTML = '';
      const filter = filterCategory.value;

      const filteredExpenses = filter === "All" ? expenses : expenses.filter(e => e.category === filter);

      filteredExpenses.forEach((expense, index) => {
        const div = document.createElement('div');
        div.className = 'expense';
        div.innerHTML = `
          <div class="expense-top">
            <span><strong>${expense.category}:</strong> ${expense.description}</span>
            <span>-₹${expense.amount.toFixed(2)}</span>
          </div>
          <span style="font-size: 12px;">🕒 ${expense.time}</span>
          <div class="actions">
            <button class="edit-btn" onclick="editExpense(${index})">Edit</button>
            <button class="delete-btn" onclick="deleteExpense(${index})">Delete</button>
          </div>
        `;
        expensesList.appendChild(div);
      });

      updateCategorySummary();
    }

    function updateCategorySummary() {
      const summary = {};
      expenses.forEach(e => {
        summary[e.category] = (summary[e.category] || 0) + e.amount;
      });

      categorySummary.innerHTML = Object.entries(summary)
        .map(([cat, amt]) => `${cat}: ₹${amt.toFixed(2)}`)
        .join(' | ');
    }

    function editExpense(index) {
      const expense = expenses[index];
      descriptionInput.value = expense.description;
      amountInput.value = expense.amount;
      categoryInput.value = expense.category;
      editIndex = index;
    }

    function deleteExpense(index) {
      expenses.splice(index, 1);
      saveToLocal();
      renderExpenses();
      updateBalance();
    }

    function saveToLocal() {
      localStorage.setItem("expenses", JSON.stringify(expenses));
    }

    form.addEventListener('submit', function (e) {
      e.preventDefault();
      const desc = descriptionInput.value.trim();
      const amount = parseFloat(amountInput.value);
      const category = categoryInput.value;
      const time = new Date().toLocaleString();

      if (desc && amount > 0 && category) {
        const expenseData = { description: desc, amount, category, time };

        if (editIndex >= 0) {
          expenses[editIndex] = expenseData;
          editIndex = -1;
        } else {
          expenses.unshift(expenseData);
        }

        descriptionInput.value = '';
        amountInput.value = '';
        categoryInput.value = '';
        saveToLocal();
        renderExpenses();
        updateBalance();
      }
    });

    function ThemeTheme() {
      const current = body.getAttribute('data-theme');
      body.setAttribute('data-theme', current === 'light' ? 'dark' : 'light');
    }

    filterCategory.addEventListener('change', renderExpenses);

    renderExpenses();
    updateBalance();
  </script>
</body>
</html>
