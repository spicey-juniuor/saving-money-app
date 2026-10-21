const STORAGE_KEY = 'savvynest-state';

const initialState = {
  goals: [
    { id: 1, name: 'Emergency fund', category: 'Emergency', target: 8000, saved: 4600 },
    { id: 2, name: 'Summer trip', category: 'Travel', target: 2400, saved: 1380 },
    { id: 3, name: 'Home office', category: 'Home', target: 3000, saved: 820 }
  ],
  transactions: [
    { id: 1, title: 'Paycheck', category: 'Income', amount: 3200, type: 'income', date: '2026-10-19' },
    { id: 2, title: 'Weekly groceries', category: 'Food', amount: 120, type: 'expense', date: '2026-10-18' },
    { id: 3, title: 'Rent', category: 'Housing', amount: 1400, type: 'expense', date: '2026-10-16' },
    { id: 4, title: 'Freelance work', category: 'Income', amount: 480, type: 'income', date: '2026-10-15' },
    { id: 5, title: 'Train ticket', category: 'Transport', amount: 42, type: 'expense', date: '2026-10-12' }
  ],
  budgets: [
    { category: 'Housing', spent: 1400, limit: 1800 },
    { category: 'Food', spent: 340, limit: 500 },
    { category: 'Transport', spent: 180, limit: 250 },
    { category: 'Utilities', spent: 240, limit: 270 }
  ]
};

const state = loadState();

const statsGrid = document.querySelector('#statsGrid');
const goalsList = document.querySelector('#goalsList');
const budgetList = document.querySelector('#budgetList');
const transactionsTable = document.querySelector('#transactionsTable');
const goalModal = document.querySelector('#goalModal');
const transactionModal = document.querySelector('#transactionModal');
const goalForm = document.querySelector('#goalForm');
const transactionForm = document.querySelector('#transactionForm');

function loadState() {
  const saved = localStorage.getItem(STORAGE_KEY);
  if (!saved) {
    return structuredClone(initialState);
  }

  try {
    return JSON.parse(saved);
  } catch {
    return structuredClone(initialState);
  }
}

function persistState() {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
}

function formatCurrency(value) {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
    maximumFractionDigits: 0
  }).format(value);
}

function computeSummary() {
  const totalSaved = state.goals.reduce((sum, goal) => sum + goal.saved, 0);
  const totalTarget = state.goals.reduce((sum, goal) => sum + goal.target, 0);
  const income = state.transactions
    .filter((item) => item.type === 'income')
    .reduce((sum, item) => sum + item.amount, 0);
  const expense = state.transactions
    .filter((item) => item.type === 'expense')
    .reduce((sum, item) => sum + item.amount, 0);
  const monthlySaved = income - expense;

  return {
    totalSaved,
    monthlySaved,
    totalTarget,
    expense
  };
}

function renderStats() {
  const summary = computeSummary();

  const cards = [
    { label: 'Total saved', value: formatCurrency(summary.totalSaved), delta: '+12.4%', tone: 'positive' },
    { label: 'Monthly save', value: formatCurrency(summary.monthlySaved), delta: '+8.1%', tone: 'positive' },
    { label: 'Total spending', value: formatCurrency(summary.expense), delta: '-3.2%', tone: 'negative' },
    { label: 'Goal progress', value: `${Math.round((summary.totalSaved / summary.totalTarget) * 100)}%`, delta: '+14.6%', tone: 'positive' }
  ];

  statsGrid.innerHTML = cards
    .map(
      (card) => `
        <article class="stat-card">
          <div class="stat-top">
            <h4>${card.label}</h4>
            <div class="stat-icon">$</div>
          </div>
          <p class="stat-value">${card.value}</p>
          <span class="stat-delta ${card.tone === 'positive' ? 'delta-positive' : 'delta-negative'}">${card.delta}</span>
        </article>
      `
    )
    .join('');
}

function renderGoals() {
  goalsList.innerHTML = state.goals
    .map((goal) => {
      const progress = Math.min((goal.saved / goal.target) * 100, 100);
      return `
        <article class="goal-item">
          <div class="goal-item-top">
            <div>
              <h4>${goal.name}</h4>
            </div>
            <span class="goal-pill">${goal.category}</span>
          </div>
          <div class="goal-meta">
            <span>${formatCurrency(goal.saved)} saved</span>
            <span>${formatCurrency(goal.target)}</span>
          </div>
          <div class="goal-progress-bar">
            <span style="width: ${progress}%"></span>
          </div>
          <div class="goal-meta">
            <span>${Math.round(progress)}% complete</span>
            <span>${formatCurrency(goal.target - goal.saved)} left</span>
          </div>
        </article>
      `;
    })
    .join('');
}

function renderBudget() {
  budgetList.innerHTML = state.budgets
    .map((budget) => {
      const ratio = (budget.spent / budget.limit) * 100;
      return `
        <div class="budget-item">
          <div class="budget-item-top">
            <h4>${budget.category}</h4>
            <span class="budget-pill">${Math.round(ratio)}%</span>
          </div>
          <div class="budget-meta">
            <span>${formatCurrency(budget.spent)} spent</span>
            <span>${formatCurrency(budget.limit)} limit</span>
          </div>
          <div class="budget-progress">
            <span style="width: ${Math.min(ratio, 100)}%"></span>
          </div>
        </div>
      `;
    })
    .join('');
}

function renderTransactions() {
  transactionsTable.innerHTML = state.transactions
    .slice()
    .sort((a, b) => new Date(b.date) - new Date(a.date))
    .map(
      (item) => `
        <tr>
          <td>${item.title}</td>
          <td>${item.category}</td>
          <td>${new Date(item.date).toLocaleDateString('en-US', { month: 'short', day: 'numeric', year: 'numeric' })}</td>
          <td>${item.type === 'income' ? 'Income' : 'Expense'}</td>
          <td class="align-right amount ${item.type === 'income' ? 'income' : 'expense'}">
            ${item.type === 'income' ? '+' : '-'}${formatCurrency(item.amount)}
          </td>
        </tr>
      `
    )
    .join('');
}

function render() {
  renderStats();
  renderGoals();
  renderBudget();
  renderTransactions();
}

function openModal(modal) {
  modal.classList.remove('hidden');
}

function closeModal(modal) {
  modal.classList.add('hidden');
}

document.querySelector('#openGoalModal').addEventListener('click', () => openModal(goalModal));
document.querySelector('#openTransactionModal').addEventListener('click', () => openModal(transactionModal));

document.querySelectorAll('[data-close]').forEach((button) => {
  button.addEventListener('click', () => closeModal(document.querySelector(`#${button.dataset.close}`)));
});

goalForm.addEventListener('submit', (event) => {
  event.preventDefault();

  const formData = new FormData(goalForm);
  const goal = {
    id: Date.now(),
    name: formData.get('name'),
    category: formData.get('category'),
    target: Number(formData.get('target')),
    saved: Number(formData.get('saved'))
  };

  state.goals.push(goal);
  persistState();
  render();
  goalForm.reset();
  closeModal(goalModal);
});

transactionForm.addEventListener('submit', (event) => {
  event.preventDefault();

  const formData = new FormData(transactionForm);
  const transaction = {
    id: Date.now(),
    title: formData.get('title'),
    category: formData.get('category'),
    amount: Number(formData.get('amount')),
    type: formData.get('type'),
    date: new Date().toISOString().slice(0, 10)
  };

  state.transactions.push(transaction);

  if (transaction.type === 'expense') {
    const budget = state.budgets.find((item) => item.category === transaction.category);
    if (budget) {
      budget.spent += transaction.amount;
    }
  }

  persistState();
  render();
  transactionForm.reset();
  closeModal(transactionModal);
});

window.addEventListener('click', (event) => {
  if (event.target.classList.contains('modal')) {
    event.target.classList.add('hidden');
  }
});

render();

