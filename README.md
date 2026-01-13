<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>تطبيق جمعية الحلاقة - المطور</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    
    <!-- Firebase SDK (Compat Version 8.10.1 - الأكثر استقراراً للملفات المدمجة) -->
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-firestore.js"></script>

    <style>
        :root {
            --primary-color: #2c3e50;
            --accent-color: #e67e22;
            --success-color: #27ae60;
            --danger-color: #c0392b;
            --warning-color: #f39c12;
            --partial-color: #d35400;
            --bg-color: #f4f6f7;
            --card-bg: #ffffff;
            --text-color: #333;
            --nav-height: 60px;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 0;
            color: var(--text-color);
            padding-bottom: calc(var(--nav-height) + 20px);
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }

        /* --- UI Components --- */
        .mobile-header {
            background-color: var(--primary-color); color: white; padding: 15px;
            text-align: center; box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            position: sticky; top: 0; z-index: 100;
        }
        
        .container { padding: 15px; }
        .section { display: none; animation: fadeIn 0.3s; }
        .section.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(5px); } to { opacity: 1; transform: translateY(0); } }

        .card {
            background: var(--card-bg); border-radius: 12px; padding: 15px;
            margin-bottom: 15px; box-shadow: 0 2px 8px rgba(0,0,0,0.03);
        }

        /* --- Forms --- */
        .form-group { margin-bottom: 12px; }
        .form-group label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 0.9rem; }
        .form-group input, .form-group select {
            width: 100%; padding: 12px; border: 1px solid #ddd;
            border-radius: 8px; box-sizing: border-box; background: #fafafa;
        }
        .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }

        button.btn-primary {
            background-color: var(--accent-color); color: white; border: none; padding: 12px;
            border-radius: 8px; width: 100%; font-size: 1rem; font-weight: bold; cursor: pointer;
        }

        /* --- Tables --- */
        .table-responsive { overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; min-width: 100%; }
        th, td { text-align: right; padding: 10px; border-bottom: 1px solid #eee; font-size: 0.9rem; }
        th { color: #777; font-weight: 600; white-space: nowrap; }
        .clickable-name { font-weight: bold; color: var(--primary-color); border-bottom: 1px dashed #ccc; cursor: pointer; }

        /* --- Badges & Lists --- */
        .badge { padding: 4px 8px; border-radius: 12px; font-size: 0.7rem; font-weight: bold; display: inline-block; min-width: 50px; text-align: center; }
        .badge-paid { background: #d4edda; color: #155724; }
        .badge-partial { background: #fff3cd; color: #856404; }
        .badge-due { background: #f8d7da; color: #721c24; }

        .supplies-list { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 5px; }
        .supply-item { background: #f0f2f5; padding: 8px 12px; border-radius: 20px; border: 1px solid #ddd; display: flex; align-items: center; font-size: 0.85rem; }
        .supply-item input { margin-left: 8px; width: auto; }

        /* --- Navigation --- */
        .bottom-nav {
            position: fixed; bottom: 0; left: 0; width: 100%; height: var(--nav-height);
            background-color: #fff; display: flex; justify-content: space-around;
            align-items: center; box-shadow: 0 -2px 10px rgba(0,0,0,0.05); z-index: 1000;
        }
        .nav-item { border: none; background: none; color: #95a5a6; font-size: 0.75rem; display: flex; flex-direction: column; align-items: center; }
        .nav-item.active { color: var(--accent-color); font-weight: bold; }
        .nav-item i { font-size: 1.2rem; margin-bottom: 4px; }

        /* --- Stats --- */
        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px; }
        .stat-box { background: white; padding: 15px; border-radius: 10px; text-align: center; border: 1px solid #eee; }
        .stat-box p { margin: 5px 0 0; font-size: 1.2rem; font-weight: bold; color: var(--primary-color); }

        /* --- Modals --- */
        .modal { display: none; position: fixed; z-index: 2000; left: 0; top: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); align-items: flex-end; }
        .modal-content { background: white; width: 100%; height: 90vh; border-radius: 20px 20px 0 0; padding: 20px; box-sizing: border-box; overflow-y: auto; }

        /* --- Loading --- */
        #loading-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #fff; display: flex; justify-content: center; align-items: center; z-index: 9999; flex-direction: column; }
        .spinner { border: 4px solid #f3f3f3; border-top: 4px solid var(--accent-color); border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

    </style>
</head>
<body>

    <div id="loading-overlay">
        <div class="spinner"></div>
        <p style="margin-top:15px">جاري الاتصال بقاعدة البيانات...</p>
    </div>

    <div class="mobile-header">
        <h2><i class="fas fa-cut"></i> إدارة الجمعية</h2>
    </div>

    <div class="container">
        
        <!-- صفحة الرئيسية -->
        <div id="home" class="section active">
            <div class="card" style="background: linear-gradient(135deg, #2c3e50, #34495e); color: white; text-align: center;">
                <p style="margin:0; opacity:0.8">الرصيد الصافي</p>
                <h1 id="netBalance" style="margin:5px 0">0</h1>
                <p style="margin:0">درهم</p>
            </div>

            <div class="stats-grid">
                <div class="stat-box" onclick="navTo('students', document.querySelectorAll('.nav-item')[1])">
                    <h4>المتدربين</h4><p id="totalStudents">0</p>
                </div>
                <div class="stat-box">
                    <h4>المداخيل</h4><p id="totalIncome" style="color:green">0</p>
                </div>
            </div>

            <div class="card" style="border-right: 4px solid var(--danger-color);">
                <h3><i class="fas fa-file-excel"></i> نواقص لوازم التسجيل</h3>
                <div class="table-responsive"><table id="missingTable"><tbody></tbody></table></div>
            </div>

            <div class="card" style="border-right: 4px solid var(--warning-color);">
                <h3><i class="fas fa-clock"></i> استحقاقات قريبة/متأخرة</h3>
                <div class="table-responsive"><table id="overdueTable"><tbody></tbody></table></div>
            </div>
        </div>

        <!-- صفحة المتدربين -->
        <div id="students" class="section">
            <div class="card">
                <button class="btn-primary" onclick="toggleForm('studentForm')"><i class="fas fa-plus"></i> إضافة متدرب جديد</button>
                
                <form id="studentForm" style="display:none; margin-top:20px;" onsubmit="addStudent(event)">
                    
                    <h4 style="color:var(--primary-color); border-bottom:1px solid #eee; padding-bottom:5px;">1. المعلومات الشخصية</h4>
                    <div class="form-group">
                        <label>الاسم الكامل</label>
                        <input type="text" id="sName" required>
                    </div>
                    <div class="form-row">
                        <div class="form-group">
                            <label>تاريخ الازدياد <span id="ageDisplay" style="color:var(--accent-color); font-size:0.8em"></span></label>
                            <input type="date" id="sDob" required onchange="calcAge(this.value)">
                        </div>
                        <div class="form-group">
                            <label>رقم الهاتف</label>
                            <input type="tel" id="sPhone" required>
                        </div>
                    </div>
                    <div class="form-group">
                        <label>رقم البطاقة الوطنية (CINE)</label>
                        <input type="text" id="sCNIE">
                    </div>

                    <h4 style="color:var(--primary-color); border-bottom:1px solid #eee; padding-bottom:5px; margin-top:15px;">2. بيانات التسجيل والتدريب</h4>
                    <div class="form-row">
                        <div class="form-group">
                            <label>تاريخ التسجيل</label>
                            <input type="date" id="sRegDate" required>
                        </div>
                        <div class="form-group">
                            <label>مدة التدريب (أشهر)</label>
                            <input type="number" id="sDuration" value="9" required>
                        </div>
                    </div>

                    <h4 style="color:var(--primary-color); border-bottom:1px solid #eee; padding-bottom:5px; margin-top:15px;">3. الرسوم والأقساط</h4>
                    <div class="form-row">
                        <div class="form-group">
                            <label>رسوم التسجيل</label>
                            <input type="number" id="sRegFee" placeholder="أدخل المبلغ">
                        </div>
                        <div class="form-group">
                            <label>قسط التدريب الشهري</label>
                            <input type="number" id="sTrainFee" placeholder="أدخل المبلغ">
                        </div>
                    </div>
                    <div class="form-group">
                        <label>مبلغ مواد الحلاقة</label>
                        <input type="number" id="sMatFee" placeholder="0 إذا لا يوجد">
                    </div>
                    
                    <div style="background:#f9f9f9; padding:10px; border-radius:8px; border:1px solid #eee;">
                        <div class="form-row">
                            <div class="form-group">
                                <label>مبلغ معدات الحلاقة</label>
                                <input type="number" id="sEquipFee" placeholder="0 إذا لا يوجد">
                            </div>
                            <div class="form-group">
                                <label>مدة تقسيط المعدات (أشهر)</label>
                                <input type="number" id="sEquipDuration" value="1" min="1">
                            </div>
                        </div>
                    </div>

                    <h4 style="color:var(--primary-color); border-bottom:1px solid #eee; padding-bottom:5px; margin-top:15px;">4. اللوازم والدفع</h4>
                    <div class="form-group">
                        <label>لوازم التسجيل المقدمة:</label>
                        <div id="suppliesChecklist" class="supplies-list"></div>
                    </div>

                    <div class="form-group" style="background:#e8f6f3; padding:10px; border-radius:8px; border:1px dashed var(--success-color);">
                        <label style="color:var(--success-color)">المبلغ المدفوع الآن (تسبيق)</label>
                        <input type="number" id="sInitialPay" placeholder="0">
                    </div>

                    <button type="submit" class="btn-primary" style="margin-top:10px">حفظ البيانات</button>
                </form>
            </div>

            <div style="margin-top:15px">
                <input type="text" id="searchBox" placeholder="بحث بالاسم أو الهاتف..." onkeyup="renderStudents()" style="width:100%; padding:10px; border-radius:20px; border:1px solid #ccc;">
            </div>

            <div class="card table-responsive" style="margin-top:10px">
                <table id="studentsTable"><thead><tr><th>الاسم</th><th>التقدم</th><th></th></tr></thead><tbody></tbody></table>
            </div>
        </div>

        <!-- المصاريف -->
        <div id="expenses" class="section">
            <div class="card">
                <h3>إضافة مصروف</h3>
                <form onsubmit="addExpense(event)">
                    <div class="form-row">
                        <div class="form-group"><input type="text" id="exDesc" placeholder="البيان" required></div>
                        <div class="form-group"><input type="number" id="exAmount" placeholder="المبلغ" required></div>
                    </div>
                    <div class="form-group"><input type="date" id="exDate" required></div>
                    <button type="submit" class="btn-primary">تسجيل</button>
                </form>
            </div>
            <div class="card table-responsive">
                <table id="expensesTable"><tbody></tbody></table>
            </div>
        </div>

        <!-- الإعدادات -->
        <div id="settings" class="section">
            <div class="card">
                <h3>الإعدادات</h3>
                <div class="form-group">
                    <label>قسط التدريب الشهري الافتراضي</label>
                    <input type="number" id="defTrainFee" onchange="saveSettings()">
                </div>
                
                <hr>
                <h4>إدارة لوازم التسجيل</h4>
                <div style="display:flex; gap:10px; margin-bottom:10px;">
                    <input type="text" id="newSupply" placeholder="اسم اللازمة (مثلا: صورتان)">
                    <button class="btn-primary" style="width:auto" onclick="addSupplyItem()">+</button>
                </div>
                <ul id="settingsSupplies" style="list-style:none; padding:0"></ul>
                
                <hr>
                <button class="btn-primary" style="background:var(--danger-color)" onclick="clearAllData()">تهيئة المصنع (حذف الكل)</button>
            </div>
        </div>

    </div>

    <!-- القائمة السفلية -->
    <div class="bottom-nav">
        <button class="nav-item active" onclick="navTo('home', this)"><i class="fas fa-home"></i> الرئيسية</button>
        <button class="nav-item" onclick="navTo('students', this)"><i class="fas fa-user-graduate"></i> المتدربون</button>
        <button class="nav-item" onclick="navTo('expenses', this)"><i class="fas fa-wallet"></i> المصاريف</button>
        <button class="nav-item" onclick="navTo('settings', this)"><i class="fas fa-cog"></i> الإعدادات</button>
    </div>

    <!-- النوافذ المنبثقة -->
    <div id="profileModal" class="modal">
        <div class="modal-content">
            <span onclick="document.getElementById('profileModal').style.display='none'" style="float:left; font-size:24px; cursor:pointer">&times;</span>
            <h3 style="margin-top:0">ملف المتدرب</h3>
            <div id="profileContent"></div>
            <button class="btn-primary" style="margin-top:15px" onclick="goToPayFromProfile()">سجل الأداءات</button>
        </div>
    </div>

    <div id="paymentModal" class="modal">
        <div class="modal-content">
            <span onclick="document.getElementById('paymentModal').style.display='none'" style="float:left; font-size:24px; cursor:pointer">&times;</span>
            <h3 id="payModalName" style="margin-top:0"></h3>
            <div class="table-responsive">
                <table><thead><tr><th>البيان</th><th>تاريخ</th><th>الحالة</th><th></th></tr></thead><tbody id="payTableBody"></tbody></table>
            </div>
        </div>
    </div>

    <script>
        // --- تهيئة Firebase بإعداداتك ---
        const firebaseConfig = {
            apiKey: "AIzaSyB5a4zFSlo_Td4wto1_HCNeb7CpI4AzJkM",
            authDomain: "hila9a-64416.firebaseapp.com",
            projectId: "hila9a-64416",
            storageBucket: "hila9a-64416.firebasestorage.app",
            messagingSenderId: "869332200810",
            appId: "1:869332200810:web:46b8f9f634d0cbba41c913",
            measurementId: "G-HNVEL5XJJP"
        };

        // Initialize Firebase
        if (!firebase.apps.length) {
            firebase.initializeApp(firebaseConfig);
        }
        const db = firebase.firestore();
        db.enablePersistence().catch(err => console.log("Persistence error:", err));

        // --- المتغيرات العامة ---
        let allStudents = [];
        let allExpenses = [];
        let appSettings = { trainFee: 300, supplies: ['صورة شمسية', 'نسخة بطاقة التعريف'] };
        let currentStudentId = null;

        const arabicMonths = ["يناير", "فبراير", "مارس", "أبريل", "مايو", "يونيو", "يوليو", "غشت", "شتنبر", "أكتوبر", "نونبر", "دجنبر"];

        // --- تحميل البيانات ---
        window.onload = function() {
            const today = new Date().toISOString().split('T')[0];
            if(document.getElementById('sRegDate')) document.getElementById('sRegDate').value = today;
            if(document.getElementById('exDate')) document.getElementById('exDate').value = today;

            // Settings Listener
            db.collection("settings").doc("config").onSnapshot(doc => {
                if(doc.exists) {
                    appSettings = doc.data();
                    window.updateSettingsUI();
                } else {
                    db.collection("settings").doc("config").set(appSettings);
                }
            });

            // Students Listener
            db.collection("students").onSnapshot(snap => {
                allStudents = snap.docs.map(d => ({id: d.id, ...d.data()}));
                document.getElementById('loading-overlay').style.display = 'none';
                window.renderStudents();
                window.updateDashboard();
                if(currentStudentId && document.getElementById('paymentModal').style.display === 'flex') {
                    // إعادة تحميل النافذة لتحديث الألوان والأرقام
                    window.openPayModal(currentStudentId);
                }
            });

            // Expenses Listener
            db.collection("expenses").onSnapshot(snap => {
                allExpenses = snap.docs.map(d => ({id: d.id, ...d.data()}));
                window.renderExpenses();
                window.updateDashboard();
            });
        };

        // --- الدوال العامة ---
        window.navTo = function(id, btn) {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.querySelectorAll('.nav-item').forEach(b => b.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            if(btn) btn.classList.add('active');
        }

        window.toggleForm = function(id) {
            const el = document.getElementById(id);
            if(el.style.display === 'none') {
                el.style.display = 'block';
                const container = document.getElementById('suppliesChecklist');
                container.innerHTML = '';
                (appSettings.supplies || []).forEach(s => {
                    container.innerHTML += `<label class="supply-item"><input type="checkbox" value="${s}"> ${s}</label>`;
                });
            } else {
                el.style.display = 'none';
            }
        }

        window.calcAge = function(val) {
            if(!val) return;
            const age = Math.abs(new Date(Date.now() - new Date(val).getTime()).getUTCFullYear() - 1970);
            document.getElementById('ageDisplay').innerText = `(${age} سنة)`;
        }

        // --- إضافة متدرب ---
        window.addStudent = function(e) {
            e.preventDefault();
            const name = document.getElementById('sName').value;
            const dob = document.getElementById('sDob').value;
            const phone = document.getElementById('sPhone').value;
            const cnie = document.getElementById('sCNIE').value;
            const regDate = document.getElementById('sRegDate').value;
            const duration = parseInt(document.getElementById('sDuration').value) || 9;
            
            const regFee = parseFloat(document.getElementById('sRegFee').value) || 0;
            const trainFee = parseFloat(document.getElementById('sTrainFee').value) || 0;
            const matFee = parseFloat(document.getElementById('sMatFee').value) || 0;
            const equipFee = parseFloat(document.getElementById('sEquipFee').value) || 0;
            const equipDuration = parseInt(document.getElementById('sEquipDuration').value) || 1;
            let initPay = parseFloat(document.getElementById('sInitialPay').value) || 0;

            const supplies = [];
            document.querySelectorAll('#suppliesChecklist input').forEach(cb => {
                supplies.push({name: cb.value, checked: cb.checked});
            });

            const payments = [];
            if(regFee > 0) payments.push({ type: 'reg', label: 'رسوم التسجيل', amount: regFee, paid: 0, due: regDate });
            if(matFee > 0) payments.push({ type: 'mat', label: 'مواد الحلاقة', amount: matFee, paid: 0, due: regDate });

            const d = new Date(regDate);
            for(let i=0; i<duration; i++) {
                const dueDate = new Date(d);
                dueDate.setMonth(d.getMonth() + i);
                payments.push({ type: 'train', label: `تدريب شهر ${i+1}`, amount: trainFee, paid: 0, due: dueDate.toISOString() });
            }

            if(equipFee > 0) {
                const monthlyEquip = equipFee / equipDuration;
                for(let i=0; i<equipDuration; i++) {
                    const dueDate = new Date(d);
                    dueDate.setMonth(d.getMonth() + i);
                    payments.push({ type: 'equip', label: `معدات قسط ${i+1}/${equipDuration}`, amount: monthlyEquip, paid: 0, due: dueDate.toISOString() });
                }
            }

            if(initPay > 0) {
                for(let p of payments) {
                    if(initPay <= 0.1) break;
                    let remaining = p.amount - p.paid;
                    let toPay = Math.min(remaining, initPay);
                    p.paid += toPay;
                    initPay -= toPay;
                }
            }

            db.collection("students").add({
                name, dob, phone, cnie, regDate,
                trainFee, duration, equipFee, equipDuration,
                supplies, payments
            }).then(() => {
                e.target.reset();
                window.toggleForm('studentForm');
                alert("تمت إضافة المتدرب");
            }).catch(err => alert("خطأ: " + err.message));
        }

        window.renderStudents = function() {
            const tbody = document.querySelector('#studentsTable tbody');
            tbody.innerHTML = '';
            const q = (document.getElementById('searchBox').value || '').toLowerCase();

            allStudents.forEach(s => {
                if(s.name.toLowerCase().includes(q) || s.phone.includes(q)) {
                    const trainParams = s.payments.filter(p => p.type === 'train');
                    const paidCount = trainParams.filter(p => p.paid >= p.amount - 1).length;
                    const percent = trainParams.length ? Math.round((paidCount / trainParams.length)*100) : 0;

                    tbody.innerHTML += `
                        <tr>
                            <td><span class="clickable-name" onclick="window.openProfile('${s.id}')">${s.name}</span><br><small>${s.phone}</small></td>
                            <td><div style="font-size:0.7em">${paidCount}/${trainParams.length} أشهر</div>
                            <div style="height:5px; background:#eee; width:100%"><div style="height:100%; background:green; width:${percent}%"></div></div></td>
                            <td>
                                <button class="btn-primary" style="padding:5px 10px; font-size:0.8rem; width:auto" onclick="window.openPayModal('${s.id}')">أداء</button>
                                <button class="btn-primary" style="padding:5px 10px; font-size:0.8rem; width:auto; background:red; margin-right:5px" onclick="window.deleteStudent('${s.id}')">حذف</button>
                            </td>
                        </tr>
                    `;
                }
            });
        }

        window.deleteStudent = function(id) {
            if(confirm("حذف نهائي؟")) db.collection("students").doc(id).delete();
        }

        window.openProfile = function(id) {
            currentStudentId = id;
            const s = allStudents.find(x => x.id === id);
            if(!s) return;

            const age = Math.abs(new Date(Date.now() - new Date(s.dob).getTime()).getUTCFullYear() - 1970);
            let totalReq = 0, totalPaid = 0;
            s.payments.forEach(p => { totalReq += p.amount; totalPaid += p.paid; });

            let suppliesHtml = '<div><strong>اللوازم:</strong><br>';
            if(s.supplies) {
                s.supplies.forEach(sup => {
                    suppliesHtml += sup.checked 
                        ? `<span class="badge" style="background:#d4edda; color:#155724; margin:2px">✔ ${sup.name}</span>` 
                        : `<span class="badge" style="background:#f8d7da; color:#721c24; margin:2px">✖ ${sup.name}</span>`;
                });
            }
            suppliesHtml += '</div>';

            document.getElementById('profileContent').innerHTML = `
                <p><strong>الاسم:</strong> ${s.name}</p>
                <p><strong>الهاتف:</strong> ${s.phone}</p>
                <p><strong>السن:</strong> ${age} سنة</p>
                <p><strong>البطاقة:</strong> ${s.cnie || '-'}</p>
                <div style="background:#f9f9f9; padding:10px; border-radius:5px; margin:10px 0;">
                    <p><strong>المدفوع:</strong> <span style="color:green">${Math.round(totalPaid)}</span></p>
                    <p><strong>المتبقي:</strong> <span style="color:red">${Math.round(totalReq - totalPaid)}</span></p>
                </div>
                ${suppliesHtml}
            `;
            document.getElementById('profileModal').style.display = 'flex';
        }

        window.goToPayFromProfile = function() {
            document.getElementById('profileModal').style.display = 'none';
            window.openPayModal(currentStudentId);
        }

        // --- نظام الأداء (الاستخلاص) ---
        window.openPayModal = function(id) {
            currentStudentId = id;
            const s = allStudents.find(x => x.id === id);
            if(!s) return;

            document.getElementById('payModalName').innerText = s.name;
            const tbody = document.getElementById('payTableBody');
            tbody.innerHTML = '';
            
            const today = new Date(); today.setHours(0,0,0,0);

            s.payments.forEach((p, idx) => {
                const due = new Date(p.due); due.setHours(0,0,0,0);
                const diff = Math.ceil((due - today) / (1000*60*60*24));
                const remaining = p.amount - p.paid;

                let statusBadge = '';
                if(remaining <= 0.5) statusBadge = '<span class="badge badge-paid">مدفوع</span>';
                else if(p.paid > 0) statusBadge = `<span class="badge badge-partial">باقي ${Math.round(remaining)}</span>`;
                else if(diff <= 0) statusBadge = '<span class="badge badge-due">واجب</span>';
                else if(diff <= 3) statusBadge = '<span class="badge" style="background:#fff3cd">قريب</span>';
                else statusBadge = '<span class="badge" style="background:#eee">قادم</span>';

                let actionBtn = '';
                if(remaining <= 0.5) {
                    actionBtn = `<button class="btn-primary" style="background:none; color:red; padding:2px; font-size:0.7em; width:auto" onclick="window.cancelPayment('${s.id}', ${idx})">إلغاء</button>`;
                } else {
                    // الزر الآن type="button" لمنع إعادة تحميل الصفحة
                    const btnText = p.paid > 0 ? 'إتمام' : 'استخلاص';
                    const btnColor = p.paid > 0 ? 'var(--partial-color)' : 'var(--accent-color)';
                    actionBtn = `<button type="button" class="btn-primary" style="background:${btnColor}; padding:5px 10px; width:auto; font-size:0.8rem" onclick="window.processPayment('${s.id}', ${idx})">${btnText}</button>`;
                }

                tbody.innerHTML += `
                    <tr>
                        <td><b>${p.label}</b><br><small style="color:#777">${Math.round(p.paid)} / ${Math.round(p.amount)}</small></td>
                        <td style="font-size:0.8rem">${new Date(p.due).toLocaleDateString('ar-MA')}</td>
                        <td>${statusBadge}</td>
                        <td>${actionBtn}</td>
                    </tr>
                `;
            });
            document.getElementById('paymentModal').style.display = 'flex';
        }

        window.processPayment = async function(studentId, idx) {
            console.log("Start payment for:", studentId, "index:", idx); // Debug log
            
            const s = allStudents.find(x => x.id === studentId);
            if(!s) {
                alert("لم يتم العثور على المتدرب!");
                return;
            }

            const p = s.payments[idx];
            const remaining = p.amount - p.paid;

            let input = prompt(`المبلغ المتبقي: ${Math.round(remaining)} درهم.\nأدخل المبلغ المدفوع الآن:`, Math.round(remaining));
            
            if(input !== null) {
                let amount = parseFloat(input);
                if(isNaN(amount) || amount < 0) {
                    alert("مبلغ غير صحيح");
                    return;
                }
                
                if(amount > remaining) amount = remaining;

                // تحديث محلي
                p.paid += amount;
                
                try {
                    // حفظ في Firebase
                    await db.collection("students").doc(studentId).update({ payments: s.payments });
                    console.log("Payment saved successfully");
                    // التحديث سيتم تلقائياً عبر المستمع، ولكن للاحتياط:
                    window.openPayModal(studentId);
                } catch (error) {
                    console.error("Error updating payment:", error);
                    alert("حدث خطأ أثناء حفظ الدفع: " + error.message);
                }
            }
        }

        window.cancelPayment = async function(studentId, idx) {
            if(confirm("هل أنت متأكد من إلغاء الدفع؟")) {
                const s = allStudents.find(x => x.id === studentId);
                s.payments[idx].paid = 0;
                try {
                    await db.collection("students").doc(studentId).update({ payments: s.payments });
                } catch (error) {
                    console.error(error);
                    alert("خطأ في الإلغاء");
                }
            }
        }

        // --- المصاريف ---
        window.addExpense = function(e) {
            e.preventDefault();
            const ex = {
                desc: document.getElementById('exDesc').value,
                amount: parseFloat(document.getElementById('exAmount').value),
                date: document.getElementById('exDate').value
            };
            db.collection("expenses").add(ex).then(() => {
                e.target.reset(); 
                document.getElementById('exDate').value = new Date().toISOString().split('T')[0];
            });
        }

        window.renderExpenses = function() {
            const tbody = document.querySelector('#expensesTable tbody');
            tbody.innerHTML = '';
            allExpenses.sort((a,b) => new Date(b.date) - new Date(a.date)).forEach(ex => {
                tbody.innerHTML += `<tr><td>${ex.date}</td><td>${ex.desc}</td><td style="color:red">-${ex.amount}</td><td><button onclick="db.collection('expenses').doc('${ex.id}').delete()" style="background:none; border:none; color:red; cursor:pointer">x</button></td></tr>`;
            });
        }

        // --- الإعدادات ---
        window.updateSettingsUI = function() {
            document.getElementById('defTrainFee').value = appSettings.trainFee;
            document.getElementById('sTrainFee').value = appSettings.trainFee;
            
            const ul = document.getElementById('settingsSupplies');
            ul.innerHTML = '';
            (appSettings.supplies || []).forEach((s, i) => {
                ul.innerHTML += `<li><span>${s}</span> <button class="btn-primary" style="background:red; width:auto; padding:2px 8px; margin-right:auto" onclick="window.remSupply(${i})">x</button></li>`;
            });
        }

        window.saveSettings = function() {
            appSettings.trainFee = document.getElementById('defTrainFee').value;
            db.collection("settings").doc("config").update(appSettings);
        }

        window.addSupplyItem = function() {
            const val = document.getElementById('newSupply').value;
            if(val) {
                if(!appSettings.supplies) appSettings.supplies = [];
                appSettings.supplies.push(val);
                db.collection("settings").doc("config").update({supplies: appSettings.supplies});
                document.getElementById('newSupply').value = '';
            }
        }

        window.remSupply = function(i) {
            appSettings.supplies.splice(i, 1);
            db.collection("settings").doc("config").update({supplies: appSettings.supplies});
        }

        window.clearAllData = function() {
            if(confirm("تحذير: سيتم مسح كل شيء!")) {
                allStudents.forEach(s => db.collection("students").doc(s.id).delete());
                allExpenses.forEach(e => db.collection("expenses").doc(e.id).delete());
                alert("تم المسح");
            }
        }

        // --- لوحة التحكم ---
        window.updateDashboard = function() {
            document.getElementById('totalStudents').innerText = allStudents.length;
            
            let inc = 0;
            allStudents.forEach(s => s.payments.forEach(p => inc += p.paid));
            let exp = 0;
            allExpenses.forEach(e => exp += e.amount);
            
            document.getElementById('totalIncome').innerText = Math.round(inc);
            document.getElementById('netBalance').innerText = Math.round(inc - exp);

            const missingTbody = document.querySelector('#missingTable tbody');
            const overdueTbody = document.querySelector('#overdueTable tbody');
            missingTbody.innerHTML = ''; overdueTbody.innerHTML = '';

            const today = new Date(); today.setHours(0,0,0,0);

            allStudents.forEach(s => {
                if(s.supplies && s.supplies.some(x => !x.checked)) {
                    const m = s.supplies.filter(x => !x.checked).map(x => x.name).join(', ');
                    missingTbody.innerHTML += `<tr><td><span class="clickable-name" onclick="window.openProfile('${s.id}')">${s.name}</span></td><td style="color:red; font-size:0.8em">${m}</td></tr>`;
                }

                s.payments.forEach(p => {
                    if(p.amount - p.paid > 0.5) {
                        const due = new Date(p.due); due.setHours(0,0,0,0);
                        const diff = Math.ceil((due - today)/(1000*60*60*24));
                        
                        if(diff <= 3) {
                            const color = diff <= 0 ? 'red' : 'orange';
                            const rem = Math.round(p.amount - p.paid);
                            overdueTbody.innerHTML += `<tr><td>${s.name}</td><td>${p.label}</td><td style="color:${color}">${rem}</td></tr>`;
                        }
                    }
                });
            });
            
            if(!missingTbody.innerHTML) missingTbody.innerHTML = '<tr><td colspan="2" style="text-align:center">لا توجد نواقص</td></tr>';
            if(!overdueTbody.innerHTML) overdueTbody.innerHTML = '<tr><td colspan="3" style="text-align:center">لا توجد استحقاقات عاجلة</td></tr>';
        }

    </script>
</body>
</html>
