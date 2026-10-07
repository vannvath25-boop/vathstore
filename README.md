<!DOCTYPE html>
<html lang="km">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vath Store - Admin</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.22.0/firebase-database-compat.js"></script>
</head>
<body class="bg-slate-900 text-slate-100 font-sans p-6">
    <div class="max-w-6xl mx-auto space-y-6">
        <h1 class="text-2xl font-bold text-purple-400">ផ្ទាំងគ្រប់គ្រង Admin</h1>
        
        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <!-- បង្ហោះទំនិញ -->
            <div class="bg-slate-800 p-5 rounded-2xl border border-slate-700">
                <h3 class="font-bold mb-3 text-white">បង្ហោះទំនិញថ្មី</h3>
                <form onsubmit="addNewProduct(event)" class="space-y-3">
                    <input type="text" id="p-name" placeholder="ឈ្មោះទំនិញ" required class="w-full bg-slate-900 border border-slate-700 p-2 rounded-xl text-white">
                    <input type="number" step="0.01" id="p-price" placeholder="តម្លៃ ($)" required class="w-full bg-slate-900 border border-slate-700 p-2 rounded-xl text-white">
                    <input type="text" id="p-img" placeholder="Image URL" required class="w-full bg-slate-900 border border-slate-700 p-2 rounded-xl text-white">
                    <textarea id="p-desc" placeholder="បរិយាយ" required class="w-full bg-slate-900 border border-slate-700 p-2 rounded-xl text-white"></textarea>
                    <button type="submit" class="w-full bg-purple-600 text-white py-2 rounded-xl">បង្ហោះ</button>
                </form>
            </div>

            <!-- បន្ថែមស្តុក -->
            <div class="bg-slate-800 p-5 rounded-2xl border border-slate-700">
                <h3 class="font-bold mb-3 text-white">បន្ថែមស្តុក (Gmail & Pass)</h3>
                <form onsubmit="addStock(event)" class="space-y-3">
                    <select id="stock-prod-select" required class="w-full bg-slate-900 border border-slate-700 p-2 rounded-xl text-white"><option value="">-- ជ្រើសរើសទំនិញ --</option></select>
                    <input type="text" id="stock-gmail" placeholder="Gmail" required class="w-full bg-slate-900 border border-slate-700 p-2 rounded-xl text-white">
                    <input type="text" id="stock-pass" placeholder="Password" required class="w-full bg-slate-900 border border-slate-700 p-2 rounded-xl text-white">
                    <button type="submit" class="w-full bg-emerald-600 text-white py-2 rounded-xl">បន្ថែមស្តុក</button>
                </form>
            </div>
        </div>

        <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
            <!-- បញ្ជីទំនិញ និងបង្ហាញចំនួនស្តុកដែលនៅសល់ -->
            <div class="bg-slate-800 p-5 rounded-2xl border border-slate-700">
                <h3 class="font-bold mb-3 text-white">បញ្ជីរាយនាមទំនិញ និងស្តុកដែលនៅសល់</h3>
                <div id="admin-product-list" class="space-y-2 max-h-80 overflow-y-auto"></div>
            </div>

            <!-- បញ្ជីអ្នកប្រើប្រាស់ និងមុខងារដាក់ប្រាក់ -->
            <div class="bg-slate-800 p-5 rounded-2xl border border-slate-700">
                <h3 class="font-bold mb-3 text-white">បញ្ជីរាយនាមអ្នកប្រើប្រាស់ និងដាក់ប្រាក់</h3>
                <div id="admin-user-list" class="space-y-3 max-h-80 overflow-y-auto"></div>
            </div>
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

        window.onload = function() {
            loadProductsWithStockCount();
            loadAdminUsers();
        };

        function loadProductsWithStockCount() {
            // ទាញយកស្តុកទាំងអស់មកឆែកមើលចំនួនសរុប និងចំនួនដែលនៅទំនេរ
            db.ref('stocks').on('value', (stockSnap) => {
                const stockCounts = {};
                if(stockSnap.exists()) {
                    stockSnap.forEach(c => {
                        const st = c.val();
                        if(st.status === 'available') {
                            stockCounts[st.productId] = (stockCounts[st.productId] || 0) + 1;
                        }
                    });
                }

                // ទាញយកទំនិញមកបង្ហាញ
                db.ref('products').on('value', (prodSnap) => {
                    const sel = document.getElementById('stock-prod-select');
                    const prodList = document.getElementById('admin-product-list');
                    
                    sel.innerHTML = '<option value="">-- ជ្រើសរើសទំនិញ --</option>';
                    prodList.innerHTML = '';

                    if(prodSnap.exists()) {
                        prodSnap.forEach(c => {
                            const p = c.val();
                            const availableCount = stockCounts[p.id] || 0;

                            sel.innerHTML += `<option value="${p.id}">${p.name} ($${p.price})</option>`;
                            prodList.innerHTML += `
                                <div class="bg-slate-900 p-3 rounded-xl border border-slate-700 flex items-center justify-between">
                                    <div class="flex items-center space-x-3">
                                        <img src="${p.image}" class="w-10 h-10 rounded-lg object-cover">
                                        <div>
                                            <h5 class="font-bold text-sm text-white">${p.name}</h5>
                                            <p class="text-xs text-indigo-400">$${p.price} | <span class="text-emerald-400 font-semibold">ស្តុកនៅសល់: ${availableCount}</span></p>
                                        </div>
                                    </div>
                                    <button onclick="deleteProduct('${p.id}')" class="bg-rose-500/10 text-rose-400 p-2 rounded-lg text-xs" title="លុបទំនិញ"><i class="fa-solid fa-trash"></i></button>
                                </div>
                            `;
                        });
                    } else {
                        prodList.innerHTML = `<p class="text-slate-400 text-sm text-center">មិនទាន់មានទំនិញ</p>`;
                    }
                });
            });
        }

        function loadAdminUsers() {
            db.ref('users').on('value', (snap) => {
                const userList = document.getElementById('admin-user-list');
                userList.innerHTML = '';
                if(snap.exists()) {
                    snap.forEach(c => {
                        const u = c.val();
                        userList.innerHTML += `
                            <div class="bg-slate-900 p-3.5 rounded-xl border border-slate-700 space-y-2">
                                <div class="flex items-center justify-between">
                                    <div class="flex items-center space-x-3">
                                        <img src="${u.avatar || ''}" class="w-9 h-9 rounded-full object-cover">
                                        <div>
                                            <h5 class="font-bold text-sm text-white">${u.name}</h5>
                                            <p class="text-xs text-slate-400">${u.email} | <span class="text-emerald-400 font-bold">កាបូប: $${(u.balance || 0).toFixed(2)}</span></p>
                                        </div>
                                    </div>
                                    <button onclick="deleteUser('${u.id}')" class="bg-rose-500/10 text-rose-400 p-2 rounded-lg text-xs" title="លុបអ្នកប្រើ"><i class="fa-solid fa-trash"></i></button>
                                </div>
                                <div class="flex space-x-2 pt-1 border-t border-slate-800">
                                    <input type="number" step="0.01" id="add-bal-${u.id}" placeholder="ចំនួនទឹកប្រាក់ ($)" class="w-full bg-slate-800 border border-slate-700 px-3 py-1.5 rounded-lg text-xs text-white">
                                    <button onclick="addUserBalance('${u.id}')" class="bg-emerald-600 hover:bg-emerald-500 text-white px-3 py-1.5 rounded-lg text-xs font-semibold whitespace-nowrap transition">ដាក់ប្រាក់</button>
                                </div>
                            </div>
                        `;
                    });
                } else {
                    userList.innerHTML = `<p class="text-slate-400 text-sm text-center">មិនទាន់មានអ្នកប្រើប្រាស់</p>`;
                }
            });
        }

        function addUserBalance(userId) {
            const amountInput = document.getElementById(`add-bal-${userId}`);
            const amount = parseFloat(amountInput.value);

            if(!amount || amount <= 0) {
                alert('សូមបញ្ចូលចំនួនទឹកប្រាក់ឱ្យបានត្រឹមត្រូវ!');
                return;
            }

            db.ref('users/' + userId + '/balance').once('value').then((snap) => {
                let currentBal = snap.val() || 0;
                let newBal = currentBal + amount;

                db.ref('users/' + userId + '/balance').set(newBal).then(() => {
                    alert(`បានបន្ថែមទឹកប្រាក់ចំនួន $${amount.toFixed(2)} ជូនអតិថិជនជោគជ័យ!`);
                    amountInput.value = '';
                });
            }).catch(err => alert('មានបញ្ហា: ' + err.message));
        }

        function addNewProduct(e) {
            e.preventDefault();
            const id = 'product_' + Date.now();
            db.ref('products/' + id).set({
                id: id,
                name: document.getElementById('p-name').value,
                price: parseFloat(document.getElementById('p-price').value),
                image: document.getElementById('p-img').value,
                description: document.getElementById('p-desc').value
            }).then(() => { alert('បង្ហោះទំនិញជោគជ័យ!'); location.reload(); });
        }

        function deleteProduct(id) {
            if(confirm('តើអ្នកពិតជាចង់លុបទំនិញនេះមែនទេ?')) {
                db.ref('products/' + id).remove().then(() => alert('លុបជោគជ័យ!'));
            }
        }

        function deleteUser(id) {
            if(confirm('តើអ្នកពិតជាចង់លុបអ្នកប្រើប្រាស់នេះមែនទេ?')) {
                db.ref('users/' + id).remove().then(() => alert('លុបអ្នកប្រើប្រាស់ជោគជ័យ!'));
            }
        }

        function addStock(e) {
            e.preventDefault();
            const id = 'stock_' + Date.now();
            db.ref('stocks/' + id).set({
                id: id,
                productId: document.getElementById('stock-prod-select').value,
                gmail: document.getElementById('stock-gmail').value,
                password: document.getElementById('stock-pass').value,
                status: 'available'
            }).then(() => { alert('បន្ថែមស្តុករួចរាល់!'); location.reload(); });
        }
    </script>
</body>
</html>
