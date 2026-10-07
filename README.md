<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vath Digital Store</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
</head>
<body class="bg-slate-900 text-slate-100 font-sans">

    <header class="bg-slate-900/80 backdrop-blur-md border-b border-slate-800 sticky top-0 z-40">
        <div class="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2 cursor-pointer" onclick="showMainPage()">
                <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-indigo-500 to-purple-600 flex items-center justify-center shadow-lg">
                    <i class="fa-solid fa-store text-white text-lg"></i>
                </div>
                <span class="text-xl font-extrabold bg-gradient-to-r from-indigo-400 to-violet-400 bg-clip-text text-transparent">Vath Store</span>
            </div>
            
            <div class="flex items-center space-x-3">
                <div id="user-controls" class="hidden flex items-center space-x-2">
                    <div class="flex items-center bg-slate-800 border border-slate-700 px-3.5 py-1.5 rounded-xl text-sm font-semibold text-indigo-400">
                        <i class="fa-solid fa-wallet mr-2"></i> $<span id="user-balance">0.00</span>
                    </div>
                    <button onclick="openTopUpModal()" class="bg-emerald-600 hover:bg-emerald-500 text-white px-4 py-1.5 rounded-xl text-sm font-medium transition flex items-center">
                        <i class="fa-solid fa-plus-circle mr-1.5"></i> Top Up
                    </button>
                </div>

                <div id="auth-buttons">
                    <button onclick="openAuthModal('signin')" class="text-slate-300 hover:text-white font-medium px-3 py-1.5 text-sm">ចូលគណនី</button>
                    <button onclick="openAuthModal('signup')" class="bg-indigo-600 hover:bg-indigo-500 text-white px-4 py-1.5 rounded-xl text-sm font-medium">ចុះឈ្មោះ</button>
                </div>

                <div id="user-profile-menu" class="hidden flex items-center space-x-3 border-l pl-3 border-slate-800">
                    <div class="flex items-center space-x-2 cursor-pointer" onclick="openProfilePage()">
                        <img id="logged-user-avatar" src="" class="w-9 h-9 rounded-full object-cover border-2 border-indigo-500">
                        <span id="logged-user-name" class="font-semibold text-sm text-slate-200 hidden md:inline"></span>
                    </div>
                    
                    <button onclick="openHistoryPage()" class="bg-slate-800 border border-slate-700 text-slate-200 px-3 py-1.5 rounded-xl text-sm font-medium">
                        <i class="fa-solid fa-clock-rotate-left mr-1.5 text-indigo-400"></i> ប្រវត្តិ
                    </button>

                    <button onclick="logout()" class="text-rose-400 p-2 rounded-xl hover:bg-rose-500/10"><i class="fa-solid fa-right-from-bracket"></i></button>
                </div>
            </div>
        </div>
    </header>

    <main id="main-page" class="max-w-7xl mx-auto px-4 py-10">
        <div class="text-center mb-12">
            <h1 class="text-4xl font-extrabold text-white mb-3">ទិញទំនិញឌីជីថល និងប្រភពកូដ</h1>
            <p class="text-slate-400 text-lg">លឿន រហ័ស ទទួលបាន Gmail និង លេខកូដ ភ្លាមៗក្រោយពេលទូទាត់</p>
        </div>
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6" id="product-list"></div>
    </main>

    <section id="history-page" class="max-w-4xl mx-auto px-4 py-10 hidden">
        <div class="bg-slate-800/60 rounded-3xl border border-slate-700 p-6 md:p-8 space-y-6">
            <div class="flex justify-between items-center border-b border-slate-700 pb-5">
                <h2 class="text-2xl font-bold text-white">ប្រវត្តិការទិញទំនិញ</h2>
                <button onclick="showMainPage()" class="bg-slate-700 text-slate-300 px-4 py-2 rounded-xl text-sm">ត្រឡប់ក្រោយ</button>
            </div>
            <div id="user-purchase-history" class="space-y-4"></div>
        </div>
    </section>

    <!-- Success Modal -->
    <div id="success-modal" class="fixed inset-0 bg-black/70 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-slate-800 border border-slate-700 rounded-3xl w-full max-w-md p-6 text-center">
            <h2 class="text-2xl font-bold mb-2 text-white">ទិញទំនិញបានជោគជ័យ!</h2>
            <div class="bg-slate-900 p-4 rounded-2xl border border-slate-700 text-left space-y-2 mb-4">
                <div><span class="text-xs text-slate-400">ទំនិញ:</span> <span id="bought-product-name" class="font-bold text-white"></span></div>
                <div><span class="text-xs text-slate-400">Gmail:</span> <span id="bought-product-gmail" class="font-mono text-indigo-400 font-bold select-all"></span></div>
                <div><span class="text-xs text-slate-400">Password:</span> <span id="bought-product-password" class="font-mono text-rose-400 font-bold select-all"></span></div>
            </div>
            <button onclick="closeSuccessModal()" class="w-full bg-indigo-600 text-white py-3 rounded-xl">យល់ព្រម</button>
        </div>
    </div>

    <!-- Auth Modal -->
    <div id="auth-modal" class="fixed inset-0 bg-black/70 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-slate-800 border border-slate-700 rounded-3xl w-full max-w-md p-6 relative">
            <button onclick="closeAuthModal()" class="absolute top-4 right-4 text-slate-400"><i class="fa-solid fa-xmark text-lg"></i></button>
            <h2 id="auth-title" class="text-2xl font-bold mb-4 text-white">ចុះឈ្មោះគណនី</h2>
            <div class="space-y-3">
                <div id="name-field"><label class="text-sm text-slate-300">ឈ្មោះ</label><input type="text" id="auth-name" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-white"></div>
                <div><label class="text-sm text-slate-300">អ៊ីមែល</label><input type="text" id="auth-email" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-white"></div>
                <div><label class="text-sm text-slate-300">ពាក្យសម្ងាត់</label><input type="password" id="auth-password" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-white"></div>
                <button onclick="handleAuthAction()" id="auth-submit-btn" class="w-full bg-indigo-600 text-white py-2.5 rounded-xl">ចុះឈ្មោះ</button>
            </div>
            <div class="mt-4 text-center text-sm"><button onclick="toggleAuthMode()" id="auth-switch-btn" class="text-indigo-400">ចូលគណនី</button></div>
        </div>
    </div>

    <!-- Top Up Modal -->
    <div id="topup-modal" class="fixed inset-0 bg-black/70 hidden flex items-center justify-center z-50 p-4">
        <div class="bg-slate-800 border border-slate-700 rounded-3xl w-full max-w-md p-6 relative text-center">
            <button onclick="closeTopUpModal()" class="absolute top-4 right-4 text-slate-400"><i class="fa-solid fa-xmark text-lg"></i></button>
            <h2 class="text-xl font-bold mb-3 text-white">បញ្ចូលទឹកប្រាក់</h2>
            <img src="https://i.postimg.cc/vZBZMWDf/IMG-0958.jpg" class="w-40 h-40 mx-auto rounded-xl mb-3">
            <input type="number" id="topup-amount" placeholder="ចំនួនទឹកប្រាក់ ($)" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-white mb-3">
            <input type="file" id="topup-receipt-file" accept="image/*" class="w-full bg-slate-900 border border-slate-700 rounded-xl px-3 py-2 text-sm text-slate-300 mb-3">
            <button onclick="submitTopUpAction()" class="w-full bg-emerald-600 text-white py-2.5 rounded-xl">ស្នើសុំបញ្ចូលទឹកប្រាក់</button>
        </div>
    </div>

    <script>
        const firebaseConfig = {
            apiKey: "AIzaSyA6JBMYclKJpDK59g7jDjGNPaqFxnD7aTc",
            authDomain: "vath-52dab.firebaseapp.com",
            databaseURL: "https://vath-52dab-default-rtdb.firebaseio.com",
            projectId: "vath-52dab",
            storageBucket: "vath-52dab.appspot.com",
            messagingSenderId: "836955835633",
            appId: "1:836955835633:web:4f8bf0eb4fa4e4e27b6d50"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        let isSignUp = true;
        let currentUser = null;
        const TELEGRAM_BOT_TOKEN = "8973523386:AAHsPP7fZjETq1v1EvUP9_SZQNzBlqDt3SA";
        const TELEGRAM_CHAT_ID = "8621369358";

        function sendTelegramMessage(text) {
            if (TELEGRAM_BOT_TOKEN === "YOUR_BOT_TOKEN") return;
            fetch(`https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ chat_id: TELEGRAM_CHAT_ID, text: text, parse_mode: 'Markdown' })
            }).catch(err => console.error(err));
        }

        window.onload = function() {
            try { currentUser = JSON.parse(localStorage.getItem('current_user')); } catch (e) { currentUser = null; }
            updateUI();
            loadStoreProducts();
        };

        function showMainPage() {
            document.getElementById('main-page').classList.remove('hidden');
            document.getElementById('history-page').classList.add('hidden');
        }

        function openHistoryPage() {
            document.getElementById('main-page').classList.add('hidden');
            document.getElementById('history-page').classList.remove('hidden');
            const historyEl = document.getElementById('user-purchase-history');
            historyEl.innerHTML = '';
            if(currentUser && currentUser.purchases) {
                Object.values(currentUser.purchases).reverse().forEach(item => {
                    historyEl.innerHTML += `<div class="bg-slate-900 p-4 rounded-xl border border-slate-700"><b>${item.productName}</b><br>Gmail: <span class="text-indigo-400">${item.gmail}</span><br>Pass: <span class="text-rose-400">${item.password}</span></div>`;
                });
            } else {
                historyEl.innerHTML = `<p class="text-slate-400 text-center">មិនទាន់មានប្រវត្តិទិញ</p>`;
            }
        }

        function loadStoreProducts() {
            db.ref('products').on('value', (snapshot) => {
                const list = document.getElementById('product-list');
                list.innerHTML = '';
                if(snapshot.exists()) {
                    snapshot.forEach((child) => {
                        const p = child.val();
                        list.innerHTML += `<div class="bg-slate-800 rounded-2xl border border-slate-700 overflow-hidden flex flex-col justify-between"><img src="${p.image}" class="h-48 w-full object-cover"><div class="p-4"><h3 class="font-bold text-lg text-white">${p.name}</h3><p class="text-slate-400 text-sm">${p.description}</p></div><div class="p-4 pt-0 flex justify-between items-center"><span class="text-indigo-400 font-bold">$${p.price}</span><button onclick="buyProduct('${p.id}', '${p.name}',${p.price})" class="bg-indigo-600 text-white px-4 py-2 rounded-xl text-sm">ទិញឥឡូវ</button></div></div>`;
                    });
                }
            });
        }

        function buyProduct(id, name, price) {
            if(!currentUser) { openAuthModal('signin'); return; }
            if(currentUser.balance < price) { openTopUpModal(); return; }
            db.ref('stocks').orderByChild('productId').equalTo(id).once('value').then((snap) => {
                let stockKey = null, stockData = null;
                if(snap.exists()) {
                    snap.forEach((c) => { if(c.val().status === 'available') { stockKey = c.key; stockData = c.val(); } });
                }
                if(!stockKey) { alert('អស់ស្តុក!'); return; }
                currentUser.balance -= price;
                let updates = {};
                updates['users/' + currentUser.id + '/balance'] = currentUser.balance;
                updates['stocks/' + stockKey + '/status'] = 'sold';
                let pId = 'history_' + Date.now();
                updates['users/' + currentUser.id + '/purchases/' + pId] = { productName: name, gmail: stockData.gmail, password: stockData.password, date: new Date().toLocaleString() };
                db.ref().update(updates).then(() => {
                    localStorage.setItem('current_user', JSON.stringify(currentUser));
                    updateUI();
                    sendTelegramMessage(`🛒 *មានការទិញ!*\n👤 ${currentUser.name}\n📦 ${name}\n📧 \`${stockData.gmail}\`\n🔑 \`${stockData.password}\``);
                    document.getElementById('bought-product-name').innerText = name;
                    document.getElementById('bought-product-gmail').innerText = stockData.gmail;
                    document.getElementById('bought-product-password').innerText = stockData.password;
                    document.getElementById('success-modal').classList.remove('hidden');
                });
            });
        }

        function closeSuccessModal() { document.getElementById('success-modal').classList.add('hidden'); }
        function openAuthModal(mode) { isSignUp = (mode === 'signup'); document.getElementById('auth-modal').classList.remove('hidden'); }
        function closeAuthModal() { document.getElementById('auth-modal').classList.add('hidden'); }
        function toggleAuthMode() { openAuthModal(isSignUp ? 'signin' : 'signup'); }

        function handleAuthAction() {
            const email = document.getElementById('auth-email').value.trim();
            const password = document.getElementById('auth-password').value.trim();
            const name = isSignUp ? document.getElementById('auth-name').value.trim() : email.split('@')[0];
            const userId = email.replace(/[^a-zA-Z0-9]/g, '_');
            db.ref('users/' + userId).once('value').then((snap) => {
                if(isSignUp) {
                    currentUser = { id: userId, name: name, email: email, password: password, balance: 0, avatar: 'https://api.dicebear.com/7.x/avataaars/svg?seed=' + name };
                    db.ref('users/' + userId).set(currentUser).then(() => {
                        localStorage.setItem('current_user', JSON.stringify(currentUser));
                        closeAuthModal(); updateUI(); alert('ជោគជ័យ!');
                    });
                } else {
                    if(snap.exists() && snap.val().password === password) {
                        currentUser = snap.val();
                        localStorage.setItem('current_user', JSON.stringify(currentUser));
                        closeAuthModal(); updateUI(); alert('ចូលជោគជ័យ!');
                    } else { alert('ខុសអ៊ីមែល ឬពាក្យសម្ងាត់!'); }
                }
            });
        }

        function logout() { currentUser = null; localStorage.removeItem('current_user'); updateUI(); }
        function updateUI() {
            if(currentUser) {
                document.getElementById('auth-buttons').classList.add('hidden');
                document.getElementById('user-profile-menu').classList.remove('hidden');
                document.getElementById('user-controls').classList.remove('hidden');
                document.getElementById('user-balance').innerText = (currentUser.balance || 0).toFixed(2);
                document.getElementById('logged-user-name').innerText = currentUser.name || '';
                document.getElementById('logged-user-avatar').src = currentUser.avatar || '';
            } else {
                document.getElementById('auth-buttons').classList.remove('hidden');
                document.getElementById('user-profile-menu').classList.add('hidden');
                document.getElementById('user-controls').classList.add('hidden');
            }
        }
        function openTopUpModal() { document.getElementById('topup-modal').classList.remove('hidden'); }
        function closeTopUpModal() { document.getElementById('topup-modal').classList.add('hidden'); }
        function submitTopUpAction() {
            const amount = parseFloat(document.getElementById('topup-amount').value);
            const file = document.getElementById('topup-receipt-file').files[0];
            if(!amount || !file) { alert('សូមបំពេញឱ្យបានគ្រប់!'); return; }
            const reader = new FileReader();
            reader.onload = function(e) {
                const topupId = 'topup_' + Date.now();
                db.ref('topups/' + topupId).set({ id: topupId, userId: currentUser.id, userName: currentUser.name, amount: amount, receiptImage: e.target.result, status: 'pending', date: new Date().toLocaleString() }).then(() => {
                    sendTelegramMessage(`💳 *មានសំណើ Top Up!*\n👤 ${currentUser.name}\n💵 $${amount}`);
                    alert('បានបញ្ជូនសំណើ!'); closeTopUpModal();
                });
            };
            reader.readAsDataURL(file);
        }
    </script>
</body>
</html>
