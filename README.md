# BAF-Ration-Store-<!DOCTYPE html>
<html lang="bn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BAF Ration Store</title>
    <style>
        :root { --baf-blue: #003366; --accent: #ffd700; --bg: #f4f7f6; }
        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--bg); margin: 0; display: flex; justify-content: center; align-items: center; min-height: 100vh; }
        
        .app-card { width: 90%; max-width: 400px; background: white; border-radius: 20px; box-shadow: 0 15px 35px rgba(0,0,0,0.2); overflow: hidden; animation: fadeIn 0.5s; }
        
        .header { background: var(--baf-blue); color: white; padding: 40px 20px; text-align: center; border-bottom: 5px solid var(--accent); }
        .header h1 { margin: 0; font-size: 24px; letter-spacing: 1px; }
        .motivation { font-size: 11px; margin-top: 10px; font-style: italic; opacity: 0.9; }

        .content { padding: 30px 20px; }
        .input-group { margin-bottom: 20px; position: relative; }
        label { font-size: 12px; font-weight: bold; color: var(--baf-blue); display: block; margin-bottom: 5px; }
        input { width: 100%; padding: 12px; border: 2px solid #eee; border-radius: 10px; box-sizing: border-box; transition: 0.3s; }
        input:focus { border-color: var(--baf-blue); outline: none; }

        .btn-main { width: 100%; padding: 14px; background: var(--baf-blue); color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; font-size: 16px; box-shadow: 0 5px 15px rgba(0,51,102,0.3); }
        .btn-main:active { transform: scale(0.98); }

        .footer-links { text-align: center; margin-top: 20px; font-size: 13px; }
        .footer-links a { color: var(--baf-blue); text-decoration: none; font-weight: bold; margin: 0 10px; }

        /* ভাউচার ভিউ স্টাইল */
        #voucherSection { display: none; padding: 20px; }
        .voucher-item { background: #fff; border-left: 5px solid var(--baf-blue); padding: 15px; border-radius: 8px; margin-bottom: 15px; box-shadow: 0 3px 10px rgba(0,0,0,0.05); display: flex; justify-content: space-between; align-items: center; }
        .action-btns { display: flex; gap: 10px; }
        .btn-action { padding: 8px 12px; border-radius: 5px; border: none; font-size: 12px; font-weight: bold; cursor: pointer; }
        .collect { background: #28a745; color: white; }
        .sell { background: #e67e22; color: white; }

        @keyframes fadeIn { from { opacity: 0; transform: translateY(20px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body>

<div class="app-card">
    <div id="loginSection">
        <div class="header">
            <h1>BAF RATION STORE</h1>
            <p class="motivation">"শৃঙ্খলা ও আধুনিক সেবায় আমরা অঙ্গীকারবদ্ধ"</p>
        </div>
        <div class="content">
            <div class="input-group">
                <label>বিডি নম্বর (BD No)</label>
                <input type="text" id="bdNo" placeholder="আপনার আইডি লিখুন">
            </div>
            <div class="input-group">
                <label>নির্ধারিত পিন (PIN)</label>
                <input type="password" id="pin" placeholder="৪ ডিজিটের পিন">
            </div>
            <button class="btn-main" onclick="handleLogin()">লগইন করুন</button>
            
            <div class="footer-links">
                <a href="#" onclick="alert('রেজিস্ট্রেশন করতে অ্যাডমিনের সাথে যোগাযোগ করুন')">রেজিস্টার</a> | 
                <a href="#" onclick="alert('আপনার পাসওয়ার্ড রিসেট করতে অ্যাডমিনে মেসেজ দিন')">পিন ভুলে গেছেন?</a>
            </div>
        </div>
    </div>

    <div id="voucherSection">
        <h2 style="color: var(--baf-blue); text-align: center;">আপনার রেশন ভাউচার</h2>
        <p id="welcomeMsg" style="text-align: center; font-size: 14px; color: #666;"></p>
        <div id="itemsContainer">
            </div>
        <button class="btn-main" style="background: #666; margin-top: 20px;" onclick="location.reload()">লগআউট</button>
    </div>
</div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
    import { getDatabase, ref, get, child } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-database.js";

    // আপনার স্ক্রিনশট থেকে এই তথ্যগুলো এখানে বসান
    const firebaseConfig = {
        apiKey: "এখানে_আপনার_ApiKey_বসান",
        authDomain: "baf-ration-store.firebaseapp.com",
        databaseURL: "https://baf-ration-store-default-rtdb.firebaseio.com",
        projectId: "baf-ration-store",
        storageBucket: "baf-ration-store.firebasestorage.app",
        messagingSenderId: "7233120201",
        appId: "1:7233120201:web:48bcce0b58c1f07bc52a18"
    };

    const app = initializeApp(firebaseConfig);
    const dbRef = ref(getDatabase(app));

    window.handleLogin = function() {
        const id = document.getElementById('bdNo').value;
        const pin = document.getElementById('pin').value;

        if(!id || !pin) return alert("আইডি এবং পিন দিন");

        get(child(dbRef, `users/${id}`)).then((snapshot) => {
            if (snapshot.exists()) {
                const data = snapshot.val();
                if(data.pin === pin) {
                    showVouchers(id, data.items);
                } else {
                    alert("ভুল পিন!");
                }
            } else {
                alert("এই বিডি নম্বরটি নিবন্ধিত নয়!");
            }
        });
    }

    function showVouchers(id, items) {
        document.getElementById('loginSection').style.display = 'none';
        document.getElementById('voucherSection').style.display = 'block';
        document.getElementById('welcomeMsg').innerText = "BD No: " + id + " এর বরাদ্দকৃত তালিকা";
        
        const container = document.getElementById('itemsContainer');
        for (let item in items) {
            container.innerHTML += `
                <div class="voucher-item">
                    <div>
                        <div style="font-weight:bold">${item}</div>
                        <div style="font-size:12px; color:#777">${items[item]}</div>
                    </div>
                    <div class="action-btns">
                        <button class="btn-action collect" onclick="alert('আবেদন পাঠানো হয়েছে')">গ্রহণ</button>
                        <button class="btn-action sell" onclick="alert('বিক্রির অনুরোধ পাঠানো হয়েছে')">বিক্রি</button>
                    </div>
                </div>
            `;
        }
    }
</script>

</body>
</html>
