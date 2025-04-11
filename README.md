
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Smart Expense Tracker</title>
  <style>
    :root {
      --bg-color: #f0f2f5;
      --text-color: #1a1a1a;
      --card-color: #ffffff;
      --accent-color: #00bfa6;
    }

    [data-theme="dark"] {
      --bg-color: #1e2a38;
      --text-color: #f1f1f1;
      --card-color: #2e3d51;
      --accent-color: #00ffc3;
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

    .toggle-btn {
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

    form {
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    input, button {
      padding: 10px;
      font-size: 16px;
      border: none;
      border-radius: 10px;
    }

    input {
      background: #e0e0e0;
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
      justify-content: space-between;
      align-items: center;
      background: #dfe6e9;
      padding: 10px;
      border-radius: 8px;
      margin-bottom: 10px;
    }

    [data-theme="dark"] .expense {
      background: #3b4d64;
    }

    .expense span {
      flex: 1;
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
  </style>
</head>
<body data-theme="light">

  <div class="tracker">
    <div class="header">
      <h1>Expense Tracker</h1>
      <button class="toggle-btn" onclick="toggleTheme()">🌗 Toggle</button>
    </div>

    <div class="balance">Balance: $<span id="balance">0.00</span></div>

    <form id="expense-form">
      <input type="text" id="description" placeholder="Expense Description" required />
      <input type="number" id="amount" placeholder="Amount" required min="0.01" step="0.01" />
      <button type="submit">Add Expense</button>
    </form>

    <div class="expenses" id="expenses-list"></div>
  </div>

  <script>
    const form = document.getElementById('expense-form');
    const descriptionInput = document.getElementById('description');
    const amountInput = document.getElementById('amount');
    const balanceDisplay = document.getElementById('balance');
    const expensesList = document.getElementById('expenses-list');
    const body = document.body;

    let balance = 0;
    let expenses = [];
    let editIndex = -1;

    function updateBalance() {
      const total = expenses.reduce((sum, exp) => sum - exp.amount, 0);
      balance = total;
      balanceDisplay.textContent = balance.toFixed(2);
    }

    function renderExpenses() {
      expensesList.innerHTML = '';
      expenses.forEach((expense, index) => {
        const div = document.createElement('div');
        div.className = 'expense';
        div.innerHTML = `
          <span>${expense.description}</span>
          <span>-$${expense.amount.toFixed(2)}</span>
          <div class="actions">
            <button class="edit-btn" onclick="editExpense(${index})">Edit</button>
            <button class="delete-btn" onclick="deleteExpense(${index})">Delete</button>
          </div>
        `;
        expensesList.appendChild(div);
      });
    }

    function editExpense(index) {
      const expense = expenses[index];
      descriptionInput.value = expense.description;
      amountInput.value = expense.amount;
      editIndex = index;
    }

    function deleteExpense(index) {
      expenses.splice(index, 1);
      renderExpenses();
      updateBalance();
    }

    form.addEventListener('submit', function (e) {
      e.preventDefault();
      const desc = descriptionInput.value.trim();
      const amount = parseFloat(amountInput.value);

      if (desc && amount > 0) {
        if (editIndex >= 0) {
          expenses[editIndex] = { description: desc, amount };
          editIndex = -1;
        } else {
          expenses.unshift({ description: desc, amount });
        }
        descriptionInput.value = '';
        amountInput.value = '';
        renderExpenses();
        updateBalance();
      }
    });

    function toggleTheme() {
      const current = body.getAttribute('data-theme');
      body.setAttribute('data-theme', current === 'light' ? 'dark' : 'light');
    }
  </script>
</body>
</html>
