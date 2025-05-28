<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Flex-Digital Billing System</title>
  <link href="https://unpkg.com/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <style>
    @media print {
      .no-print {
        display: none;
      }
      .invoice-box {
        max-width: 100%;
        box-shadow: none;
        padding: 10px;
      }
      table {
        font-size: 12px;
        width: 100%;
        border-collapse: collapse;
        color: #000 !important;
      }
      th, td {
        border: 1px solid #000;
        padding: 8px;
        color: #000 !important;
      }
      th {
        background-color: #f7dc6f;
      }
      #billTable,
      #billTable tr,
      #billTable td {
        display: table-row !important;
        visibility: visible !important;
      }
    }
    .table-container {
      max-height: 50vh;
      min-height: 200px;
      overflow-y: auto;
      overflow-x: auto;
      -webkit-overflow-scrolling: touch;
    }
    table {
      min-width: 100%;
      table-layout: auto;
      color: #000 !important;
    }
    th, td {
      white-space: nowrap;
      padding: 8px;
      color: #000 !important;
    }
    input::placeholder {
      color: #6b7280;
    }
    input.table-input {
      width: 100%;
      border: none;
      background: transparent;
      text-align: center;
      color: #000 !important;
    }
    input.table-input:focus {
      outline: none;
      background: #f7fafc;
    }
  </style>
</head>
<body class="bg-blue-900 text-white p-4">
  <div class="invoice-box">
    <h1 class="text-xl font-bold text-center mb-4">🧾 Flex-Digital Billing System</h1>

    <div class="mb-4 space-y-2">
      <input type="text" id="customerName" placeholder="Customer Name" class="w-full p-2 border rounded text-black" oninput="saveDataToStorage()" />
      <input type="date" id="date" placeholder="Date" class="w-full p-2 border rounded text-black" />
      <input type="text" id="description" placeholder="Description" list="descList" class="w-full p-2 border rounded text-black" />
      <div class="flex gap-2">
        <input type="text" id="size" placeholder="e.g. 3x2" class="w-full p-2 border rounded text-black" oninput="updateTotalForm()" />
      </div>
      <input type="number" id="quantity" placeholder="No. of Sheets" min="1" value="1" class="w-full p-2 border rounded text-black" oninput="updateRate(); updateTotalForm()" />
      <input type="number" id="rate" placeholder="Rate (per unit)" min="0" class="w-full p-2 border rounded text-black" oninput="updateTotalForm()" />
      <button onclick="addRow()" class="w-full bg-yellow-400 text-black p-2 rounded shadow mt-2 no-print">Add Item</button>
    </div>

    <div class="bg-white text-black p-4 rounded shadow mb-4 table-container">
      <table class="w-full text-sm">
        <thead>
          <tr class="border-b">
            <th>Date</th>
            <th>Description</th>
            <th>Size</th>
            <th>Qty</th>
            <th>Rate</th>
            <th>Amount</th>
            <th class="no-print"></th>
          </tr>
        </thead>
        <tbody id="billTable"></tbody>
      </table>
      <div class="text-right font-bold mt-2">Total: <span id="total">0</span></div>
    </div>

    <button onclick="printInvoice()" class="w-full bg-yellow-400 text-black p-2 rounded shadow no-print">🖨️ Print</button>
    <button onclick="downloadPDF()" class="w-full bg-green-500 text-black p-2 rounded shadow mt-2 no-print">📄 Download PDF</button>
  </div>

  <datalist id="descList">
    <option value="Digital">
    <option value="Digital Sheet 300 gsm">
    <option value="Sticker Sheet">
    <option value="Plastic Sticker">
    <option value="Transparent Sticker">
    <option value="Plastic Sheet 125 Micron">
    <option value="Plastic Sheet 200 Micron">
    <option value="Metallic Sheet">
    <option value="Lamination">
    <option value="Flex">
    <option value="Semi">
    <option value="Star">
    <option value="Backlit">
    <option value="Vinyl">
    <option value="Oneway">
  </datalist>

  <script>
    const STORAGE_KEY = "flexBillingData";

    const rateChart = {
      "digital": [20, 15, 13],
      "digital sheet 300 gsm": [21, 16, 14],
      "sticker sheet": [25, 18, 16],
      "plastic sticker": [35, 28, 26],
      "transparent sticker": [35, 28, 26],
      "plastic sheet 125 micron": [35, 30, 28],
      "plastic sheet 200 micron": [40, 35, 33],
      "metallic sheet": [35, 28, 26],
      "lamination": [5, 5, 5],
      "flex": [7, 7, 7],
      "semi": [10, 10, 10],
      "star": [14, 14, 14],
      "backlit": [20, 20, 20],
      "vinyl": [20, 20, 20],
      "oneway": [25, 25, 25]
    };

    const sizeNotRequiredItems = [
      "digital",
      "digital sheet 300 gsm",
      "sticker sheet",
      "plastic sticker",
      "transparent sticker",
      "plastic sheet 125 micron",
      "plastic sheet 200 micron",
      "metallic sheet",
      "lamination"
    ];

    const sizeRequiredItems = [
      "flex",
      "semi",
      "star",
      "backlit",
      "vinyl",
      "oneway"
    ];

    function updateRate() {
      const desc = document.getElementById('description').value.toLowerCase().trim();
      const qty = parseInt(document.getElementById('quantity').value) || 1;
      const rateInput = document.getElementById('rate');
      for (let key in rateChart) {
        if (desc.includes(key)) {
          const rates = rateChart[key];
          if (Array.isArray(rates)) {
            rateInput.value = qty === 1 ? rates[0] : qty >= 2 && qty <= 4 ? rates[1] : rates[2];
          }
          break;
        }
      }
      console.log('Updated rate:', rateInput.value); // Debug
    }

    function updateTotalForm() {
      const desc = document.getElementById('description').value.toLowerCase().trim();
      const sizeText = document.getElementById('size').value.trim().toLowerCase();
      const qty = parseInt(document.getElementById('quantity').value) || 1;
      const rate = parseFloat(document.getElementById('rate').value) || 0;

      let width = 0, height = 0;
      if (sizeText.includes('x')) {
        const parts = sizeText.split('x').map(part => parseFloat(part.trim()) || 0);
        width = parts[0];
        height = parts[1];
      }

      let amount = 0;
      const isSizeNotRequired = sizeNotRequiredItems.some(item => desc.includes(item));
      const isLamination = desc.includes("lamination");

      if (isSizeNotRequired) {
        amount = isLamination ? (qty < 20 ? 100 : qty * rate) : qty * rate;
      } else {
        amount = width * height * qty * rate;
      }

      console.log('Form amount calculated:', amount); // Debug
      return amount;
    }

    function updateTotal(el) {
      const row = el.closest('tr');
      const desc = row.cells[1].children[0].value.trim().toLowerCase();
      const sizeText = row.cells[2].children[0].value.trim().toLowerCase();
      const qty = parseFloat(row.cells[3].children[0].value) || 1;
      const rateInput = row.cells[4].children[0];

      let width = 0, height = 0;
      if (sizeText.includes('x')) {
        const parts = sizeText.split('x').map(part => parseFloat(part.trim()) || 0);
        width = parts[0];
        height = parts[1];
      }

      let rate = parseFloat(rateInput.value) || 0;
      for (let key in rateChart) {
        if (desc.includes(key)) {
          const rates = rateChart[key];
          if (Array.isArray(rates)) {
            rate = qty === 1 ? rates[0] : qty >= 2 && qty <= 4 ? rates[1] : rates[2];
            rateInput.value = rate.toFixed(2);
          }
          break;
        }
      }

      let amount = 0;
      const isSizeNotRequired = sizeNotRequiredItems.some(item => desc.includes(item));
      const isLamination = desc.includes("lamination");

      if (isSizeNotRequired) {
        amount = isLamination ? (qty < 20 ? 100 : qty * rate) : qty * rate;
      } else {
        amount = width * height * qty * rate;
      }

      row.cells[5].innerText = isNaN(amount) ? '0.00' : amount.toFixed(2);
      calculateGrandTotal();
      saveDataToStorage();
      console.log('Row updated, amount:', amount); // Debug
    }

    function addRow() {
      const date = document.getElementById('date').value;
      const desc = document.getElementById('description').value.trim();
      const sizeText = document.getElementById('size').value.trim();
      const qty = parseInt(document.getElementById('quantity').value) || 0;
      const rate = parseFloat(document.getElementById('rate').value) || 0;

      const isSizeNotRequired = sizeNotRequiredItems.some(item => desc.toLowerCase().includes(item));
      const isSizeRequired = sizeRequiredItems.some(item => desc.toLowerCase().includes(item));

      if (!desc || !date || isNaN(qty) || qty <= 0 || isNaN(rate) || rate < 0) {
        alert("Please fill all valid fields!");
        return;
      }

      if (isSizeRequired && !sizeText) {
        alert("Size is required for this item!");
        return;
      }

      let width = 0, height = 0;
      if (sizeText.includes('x')) {
        const parts = sizeText.split('x').map(part => parseFloat(part.trim()) || 0);
        width = parts[0];
        height = parts[1];
      }

      let amount = 0;
      const isLamination = desc.toLowerCase().includes("lamination");
      if (isSizeNotRequired) {
        amount = isLamination ? (qty < 20 ? 100 : qty * rate) : qty * rate;
      } else {
        if (!width || !height) {
          alert("Please enter a valid size (e.g., 3x2)!");
          return;
        }
        amount = width * height * qty * rate;
      }

      const tableBody = document.getElementById('billTable');
      const row = document.createElement('tr');
      row.className = 'border-b';
      row.innerHTML = `
        <td><input type="date" value="${date}" class="table-input" oninput="updateTotal(this)"></td>
        <td><input type="text" value="${desc}" list="descList" class="table-input" oninput="updateTotal(this)"></td>
        <td><input type="text" value="${sizeText}" class="table-input" oninput="updateTotal(this)"></td>
        <td><input type="number" value="${qty}" min="1" class="table-input" oninput="updateTotal(this)"></td>
        <td><input type="number" value="${rate.toFixed(2)}" min="0" class="table-input" oninput="updateTotal(this)"></td>
        <td class="amount">${amount.toFixed(2)}</td>
        <td class="text-center no-print"><button onclick="removeRow(this)" class="text-red-500">×</button></td>
      `;
      tableBody.appendChild(row);
      calculateGrandTotal();
      saveDataToStorage();

      ['description', 'size', 'quantity', 'rate'].forEach(id => document.getElementById(id).value = '');
      document.getElementById('quantity').value = '1';
      document.getElementById('date').value = new Date().toISOString().split('T')[0];
      console.log('Row added:', { date, desc, sizeText, qty, rate, amount }); // Debug
    }

    function removeRow(el) {
      el.closest('tr').remove();
      calculateGrandTotal();
      saveDataToStorage();
      console.log('Row removed'); // Debug
    }

    function calculateGrandTotal() {
      let total = 0;
      document.querySelectorAll('.amount').forEach(td => {
        total += parseFloat(td.innerText) || 0;
      });
      document.getElementById('total').innerText = total.toFixed(2);
      console.log('Grand total:', total); // Debug
    }

    function saveDataToStorage() {
      try {
        const customerName = document.getElementById('customerName').value;
        const tableBody = document.getElementById('billTable');
        let data = {
          customerName: customerName,
          rows: []
        };
        for (let row of tableBody.rows) {
          let rowData = {
            date: row.cells[0].children[0].value,
            description: row.cells[1].children[0].value,
            size: row.cells[2].children[0].value,
            quantity: row.cells[3].children[0].value,
            rate: row.cells[4].children[0].value,
            amount: parseFloat(row.cells[5].innerText) || 0
          };
          data.rows.push(rowData);
        }
        localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
        console.log('Data saved to storage:', data); // Debug
      } catch (e) {
        console.error('Error saving to local storage:', e); // Debug
      }
    }

    function loadDataFromStorage() {
      try {
        const savedData = localStorage.getItem(STORAGE_KEY);
        const today = new Date().toISOString().split('T')[0];
        document.getElementById('date').value = today;
        if (!savedData) {
          document.getElementById('customerName').value = `Customer - ${today}`;
          return;
        }

        const data = JSON.parse(savedData);
        document.getElementById('customerName').value = data.customerName || `Customer - ${today}`;
        const tableBody = document.getElementById('billTable');
        tableBody.innerHTML = "";

        if (data.rows && Array.isArray(data.rows)) {
          for (let r of data.rows) {
            const amount = parseFloat(r.amount) || 0;
            const row = document.createElement('tr');
            row.className = 'border-b';
            row.innerHTML = `
              <td><input type="date" value="${r.date || today}" class="table-input" oninput="updateTotal(this)"></td>
              <td><input type="text" value="${r.description || ''}" list="descList" class="table-input" oninput="updateTotal(this)"></td>
              <td><input type="text" value="${r.size || ''}" class="table-input" oninput="updateTotal(this)"></td>
              <td><input type="number" value="${r.quantity || 1}" min="1" class="table-input" oninput="updateTotal(this)"></td>
              <td><input type="number" value="${parseFloat(r.rate || 0).toFixed(2)}" min="0" class="table-input" oninput="updateTotal(this)"></td>
              <td class="amount">${amount.toFixed(2)}</td>
              <td class="text-center no-print"><button onclick="removeRow(this)" class="text-red-500">×</button></td>
            `;
            tableBody.appendChild(row);
          }
        }
        calculateGrandTotal();
        console.log('Data loaded from storage:', data); // Debug
      } catch (e) {
        console.error('Error loading from local storage:', e); // Debug
        const today = new Date().toISOString().split('T')[0];
        document.getElementById('customerName').value = `Customer - ${today}`;
      }
    }

    function printInvoice() {
      window.print();
    }

    function downloadPDF() {
      const customerName = document.getElementById('customerName').value;
      const tableBody = document.getElementById('billTable');
      if (!customerName) return alert("Please enter customer name before downloading PDF!");
      if (!tableBody.rows.length) return alert("Please add at least one item to the table!");

      const printContent = document.createElement('div');
      printContent.style.padding = "20px";
      printContent.style.background = "#fff";
      printContent.style.color = "#000";

      const heading = document.createElement('h2');
      heading.innerText = `Customer Name: ${customerName}`;
      heading.style.textAlign = "center";
      heading.style.marginBottom = "20px";
      printContent.appendChild(heading);

      const table = document.createElement('table');
      table.style.width = "100%";
      table.style.borderCollapse = "collapse";
      table.style.fontSize = "12px";

      const thead = document.createElement('thead');
      thead.innerHTML = `
        <tr style="background:#f7dc6f;">
          <th style="border:1px solid #000; padding:8px;">Date</th>
          <th style="border:1px solid #000; padding:8px;">Description</th>
          <th style="border:1px solid #000; padding:8px;">Size</th>
          <th style="border:1px solid #000; padding:8px;">Qty</th>
          <th style="border:1px solid #000; padding:8px;">Rate</th>
          <th style="border:1px solid #000; padding:8px;">Amount</th>
        </tr>
      `;
      table.appendChild(thead);

      const tbody = document.createElement('tbody');
      const rows = document.querySelectorAll('#billTable tr');
      rows.forEach(row => {
        const tds = row.querySelectorAll('td');
        const tr = document.createElement('tr');
        tr.style.border = "1px solid #000";
        tr.style.padding = "8px";

        for (let i = 0; i < 6; i++) {
          const td = document.createElement('td');
          td.style.border = "1px solid #000";
          td.style.padding = "8px";
          td.innerText = tds[i].children[0] && tds[i].children[0].value !== undefined ? tds[i].children[0].value : tds[i].innerText;
          tr.appendChild(td);
        }
        tbody.appendChild(tr);
      });
      table.appendChild(tbody);
      printContent.appendChild(table);

      const totalDiv = document.createElement('div');
      totalDiv.style.textAlign = "right";
      totalDiv.style.marginTop = "15px";
      totalDiv.style.fontWeight = "bold";
      totalDiv.innerText = `Total: ${document.getElementById('total').innerText}`;
      printContent.appendChild(totalDiv);

      html2pdf().set({
        margin: 10,
        filename: `Bill_${customerName.replace(/\s+/g, '_')}.pdf`,
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2 },
        jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
      }).from(printContent).save();
      console.log('PDF generated for:', customerName); // Debug
    }

    window.onload = () => {
      loadDataFromStorage();
    };
  </script>
</body>
</html>
