
# IFIC-BANK-
<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>আইপিসি ব্যাংক ডিপোজিট ট্র্যাকার</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        body {
            background-color: #f4f7f6;
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 800px;
            margin: 0 auto;
            background: #ffffff;
            padding: 25px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
        }
        h2 {
            text-align: center;
            color: #1a5276;
            margin-bottom: 20px;
        }
        .summary-box {
            display: flex;
            justify-content: space-between;
            background: #ebf5fb;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
            border-left: 5px solid #2980b9;
        }
        .summary-box div {
            font-size: 18px;
            font-weight: bold;
            color: #2c3e50;
        }
        form {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 25px;
        }
        .form-group {
            display: flex;
            flex-direction: column;
        }
        .form-group.full-width {
            grid-column: span 2;
        }
        label {
            margin-bottom: 5px;
            font-weight: 600;
            color: #34495e;
        }
        input, select {
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            font-size: 15px;
        }
        button {
            grid-column: span 2;
            padding: 12px;
            background-color: #27ae60;
            color: white;
            border: none;
            border-radius: 6px;
            font-size: 16px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.3s;
        }
        button:hover {
            background-color: #219150;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
        }
        th, td {
            padding: 12px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        th {
            background-color: #34495e;
            color: white;
        }
        tr:hover {
            background-color: #f1f1f1;
        }
        .delete-btn {
            background-color: #e74c3c;
            color: white;
            border: none;
            padding: 6px 10px;
            border-radius: 4px;
            cursor: pointer;
        }
        .delete-btn:hover {
            background-color: #c0392b;
        }
        @media (max-width: 600px) {
            form {
                grid-template-columns: 1fr;
            }
            button, .form-group.full-width {
                grid-column: span 1;
            }
        }
    </style>
</head>
<body>

<div class="container">
    <h2>আইপিসি ব্যাংক - ডিপোজিট ট্র্যাকার</h2>

    <div class="summary-box">
        <div>মোট জমা: <span id="total-amount" style="color: #27ae60;">০</span> টাকা</div>
        <div>মোট এন্ট্রি: <span id="total-count">০</span> টি</div>
    </div>

    <form id="deposit-form">
        <div class="form-group">
            <label for="date">তারিখ:</label>
            <input type="date" id="date" required>
        </div>
        <div class="form-group">
            <label for="account">হিসাব নম্বর / নাম:</label>
            <input type="text" id="account" placeholder="যেমন: DPS-1025" required>
        </div>
        <div class="form-group">
            <label for="amount">টাকার পরিমাণ (BDT):</label>
            <input type="number" id="amount" placeholder="যেমন: 5000" min="1" required>
        </div>
        <div class="form-group">
            <label for="type">ডিপোজিটের ধরন:</label>
            <select id="type">
                <option value="DPS">DPS (ডিপিএস)</option>
                <option value="FDR">FDR (এফডিআর)</option>
                <option value="Savings">সাধারণ সঞ্চয়</option>
            </select>
        </div>
        <div class="form-group full-width">
            <label for="note">নোট/মন্তব্য (ঐচ্ছিক):</label>
            <input type="text" id="note" placeholder="যেমন: জানুয়ারী মাসের কিস্তি">
        </div>
        <button type="submit">জমা যুক্ত করুন</button>
    </form>

    <h3>জমা খাতার তালিকা</h3>
    <table>
        <thead>
            <tr>
                <th>তারিখ</th>
                <th>হিসাব</th>
                <th>ধরন</th>
                <th>পরিমাণ</th>
                <th>নোট</th>
                <th>অ্যাকশন</th>
            </tr>
        </thead>
        <tbody id="deposit-list">
            <!-- তথ্য এখানে যুক্ত হবে -->
        </tbody>
    </table>
</div>

<script>
    const form = document.getElementById('deposit-form');
    const depositList = document.getElementById('deposit-list');
    const totalAmountEl = document.getElementById('total-amount');
    const totalCountEl = document.getElementById('total-count');

    // স্থানীয় ডাটাবেজ (LocalStorage) থেকে তথ্য লোড করা
    let deposits = JSON.parse(localStorage.getItem('ipc_deposits')) || [];

    // আজকের তারিখ ডিফল্টভাবে সেট করা
    document.getElementById('date').valueAsDate = new Date();

    function updateUI() {
        depositList.innerHTML = '';
        let total = 0;

        deposits.forEach((item, index) => {
            total += parseFloat(item.amount);
            const row = document.createElement('tr');
            row.innerHTML = `
                <td>${item.date}</td>
                <td>${item.account}</td>
                <td>${item.type}</td>
                <td>${parseFloat(item.amount).toLocaleString('bn-BD')} ৳</td>
                <td>${item.note || '-'}</td>
                <td><button class="delete-btn" onclick="deleteDeposit(${index})">মুছুন</button></td>
            `;
            depositList.appendChild(row);
        });

        totalAmountEl.textContent = total.toLocaleString('bn-BD');
        totalCountEl.textContent = deposits.length.toLocaleString('bn-BD');

        // লোকাল স্টোরেজে সেভ করা
        localStorage.setItem('ipc_deposits', JSON.stringify(deposits));
    }

    form.addEventListener('submit', (e) => {
        e.preventDefault();

        const newDeposit = {
            date: document.getElementById('date').value,
            account: document.getElementById('account').value,
            amount: document.getElementById('amount').value,
            type: document.getElementById('type').value,
            note: document.getElementById('note').value
        };

        deposits.push(newDeposit);
        updateUI();

        // ফর্ম রিসেট
        document.getElementById('account').value = '';
        document.getElementById('amount').value = '';
        document.getElementById('note').value = '';
    });

    function deleteDeposit(index) {
        if (confirm('আপনি কি এই হিসাবটি মুছে ফেলতে চান?')) {
            deposits.splice(index, 1);
            updateUI();
        }
    }

    // প্রথমবার পেজ লোড হলে UI দেখানো
    updateUI();
</script>

</body>
</html>
