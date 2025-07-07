
new_sender_html = """
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <title>تقرير المروجين</title>
    <script type="module" src="./firebase-config.js"></script>
    <style>
        body {
            font-family: 'Cairo', sans-serif;
            background: linear-gradient(135deg, #007BFF, #00C3FF);
            color: #333;
            padding: 20px;
            direction: rtl;
        }

        .container {
            max-width: 900px;
            background: #fff;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            margin: auto;
        }

        h1 {
            text-align: center;
            color: #007BFF;
            margin-bottom: 20px;
        }

        label {
            display: block;
            margin-top: 15px;
            font-weight: bold;
        }

        input {
            width: 100%;
            padding: 10px;
            margin-top: 5px;
            border-radius: 8px;
            border: 1px solid #ccc;
            font-size: 16px;
        }

        .buttons {
            margin-top: 20px;
            text-align: center;
        }

        button {
            padding: 12px 20px;
            margin: 5px;
            border: none;
            border-radius: 8px;
            background: #007BFF;
            color: #fff;
            font-size: 16px;
            cursor: pointer;
        }

        button:hover {
            background: #0056b3;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 25px;
        }

        th, td {
            padding: 10px;
            border: 1px solid #ccc;
            text-align: center;
        }

        th {
            background: #007BFF;
            color: white;
        }

        .totals {
            font-weight: bold;
            margin-top: 15px;
            background: #f1f1f1;
            padding: 10px;
            border-radius: 8px;
        }

        .highlight {
            font-weight: bold;
            color: #007BFF;
        }
    </style>
</head>
<body>
<div class="container">
    <h1>تقرير المروجين</h1>

    <label>اسم المروج</label>
    <input type="text" id="promoter">

    <label>تاريخ اليوم</label>
    <input type="date" id="date">

    <label>نوع المنتج</label>
    <input type="text" id="productType">

    <label>اسم الحساب الإعلاني</label>
    <input type="text" id="adAccount">

    <label>اسم الصفحة</label>
    <input type="text" id="pageName">

    <label>عدد الرسائل</label>
    <input type="number" id="messages" value="0">

    <label>عدد الحجوزات</label>
    <input type="number" id="bookings" value="0">

    <label>الصرف بالدولار</label>
    <input type="number" id="spend" value="0">

    <div class="buttons">
        <button onclick="addRow()">إضافة حساب إعلاني</button>
        <button onclick="sendReport()">📤 إرسال التقرير</button>
    </div>

    <div id="reportMeta" class="highlight"></div>

    <table id="reportTable">
        <thead>
            <tr>
                <th>نوع المنتج</th>
                <th>اسم الحساب الإعلاني</th>
                <th>اسم الصفحة</th>
                <th>عدد الرسائل</th>
                <th>عدد الحجوزات</th>
                <th>الصرف بالدولار</th>
            </tr>
        </thead>
        <tbody></tbody>
    </table>

    <div class="totals">
        المجموع → رسائل: <span id="totalMessages">0</span> |
        حجوزات: <span id="totalBookings">0</span> |
        الصرف بالدولار: <span id="totalSpend">0</span>
    </div>
</div>

<script type="module">
    import { db, ref, set } from "./firebase-config.js";

    let totalMessages = 0;
    let totalBookings = 0;
    let totalSpend = 0;

    function updateTotals(messages, bookings, spend) {
        totalMessages += messages;
        totalBookings += bookings;
        totalSpend += spend;

        document.getElementById("totalMessages").innerText = totalMessages;
        document.getElementById("totalBookings").innerText = totalBookings;
        document.getElementById("totalSpend").innerText = totalSpend.toFixed(2);
    }

    window.addRow = function () {
        const promoter = document.getElementById("promoter").value.trim();
        const date = document.getElementById("date").value;
        const productType = document.getElementById("productType").value;
        const adAccount = document.getElementById("adAccount").value;
        const pageName = document.getElementById("pageName").value;
        const messages = parseInt(document.getElementById("messages").value) || 0;
        const bookings = parseInt(document.getElementById("bookings").value) || 0;
        const spend = parseFloat(document.getElementById("spend").value) || 0;

        if (!promoter || !date || !productType || !adAccount || !pageName) {
            alert("يرجى ملء جميع الحقول.");
            return;
        }

        document.getElementById("reportMeta").innerText = `اسم المروج: ${promoter} | التاريخ: ${date}`;

        const table = document.querySelector("#reportTable tbody");
        const row = table.insertRow();
        [productType, adAccount, pageName, messages, bookings, spend.toFixed(2)].forEach(text => {
            const cell = row.insertCell();
            cell.innerText = text;
        });

        updateTotals(messages, bookings, spend);

        document.getElementById("productType").value = '';
        document.getElementById("adAccount").value = '';
        document.getElementById("pageName").value = '';
        document.getElementById("messages").value = 0;
        document.getElementById("bookings").value = 0;
        document.getElementById("spend").value = 0;
    }

    window.sendReport = async function () {
        const promoter = document.getElementById("promoter").value.trim();
        const date = document.getElementById("date").value;
        if (!promoter || !date) {
            alert("يرجى إدخال اسم المروج والتاريخ.");
            return;
        }

        const rows = document.querySelectorAll("#reportTable tbody tr");
        if (rows.length === 0) {
            alert("لا يوجد تقارير لإرسالها.");
            return;
        }

        const data = Array.from(rows).map(row => Array.from(row.cells).map(cell => cell.innerText));
        await set(ref(db, `reports/${promoter}_${date}`), data);
        alert("✅ تم إرسال التقرير بنجاح.");
    }
</script>
</body>
</html>
