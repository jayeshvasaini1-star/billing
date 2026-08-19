<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Shivam Traders - POS Billing System</title>
  <!-- Camera Barcode Scanner Library -->
  <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
  <!-- HTML to Canvas Library for Generating Receipt Image -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <style>
    :root {
      --primary: #4f46e5;
      --primary-dark: #3730a3;
      --secondary: #10b981;
      --whatsapp: #25d366;
      --warning: #f59e0b;
      --danger: #ef4444;
      --bg: #f8fafc;
      --card-bg: #ffffff;
      --text: #0f172a;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', system-ui, -apple-system, sans-serif; }
    body { background-color: var(--bg); color: var(--text); padding-bottom: 40px; }

    /* Header & Navbar */
    header {
      background: linear-gradient(135deg, #4f46e5 0%, #7c3aed 100%);
      color: white; padding: 15px 20px;
      display: flex; flex-wrap: wrap; justify-content: space-between; align-items: center;
      gap: 12px; box-shadow: 0 4px 15px rgba(0,0,0,0.1);
    }
    header h1 { font-size: 1.5rem; font-weight: 800; letter-spacing: 0.5px; }

    nav { display: flex; gap: 8px; background: rgba(255,255,255,0.2); padding: 4px; border-radius: 10px; }
    nav button {
      background: transparent; border: none; color: white; padding: 10px 16px;
      font-weight: 700; cursor: pointer; border-radius: 8px; transition: 0.2s; font-size: 0.9rem;
    }
    nav button.active { background: white; color: var(--primary-dark); }

    /* Layout */
    .container { max-width: 1280px; margin: 20px auto; padding: 0 15px; }
    .page { display: none; }
    .page.active { display: block; }

    .card {
      background: var(--card-bg); padding: 20px; border-radius: 14px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05); margin-bottom: 20px; border: 1px solid #e2e8f0;
    }

    /* Billing Split Layout (With Mobile Top Reordering) */
    .billing-grid { display: grid; grid-template-columns: 1fr 440px; gap: 20px; }
    @media (max-width: 900px) { 
      .billing-grid { grid-template-columns: 1fr; } 
      .sidebar-bill { order: -1; } /* Moves bill/cart to top on mobile phones */
    }

    /* Form Controls */
    h2 { margin-bottom: 16px; color: var(--primary-dark); font-size: 1.2rem; font-weight: 700; display: flex; align-items: center; justify-content: space-between; gap: 8px; }
    .form-group { margin-bottom: 12px; }
    label { display: block; font-size: 0.8rem; font-weight: 700; margin-bottom: 4px; color: #64748b; text-transform: uppercase; }
    input, select {
      width: 100%; padding: 10px 12px; border: 2px solid #e2e8f0; border-radius: 8px; font-size: 0.95rem;
      transition: border-color 0.2s;
    }
    input:focus, select:focus { outline: none; border-color: var(--primary); }

    /* Product Cards Grid in Billing Page */
    .product-cards-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
      gap: 15px;
      max-height: 520px;
      overflow-y: auto;
      padding: 5px;
    }

    .item-card {
      background: #ffffff;
      border: 2px solid #e2e8f0;
      border-radius: 12px;
      padding: 12px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      transition: transform 0.15s, border-color 0.2s;
      box-shadow: 0 2px 6px rgba(0,0,0,0.03);
    }
    .item-card:hover { border-color: var(--primary); transform: translateY(-2px); }
    .item-title { font-weight: 700; font-size: 0.95rem; color: #1e293b; margin-bottom: 4px; }
    .item-barcode { font-size: 0.75rem; color: #94a3b8; margin-bottom: 8px; }
    .price-tag { display: flex; align-items: baseline; gap: 6px; margin-bottom: 10px; }
    .mrp-price { font-size: 0.8rem; color: #94a3b8; text-decoration: line-through; }
    .sell-price { font-size: 1.1rem; font-weight: 800; color: var(--secondary); }

    /* Quantity Box inside Card */
    .qty-controls {
      display: flex;
      align-items: center;
      justify-content: space-between;
      background: #f1f5f9;
      border-radius: 8px;
      padding: 4px;
      margin-bottom: 8px;
    }
    .btn-qty {
      background: white; border: 1px solid #cbd5e1; width: 28px; height: 28px;
      border-radius: 6px; font-weight: 800; font-size: 1rem; cursor: pointer;
      display: flex; align-items: center; justify-content: center; color: var(--text);
    }
    .btn-qty:active { background: #e2e8f0; }
    .qty-display { font-weight: 700; font-size: 0.95rem; width: 30px; text-align: center; }

    /* Buttons */
    .btn {
      display: inline-flex; align-items: center; justify-content: center; gap: 6px;
      padding: 10px 14px; border: none; border-radius: 8px; font-weight: 700;
      cursor: pointer; transition: 0.2s; font-size: 0.9rem; width: 100%;
    }
    .btn-primary { background: var(--primary); color: white; }
    .btn-primary:hover { background: var(--primary-dark); }
    .btn-success { background: var(--secondary); color: white; }
    .btn-whatsapp { background: var(--whatsapp); color: white; }
    .btn-warning { background: var(--warning); color: white; }
    .btn-danger { background: var(--danger); color: white; }
    .btn-outline { background: transparent; border: 2px solid var(--primary); color: var(--primary); }
    .btn-sm { width: auto; padding: 4px 8px; font-size: 0.8rem; }

    /* Camera Feed Container */
    .camera-box {
      display: none; width: 100%; min-height: 220px; border-radius: 10px; overflow: hidden;
      border: 3px solid var(--primary); margin: 10px 0; background: #000;
    }

    /* Tables */
    table { width: 100%; border-collapse: collapse; margin-top: 8px; }
    th, td { text-align: left; padding: 6px 4px; border-bottom: 1px solid #f1f5f9; font-size: 0.82rem; }
    th { background: #f8fafc; color: #64748b; font-weight: 700; }

    /* Sidebar Receipt Area */
    .sidebar-bill {
      background: #ffffff; border: 2px solid #e2e8f0; border-radius: 12px; padding: 15px;
      position: sticky; top: 20px;
    }
    .bill-scroll-table { max-height: 220px; overflow-y: auto; margin-bottom: 10px; }

    /* Modal for Held Bills */
    .modal-overlay {
      display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
      background: rgba(0,0,0,0.5); z-index: 1000; justify-content: center; align-items: center;
    }
    .modal-overlay.active { display: flex; }
    .modal-box { background: white; padding: 20px; border-radius: 12px; width: 90%; max-width: 550px; max-height: 80vh; overflow-y: auto; }

    /* Thermal Receipt Formatting */
    #receipt-area {
      position: absolute;
      left: -9999px;
      top: -9999px;
      width: 280px;
      background: #fff;
      padding: 10px;
      font-family: 'Courier New', monospace;
      font-size: 10px;
      color: black;
    }
    .t-center { text-align: center; }
    .t-bold { font-weight: bold; }
    .t-line { border-bottom: 1px dashed black; margin: 6px 0; }
    .t-table { width: 100%; margin: 4px 0; }
    .t-table th, .t-table td { padding: 2px 0; font-size: 9px; border: none; }

    @media print {
      body * { visibility: hidden; }
      #receipt-area, #receipt-area * { visibility: visible; }
      #receipt-area {
        display: block !important; position: absolute; left: 0; top: 0;
        width: 58mm; padding: 2mm; font-family: 'Courier New', monospace; font-size: 9px; color: black;
      }
      @page { size: 58mm auto; margin: 0; }
    }
  </style>
</head>
<body>

  <header>
    <h1>Shivam Traders</h1>
    <nav>
      <button class="active" onclick="switchTab('page-inventory', this)">📦 Inventory</button>
      <button onclick="switchTab('page-billing', this)">🧾 Make Bill</button>
      <button onclick="switchTab('page-history', this)">📜 Bill History</button>
    </nav>
  </header>

  <div class="container">
    
    <!-- PAGE 1: INVENTORY -->
    <div id="page-inventory" class="page active">
      <div style="display: grid; grid-template-columns: 350px 1fr; gap: 20px;">
        <div class="card">
          <h2>➕ Add / Update Product</h2>
          
          <div class="form-group">
            <label>Barcode</label>
            <input type="text" id="inv-barcode" placeholder="Type or scan barcode">
            <button class="btn btn-warning" style="margin-top: 6px;" onclick="toggleCameraScanner('inv-camera', 'inv-barcode')">
              📷 Camera Scan Barcode
            </button>
            <div id="inv-camera" class="camera-box"><div id="inv-reader"></div></div>
          </div>

          <div class="form-group">
            <label>Item Name</label>
            <input type="text" id="inv-name" placeholder="e.g. Fortune Oil 1L">
          </div>
          
          <div style="display: flex; gap: 10px;">
            <div class="form-group" style="flex:1;">
              <label>MRP (₹)</label>
              <input type="number" id="inv-mrp" placeholder="0.00">
            </div>
            <div class="form-group" style="flex:1;">
              <label>Selling Price (₹)</label>
              <input type="number" id="inv-price" placeholder="0.00">
            </div>
          </div>

          <button class="btn btn-primary" onclick="saveInventoryItem()">Save / Update Item</button>
        </div>

        <div class="card">
          <h2>📋 Stock Inventory List</h2>
          <table>
            <thead>
              <tr><th>Barcode</th><th>Name</th><th>MRP</th><th>Selling Price</th><th>Action</th></tr>
            </thead>
            <tbody id="tbl-inventory-body"></tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- PAGE 2: MAKE BILL -->
    <div id="page-billing" class="page">
      <div class="billing-grid">
        
        <!-- Search & Catalog Cards -->
        <div>
          <div class="card" style="padding: 15px; margin-bottom: 15px;">
            <div style="display: flex; gap: 10px; margin-bottom: 10px;">
              <input type="text" id="bill-search" placeholder="🔍 Search item name or barcode..." oninput="renderProductCards()">
              <button class="btn btn-warning" style="width: auto; white-space: nowrap;" onclick="toggleCameraScanner('bill-camera', null, true)">
                📷 Camera Scan
              </button>
            </div>
            <div id="bill-camera" class="camera-box"><div id="bill-reader"></div></div>
          </div>

          <!-- Items Grid Boxes -->
          <div id="product-cards-container" class="product-cards-grid"></div>
        </div>

        <!-- Live Bill Summary (Appears on top on mobile screens) -->
        <div class="sidebar-bill">
          <div style="display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #e2e8f0; padding-bottom: 8px;">
            <h2 style="margin:0;">🛒 Current Bill</h2>
            <button class="btn btn-outline btn-sm" onclick="openHeldBillsModal()">
              ⏸️ Held (<span id="held-count-badge">0</span>)
            </button>
          </div>
          
          <div class="form-group" style="margin-top: 10px;">
            <label>Customer Name</label>
            <input type="text" id="cust-name" placeholder="Customer Name">
          </div>
          <div class="form-group">
            <label>Customer Mobile No.</label>
            <input type="text" id="cust-phone" placeholder="WhatsApp Number (e.g. 9876543210)">
          </div>

          <div class="bill-scroll-table">
            <table>
              <thead>
                <tr>
                  <th>Item</th>
                  <th>MRP</th>
                  <th>Rate</th>
                  <th style="width: 45px;">Qty</th>
                  <th style="text-align: right;">Total</th>
                  <th></th>
                </tr>
              </thead>
              <tbody id="tbl-bill-items-body"></tbody>
            </table>
          </div>

          <div style="border-top: 2px dashed #e2e8f0; padding-top: 10px;">
            <div class="form-group">
              <label>Discount (in ₹)</label>
              <input type="number" id="cust-discount" value="0" min="0" oninput="renderBillTotals()">
            </div>

            <div style="font-size: 1.2rem; font-weight: 800; text-align: right; margin-bottom: 12px; color: var(--primary-dark);">
              Grand Total: ₹<span id="txt-grand-total">0.00</span>
            </div>

            <div style="display: flex; flex-direction: column; gap: 8px;">
              <button class="btn btn-warning" onclick="holdCurrentBill()">⏸️ Hold This Bill</button>
              <button class="btn btn-primary" onclick="generateAndPrintBill()">🖨️ Print Thermal Receipt</button>
              <button class="btn btn-whatsapp" onclick="shareBillOnWhatsApp()">🖼️ Share Thermal Receipt Image</button>
            </div>
          </div>
        </div>

      </div>
    </div>

    <!-- PAGE 3: BILL HISTORY -->
    <div id="page-history" class="page">
      <div class="card">
        <h2>📜 Past Bill History</h2>
        <table>
          <thead>
            <tr><th>Bill ID</th><th>Date</th><th>Customer</th><th>Mobile</th><th>Total</th><th>Actions</th></tr>
          </thead>
          <tbody id="tbl-history-body"></tbody>
        </table>
      </div>
    </div>

  </div>

  <!-- Modal for Viewing Held Bills -->
  <div id="held-bills-modal" class="modal-overlay">
    <div class="modal-box">
      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 15px;">
        <h3 style="color: var(--primary-dark);">⏸️ Held Bills</h3>
        <button class="btn btn-danger btn-sm" onclick="closeHeldBillsModal()">✕ Close</button>
      </div>
      <div id="held-bills-list"></div>
    </div>
  </div>

  <!-- Hidden Printable Thermal Receipt Layout -->
  <div id="receipt-area">
    <div class="t-center t-bold" style="font-size: 13px;">SHIVAM TRADERS</div>
    <div class="t-center">Retail Invoice / Bill</div>
    <div class="t-line"></div>
    <div>Bill No: <span id="rec-id"></span></div>
    <div>Date: <span id="rec-date"></span></div>
    <div>Customer: <span id="rec-cust"></span></div>
    <div>Mobile: <span id="rec-phone"></span></div>
    <div class="t-line"></div>
    <table class="t-table">
      <thead>
        <tr>
          <th>Item</th>
          <th>MRP</th>
          <th>Rate</th>
          <th style="text-align: center;">Qty</th>
          <th style="text-align: right;">Total</th>
        </tr>
      </thead>
      <tbody id="rec-items"></tbody>
    </table>
    <div class="t-line"></div>
    <div style="text-align: right;">Subtotal: ₹<span id="rec-sub">0.00</span></div>
    <div style="text-align: right;">Discount: -₹<span id="rec-disc">0.00</span></div>
    <div class="t-bold" style="text-align: right; font-size: 11px;">TOTAL: ₹<span id="rec-total">0.00</span></div>
    <div class="t-line"></div>
    <div class="t-center">Thank You! Visit Again</div>
  </div>

  <script>
    let db = {
      inventory: JSON.parse(localStorage.getItem('st_inventory') || '[]'),
      bills: JSON.parse(localStorage.getItem('st_bills') || '[]'),
      heldBills: JSON.parse(localStorage.getItem('st_held_bills') || '[]')
    };

    let activeBillItems = [];
    let activeEditBillId = null;
    let cardQuantities = {};
    let html5QrCode = null;

    function saveState() {
      localStorage.setItem('st_inventory', JSON.stringify(db.inventory));
      localStorage.setItem('st_bills', JSON.stringify(db.bills));
      localStorage.setItem('st_held_bills', JSON.stringify(db.heldBills));
      updateHeldBadge();
    }

    function updateHeldBadge() {
      const badge = document.getElementById('held-count-badge');
      if (badge) badge.innerText = db.heldBills.length;
    }

    function switchTab(pageId, btn) {
      stopCameraScanner();
      document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
      document.querySelectorAll('nav button').forEach(b => b.classList.remove('active'));
      document.getElementById(pageId).classList.add('active');
      btn.classList.add('active');

      if (pageId === 'page-inventory') renderInventoryTable();
      else if (pageId === 'page-billing') renderProductCards();
      else if (pageId === 'page-history') renderHistoryTable();
    }

    /* Robust Mobile Camera Barcode Scanner */
    async function toggleCameraScanner(containerId, targetInputId, isAutoBill = false) {
      const box = document.getElementById(containerId);
      const readerId = containerId === 'inv-camera' ? 'inv-reader' : 'bill-reader';

      if (box.style.display === 'block') {
        stopCameraScanner();
        box.style.display = 'none';
        return;
      }

      box.style.display = 'block';

      try {
        html5QrCode = new Html5Qrcode(readerId);
        
        const qrConfig = {
          fps: 15,
          qrbox: (viewfinderWidth, viewfinderHeight) => {
            const minDim = Math.min(viewfinderWidth, viewfinderHeight);
            return { width: Math.floor(minDim * 0.75), height: Math.floor(minDim * 0.5) };
          }
        };

        const onScanSuccess = (barcode) => {
          if (targetInputId) document.getElementById(targetInputId).value = barcode;
          if (isAutoBill) autoAddScannedBarcode(barcode);
          stopCameraScanner();
          box.style.display = 'none';
        };

        try {
          await html5QrCode.start({ facingMode: "environment" }, qrConfig, onScanSuccess, () => {});
        } catch (e1) {
          const cameras = await Html5Qrcode.getCameras();
          if (cameras && cameras.length > 0) {
            const backCamera = cameras.find(c => c.label.toLowerCase().includes('back') || c.label.toLowerCase().includes('rear')) || cameras[cameras.length - 1];
            await html5QrCode.start(backCamera.id, qrConfig, onScanSuccess, () => {});
          } else {
            throw new Error("No camera found.");
          }
        }
      } catch (err) {
        alert("Unable to access camera. Please ensure camera permissions are granted and the website is accessed over HTTPS.");
        box.style.display = 'none';
      }
    }

    function stopCameraScanner() {
      if (html5QrCode && html5QrCode.isScanning) {
        html5QrCode.stop().then(() => html5QrCode.clear()).catch(() => {});
      }
      document.querySelectorAll('.camera-box').forEach(b => b.style.display = 'none');
    }

    /* Inventory Logic */
    function saveInventoryItem() {
      const barcode = document.getElementById('inv-barcode').value.trim();
      const name = document.getElementById('inv-name').value.trim();
      const mrp = parseFloat(document.getElementById('inv-mrp').value) || 0;
      const price = parseFloat(document.getElementById('inv-price').value);

      if (!name || isNaN(price) || price <= 0) {
        alert('Please fill product name and selling price.');
        return;
      }

      const existingIdx = db.inventory.findIndex(i => 
        (barcode && i.barcode === barcode) || 
        (i.name.toLowerCase() === name.toLowerCase())
      );

      const itemObj = { 
        barcode: barcode || (existingIdx > -1 ? db.inventory[existingIdx].barcode : 'BAR-' + Date.now()), 
        name, 
        mrp: mrp || price, 
        price 
      };

      if (existingIdx > -1) {
        db.inventory[existingIdx] = itemObj;
        alert(`Product "${name}" updated successfully!`);
      } else {
        db.inventory.push(itemObj);
      }

      saveState();
      renderInventoryTable();
      document.getElementById('inv-barcode').value = '';
      document.getElementById('inv-name').value = '';
      document.getElementById('inv-mrp').value = '';
      document.getElementById('inv-price').value = '';
    }

    function deleteInventoryItem(index) {
      db.inventory.splice(index, 1);
      saveState();
      renderInventoryTable();
    }

    function renderInventoryTable() {
      const tbody = document.getElementById('tbl-inventory-body');
      tbody.innerHTML = '';
      db.inventory.forEach((item, idx) => {
        tbody.innerHTML += `
          <tr>
            <td><b>${item.barcode}</b></td>
            <td>${item.name}</td>
            <td>₹${item.mrp.toFixed(2)}</td>
            <td>₹${item.price.toFixed(2)}</td>
            <td><button class="btn btn-danger btn-sm" onclick="deleteInventoryItem(${idx})">Delete</button></td>
          </tr>`;
      });
    }

    /* Billing Box/Card UI Rendering */
    function renderProductCards() {
      const query = document.getElementById('bill-search').value.toLowerCase();
      const container = document.getElementById('product-cards-container');
      container.innerHTML = '';

      const filtered = db.inventory.filter(i => 
        i.name.toLowerCase().includes(query) || i.barcode.toLowerCase().includes(query)
      );

      if (filtered.length === 0) {
        container.innerHTML = `<div style="grid-column: 1/-1; text-align:center; color:#94a3b8; padding:20px;">No products found in inventory.</div>`;
        return;
      }

      filtered.forEach(item => {
        if (!cardQuantities[item.barcode]) cardQuantities[item.barcode] = 1;

        container.innerHTML += `
          <div class="item-card">
            <div>
              <div class="item-title">${item.name}</div>
              <div class="item-barcode">BC: ${item.barcode}</div>
              <div class="price-tag">
                ${item.mrp > item.price ? `<span class="mrp-price">₹${item.mrp.toFixed(2)}</span>` : ''}
                <span class="sell-price">₹${item.price.toFixed(2)}</span>
              </div>
            </div>
            <div>
              <div class="qty-controls">
                <button class="btn-qty" onclick="changeCardQty('${item.barcode}', -1)">-</button>
                <span class="qty-display" id="card-qty-${item.barcode}">${cardQuantities[item.barcode]}</span>
                <button class="btn-qty" onclick="changeCardQty('${item.barcode}', 1)">+</button>
              </div>
              <button class="btn btn-success btn-sm" style="width:100%;" onclick="addCardToBill('${item.barcode}')">+ Add to Bill</button>
            </div>
          </div>`;
      });
    }

    function changeCardQty(barcode, change) {
      cardQuantities[barcode] = Math.max(1, (cardQuantities[barcode] || 1) + change);
      const el = document.getElementById(`card-qty-${barcode}`);
      if (el) el.innerText = cardQuantities[barcode];
    }

    function addCardToBill(barcode) {
      const item = db.inventory.find(i => i.barcode === barcode);
      const qty = cardQuantities[barcode] || 1;
      if (item) addItemToBillArray(item, qty);
    }

    function autoAddScannedBarcode(barcode) {
      const item = db.inventory.find(i => i.barcode === barcode);
      if (item) addItemToBillArray(item, 1);
      else alert('Barcode not found in inventory!');
    }

    function addItemToBillArray(item, qty) {
      const existing = activeBillItems.find(i => i.barcode === item.barcode);
      if (existing) existing.qty += qty;
      else activeBillItems.push({ barcode: item.barcode, name: item.name, price: item.price, mrp: item.mrp, qty });
      
      renderBillTotals();
    }

    function removeBillItem(idx) {
      activeBillItems.splice(idx, 1);
      renderBillTotals();
    }

    function renderBillTotals() {
      const tbody = document.getElementById('tbl-bill-items-body');
      tbody.innerHTML = '';
      let subtotal = 0;

      activeBillItems.forEach((item, idx) => {
        const itemTotal = item.price * item.qty;
        subtotal += itemTotal;
        tbody.innerHTML += `
          <tr>
            <td><b>${item.name}</b></td>
            <td>₹${item.mrp.toFixed(2)}</td>
            <td>₹${item.price.toFixed(2)}</td>
            <td>
              <input type="number" value="${item.qty}" min="1" style="width:40px; padding:2px;" 
                     onchange="activeBillItems[${idx}].qty = parseInt(this.value) || 1; renderBillTotals();">
            </td>
            <td style="text-align: right; font-weight: 700;">₹${itemTotal.toFixed(2)}</td>
            <td style="text-align: right;"><button class="btn btn-danger btn-sm" onclick="removeBillItem(${idx})">✕</button></td>
          </tr>`;
      });

      const discount = parseFloat(document.getElementById('cust-discount').value) || 0;
      const grandTotal = Math.max(0, subtotal - discount);
      document.getElementById('txt-grand-total').innerText = grandTotal.toFixed(2);
      return { subtotal, discount, grandTotal };
    }

    /* HOLD BILL FEATURE */
    function holdCurrentBill() {
      if (activeBillItems.length === 0) {
        alert('Cannot hold an empty bill.');
        return;
      }

      const heldObj = {
        id: 'HOLD-' + Math.floor(1000 + Math.random() * 9000),
        time: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
        customer: document.getElementById('cust-name').value.trim() || 'Cash Customer',
        phone: document.getElementById('cust-phone').value.trim(),
        discount: document.getElementById('cust-discount').value || '0',
        items: [...activeBillItems]
      };

      db.heldBills.push(heldObj);
      saveState();
      resetBillingForm();
      alert(`Bill held successfully as ${heldObj.id}`);
    }

    function openHeldBillsModal() {
      const list = document.getElementById('held-bills-list');
      list.innerHTML = '';

      if (db.heldBills.length === 0) {
        list.innerHTML = `<p style="text-align:center; color:#94a3b8; padding:10px;">No held bills currently.</p>`;
      } else {
        db.heldBills.forEach((hb, idx) => {
          let itemCount = hb.items.reduce((sum, i) => sum + i.qty, 0);
          list.innerHTML += `
            <div style="border: 1px solid #e2e8f0; border-radius: 8px; padding: 12px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; background: #f8fafc;">
              <div>
                <strong style="color: var(--primary-dark);">${hb.id} (${hb.time})</strong><br>
                <small>Cust: ${hb.customer} (${hb.phone || 'No Phone'}) | Items: ${itemCount}</small>
              </div>
              <div style="display: flex; gap: 6px;">
                <button class="btn btn-success btn-sm" onclick="restoreHeldBill(${idx})">Restore</button>
                <button class="btn btn-danger btn-sm" onclick="deleteHeldBill(${idx})">✕</button>
              </div>
            </div>`;
        });
      }

      document.getElementById('held-bills-modal').classList.add('active');
    }

    function closeHeldBillsModal() {
      document.getElementById('held-bills-modal').classList.remove('active');
    }

    function restoreHeldBill(index) {
      const hb = db.heldBills[index];
      activeBillItems = [...hb.items];
      document.getElementById('cust-name').value = hb.customer === 'Cash Customer' ? '' : hb.customer;
      document.getElementById('cust-phone').value = hb.phone || '';
      document.getElementById('cust-discount').value = hb.discount || '0';

      db.heldBills.splice(index, 1);
      saveState();
      renderBillTotals();
      closeHeldBillsModal();
    }

    function deleteHeldBill(index) {
      db.heldBills.splice(index, 1);
      saveState();
      openHeldBillsModal();
    }

    /* Receipts & Customer Phone Mapping */
    function populateReceiptDOM() {
      const { subtotal, discount, grandTotal } = renderBillTotals();
      const customer = document.getElementById('cust-name').value.trim() || 'Cash Customer';
      const phone = document.getElementById('cust-phone').value.trim() || 'N/A';
      const billId = activeEditBillId || 'ST-' + Math.floor(10000 + Math.random() * 90000);
      const dateStr = new Date().toLocaleString();

      document.getElementById('rec-id').innerText = billId;
      document.getElementById('rec-date').innerText = dateStr;
      document.getElementById('rec-cust').innerText = customer;
      document.getElementById('rec-phone').innerText = phone;
      
      const recTbody = document.getElementById('rec-items');
      recTbody.innerHTML = '';
      activeBillItems.forEach(i => {
        recTbody.innerHTML += `
          <tr>
            <td>${i.name}</td>
            <td>₹${i.mrp.toFixed(2)}</td>
            <td>₹${i.price.toFixed(2)}</td>
            <td style="text-align: center;">${i.qty}</td>
            <td style="text-align: right;">₹${(i.price * i.qty).toFixed(2)}</td>
          </tr>`;
      });

      document.getElementById('rec-sub').innerText = subtotal.toFixed(2);
      document.getElementById('rec-disc').innerText = discount.toFixed(2);
      document.getElementById('rec-total').innerText = grandTotal.toFixed(2);

      return { billId, dateStr, customer, phone, subtotal, discount, grandTotal };
    }

    function generateAndPrintBill() {
      if (activeBillItems.length === 0) {
        alert('Please add items to bill first.');
        return;
      }

      const { billId, dateStr, customer, phone, subtotal, discount, grandTotal } = populateReceiptDOM();

      const billData = {
        id: billId, date: dateStr, customer, phone,
        items: [...activeBillItems], subtotal, discount, grandTotal
      };

      if (activeEditBillId) {
        const idx = db.bills.findIndex(b => b.id === activeEditBillId);
        if (idx > -1) db.bills[idx] = billData;
        activeEditBillId = null;
      } else {
        db.bills.unshift(billData);
      }

      saveState();
      window.print();
      resetBillingForm();
    }

    function shareBillOnWhatsApp() {
      if (activeBillItems.length === 0) {
        alert('Please add items to the bill first.');
        return;
      }

      const { phone } = populateReceiptDOM();
      const receiptEl = document.getElementById('receipt-area');
      
      receiptEl.style.left = '0';
      receiptEl.style.top = '0';
      receiptEl.style.zIndex = '-9999';

      html2canvas(receiptEl, { scale: 2 }).then(canvas => {
        receiptEl.style.left = '-9999px';
        receiptEl.style.top = '-9999px';

        canvas.toBlob(blob => {
          const item = new ClipboardItem({ "image/png": blob });
          const targetPhone = phone !== 'N/A' ? phone : '';
          
          navigator.clipboard.write([item]).then(() => {
            alert('Receipt image copied to clipboard! Opening WhatsApp... Just PASTE (Ctrl+V) into the chat window.');
            let waUrl = targetPhone ? `https://wa.me/91${targetPhone}` : `https://api.whatsapp.com/send`;
            window.open(waUrl, '_blank');
          }).catch(() => {
            const a = document.createElement('a');
            a.href = canvas.toDataURL('image/png');
            a.download = `Receipt_${Date.now()}.png`;
            a.click();
            alert('Receipt image downloaded! You can attach and send it on WhatsApp.');
            let waUrl = targetPhone ? `https://wa.me/91${targetPhone}` : `https://api.whatsapp.com/send`;
            window.open(waUrl, '_blank');
          });
        });
      });
    }

    function resetBillingForm() {
      activeBillItems = [];
      document.getElementById('cust-name').value = '';
      document.getElementById('cust-phone').value = '';
      document.getElementById('cust-discount').value = '0';
      renderBillTotals();
    }

    /* History & Editing */
    function renderHistoryTable() {
      const tbody = document.getElementById('tbl-history-body');
      tbody.innerHTML = '';

      db.bills.forEach((bill) => {
        tbody.innerHTML += `
          <tr>
            <td><b>${bill.id}</b></td>
            <td>${bill.date}</td>
            <td>${bill.customer}</td>
            <td>${bill.phone || 'N/A'}</td>
            <td><b>₹${bill.grandTotal.toFixed(2)}</b></td>
            <td>
              <button class="btn btn-warning btn-sm" onclick="editBill('${bill.id}')">✏️ Edit Bill</button>
            </td>
          </tr>`;
      });
    }

    function editBill(billId) {
      const bill = db.bills.find(b => b.id === billId);
      if (!bill) return;

      activeEditBillId = bill.id;
      activeBillItems = [...bill.items];
      
      switchTab('page-billing', document.querySelectorAll('nav button')[1]);
      document.getElementById('cust-name').value = bill.customer;
      document.getElementById('cust-phone').value = bill.phone === 'N/A' ? '' : (bill.phone || '');
      document.getElementById('cust-discount').value = bill.discount;
      renderBillTotals();
    }

    // Initialize on page start
    renderInventoryTable();
    updateHeldBadge();
  </script>
</body>
</html>
