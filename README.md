<!DOCTYPE html>
<html lang="te">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>తాటి కల్లు లైవ్ స్టాక్</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f4f4;
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
        }
        .app-container {
            width: 100%;
            max-width: 450px;
            background-color: white;
            min-height: 100vh;
            box-shadow: 0 0 15px rgba(0,0,0,0.1);
            position: relative;
        }
        .app-bar {
            background-color: #388E3C; /* Green */
            color: white;
            padding: 15px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .app-bar.admin-mode {
            background-color: #448AFF; /* Blue for Admin */
        }
        .app-title {
            font-size: 20px;
            font-weight: bold;
            cursor: pointer;
            user-select: none;
        }
        .lang-btn {
            background: none;
            border: none;
            color: white;
            font-size: 16px;
            font-weight: bold;
            cursor: pointer;
        }
        .content {
            padding: 30px 20px;
            text-align: center;
        }
        .admin-msg-card {
            background-color: #ffe0b2;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 30px;
            display: none; /* Hidden by default */
            text-align: left;
            font-weight: bold;
            color: #e65100;
        }
        .stock-label {
            font-size: 22px;
            font-weight: 500;
            color: #333;
        }
        .stock-value {
            font-size: 60px;
            font-weight: bold;
            color: #2E7D32;
            margin: 15px 0 40px 0;
        }
        .wa-btn {
            background-color: #43A047;
            color: white;
            border: none;
            width: 100%;
            padding: 15px;
            font-size: 18px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
        }
        /* Admin Panel Styles */
        .admin-panel {
            display: none; /* Hidden by default */
            text-align: left;
        }
        .input-group {
            margin-bottom: 20px;
        }
        .input-group label {
            display: block;
            margin-bottom: 8px;
            font-weight: bold;
        }
        .input-group input {
            width: 100%;
            padding: 12px;
            font-size: 16px;
            border: 1px solid #ccc;
            border-radius: 6px;
            box-sizing: border-box;
        }
        .update-btn {
            background-color: #448AFF;
            color: white;
            border: none;
            width: 100%;
            padding: 15px;
            font-size: 18px;
            border-radius: 8px;
            cursor: pointer;
            font-weight: bold;
        }
        .close-admin {
            margin-top: 15px;
            background-color: #d32f2f;
        }
    </style>
</head>
<body>

<div class="app-container">
    <div class="app-bar" id="appBar">
        <div class="app-title" id="appTitle">తాజా స్టాక్</div>
        <button class="lang-btn" id="langBtn">English</button>
    </div>

    <div class="content">
        <div id="customerDashboard">
            <div class="admin-msg-card" id="adminMsgCard"></div>
            
            <div class="stock-label" id="stockLabel">ప్రస్తుతం ఉన్న తాటి కల్లు:</div>
            <div class="stock-value" id="stockValue">0 లీటర్లు</div>
            
            <button class="wa-btn" id="waBtn">వాట్సాప్ ద్వారా ఆర్డర్ చేయండి</button>
        </div>

        <div class="admin-panel" id="adminPanel">
            <h3 id="adminHeader">స్టాక్ అప్‌డేట్ చేయండి:</h3>
            
            <div class="input-group">
                <label id="kalluInputLabel">తాటి కల్లు (లీటర్లలో)</label>
                <input type="number" id="kalluInput" placeholder="0">
            </div>
            
            <div class="input-group">
                <label id="msgInputLabel">కస్టమర్లకు ప్రకటన (ఉదా: ఫ్రెష్ కల్లు వచ్చింది)</label>
                <input type="text" id="msgInput" placeholder="Message...">
            </div>
            
            <button class="update-btn" id="updateBtn">అప్‌డేట్ చేయండి</button>
            <button class="update-btn close-admin" id="closeAdminBtn">క్లోజ్ (Close Admin)</button>
        </div>
    </div>
</div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getDatabase, ref, onValue, update } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-database.js";

    // 🔴 ముఖ్య గమనిక: ఇక్కడ మీ Firebase ప్రాజెక్ట్ Config వివరాలు పెట్టాలి
    const firebaseConfig = {
        apiKey: "YOUR_API_KEY",
        authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
        databaseURL: "https://YOUR_PROJECT_ID-default-rtdb.firebaseio.com",
        projectId: "YOUR_PROJECT_ID",
        storageBucket: "YOUR_PROJECT_ID.appspot.com",
        messagingSenderId: "YOUR_SENDER_ID",
        appId: "YOUR_APP_ID"
    };

    // Initialize Firebase
    const app = initializeApp(firebaseConfig);
    const database = getDatabase(app);
    const shopRef = ref(database, 'shop_data');

    // State Variables
    let isTelugu = true;
    let isAdminMode = false;
    let currentKalluLiters = 0;
    
    // DOM Elements
    const appTitle = document.getElementById('appTitle');
    const langBtn = document.getElementById('langBtn');
    const appBar = document.getElementById('appBar');
    
    const customerDashboard = document.getElementById('customerDashboard');
    const adminMsgCard = document.getElementById('adminMsgCard');
    const stockLabel = document.getElementById('stockLabel');
    const stockValue = document.getElementById('stockValue');
    const waBtn = document.getElementById('waBtn');
    
    const adminPanel = document.getElementById('adminPanel');
    const adminHeader = document.getElementById('adminHeader');
    const kalluInputLabel = document.getElementById('kalluInputLabel');
    const msgInputLabel = document.getElementById('msgInputLabel');
    const kalluInput = document.getElementById('kalluInput');
    const msgInput = document.getElementById('msgInput');
    const updateBtn = document.getElementById('updateBtn');
    const closeAdminBtn = document.getElementById('closeAdminBtn');

    // Language Dictionary
    const texts = {
        te: {
            title: "తాజా స్టాక్",
            adminTitle: "అడ్మిన్ డ్యాష్‌బోర్డ్",
            langText: "English",
            stockLabel: "ప్రస్తుతం ఉన్న తాటి కల్లు:",
            liters: "లీటర్లు",
            waBtn: "వాట్సాప్ ద్వారా ఆర్డర్ చేయండి",
            adminHeader: "స్టాక్ అప్‌డేట్ చేయండి:",
            kalluInputLabel: "తాటి కల్లు (లీటర్లలో)",
            msgInputLabel: "కస్టమర్లకు ప్రకటన (ఉదా: ఫ్రెష్ కల్లు వచ్చింది)",
            updateBtn: "అప్‌డేట్ చేయండి",
            waMsg: "హలో, నాకు తాటి కల్లు కావాలి.",
            successAlert: "స్టాక్ అప్‌డేట్ చేయబడింది!"
        },
        en: {
            title: "Live Stock",
            adminTitle: "Admin Dashboard",
            langText: "తెలుగు",
            stockLabel: "Available Thati Kallu:",
            liters: "Liters",
            waBtn: "Order via WhatsApp",
            adminHeader: "Update Stock:",
            kalluInputLabel: "Thati Kallu (in Liters)",
            msgInputLabel: "Announcement (e.g., Fresh Kallu available)",
            updateBtn: "Update",
            waMsg: "Hello, I want Thati Kallu.",
            successAlert: "Stock Successfully Updated!"
        }
    };

    // Update UI text based on language
    function updateLanguageUI() {
        const lang = isTelugu ? 'te' : 'en';
        appTitle.innerText = isAdminMode ? texts[lang].adminTitle : texts[lang].title;
        langBtn.innerText = texts[lang].langText;
        
        // Customer UI
        stockLabel.innerText = texts[lang].stockLabel;
        stockValue.innerText = `${currentKalluLiters} ${texts[lang].liters}`;
        waBtn.innerText = texts[lang].waBtn;
        
        // Admin UI
        adminHeader.innerText = texts[lang].adminHeader;
        kalluInputLabel.innerText = texts[lang].kalluInputLabel;
        msgInputLabel.innerText = texts[lang].msgInputLabel;
        updateBtn.innerText = texts[lang].updateBtn;
    }

    // Toggle Language
    langBtn.addEventListener('click', () => {
        isTelugu = !isTelugu;
        updateLanguageUI();
    });

    // --- Firebase Live Data Listener ---
    onValue(shopRef, (snapshot) => {
        const data = snapshot.val();
        if (data) {
            currentKalluLiters = data.thati_kallu || 0;
            const adminMessage = data.admin_message || "";
            
            // Update Liters
            const lang = isTelugu ? 'te' : 'en';
            stockValue.innerText = `${currentKalluLiters} ${texts[lang].liters}`;
            
            // Update Admin Message Card
            if (adminMessage.trim() !== "") {
                adminMsgCard.style.display = "block";
                adminMsgCard.innerHTML = `📢 ${adminMessage}`;
            } else {
                adminMsgCard.style.display = "none";
            }
        }
    });

    // --- WhatsApp Order Button ---
    waBtn.addEventListener('click', () => {
        const adminNumber = "919989471413"; // ఇక్కడ మీ వాట్సాప్ నెంబర్ ఇవ్వండి
        const lang = isTelugu ? 'te' : 'en';
        const message = texts[lang].waMsg;
        const url = `https://wa.me/${adminNumber}?text=${encodeURIComponent(message)}`;
        window.open(url, '_blank');
    });

    // --- Secret Admin Login (Double Click/Tap on Title) ---
    // ఫ్లట్టర్‌లో లాంగ్ ప్రెస్ వాడాము, వెబ్‌లో డబుల్ క్లిక్ సులువుగా ఉంటుంది
    let clickCount = 0;
    appTitle.addEventListener('click', () => {
        clickCount++;
        if (clickCount === 2) {
            const password = prompt("Enter Admin Password:");
            if (password === "1985") {
                openAdminPanel();
            } else if (password !== null) {
                alert("Wrong Password!");
            }
            clickCount = 0; // Reset
        }
        // Reset count after 1 second if not double clicked
        setTimeout(() => clickCount = 0, 1000); 
    });

    function openAdminPanel() {
        isAdminMode = true;
        customerDashboard.style.display = "none";
        adminPanel.style.display = "block";
        appBar.classList.add("admin-mode");
        updateLanguageUI();
    }

    closeAdminBtn.addEventListener('click', () => {
        isAdminMode = false;
        customerDashboard.style.display = "block";
        adminPanel.style.display = "none";
        appBar.classList.remove("admin-mode");
        updateLanguageUI();
    });

    // --- Update Stock to Firebase ---
    updateBtn.addEventListener('click', () => {
        const newStock = parseInt(kalluInput.value) || 0;
        const newMsg = msgInput.value || "";

        update(shopRef, {
            thati_kallu: newStock,
            admin_message: newMsg
        }).then(() => {
            const lang = isTelugu ? 'te' : 'en';
            alert(texts[lang].successAlert);
            kalluInput.value = "";
            msgInput.value = "";
        }).catch((error) => {
            alert("Error updating stock: " + error);
        });
    });

    // Initialize UI
    updateLanguageUI();
</script>

</body>
</html>
