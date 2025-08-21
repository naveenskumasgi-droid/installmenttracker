<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Transaction Tracker Pro</title>
  <style>
    body { font-family: Arial, sans-serif; margin:0; padding:0; background:#f5f5f5; color:#333; }
    header { display:flex; justify-content:space-between; align-items:center; padding:1rem; background:#fff; box-shadow:0 2px 4px rgba(0,0,0,0.1); position:relative; }
    header h1 { margin:0 auto; }
    .notification { position:absolute; right:1rem; top:1rem; cursor:pointer; }
    .badge { background:red; color:white; font-size:0.7rem; padding:2px 6px; border-radius:50%; vertical-align:top; margin-left:2px; display:none; }
    #notifList { display:none; position:absolute; right:1rem; top:2.5rem; background:#fff; border:1px solid #ccc; padding:0.5rem; font-size:0.9rem; border-radius:5px; box-shadow:0 2px 5px rgba(0,0,0,0.2); }
    .summary { display:flex; gap:1rem; justify-content:space-around; margin:1rem; }
    .box { flex:1; padding:1rem; border-radius:8px; box-shadow:0 2px 5px rgba(0,0,0,0.2); text-align:center; font-weight:bold; }
    .green { background:#d4edda; color:#155724; } .red { background:#f8d7da; color:#721c24; } .neutral { background:#e2e3e5; color:#383d41; }
    table { width:100%; border-collapse:collapse; margin:1rem 0; background:#fff; }
    th,td { border:1px solid #ddd; padding:0.5rem; text-align:center; }
    tr.lent { background:#eaf9ea; } tr.borrowed { background:#fceaea; }
    .form-section { display:none; padding:1rem; background:#fff; margin:1rem; border-radius:8px; box-shadow:0 2px 4px rgba(0,0,0,0.2); }
    .form-section input,.form-section select,.form-section button { display:block; margin:0.5rem 0; padding:0.5rem; width:100%; }
    .controls { display:flex; justify-content:center; gap:1rem; margin:1rem; }
    .controls button { padding:0.8rem 1.2rem; font-size:1.2rem; border:none; border-radius:50%; cursor:pointer; }
    #filterBorrowed { background:#ffcccc; } #showForm,#showInstallmentForm { background:#cce5ff; }
    .repayments { background:#fafafa; border:1px solid #ddd; margin-top:0.5rem; padding:0.5rem; font-size:0.9rem; }
  </style>
</head>
<body>
  <header>
    <h1>💰 Transaction Tracker Pro</h1>
    <div class="date" id="currentDate"></div>
    <div class="notification" id="notifBtn">
      🔔<span class="badge" id="notifBadge">●</span>
      <div id="notifList"></div>
    </div>
  </header>

  <!-- Summary -->
  <section class="summary">
    <div class="box green">Total Lent: <span id="totalLent">0</span></div>
    <div class="box red">Total Borrowed: <span id="totalBorrowed">0</span></div>
    <div class="box neutral">Net Balance: <span id="netBalance">0</span></div>
    <div class="box neutral">Net Balance w/ Interest: <span id="netBalanceInterest">0</span></div>
  </section>

  <!-- Filter -->
  <div style="text-align:center;margin:1rem;">
    <label>Filter by Name: </label>
    <select id="nameFilter"><option value="">All</option></select>
  </div>

  <!-- Transactions Table -->
  <h2 style="text-align:center;">Normal Transactions</h2>
  <table id="transactionTable">
    <thead>
      <tr>
        <th>No</th><th>Name</th><th>Type</th><th>Amount</th><th>Interest %</th>
        <th>Start Date</th><th>Until Date</th><th>Days Passed</th><th>Interest</th><th>Net Balance</th><th>Action</th>
      </tr>
    </thead>
    <tbody id="tableBody"></tbody>
  </table>

  <!-- Installments -->
  <h2 style="text-align:center;">Installment Payments</h2>
  <table id="installmentTable">
    <thead>
      <tr>
        <th>No</th><th>Name</th><th>Loan Amount</th><th>Start Date</th><th>Interval</th><th>Outstanding</th><th>Repayments</th><th>Add Payment</th><th>Action</th>
      </tr>
    </thead>
    <tbody id="installmentBody"></tbody>
  </table>

  <!-- Transaction Form -->
  <section class="form-section" id="formSection">
    <h2>Add Transaction</h2>
    <form id="transactionForm">
      <input type="text" id="name" placeholder="Name" required>
      <select id="type" required><option value="Lent">Lent</option><option value="Borrowed">Borrowed</option></select>
      <input type="number" id="amount" placeholder="Amount" required>
      <input type="number" step="0.01" id="interest" placeholder="Interest %">
      <input type="date" id="startDate" required>
      <input type="date" id="untilDate" required>
      <button type="submit">Add</button>
    </form>
  </section>

  <!-- Installment Loan Form -->
  <section class="form-section" id="installmentFormSection">
    <h2>Add Installment Loan</h2>
    <form id="installmentForm">
      <input type="text" id="instName" placeholder="Name" required>
      <input type="number" id="instAmount" placeholder="Loan Amount" required>
      <input type="number" id="instPayment" placeholder="Installment Amount" required>
      <select id="instInterval">
        <option value="7">Weekly</option>
        <option value="10">Every 10 Days</option>
      </select>
      <input type="date" id="instStartDate" required>
      <button type="submit">Add Installment Loan</button>
    </form>
  </section>

  <!-- Buttons -->
  <div class="controls">
    <button id="filterBorrowed">B</button>
    <button id="showForm">+</button>
    <button id="showInstallmentForm">I</button>
  </div>

  <script>
    const form = document.getElementById('transactionForm');
    const tableBody = document.getElementById('tableBody');
    const installmentBody = document.getElementById('installmentBody');
    const showFormBtn = document.getElementById('showForm');
    const showInstallmentBtn = document.getElementById('showInstallmentForm');
    const formSection = document.getElementById('formSection');
    const instFormSection = document.getElementById('installmentFormSection');
    const notifBadge = document.getElementById('notifBadge');
    const notifList = document.getElementById('notifList');
    const nameFilter = document.getElementById('nameFilter');

    let transactions = JSON.parse(localStorage.getItem('transactions')) || [];
    let installments = JSON.parse(localStorage.getItem('installments')) || [];
    let showBorrowedOnly = false;

    document.getElementById('currentDate').innerText = new Date().toDateString();

    function calculateDays(start, until) {
      const s = new Date(start), u = new Date(until);
      return Math.max(0, Math.floor((u-s)/(1000*60*60*24)));
    }

    function checkReminders() {
      const today = new Date().toISOString().split("T")[0];
      let dueNames = [];

      installments.forEach(inst => {
        inst.schedule.forEach(d => {
          if(d === today && !inst.repayments.find(r=>r.date===d)) {
            dueNames.push(inst.name);
          }
        });
      });

      if(dueNames.length>0){
        notifBadge.style.display="inline-block";
        notifList.innerHTML = dueNames.map(n=>`<div>⚠️ ${n} payment due today</div>`).join("");
      } else { notifBadge.style.display="none"; notifList.innerHTML=""; }
    }

    function renderTransactions() {
      tableBody.innerHTML = ''; let totalLent=0,totalBorrowed=0,net=0,netI=0; let namesSet=new Set();
      transactions.forEach((t,i)=>{
        if(showBorrowedOnly&&t.type!=="Borrowed")return;
        if(nameFilter.value&&nameFilter.value!==t.name)return;
        const row=document.createElement('tr'); row.className=t.type.toLowerCase();
        const daysPassed=calculateDays(t.startDate,new Date());
        const months=daysPassed/30;
        const interestAmount=(t.amount*(parseFloat(t.interest)/100))*months;
        const netVal=t.type==="Lent"?t.amount+interestAmount:-(t.amount+interestAmount);
        if(t.type==="Lent")totalLent+=t.amount; else totalBorrowed+=t.amount;
        net+=t.type==="Lent"?t.amount:-t.amount; netI+=netVal;
        row.innerHTML=`
          <td>${i+1}</td><td>${t.name}</td><td>${t.type}</td><td>${t.amount}</td>
          <td>${t.interest}</td><td>${t.startDate}</td><td>${t.untilDate}</td>
          <td>${daysPassed}</td><td>${interestAmount.toFixed(2)}</td><td>${netVal.toFixed(2)}</td>
          <td><button onclick="deleteTransaction(${i})">❌</button></td>`;
        tableBody.appendChild(row); namesSet.add(t.name);
      });
      document.getElementById('totalLent').innerText=totalLent;
      document.getElementById('totalBorrowed').innerText=totalBorrowed;
      document.getElementById('netBalance').innerText=net.toFixed(2);
      document.getElementById('netBalanceInterest').innerText=netI.toFixed(2);
      localStorage.setItem('transactions',JSON.stringify(transactions));
      updateNameFilter(namesSet); checkReminders();
    }

    function renderInstallments() {
      installmentBody.innerHTML=''; let namesSet=new Set();
      installments.forEach((inst,i)=>{
        if(nameFilter.value&&nameFilter.value!==inst.name)return;
        const repaid=inst.repayments.reduce((s,r)=>s+r.amount,0);
        const outstanding=inst.amount-repaid;
        const row=document.createElement('tr');
        row.innerHTML=`
          <td>${i+1}</td><td>${inst.name}</td><td>${inst.amount}</td><td>${inst.startDate}</td>
          <td>${inst.interval==="7"?"Weekly":"Every 10 Days"}</td>
          <td>${outstanding}</td>
          <td><div class="repayments">${inst.repayments.map(r=>`<div>${r.date}: ${r.amount}</div>`).join("")||"No repayments yet"}</div></td>
          <td><button onclick="addRepayment(${i})">+Pay</button></td>
          <td><button onclick="deleteInstallment(${i})">❌</button></td>`;
        installmentBody.appendChild(row); namesSet.add(inst.name);
      });
      localStorage.setItem('installments',JSON.stringify(installments));
      updateNameFilter(namesSet); checkReminders();
    }

    function updateNameFilter(names){names.forEach(n=>{if(![...nameFilter.options].some(o=>o.value===n)){const op=document.createElement('option');op.value=n;op.textContent=n;nameFilter.appendChild(op);}})}

    form.addEventListener('submit',e=>{
      e.preventDefault();
      transactions.push({
        name:document.getElementById('name').value,
        type:document.getElementById('type').value,
        amount:parseFloat(document.getElementById('amount').value),
        interest:parseFloat(document.getElementById('interest').value||0),
        startDate:document.getElementById('startDate').value,
        untilDate:document.getElementById('untilDate').value
      });
      renderTransactions(); form.reset(); formSection.style.display='none';
    });

    document.getElementById('installmentForm').addEventListener('submit',e=>{
      e.preventDefault();
      const name=document.getElementById('instName').value;
      const amt=parseFloat(document.getElementById('instAmount').value);
      const pay=parseFloat(document.getElementById('instPayment').value);
      const interval=document.getElementById('instInterval').value;
      const start=document.getElementById('instStartDate').value;
      let schedule=[]; let remaining=amt; let current=new Date(start);
      while(remaining>0){schedule.push(current.toISOString().split("T")[0]);remaining-=pay;current.setDate(current.getDate()+parseInt(interval));}
      installments.push({name,amount:amt,payment:pay,interval,startDate:start,repayments:[],schedule});
      renderInstallments(); e.target.reset(); instFormSection.style.display='none';
    });

    function deleteTransaction(i){transactions.splice(i,1);renderTransactions();}
    function deleteInstallment(i){installments.splice(i,1);renderInstallments();}
    function addRepayment(i){
      const amount=parseFloat(prompt("Enter repayment amount:"));
      if(!isNaN(amount)&&amount>0){
        const today=new Date().toISOString().split("T")[0];
        installments[i].repayments.push({amount,date:today});
        renderInstallments();
      }
    }

    document.getElementById('filterBorrowed').addEventListener('click',()=>{showBorrowedOnly=!showBorrowedOnly;renderTransactions();});
    showFormBtn.addEventListener('click',()=>{formSection.style.display=formSection.style.display==='block'?'none':'block';});
    showInstallmentBtn.addEventListener('click',()=>{instFormSection.style.display=instFormSection.style.display==='block'?'none':'block';});
    document.getElementById('notifBtn').addEventListener('click',()=>{notifList.style.display=notifList.style.display==='block'?'none':'block';});
    nameFilter.addEventListener('change',()=>{renderTransactions();renderInstallments();});

    renderTransactions(); renderInstallments();
  </script>
</body>
</html>
