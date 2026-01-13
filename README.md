<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>تطبيق جمعية الحلاقة</title>
    <!-- أيقونات -->
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    
    <!-- Firebase SDK -->
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-firestore.js"></script>

    <style>
        :root {
            /* ألوان عصرية */
            --primary: #2563eb;       /* أزرق ملكي */
            --secondary: #1e293b;     /* كحلي غامق */
            --accent: #f59e0b;        /* برتقالي */
            --success: #10b981;       /* أخضر */
            --danger: #ef4444;        /* أحمر */
            --bg-body: #f1f5f9;       /* رمادي فاتح للخلفية */
            --bg-card: #ffffff;
            --text-main: #1e293b;
            --text-sub: #64748b;
            --radius: 16px;
            --nav-height: 70px;
            --shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
        }

        * { box-sizing: border-box; -webkit-tap-highlight-color: transparent; outline: none; }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-body);
            margin: 0;
            padding: 0;
            color: var(--text-main);
            padding-bottom: calc(var(--nav-height) + 20px); /* مساحة للقائمة السفلية */
            font-size: 15px;
        }

        /* --- Header --- */
        .app-header {
            background-color: var(--primary);
            color: white;
            padding: 15px 20px;
            position: sticky;
            top: 0;
            z-index: 100;
            box-shadow: 0 4px 10px rgba(37, 99, 235, 0.2);
            border-bottom-left-radius: 20px;
            border-bottom-right-radius: 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .app-header h1 { margin: 0; font-size: 1.2rem; font-weight: 700; }
        .refresh-btn { background: rgba(255,255,255,0.2); border: none; color: white; width: 35px; height: 35px; border-radius: 50%; cursor: pointer; }

        /* --- Layout --- */
        .container { padding: 20px 15px; max-width: 100%; margin: 0 auto; }
        .section { display: none; animation: slideUpFade 0.4s ease; }
        .section.active { display: block; }

        @keyframes slideUpFade {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }

        /* --- Cards & Stats --- */
        .card {
            background: var(--bg-card);
            border-radius: var(--radius);
            padding: 20px;
            margin-bottom: 15px;
            box-shadow: var(--shadow);
            border: 1px solid #e2e8f0;
        }

        .balance-card {
            background: linear-gradient(135deg, var(--primary), #1d4ed8);
            color: white;
            text-align: center;
            padding: 30px 20px;
            margin-bottom: 25px;
            position: relative;
            overflow: hidden;
        }
        .balance-card::before {
            content: ''; position: absolute; top: -50%; left: -50%; width: 200%; height: 200%;
            background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, transparent 60%);
            pointer-events: none;
        }

        .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; margin-bottom: 20px; }
        .stat-item {
            background: var(--bg-card); padding: 15px; border-radius: var(--radius);
            text-align: center; box-shadow: var(--shadow); cursor: pointer;
            transition: transform 0.2s; border: 1px solid #e2e8f0;
        }
        .stat-item:active { transform: scale(0.96); }
        .stat-item h4 { margin: 0 0 5px; color: var(--text-sub); font-size: 0.8rem; }
        .stat-item p { margin: 0; font-size: 1.4rem; font-weight: 800; color: var(--secondary); }

        /* --- Forms --- */
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 8px; font-weight: 600; font-size: 0.9rem; color: var(--text-sub); }
        input, select {
            width: 100%; padding: 14px; border: 1px solid #cbd5e1;
            border-radius: 12px; font-size: 1rem; background-color: #f8fafc;
            transition: 0.3s;
        }
        input:focus { border-color: var(--primary); background: #fff; box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.1); }
        .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }

        button.btn-primary {
            background-color: var(--accent); color: white; border: none; padding: 15px;
            border-radius: 12px; width: 100%; font-size: 1rem; font-weight: 700; cursor: pointer;
            box-shadow: 0 4px 10px rgba(245, 158, 11, 0.3); transition: 0.2s;
        }
        button.btn-primary:active { transform: translateY(2px); }

        /* --- Lists & Tables --- */
        .list-group { display: flex; flex-direction: column; gap: 10px; }
        .list-item {
            background: var(--bg-card); padding: 15px; border-radius: 12px;
            display: flex; justify-content: space-between; align-items: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.03); border: 1px solid #f1f5f9;
        }
        .item-info h4 { margin: 0 0 3px; font-size: 1rem; color: var(--text-main); }
        .item-info p { margin: 0; font-size: 0.8rem; color: var(--text-sub); }
        
        .table-responsive { overflow-x: auto; border-radius: 12px; border: 1px solid #eee; }
        table { width: 100%; border-collapse: collapse; min-width: 500px; }
        th { background: #f8fafc; color: var(--text-sub); font-weight: 600; padding: 12px; text-align: right; font-size: 0.85rem; }
        td { padding: 12px; border-bottom: 1px solid #f1f5f9; font-size: 0.9rem; }
        tr:last-child td { border-bottom: none; }

        /* --- Bottom Navigation (Mobile) --- */
        .bottom-nav {
            position: fixed; bottom: 0; left: 0; width: 100%; height: var(--nav-height);
            background: #fff; display: flex; justify-content: space-around; align-items: center;
            box-shadow: 0 -5px 20px rgba(0,0,0,0.05); z-index: 900;
            border-top-left-radius: 20px; border-top-right-radius: 20px;
            padding-bottom: env(safe-area-inset-bottom);
        }
        .nav-btn {
            background: none; border: none; color: #94a3b8; font-size: 0.7rem; font-weight: 600;
            display: flex; flex-direction: column; align-items: center; gap: 4px; flex: 1; cursor: pointer; transition: 0.3s;
        }
        .nav-btn i { font-size: 1.4rem; transition: 0.3s; }
        .nav-btn.active { color: var(--primary); }
        .nav-btn.active i { transform: translateY(-2px); }

        /* --- Modals (Bottom Sheet) --- */
        .modal {
            display: none; position: fixed; z-index: 2000; left: 0; top: 0; width: 100%; height: 100%;
            background-color: rgba(0,0,0,0.6); align-items: flex-end; backdrop-filter: blur(2px);
        }
        .modal.show { display: flex; animation: fadeInModal 0.2s forwards; }
        .modal-content {
            background: #fff; width: 100%; max-height: 85vh;
            border-radius: 25px 25px 0 0; padding: 25px 20px;
            overflow-y: auto; box-shadow: 0 -10px 40px rgba(0,0,0,0.2);
            animation: slideUpModal 0.3s cubic-bezier(0.16, 1, 0.3, 1);
        }
        .modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
        .modal-header h3 { margin: 0; font-size: 1.2rem; }
        .close-btn { background: #f1f5f9; width: 32px; height: 32px; border-radius: 50%; display: flex; align-items: center; justify-content: center; cursor: pointer; font-size: 1.2rem; }

        @keyframes fadeInModal { from { opacity: 0; } to { opacity: 1; } }
        @keyframes slideUpModal { from { transform: translateY(100%); } to { transform: translateY(0); } }

        /* --- Helper Classes --- */
        .text-success { color: var(--success) !important; }
        .text-danger { color: var(--danger) !important; }
        .text-warning { color: var(--warning) !important; }
        .bg-danger-light { background-color: #fef2f2 !important; color: #991b1b; }
        .bg-warning-light { background-color: #fffbeb !important; color: #92400e; }
        .bg-success-light { background-color: #ecfdf5 !important; color: #065f46; }
        
        .badge { padding: 4px 10px; border-radius: 20px; font-size: 0.75rem; font-weight: 700; }
        
        /* Actions */
        .action-btn { width: 32px; height: 32px; border-radius: 8px; border: none; color: white; display: inline-flex; align-items: center; justify-content: center; cursor: pointer; margin-left: 5px; }
        .act-view { background: #3b82f6; }
        .act-del { background: #ef4444; }

        /* Report Accordion */
        .report-header-row { display: flex; justify-content: space-between; padding: 15px; background: #fff; cursor: pointer; border-bottom: 1px solid #f1f5f9; font-weight: 600; }
        .report-body { display: none; padding: 15px; background: #f8fafc; border-bottom: 1px solid #e2e8f0; }
        .report-body.open { display: block; animation: slideDown 0.3s; }
        @keyframes slideDown { from { opacity:0; transform:translateY(-10px); } to { opacity:1; transform:translateY(0); } }

        /* --- Desktop Responsiveness --- */
        @media (min-width: 992px) {
            body { padding-bottom: 0; padding-right: 260px; /* Sidebar space */ }
            
            .bottom-nav {
                top: 0; right: 0; width: 260px; height: 100vh;
                flex-direction: column; justify-content: flex-start; align-items: flex-start;
                padding-top: 100px; border-radius: 0; border-left: 1px solid #e2e8f0;
            }
            .nav-btn {
                flex-direction: row; width: 100%; padding: 15px 30px; font-size: 1rem;
                flex: 0 0 auto; color: var(--text-sub);
            }
            .nav-btn i { margin-left: 15px; font-size: 1.2rem; width: 25px; text-align: center; }
            .nav-btn.active { background: #eff6ff; border-right: 4px solid var(--primary); color: var(--primary); }
            .nav-btn span { display: inline; }

            .app-header { display: none; } /* Hide mobile header on desktop */
            .container { max-width: 1200px; padding: 40px; }
            
            .stats-grid { grid-template-columns: repeat(4, 1fr); }
            .stat-item.wide { grid-column: span 1; } /* Reset span */
            
            /* Modal Centered */
            .modal { align-items: center; justify-content: center; }
            .modal-content { width: 500px; max-height: 85vh; border-radius: 16px; transform: none !important; animation: fadeInModal 0.3s; }
        }

        /* Loading */
        #loader {
            position: fixed; inset: 0; background: white; z-index: 9999;
            display: flex; flex-direction: column; align-items: center; justify-content: center;
        }
        .spinner { width: 40px; height: 40px; border: 4px solid #f3f3f3; border-top-color: var(--primary); border-radius: 50%; animation: spin 1s linear infinite; }
        @keyframes spin { to { transform: rotate(360deg); } }

    </script>
</style>
</head>
<body>

    <!-- Loader -->
    <div id="loader">
        <div class="spinner"></div>
        <p style="margin-top:15px; color:#64748b; font-weight:600">جاري التحميل...</p>
    </div>

    <!-- Header (Mobile) -->
    <div class="app-header">
        <h1>جمعية الحلاقة</h1>
        <button class="refresh-btn" onclick="location.reload()"><i class="fas fa-sync-alt"></i></button>
    </div>

    <div class="container">
        
        <!-- HOME PAGE -->
        <div id="home" class="section active">
            <!-- Balance Card -->
            <div class="card balance-card" onclick="showBalanceDetails()">
                <p style="opacity:0.9; margin:0">الرصيد الصافي</p>
                <h1 id="netBalance" style="margin:10px 0; font-size:2.8rem; font-weight:800">0</h1>
                <span style="background:rgba(255,255,255,0.2); padding:5px 15px; border-radius:20px; font-size:0.9rem">درهم مغربي</span>
            </div>

            <!-- Stats -->
            <div class="stats-grid">
                <div class="stat-item" onclick="navTo('students', document.querySelectorAll('.nav-btn')[1])">
                    <h4>المتدربين</h4><p id="totalStudents">0</p>
                </div>
                <div class="stat-item" onclick="document.getElementById('reportsSection').scrollIntoView({behavior:'smooth'})">
                    <h4>مداخيل كلية</h4><p id="totalIncome" class="text-success">0</p>
                </div>
                <div class="stat-item wide" style="grid-column: span 2;">
                    <h4>مصاريف هذا الشهر</h4><p id="monthExpenses" class="text-danger">0</p>
                </div>
            </div>

            <!-- Alerts -->
            <div class="card" style="border-right: 5px solid var(--danger);">
                <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px">
                    <h3 style="margin:0; font-size:1.1rem; color:var(--danger)">واجب الأداء</h3>
                    <span class="badge bg-danger-light" id="overdueCount">0</span>
                </div>
                <div class="table-responsive"><table id="overdueTable"><tbody></tbody></table></div>
            </div>

            <div class="card" style="border-right: 5px solid var(--warning);">
                <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px">
                    <h3 style="margin:0; font-size:1.1rem; color:var(--warning)">استحقاق قريب</h3>
                    <span class="badge bg-warning-light" id="upcomingCount">0</span>
                </div>
                <div class="table-responsive"><table id="upcomingTable"><tbody></tbody></table></div>
            </div>

            <div class="card" style="border-right: 5px solid #3b82f6;">
                <h3 style="margin:0 0 15px; font-size:1.1rem; color:#3b82f6">نواقص الملفات</h3>
                <div class="table-responsive"><table id="missingTable"><tbody></tbody></table></div>
            </div>

            <!-- Reports -->
            <div class="card" id="reportsSection">
                <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
                    <h3 style="margin:0; font-size:1.1rem">التقرير المالي</h3>
                    <select id="yearFilter" onchange="renderReports()" style="padding:8px; border-radius:8px; border:1px solid #ddd; outline:none;"></select>
                </div>
                <div id="reportsContainer"></div>
            </div>
        </div>

        <!-- STUDENTS PAGE -->
        <div id="students" class="section">
            <button class="btn-primary" onclick="toggleForm('addStudentForm')" style="margin-bottom:15px">
                <i class="fas fa-user-plus"></i> إضافة متدرب جديد
            </button>

            <!-- Add Form -->
            <div id="addStudentForm" class="card" style="display:none; border-top: 4px solid var(--accent);">
                <form onsubmit="addStudent(event)">
                    <div class="form-group">
                        <label>الاسم الكامل</label>
                        <input type="text" id="sName" required>
                    </div>
                    <div class="form-row">
                        <div class="form-group">
                            <label>تاريخ الازدياد</label>
                            <input type="date" id="sDob" required onchange="calcAge(this.value)">
                            <small id="ageCalc" style="color:var(--primary); font-weight:bold"></small>
                        </div>
                        <div class="form-group">
                            <label>الهاتف</label>
                            <input type="tel" id="sPhone" required>
                        </div>
                    </div>
                    <div class="form-group">
                        <label>رقم البطاقة (اختياري)</label>
                        <input type="text" id="sCNIE">
                    </div>
                    <div class="form-row">
                        <div class="form-group">
                            <label>تاريخ التسجيل</label>
                            <input type="date" id="sRegDate" required>
                        </div>
                        <div class="form-group">
                            <label>مدة (أشهر)</label>
                            <input type="number" id="sDuration" value="9">
                        </div>
                    </div>
                    
                    <div style="background:#f8fafc; padding:15px; border-radius:12px; margin:15px 0; border:1px dashed #cbd5e1;">
                        <div class="form-row">
                            <div class="form-group"><label>واجب التسجيل</label><input type="number" id="sRegFee"></div>
                            <div class="form-group"><label>واجب شهري</label><input type="number" id="sTrainFee"></div>
                        </div>
                        <div class="form-group"><label>مواد (مرة واحدة)</label><input type="number" id="sMatFee"></div>
                        <div class="form-row">
                            <div class="form-group"><label>ثمن المعدات</label><input type="number" id="sEquipFee"></div>
                            <div class="form-group"><label>تقسيط المعدات</label><input type="number" id="sEquipDur" value="1"></div>
                        </div>
                    </div>

                    <div class="form-group">
                        <label>اللوازم:</label>
                        <div id="suppliesChecklist" style="display:flex; flex-wrap:wrap; gap:10px"></div>
                    </div>

                    <div class="form-group">
                        <label class="text-success">دفع مسبق (الآن)</label>
                        <input type="number" id="sInitPay" placeholder="0">
                    </div>

                    <button type="submit" class="btn-primary">حفظ البيانات</button>
                </form>
            </div>

            <!-- Search -->
            <div style="position:relative; margin-bottom:15px">
                <input type="text" id="searchBox" placeholder="بحث بالاسم أو الهاتف..." onkeyup="renderStudents()" style="padding-right:40px">
                <i class="fas fa-search" style="position:absolute; right:15px; top:15px; color:#94a3b8"></i>
            </div>

            <!-- List -->
            <div class="list-group" id="studentsList"></div>
        </div>

        <!-- EXPENSES PAGE -->
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

        <!-- SETTINGS PAGE -->
        <div id="settings" class="section">
            <div class="card">
                <h3>الإعدادات العامة</h3>
                <div class="form-group">
                    <label>القسط الشهري الافتراضي</label>
                    <input type="number" id="defTrainFee" onchange="saveSettings()">
                </div>
                
                <hr style="border-color:#eee; margin:20px 0">
                
                <h4>لوازم التسجيل</h4>
                <div style="display:flex; gap:10px; margin-bottom:10px">
                    <input type="text" id="newSupply" placeholder="اسم اللازمة">
                    <button class="btn-primary" style="width:auto" onclick="addSupplyItem()">+</button>
                </div>
                <div id="settingsSupplies" style="display:flex; flex-direction:column; gap:5px"></div>
                
                <hr style="border-color:#eee; margin:20px 0">
                
                <button class="btn-primary" style="background:var(--danger)" onclick="clearAllData()">
                    <i class="fas fa-trash"></i> إعادة ضبط المصنع
                </button>
            </div>
        </div>

    </div>

    <!-- Navigation -->
    <div class="bottom-nav">
        <button class="nav-btn active" onclick="navTo('home', this)">
            <i class="fas fa-chart-pie"></i><span>الرئيسية</span>
        </button>
        <button class="nav-btn" onclick="navTo('students', this)">
            <i class="fas fa-users"></i><span>المتدربون</span>
        </button>
        <button class="nav-btn" onclick="navTo('expenses', this)">
            <i class="fas fa-wallet"></i><span>المصاريف</span>
        </button>
        <button class="nav-btn" onclick="navTo('settings', this)">
            <i class="fas fa-cog"></i><span>إعدادات</span>
        </button>
    </div>

    <!-- Modals -->
    
    <!-- Balance Modal -->
    <div id="balanceModal" class="modal modal-small">
        <div class="modal-content">
            <div class="modal-header">
                <h3>تفاصيل الرصيد</h3>
                <div class="close-btn" onclick="closeModal('balanceModal')">&times;</div>
            </div>
            <div class="list-group">
                <div class="list-item">
                    <span>مجموع المداخيل</span>
                    <span class="text-success" style="font-weight:bold" id="modInc">0</span>
                </div>
                <div class="list-item">
                    <span>مجموع المصاريف</span>
                    <span class="text-danger" style="font-weight:bold" id="modExp">0</span>
                </div>
                <div class="list-item" style="background:#f8fafc">
                    <span>الرصيد الصافي</span>
                    <span style="font-weight:bold; color:var(--primary)" id="modNet">0</span>
                </div>
            </div>
        </div>
    </div>

    <!-- Profile Modal -->
    <div id="profileModal" class="modal">
        <div class="modal-content">
            <div class="modal-header">
                <h3>ملف المتدرب</h3>
                <div class="close-btn" onclick="closeModal('profileModal')">&times;</div>
            </div>
            <div id="profileContent"></div>
            <h4 style="margin:20px 0 10px; border-bottom:1px solid #eee; padding-bottom:5px">لوازم التسجيل</h4>
            <div id="profileSupplies"></div>
            <button class="btn-primary" style="margin-top:20px" onclick="toPay()">سجل الأداءات</button>
        </div>
    </div>

    <!-- Payment Modal -->
    <div id="paymentModal" class="modal">
        <div class="modal-content">
            <div class="modal-header">
                <h3 id="payName"></h3>
                <div class="close-btn" onclick="closeModal('paymentModal')">&times;</div>
            </div>
            <div class="table-responsive">
                <table>
                    <thead><tr><th>البيان</th><th>تاريخ</th><th>الحالة</th><th></th></tr></thead>
                    <tbody id="payBody"></tbody>
                </table>
            </div>
        </div>
    </div>

    <script>
        // --- تهيئة Firebase ---
        const firebaseConfig = {
            apiKey: "AIzaSyB5a4zFSlo_Td4wto1_HCNeb7CpI4AzJkM",
            authDomain: "hila9a-64416.firebaseapp.com",
            projectId: "hila9a-64416",
            storageBucket: "hila9a-64416.firebasestorage.app",
            messagingSenderId: "869332200810",
            appId: "1:869332200810:web:46b8f9f634d0cbba41c913",
            measurementId: "G-HNVEL5XJJP"
        };

        firebase.initializeApp(firebaseConfig);
        const db = firebase.firestore();
        db.enablePersistence().catch(err => console.log(err));

        let students = [], expenses = [], settings = { trainFee: 300, supplies: ['صورة'] }, currId = null;
        const months = ["يناير","فبراير","مارس","أبريل","مايو","يونيو","يوليو","غشت","شتنبر","أكتوبر","نونبر","دجنبر"];

        // --- تحميل البيانات ---
        window.onload = () => {
            const today = new Date().toISOString().split('T')[0];
            if(document.getElementById('sRegDate')) document.getElementById('sRegDate').value = today;
            if(document.getElementById('exDate')) document.getElementById('exDate').value = today;

            db.collection("settings").doc("config").onSnapshot(d => {
                if(d.exists) { settings = d.data(); updateSettingsUI(); }
                else db.collection("settings").doc("config").set(settings);
            });

            db.collection("students").onSnapshot(snap => {
                students = snap.docs.map(d => ({id: d.id, ...d.data()}));
                document.getElementById('loader').style.display = 'none';
                renderStudents();
                updateDash();
                if(currId && document.getElementById('paymentModal').classList.contains('show')) openPay(currId);
                if(currId && document.getElementById('profileModal').classList.contains('show')) openProfile(currId);
            });

            db.collection("expenses").onSnapshot(snap => {
                expenses = snap.docs.map(d => ({id: d.id, ...d.data()}));
                renderExpenses();
                updateDash();
            });
        };

        // --- Navigation ---
        window.navTo = (id, btn) => {
            document.querySelectorAll('.section').forEach(s => s.classList.remove('active'));
            document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
            document.getElementById(id).classList.add('active');
            if(btn) btn.classList.add('active');
        };

        window.toggleForm = (id) => {
            const el = document.getElementById(id);
            if(el.style.display === 'none') {
                el.style.display = 'block';
                const div = document.getElementById('suppliesChecklist');
                div.innerHTML = '';
                (settings.supplies || []).forEach(s => {
                    div.innerHTML += `<label class="supply-checkbox-item" style="background:#fff; border:1px solid #eee; padding:8px; border-radius:8px"><input type="checkbox" value="${s}"> ${s}</label>`;
                });
            } else el.style.display = 'none';
        };

        window.calcAge = (val) => {
            if(!val) return;
            const age = Math.abs(new Date(Date.now() - new Date(val).getTime()).getUTCFullYear() - 1970);
            document.getElementById('ageDisplay').innerText = `(${age} سنة)`;
        };

        // --- Students ---
        window.addStudent = (e) => {
            e.preventDefault();
            const getVal = (id) => document.getElementById(id).value;
            const getNum = (id) => parseFloat(document.getElementById(id).value) || 0;

            const name = getVal('sName');
            const supplies = [];
            document.querySelectorAll('#suppliesChecklist input').forEach(c => supplies.push({name: c.value, checked: c.checked}));

            const payments = [];
            const rFee = getNum('sRegFee');
            const tFee = getNum('sTrainFee');
            const mFee = getNum('sMatFee');
            const eFee = getNum('sEquipFee');
            const dur = parseInt(getVal('sDuration')) || 9;
            const eDur = parseInt(getVal('sEquipDur')) || 1;
            const regDate = getVal('sRegDate');
            let init = getNum('sInitialPay');

            if(rFee > 0) payments.push({type:'reg', label:'واجب التسجيل', amount:rFee, paid:0, due:regDate});
            if(mFee > 0) payments.push({type:'mat', label:'مواد الحلاقة', amount:mFee, paid:0, due:regDate});

            const d = new Date(regDate);
            for(let i=0; i<dur; i++) {
                const due = new Date(d); due.setMonth(d.getMonth() + i);
                payments.push({type:'train', label:`تدريب شهر ${i+1}`, amount:tFee, paid:0, due:due.toISOString()});
            }

            if(eFee > 0) {
                const ep = eFee / eDur;
                for(let i=0; i<eDur; i++) {
                    const due = new Date(d); due.setMonth(d.getMonth() + i);
                    payments.push({type:'equip', label:`معدات ${i+1}/${eDur}`, amount:ep, paid:0, due:due.toISOString()});
                }
            }

            if(init > 0) {
                for(let p of payments) {
                    if(init <= 0.1) break;
                    let rem = p.amount - p.paid;
                    let pay = Math.min(rem, init);
                    p.paid += pay;
                    init -= pay;
                }
            }

            db.collection("students").add({
                name, dob: getVal('sDob'), phone: getVal('sPhone'), cnie: getVal('sCNIE'),
                regDate, trainFee: tFee, duration: dur, supplies, payments
            }).then(() => {
                e.target.reset(); toggleForm('addStudentForm'); alert("تم الحفظ");
            });
        };

        window.renderStudents = () => {
            const list = document.getElementById('studentsList');
            list.innerHTML = '';
            const q = (document.getElementById('searchBox').value || '').toLowerCase();

            students.forEach(s => {
                if(s.name.toLowerCase().includes(q) || s.phone.includes(q)) {
                    const tPay = s.payments.filter(p => p.type === 'train');
                    const paid = tPay.filter(p => p.paid >= p.amount - 1).length;
                    const pct = tPay.length ? Math.round((paid/tPay.length)*100) : 0;

                    const item = document.createElement('div');
                    item.className = 'list-item';
                    item.innerHTML = `
                        <div class="item-info" onclick="openProfile('${s.id}')" style="cursor:pointer; flex:1">
                            <h4>${s.name}</h4>
                            <p>${s.phone}</p>
                            <div style="height:4px; background:#f1f5f9; width:100px; margin-top:5px; border-radius:2px">
                                <div style="height:100%; width:${pct}%; background:${pct===100?'var(--success)':'var(--accent)'}; border-radius:2px"></div>
                            </div>
                        </div>
                        <div style="display:flex;">
                            <button class="action-btn act-view" onclick="openPay('${s.id}')"><i class="fas fa-money-bill"></i></button>
                            <button class="action-btn act-del" onclick="delStudent('${s.id}')"><i class="fas fa-trash"></i></button>
                        </div>
                    `;
                    list.appendChild(item);
                }
            });
        };

        window.delStudent = (id) => { if(confirm("حذف؟")) db.collection("students").doc(id).delete(); };

        // --- Profile ---
        window.openProfile = (id) => {
            currId = id;
            const s = students.find(x => x.id === id);
            if(!s) return;

            const age = Math.abs(new Date(Date.now() - new Date(s.dob).getTime()).getUTCFullYear() - 1970);
            let req = 0, pd = 0;
            s.payments.forEach(p => { req += p.amount; pd += p.paid; });

            document.getElementById('profileContent').innerHTML = `
                <div style="text-align:center; margin-bottom:15px">
                    <div style="width:60px; height:60px; background:#eff6ff; color:var(--primary); font-size:24px; font-weight:bold; display:flex; align-items:center; justify-content:center; border-radius:50%; margin:0 auto">${s.name.charAt(0)}</div>
                    <h3>${s.name}</h3>
                    <p style="color:#64748b">${s.phone}</p>
                </div>
                <div style="display:grid; grid-template-columns:1fr 1fr; gap:10px; margin-bottom:15px">
                    <div style="background:#ecfdf5; padding:10px; border-radius:8px; text-align:center">
                        <span style="font-size:0.8rem">المدفوع</span>
                        <div style="color:var(--success); font-weight:bold">${Math.round(pd)}</div>
                    </div>
                    <div style="background:#fef2f2; padding:10px; border-radius:8px; text-align:center">
                        <span style="font-size:0.8rem">المتبقي</span>
                        <div style="color:var(--danger); font-weight:bold">${Math.round(req - pd)}</div>
                    </div>
                </div>
                <p style="margin:5px 0"><strong>السن:</strong> ${age} سنة</p>
                <p style="margin:5px 0"><strong>تاريخ التسجيل:</strong> ${s.regDate}</p>
            `;

            const supDiv = document.getElementById('profileSupplies');
            supDiv.innerHTML = '';
            if(s.supplies) {
                s.supplies.forEach((sup, i) => {
                    supDiv.innerHTML += `
                        <div style="display:flex; justify-content:space-between; padding:8px; border-bottom:1px solid #f1f5f9; align-items:center">
                            <span>${sup.name}</span>
                            <input type="checkbox" style="width:20px; height:20px" ${sup.checked?'checked':''} onchange="togSup('${s.id}', ${i}, this.checked)">
                        </div>
                    `;
                });
            }
            
            document.getElementById('profileModal').classList.add('show');
        };

        window.togSup = (sid, idx, val) => {
            const s = students.find(x => x.id === sid);
            s.supplies[idx].checked = val;
            db.collection("students").doc(sid).update({supplies: s.supplies});
        };

        window.toPay = () => {
            window.closeModal('profileModal');
            window.openPay(currId);
        };

        // --- Payments ---
        window.openPay = (id) => {
            currId = id;
            const s = students.find(x => x.id === id);
            if(!s) return;

            document.getElementById('payModalName').innerText = s.name;
            const tbody = document.getElementById('payBody');
            tbody.innerHTML = '';
            const today = new Date(); today.setHours(0,0,0,0);

            s.payments.forEach((p, idx) => {
                const rem = p.amount - p.paid;
                const due = new Date(p.due); due.setHours(0,0,0,0);
                const diff = Math.ceil((due - today) / (1000*60*60*24));

                let badge = '', btn = '', rowClass = '';

                if(rem <= 0.5) {
                    rowClass = 'background-color:#ecfdf5';
                    badge = '<span class="badge bg-success-light text-success">مدفوع</span>';
                    btn = `<button style="background:none; border:none; color:var(--success); font-size:1.2rem; cursor:pointer" onclick="cancelPay('${s.id}', ${idx})"><i class="fas fa-check-circle"></i></button>`;
                } else {
                    if(diff <= 0) badge = '<span class="badge bg-danger-light text-danger">واجب</span>';
                    else if(diff <= 3) badge = '<span class="badge bg-warning-light text-warning">قريب</span>';
                    else badge = '<span class="badge" style="background:#f1f5f9; color:#64748b">قادم</span>';

                    const txt = p.paid > 0 ? 'إتمام' : 'دفع';
                    const col = p.paid > 0 ? 'var(--warning)' : 'var(--primary)';
                    if(p.paid > 0) rowClass = 'background-color:#fffbeb';

                    btn = `<button style="background:${col}; color:white; border:none; padding:5px 12px; border-radius:8px; font-weight:bold; font-size:0.8rem" onclick="pay('${s.id}', ${idx})">${txt}</button>`;
                }

                tbody.innerHTML += `
                    <tr style="${rowClass}">
                        <td><b>${p.label}</b><br><small style="color:#64748b">${Math.round(p.paid)} / ${Math.round(p.amount)}</small></td>
                        <td>${new Date(p.due).toLocaleDateString('ar-MA')}</td>
                        <td>${badge}</td>
                        <td>${btn}</td>
                    </tr>
                `;
            });
            document.getElementById('paymentModal').classList.add('show');
        };

        window.pay = (sid, idx) => {
            const s = students.find(x => x.id === sid);
            const p = s.payments[idx];
            const rem = p.amount - p.paid;
            const input = prompt(`المبلغ المتبقي: ${Math.round(rem)} درهم.\nأدخل المبلغ:`, Math.round(rem));
            
            if(input) {
                const val = parseFloat(input);
                if(val > 0) {
                    p.paid += Math.min(val, rem);
                    db.collection("students").doc(sid).update({payments: s.payments});
                }
            }
        };

        window.cancelPay = (sid, idx) => {
            if(confirm("إلغاء الدفع؟")) {
                const s = students.find(x => x.id === sid);
                s.payments[idx].paid = 0;
                db.collection("students").doc(sid).update({payments: s.payments});
            }
        };

        // --- Expenses ---
        window.addExpense = (e) => {
            e.preventDefault();
            db.collection("expenses").add({
                desc: document.getElementById('exDesc').value,
                amount: parseFloat(document.getElementById('exAmount').value),
                date: document.getElementById('exDate').value
            }).then(() => e.target.reset());
        };

        window.renderExpenses = () => {
            const tbody = document.querySelector('#expensesTable tbody');
            tbody.innerHTML = '';
            expenses.sort((a,b) => new Date(b.date)-new Date(a.date)).forEach(ex => {
                tbody.innerHTML += `<tr><td>${ex.date}</td><td>${ex.desc}</td><td class="text-danger">-${ex.amount}</td><td><i class="fas fa-trash text-danger" onclick="db.collection('expenses').doc('${ex.id}').delete()" style="cursor:pointer"></i></td></tr>`;
            });
        };

        // --- Dashboard & Reports ---
        window.updateDash = () => {
            document.getElementById('totalStudents').innerText = students.length;
            let inc = 0; students.forEach(s => s.payments.forEach(p => inc += p.paid));
            let exp = 0; expenses.forEach(e => exp += e.amount);
            
            document.getElementById('totalIncome').innerText = Math.round(inc);
            document.getElementById('netBalance').innerText = Math.round(inc - exp);
            document.getElementById('modInc').innerText = Math.round(inc);
            document.getElementById('modExp').innerText = Math.round(exp);
            document.getElementById('modNet').innerText = Math.round(inc - exp);

            // Month Expenses
            const now = new Date();
            let mExp = 0;
            expenses.forEach(e => {
                const d = new Date(e.date);
                if(d.getMonth() === now.getMonth() && d.getFullYear() === now.getFullYear()) mExp += e.amount;
            });
            document.getElementById('monthExpenses').innerText = Math.round(mExp);

            // Alerts Tables
            const t1 = document.querySelector('#overdueTable tbody');
            const t2 = document.querySelector('#upcomingTable tbody');
            const t3 = document.querySelector('#missingTable tbody');
            t1.innerHTML = ''; t2.innerHTML = ''; t3.innerHTML = '';
            
            const today = new Date(); today.setHours(0,0,0,0);

            students.forEach(s => {
                // Missing
                if(s.supplies && s.supplies.some(x => !x.checked)) {
                    const m = s.supplies.filter(x => !x.checked).map(x => x.name).join(', ');
                    t3.innerHTML += `<tr onclick="openProfile('${s.id}')"><td><span class="clickable-name">${s.name}</span></td><td class="text-danger">${m}</td></tr>`;
                }
                
                // Due
                s.payments.forEach(p => {
                    if(p.amount - p.paid > 0.5) {
                        const due = new Date(p.due); due.setHours(0,0,0,0);
                        const diff = Math.ceil((due - today) / (1000*60*60*24));
                        const row = `<tr onclick="openPay('${s.id}')"><td>${s.name}</td><td>${p.label}</td><td style="font-weight:bold">${Math.round(p.amount-p.paid)}</td></tr>`;
                        if(diff <= 0) t1.innerHTML += row;
                        else if(diff <= 3) t2.innerHTML += row;
                    }
                });
            });

            if(!t1.innerHTML) t1.innerHTML = '<tr><td colspan="3" style="text-align:center; color:#94a3b8">لا توجد متأخرات</td></tr>';
            if(!t2.innerHTML) t2.innerHTML = '<tr><td colspan="3" style="text-align:center; color:#94a3b8">لا توجد استحقاقات</td></tr>';
            if(!t3.innerHTML) t3.innerHTML = '<tr><td colspan="2" style="text-align:center; color:#94a3b8">الملفات مكتملة</td></tr>';

            renderReports();
        };

        window.renderReports = () => {
            const years = new Set();
            students.forEach(s => s.payments.forEach(p => { if(p.paid > 0) years.add(new Date(p.due).getFullYear()) }));
            expenses.forEach(e => years.add(new Date(e.date).getFullYear()));
            years.add(new Date().getFullYear());

            const sel = document.getElementById('yearFilter');
            const curr = sel.value || new Date().getFullYear();
            sel.innerHTML = '';
            Array.from(years).sort().reverse().forEach(y => sel.innerHTML += `<option value="${y}" ${y==curr?'selected':''}>${y}</option>`);

            const div = document.getElementById('reportsContainer');
            div.innerHTML = '';
            
            const stats = {};
            students.forEach(s => s.payments.forEach(p => {
                if(p.paid > 0) {
                    const d = new Date(p.due);
                    if(d.getFullYear() == curr) {
                        const m = d.getMonth();
                        if(!stats[m]) stats[m] = {inc:0, exp:0, iDet:[], eDet:[]};
                        stats[m].inc += p.paid;
                        stats[m].iDet.push({n: s.name, t: p.label, a: p.paid});
                    }
                }
            }));
            expenses.forEach(e => {
                const d = new Date(e.date);
                if(d.getFullYear() == curr) {
                    const m = d.getMonth();
                    if(!stats[m]) stats[m] = {inc:0, exp:0, iDet:[], eDet:[]};
                    stats[m].exp += e.amount;
                    stats[m].eDet.push({n: e.desc, a: e.amount});
                }
            });

            if(Object.keys(stats).length === 0) {
                div.innerHTML = '<p style="text-align:center; color:#94a3b8; padding:20px">لا توجد بيانات</p>';
                return;
            }

            Object.keys(stats).sort((a,b)=>b-a).forEach(m => {
                const st = stats[m];
                let dets = '';
                
                if(st.iDet.length) {
                    dets += `<div style="margin-top:10px"><strong class="text-success">المداخيل:</strong>`;
                    st.iDet.forEach(x => dets += `<div class="report-row"><span>${x.n} (${x.t})</span><span>+${Math.round(x.a)}</span></div>`);
                    dets += `</div>`;
                }
                if(st.eDet.length) {
                    dets += `<div style="margin-top:10px"><strong class="text-danger">المصاريف:</strong>`;
                    st.eDet.forEach(x => dets += `<div class="report-row"><span>${x.n}</span><span>-${Math.round(x.a)}</span></div>`);
                    dets += `</div>`;
                }

                div.innerHTML += `
                    <div class="report-item">
                        <div class="report-header" onclick="this.nextElementSibling.classList.toggle('show')">
                            <span>${arabicMonths[m]}</span>
                            <span>${Math.round(st.inc - st.exp)} dh</span>
                        </div>
                        <div class="report-details">
                            <div style="display:flex; justify-content:space-between; font-weight:bold; padding-bottom:5px; border-bottom:1px solid #eee">
                                <span class="text-success">+${Math.round(st.inc)}</span>
                                <span class="text-danger">-${Math.round(st.exp)}</span>
                            </div>
                            ${dets}
                        </div>
                    </div>
                `;
            });
        };

        // --- Settings & Utils ---
        window.updateSettingsUI = () => {
            document.getElementById('defTrainFee').value = settings.trainFee;
            document.getElementById('sTrainFee').value = settings.trainFee;
            const div = document.getElementById('settingsSupplies');
            div.innerHTML = '';
            (settings.supplies||[]).forEach((s,i) => {
                div.innerHTML += `<div class="list-item" style="padding:10px"><span>${s}</span><button class="btn-icon btn-red" onclick="remSupply(${i})">x</button></div>`;
            });
        };

        window.saveSettings = () => {
            settings.trainFee = document.getElementById('defTrainFee').value;
            db.collection("settings").doc("config").update(settings);
        };

        window.addSupplyItem = () => {
            const v = document.getElementById('newSupply').value;
            if(v) {
                settings.supplies = [...(settings.supplies||[]), v];
                db.collection("settings").doc("config").update({supplies: settings.supplies});
                document.getElementById('newSupply').value = '';
            }
        };

        window.remSupply = (i) => {
            settings.supplies.splice(i, 1);
            db.collection("settings").doc("config").update({supplies: settings.supplies});
        };

        window.clearAllData = () => {
            if(confirm("حذف الكل؟")) {
                students.forEach(s => db.collection("students").doc(s.id).delete());
                expenses.forEach(e => db.collection("expenses").doc(e.id).delete());
            }
        };

        window.showBalanceDetails = () => {
            document.getElementById('balanceModal').classList.add('show');
        };

        window.closeModal = (id) => {
            document.getElementById(id).classList.remove('show');
        };

    </script>
</body>
</html>
