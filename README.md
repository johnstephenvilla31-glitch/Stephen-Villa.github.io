<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>My Business System</title>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, sans-serif;
  background: #f1f4f9;
  color: #263449;
}

header {
  background: #142b49;
  color: white;
  padding: 22px;
}

header h2 { margin-bottom: 6px; }

.container {
  max-width: 1100px;
  margin: auto;
  padding: 18px;
}

nav {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 20px;
}

button {
  border: none;
  padding: 11px 15px;
  border-radius: 7px;
  background: #142b49;
  color: white;
  cursor: pointer;
  margin: 3px;
}

nav button {
  background: white;
  color: #142b49;
  border: 1px solid #ddd;
}

nav button.active {
  background: #142b49;
  color: white;
}

input, select {
  width: 100%;
  padding: 12px;
  margin: 8px 0;
  border: 1px solid #ccd3df;
  border-radius: 7px;
  font-size: 15px;
}

.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit,minmax(145px,1fr));
  gap: 12px;
  margin: 15px 0;
}

.card, .panel {
  background: white;
  padding: 18px;
  border-radius: 12px;
  box-shadow: 0 2px 8px #0000000c;
  margin-bottom: 14px;
}

.card p {
  color: #718096;
  margin-bottom: 10px;
}

.card h2 {
  color: #142b49;
  overflow-wrap: anywhere;
}

h2, h3 { margin-bottom: 12px; }

.item {
  border-bottom: 1px solid #eee;
  padding: 12px 0;
  overflow-wrap: anywhere;
}

.small {
  color: #718096;
  font-size: 13px;
}

.danger { background: #c0392b; }
.secondary { background: #64748b; }
.success { background: #16865c; }

footer {
  text-align: center;
  padding: 25px;
  color: #777;
}

@media(max-width:480px) {
  .container { padding: 12px; }
  header { padding: 18px; }
  button { font-size: 13px; }
}
</style>
</head>

<body>

<header>
  <h2 id="businessTitle">MY BUSINESS SYSTEM</h2>
  <p>Business Management Dashboard</p>
</header>

<div class="container">

<nav>
  <button class="active" onclick="navigate('Dashboard',this)">Dashboard</button>
  <button onclick="navigate('Products',this)">Products</button>
  <button onclick="navigate('Inventory',this)">Inventory</button>
  <button onclick="navigate('Sales',this)">Sales</button>
  <button onclick="navigate('Expenses',this)">Expenses</button>
  <button onclick="navigate('Reports',this)">Reports</button>
  <button onclick="navigate('Settings',this)">Settings</button>
</nav>

<main id="content"></main>

</div>

<footer>
  My Business System | Version 2.0
</footer>

<script>

// DATABASE
let db = JSON.parse(
  localStorage.getItem("myBusinessDB") || 
  '{"products":[],"sales":[],"expenses":[],"businessName":"MY BUSINESS SYSTEM"}'
);

let currentPage = "Dashboard";
let editingId = null;

function saveDB() {
  localStorage.setItem("myBusinessDB", JSON.stringify(db));
}

function money(number) {
  return "₱" + Number(number || 0).toLocaleString("en-PH", {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  });
}

function safe(value) {
  return String(value).replace(/[&<>"']/g, c => ({
    "&":"&amp;",
    "<":"&lt;",
    ">":"&gt;",
    '"':"&quot;",
    "'":"&#39;"
  })[c]);
}

function totals() {
  let sales = db.sales.reduce((s,x) => s + x.total, 0);
  let expenses = db.expenses.reduce((s,x) => s + x.amount, 0);
  let products = db.products.length;
  let profit = sales - expenses;

  return {sales, expenses, products, profit};
}

// NAVIGATION
function navigate(page, button) {
  currentPage = page;

  document.querySelectorAll("nav button").forEach(b => {
    b.classList.remove("active");
  });

  if(button) button.classList.add("active");

  render();
}

// DASHBOARD
function dashboard() {
  let t = totals();

  return `
    <h2>Business Dashboard</h2>

    <div class="cards">
      <div class="card">
        <p>Total Sales</p>
        <h2>${money(t.sales)}</h2>
      </div>

      <div class="card">
        <p>Total Expenses</p>
        <h2>${money(t.expenses)}</h2>
      </div>

      <div class="card">
        <p>Total Products</p>
        <h2>${t.products}</h2>
      </div>

      <div class="card">
        <p>Net Profit</p>
        <h2>${money(t.profit)}</h2>
      </div>
    </div>

    <div class="panel">
      <h3>Welcome to Your Business!</h3>
      <p>Manage your products, sales, inventory and expenses here.</p>
    </div>

    <div class="panel">
      <h3>Recent Sales</h3>
      ${db.sales.slice(-5).reverse().map(s => `
        <div class="item">
          ${safe(s.product)} - ${money(s.total)}
          <p class="small">${safe(s.date)}</p>
        </div>
      `).join("") || "No sales recorded yet."}
    </div>
  `;
}

// PRODUCTS
function productsPage() {
  return `
    <h2>Products Management</h2>

    <div class="panel">
      <h3>${editingId ? "Edit Product" : "Add New Product"}</h3>

      <form onsubmit="saveProduct(event)">
        <input id="pName" placeholder="Product Name" required>

        <input id="pPrice" type="number"
          placeholder="Selling Price" min="0.01"
          step="0.01" required>

        <input id="pQty" type="number"
          placeholder="Initial Stock" min="0"
          step="1" required>

        <button type="submit">
          ${editingId ? "Update Product" : "Add Product"}
        </button>

        ${editingId ? '<button type="button" class="secondary" onclick="cancelEdit()">Cancel</button>' : ""}
      </form>
    </div>

    <div class="panel">
      <h3>Product List (${db.products.length})</h3>

      ${db.products.map(p => `
        <div class="item">
          <strong>${safe(p.name)}</strong>
          <p>Price: ${money(p.price)}</p>
          <p>Stock: ${p.qty}</p>

          <button onclick="editProduct(${p.id})">Edit</button>
          <button class="danger" onclick="deleteProduct(${p.id})">Delete</button>
        </div>
      `).join("") || "No products added yet."}
    </div>
  `;
}

function saveProduct(e) {
  e.preventDefault();

  let name = document.getElementById("pName").value.trim();
  let price = Number(document.getElementById("pPrice").value);
  let qty = Number(document.getElementById("pQty").value);

  if(!name || !Number.isFinite(price) || price <= 0 ||
     !Number.isInteger(qty) || qty < 0) {
    alert("Please enter valid product details.");
    return;
  }

  if(editingId) {
    let p = db.products.find(x => x.id === editingId);

    if(p) {
      p.name = name;
      p.price = price;
      p.qty = qty;
    }

    editingId = null;
  } else {
    db.products.push({
      id: Date.now(),
      name,
      price,
      qty
    });
  }

  saveDB();
  render();
}

function editProduct(id) {
  let p = db.products.find(x => x.id === id);
  if(!p) return;

  editingId = id;
  render();

  document.getElementById("pName").value = p.name;
  document.getElementById("pPrice").value = p.price;
  document.getElementById("pQty").value = p.qty;
}

function cancelEdit() {
  editingId = null;
  render();
}

function deleteProduct(id) {
  if(!confirm("Delete this product?")) return;

  db.products = db.products.filter(p => p.id !== id);
  editingId = null;
  saveDB();
  render();
}

// INVENTORY
function inventoryPage() {
  return `
    <h2>Inventory</h2>

    <div class="panel">
      <h3>Stock Monitoring</h3>

      ${db.products.map(p => `
        <div class="item">
          <strong>${safe(p.name)}</strong>
          <p>Available Stock: ${p.qty}</p>
          <p class="small">
            ${p.qty <= 5 ? "Low Stock - Check inventory" : "Stock Available"}
          </p>
        </div>
      `).join("") || "No inventory records yet."}
    </div>
  `;
}

// SALES
function salesPage() {
  return `
    <h2>Sales Management</h2>

    <div class="panel">
      <h3>Record a Sale</h3>

      <form onsubmit="addSale(event)">
        <select id="saleProduct" required>
          <option value="">Select Product</option>
          ${db.products.map(p => `
            <option value="${p.id}">
              ${safe(p.name)} - ${money(p.price)} (Stock: ${p.qty})
            </option>
          `).join("")}
        </select>

        <input id="saleQty" type="number"
          placeholder="Quantity Sold" min="1" step="1" required>

        <button type="submit">Save Sale</button>
      </form>
    </div>

    <div class="panel">
      <h3>Sales History</h3>

      ${db.sales.slice().reverse().map(s => `
        <div class="item">
          <strong>${safe(s.product)}</strong>
          <p>Quantity: ${s.qty}</p>
          <p>Total: ${money(s.total)}</p>
          <p class="small">${safe(s.date)}</p>
        </div>
      `).join("") || "No sales recorded yet."}
    </div>
  `;
}

function addSale(e) {
  e.preventDefault();

  let id = Number(document.getElementById("saleProduct").value);
  let qty = Number(document.getElementById("saleQty").value);
  let p = db.products.find(x => x.id === id);

  if(!p || !Number.isInteger(qty) || qty <= 0) {
    alert("Please enter valid sale details.");
    return;
  }

  if(qty > p.qty) {
    alert("Not enough stock available!");
    return;
  }

  p.qty -= qty;

  db.sales.push({
    id: Date.now(),
    product: p.name,
    qty,
    total: p.price * qty,
    date: new Date().toLocaleString()
  });

  saveDB();
  render();
}

// EXPENSES
function expensesPage() {
  return `
    <h2>Expenses Management</h2>

    <div class="panel">
      <h3>Add Expense</h3>

      <form onsubmit="addExpense(event)">
        <input id="expenseName"
          placeholder="Expense Description" required>

        <input id="expenseAmount"
          type="number" min="0.01" step="0.01"
          placeholder="Amount" required>

        <button type="submit">Save Expense</button>
      </form>
    </div>

    <div class="panel">
      <h3>Expense History</h3>

      ${db.expenses.slice().reverse().map(x => `
        <div class="item">
          <strong>${safe(x.name)}</strong>
          <p>${money(x.amount)}</p>
          <p class="small">${safe(x.date)}</p>
        </div>
      `).join("") || "No expenses recorded yet."}
    </div>
  `;
}

function addExpense(e) {
  e.preventDefault();

  let name = document.getElementById("expenseName").value.trim();
  let amount = Number(document.getElementById("expenseAmount").value);

  if(!name || !Number.isFinite(amount) || amount <= 0) {
    alert("Enter valid expense details.");
    return;
  }

  db.expenses.push({
    id: Date.now(),
    name,
    amount,
    date: new Date().toLocaleString()
  });

  saveDB();
  render();
}

// REPORTS
function reportsPage() {
  let t = totals();

  return `
    <h2>Business Reports</h2>

    <div class="panel">
      <h3>Financial Summary</h3>

      <div class="item">
        Total Sales: <strong>${money(t.sales)}</strong>
      </div>

      <div class="item">
        Total Expenses: <strong>${money(t.expenses)}</strong>
      </div>

      <div class="item">
        Net Profit: <strong>${money(t.profit)}</strong>
      </div>

      <p class="small">
        Net profit here is calculated as sales minus recorded expenses.
        This is a basic estimate, not a full accounting statement.
      </p>
    </div>

    <div class="panel">
      <h3>Inventory Summary</h3>
      <p>Total Products: ${t.products}</p>
      <p>Total Units in Stock: ${
        db.products.reduce((s,p) => s + p.qty, 0)
      }</p>
    </div>

    <button onclick="exportData()">Export Business Data</button>
  `;
}

// SETTINGS
function settingsPage() {
  return `
    <h2>Settings</h2>

    <div class="panel">
      <h3>Business Information</h3>

      <form onsubmit="saveSettings(event)">
        <input id="businessName"
          value="${safe(db.businessName)}"
          placeholder="Business Name" required>

        <button type="submit">Save Settings</button>
      </form>
    </div>

    <div class="panel">
      <h3>Data Backup</h3>
      <p>Keep a backup of your business records.</p>
      <button onclick="exportData()">Download Backup</button>
    </div>
  `;
}

function saveSettings(e) {
  e.preventDefault();

  db.businessName =
    document.getElementById("businessName").value.trim();

  saveDB();
  document.getElementById("businessTitle").textContent =
    db.businessName;

  alert("Business name updated!");
  render();
}

// EXPORT
function exportData() {
  let file = new Blob(
    [JSON.stringify(db, null, 2)],
    {type: "application/json"}
  );

  let url = URL.createObjectURL(file);
  let a = document.createElement("a");

  a.href = url;
  a.download = "business-backup.json";
  a.click();

  URL.revokeObjectURL(url);
}

// MAIN RENDER
function render() {
  document.getElementById("businessTitle").textContent =
    db.businessName;

  let content = document.getElementById("content");

  if(currentPage === "Dashboard") content.innerHTML = dashboard();
  if(currentPage === "Products") content.innerHTML = productsPage();
  if(currentPage === "Inventory") content.innerHTML = inventoryPage();
  if(currentPage === "Sales") content.innerHTML = salesPage();
  if(currentPage === "Expenses") content.innerHTML = expensesPage();
  if(currentPage === "Reports") content.innerHTML = reportsPage();
  if(currentPage === "Settings") content.innerHTML = settingsPage();
}

render();

</script>
</body>
</html>
 
