<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>تطبيق جمعية الحلاقة</title>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    
    <!-- Firebase SDK -->
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
            --nav-height: 65px;
            --sidebar-width: 250px;
        }

        * {
            box-sizing: border-box;
            outline: none;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            margin: 0;
            padding: 0;
            color: var(--text-color);
            /* مساحة للقائمة السفلية في الموبايل */
            padding-bottom: calc(var(--nav-height) + 20px); 
            user-select: none;
            -webkit-tap-highlight-color: transparent;
            transition: padding 0.3s;
        }

        /* --- Header --- */
        .mobile-header {
            background-color: var(--primary-color);
            color: white;
            padding: 15px;
            text-align: center;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            position: sticky;
            top: 0;
            z-index: 100;
        }
        .mobile-header h2 { margin: 0; font-size: 1.2rem; }

        /* --- Layout & Container --- */
        .container {
            padding: 15px;
            margin: 0 auto;
            max-width: 100%;
            transition: max-width 0.3s;
        }

        .section { display: none; animation: fadeIn 0.3s; }
        .section.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }

        /* --- Cards --- */
        .card {
            background: var(--card-bg);
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.05);
            border: 1px solid #eee;
        }

        /* --- Stats Grid --- */
        .stats-grid {
            display: grid;
            grid-template-columns: 1fr 1fr; /* افتراضي للموبايل: عمودين */
            gap: 15px;
            margin-bottom: 20px;
        }
        .stat-box {
            background: white; padding: 15px; border-radius: 10px; text-align: center;
            border: 1px solid #eee; cursor: pointer; transition: 0.2s;
            box-shadow: 0 2px 5px rgba(0,0,0,0.02);
        }
        .stat-box:hover { transform: translateY(-3px); box-shadow: 0 5px 15px rgba(0,0,0,0.1); }
        .stat-box h4 { margin: 0 0 5px; color: #7f8c8d; font-size: 0.9rem; }
        .stat-box p { margin: 0; font-size: 1.4rem; font-weight: bold; color: var(--primary-color); }

        /* --- Forms --- */
        .form-group { margin-bottom: 15px; }
        .form-group label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 0.9rem; }
        .form-group input, .form-group select {
            width: 100%; padding: 12px; border: 1px solid #ddd;
            border-radius: 8px; box-sizing: border-box; background: #fafafa;
            font-size: 1rem;
        }
        .form-row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }

        button.btn-primary {
            background-color: var(--accent-color); color: white; border: none; padding: 12px;
            border-radius: 8px; width: 100%; font-size: 1rem; font-weight: bold; cursor: pointer;
            transition: 0.2s;
        }
        button.btn-primary:hover { background-color: #d35400; }

        /* --- Tables --- */
        .table-responsive { overflow-x: auto; }
        table { width: 100%; border-collapse: collapse; min-width: 100%; }
        th, td { text-align: right; padding: 12px; border-bottom: 1px solid #eee; font-size: 0.9rem; }
        th { color: #555; font-weight: 700; white-space: nowrap; background-color: #f9f9f9; }
        .clickable-name { font-weight: bold; color: var(--primary-color); border-bottom: 1px dashed #ccc; cursor: pointer; }
        tr:hover { background-color: #fcfcfc; }

        /* --- Navigation (Mobile Default) --- */
        .bottom-nav {
            position: fixed; bottom: 0; left: 0; width: 100%; height: var(--nav-height);
            background-color: #fff; display: flex; justify-content: space-around;
            align-items: center; box-shadow: 0 -2px 10px rgba(0,0,0,0.05); z-index: 1000;
            transition: all 0.3s ease;
        }
        .nav-item {
            border: none; background: none; color: #95a5a6; font-size: 0.75rem;
            display: flex; flex-direction: column; align-items: center; flex: 1; cursor: pointer;
        }
        .nav-item.active { color: var(--accent-color); font-weight: bold; border-top: 3px solid var(--accent-color); }
        .nav-item i { font-size: 1.3rem; margin-bottom: 4px; }
        .nav-item span { display: block; } /* Text label */

        /* --- Modals (Mobile Default) --- */
        .modal {
            display: none; position: fixed; z-index: 2000; left: 0; top: 0; width: 100%; height: 100%;
            background-color: rgba(0,0,0,0.5); align-items: flex-end; /* من الأسفل للموبايل */
        }
        .modal-content {
            background: white; width: 100%; height: 85vh; border-radius: 20px 20px 0 0;
            padding: 25px; box-sizing: border-box; overflow-y: auto; animation: slideUp 0.3s;
        }
        .modal-small .modal-content { height: auto; border-radius: 15px; width: 95%; margin: auto; position: relative; top: 50%; transform: translateY(-50%); }
        @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }

        /* --- Badges & Lists --- */
        .badge { padding: 4px 10px; border-radius: 12px; font-size: 0.75rem; font-weight: bold; display: inline-block; }
        .report-header { padding: 15px; display: flex; justify-content: space-between; cursor: pointer; background: #fff; border-bottom: 1px solid #eee; }
        .report-header:hover { background: #f9f9f9; }
        .report-details { display: none; padding: 15px; background: #fafafa; border-top: 1px solid #eee; }
        .report-details.show { display: block; }
        .supply-checkbox-item { display: flex; align-items: center; padding: 8px; border-bottom: 1px solid #eee; }
        .supply-checkbox-item input { width: 20px; height: 20px; margin-left: 10px; }

        /* --- Loading --- */
        #loading-overlay {
            position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #fff;
            display: flex; justify-content: center; align-items: center; z-index: 9999; flex-direction: column;
        }
        .spinner { border: 4px solid #f3f3f3; border-top: 4px solid var(--accent-color); border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }

        /* ========================================= */
        /*        MEDIA QUERIES (DESKTOP/TABLET)     */
        /* ========================================= */
        
        @media (min-width: 992px) {
            /* 1. تعديل الجسم والبادينج */
            body {
                padding-bottom: 0; /* إلغاء مساحة القائمة السفلية */
                background-color: #f0f2f5;
            }

            /* 2. تحويل القائمة السفلية إلى قائمة جانبية */
            .bottom-nav {
                top: 0; right: 0; /* تثبيت على اليمين */
                width: var(--sidebar-width);
                height: 100vh;
                flex-direction: column;
                justify-content: flex-start;
                align-items: flex-start;
                padding-top: 80px; /* مساحة تحت الهيدر */
                border-top: none;
                border-left: 1px solid #ddd;
                box-shadow: 2px 0 5px rgba(0,0,0,0.05);
            }

            .nav-item {
                flex: 0 0 auto;
                width: 100%;
                flex-direction: row;
                padding: 15px 25px;
                font-size: 1rem;
                color: #555;
                border-bottom: 1px solid #f9f9f9;
            }

            .nav-item:hover { background-color: #f8f9fa; color: var(--primary-color); }
            
            .nav-item.active {
                background-color: #e3f2fd;
                color: var(--primary-color);
                border-top: none;
                border-right: 4px solid var(--accent-color);
            }

            .nav-item i { margin-left: 15px; margin-bottom: 0; font-size: 1.2rem; width: 30px; text-align: center; }
            .nav-item span { display: inline; font-weight: 600; }

            /* 3. تعديل الهيدر */
            .mobile-header {
                width: calc(100% - var(--sidebar-width));
                margin-right: var(--sidebar-width); /* إزاحة لليسار */
                text-align: right;
                padding-right: 30px;
                background-color: #fff;
                color: var(--primary-color);
                border-bottom: 1px solid #ddd;
            }

            /* 4. تعديل المحتوى الرئيسي */
            .container {
                margin-right: var(--sidebar-width); /* إزاحة المحتوى */
                padding: 30px;
                max-width: 1200px; /* عرض أكبر للكمبيوتر */
            }

            /* 5. تعديل الشبكات والبطاقات */
            .stats-grid {
                grid-template-columns: repeat(4, 1fr); /* 4 أعمدة للإحصائيات */
            }
            .stat-box { grid-column: span 1 !important; } /* إجبار كل صندوق يأخذ عمود واحد */

            .card {
                padding: 30px;
                border-radius: 8px;
                box-shadow: 0 1px 3px rgba(0,0,0,0.1);
            }

            /* 6. تعديل النوافذ المنبثقة لتكون في الوسط */
            .modal {
                align-items: center; /* توسيط عمودي */
                justify-content: center; /* توسيط أفقي */
            }
            .modal-content {
                width: 600px;
                height: auto;
                max-height: 85vh;
                border-radius: 10px;
                box-shadow: 0 10px 30px rgba(0,0,0,0.2);
            }
            
            /* أزرار الحذف والتعديل أصغر */
            .btn-icon { width: 28px; height: 28px; font-size: 0.8rem; }
        }

    </style>
</head>
<body>

    <div id="loading-overlay">
        <div class="spinner"></div>
        <p style="margin-top:15px; font-weight:bold;">جاري تحميل البيانات...</p>
    </div>

    <div class="mobile-header">
        <h2><i class="fas fa-cut"></i> إدارة الجمعية</h2>
    </div>

    <div class="container">
        
        <!-- صفحة الرئيسية -->
        <div id="home" class="section active">
            <div class="card" onclick="window.showBalanceDetails()" style="background: linear-gradient(135deg, #2c3e50, #34495e); color: white; text-align: center; cursor: pointer;">
                <p style="margin:0; opacity:0.9; font-size:1.1rem">الرصيد الصافي</p>
                <h1 id="netBalance" style="margin:10px 0; font-size:3rem">0</h1>
                <p style="margin:0; opacity:0.8">درهم مغربي</p>
            </div>

            <div class="stats-grid">
                <div class="stat-box" onclick="window.navTo('students', document.querySelectorAll('.nav-item')[1])">
                    <h4>المتدربين</h4><p id="totalStudents">0</p>
                </div>
                <div class="stat-box" onclick="document.getElementById('monthlyReportCard').scrollIntoView({behavior: 'smooth'})">
                    <h4>المداخيل الكلية</h4><p id="totalIncome" style="color:green">0</p>
                </div>
                <!-- خانة مصاريف الشهر الجديد -->
                <div class="stat-box" style="grid-column: span 2;">
                    <h4>مصاريف هذا الشهر</h4><p id="currentMonthExpenses" style="color:var(--danger-color)">0</p>
                </div>
                <div class="stat-box" style="display:none"> <!-- Placeholder for desktop grid alignment --></div>
            </div>

            <div class="card" style="border-right: 5px solid var(--danger-color);">
                <h3 style="color:var(--danger-color); font-size:1.1rem"><i class="fas fa-exclamation-circle"></i> متأخرات الدفع (واجبة)</h3>
                <div class="table-responsive"><table id="overdueTable"><tbody></tbody></table></div>
            </div>

            <div class="card" style="border-right: 5px solid var(--warning-color);">
                <h3 style="color:var(--warning-color); font-size:1.1rem"><i class="fas fa-clock"></i> استحقاق قريب (3 أيام)</h3>
                <div class="table-responsive"><table id="upcomingTable"><tbody></tbody></table></div>
            </div>

            <div class="card" style="border-right: 5px solid #3498db;">
                <h3 style="color:#3498db; font-size:1.1rem"><i class="fas fa-file-contract"></i> نواقص ملفات التسجيل</h3>
                <div class="table-responsive"><table id="missingTable"><tbody></tbody></table></div>
            </div>

            <div class="card" id="monthlyReportCard">
                <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
                    <h3 style="margin:0">التقرير الشهري المفصل</h3>
                    <select id="yearFilter" onchange="window.renderDashboardReports()" style="padding:8px; border-radius:5px; border:1px solid #ddd;"></select>
                </div>
                <div id="monthlyReportContainer"></div>
            </div>
        </div>

        <!-- صفحة المتدربين -->
        <div id="students" class="section">
            <div class="card">
                <button class="btn-primary" onclick="window.toggleForm('studentForm')"><i class="fas fa-user-plus"></i> إضافة متدرب جديد</button>
                <form id="studentForm" style="display:none; margin-top:20px;" onsubmit="window.addStudent(event)">
                    <div class="form-group"><label>الاسم الكامل</label><input type="text" id="sName" required></div>
                    <div class="form-row">
                        <div class="form-group"><label>تاريخ الازدياد</label><input type="date" id="sDob" required></div>
                        <div class="form-group"><label>رقم الهاتف</label><input type="tel" id="sPhone" required></div>
                    </div>
                    <div class="form-group"><label>رقم البطاقة الوطنية</label><input type="text" id="sCNIE"></div>
                    <div class="form-row">
                        <div class="form-group"><label>تاريخ التسجيل</label><input type="date" id="sRegDate" required></div>
                        <div class="form-group"><label>مدة التدريب (أشهر)</label><input type="number" id="sDuration" value="9" required></div>
                    </div>
                    <div class="form-row">
                        <div class="form-group"><label>رسوم التسجيل</label><input type="number" id="sRegFee" placeholder="درهم"></div>
                        <div class="form-group"><label>قسط التدريب</label><input type="number" id="sTrainFee" placeholder="درهم"></div>
                    </div>
                    <div class="form-group"><label>مواد الحلاقة</label><input type="number" id="sMatFee"></div>
                    <div class="form-row">
                        <div class="form-group"><label>مبلغ المعدات</label><input type="number" id="sEquipFee"></div>
                        <div class="form-group"><label>تقسيط المعدات (أشهر)</label><input type="number" id="sEquipDuration" value="1"></div>
                    </div>
                    <div class="form-group"><label>لوازم التسجيل</label><div id="suppliesChecklist" class="supplies-list"></div></div>
                    <div class="form-group"><label style="color:var(--success-color)">تسبيق (اختياري)</label><input type="number" id="sInitialPay"></div>
                    <button type="submit" class="btn-primary">حفظ البيانات</button>
                </form>
            </div>
            <input type="text" id="searchBox" placeholder="بحث بالاسم أو الهاتف..." onkeyup="window.renderStudents()" style="width:100%; padding:15px; margin-top:10px; border-radius:30px; border:1px solid #ccc; text-align:center; box-shadow: 0 2px 5px rgba(0,0,0,0.05);">
            <div class="card table-responsive" style="margin-top:15px; padding:0;">
                <table id="studentsTable"><thead><tr><th>الاسم</th><th>التقدم</th><th>إجراء</th></tr></thead><tbody></tbody></table>
            </div>
        </div>

        <!-- المصاريف -->
        <div id="expenses" class="section">
            <div class="card">
                <h3>إضافة مصروف</h3>
                <form onsubmit="window.addExpense(event)">
                    <div class="form-row">
                        <div class="form-group"><input type="text" id="exDesc" placeholder="البيان" required></div>
                        <div class="form-group"><input type="number" id="exAmount" placeholder="المبلغ" required></div>
                    </div>
                    <div class="form-group"><input type="date" id="exDate" required></div>
                    <button type="submit" class="btn-primary">تسجيل</button>
                </form>
            </div>
            <div class="card table-responsive"><table id="expensesTable"><tbody></tbody></table></div>
        </div>

        <!-- الإعدادات -->
        <div id="settings" class="section">
            <div class="card">
                <h3>الإعدادات</h3>
                <div class="form-group"><label>قسط التدريب الشهري</label><input type="number" id="defTrainFee" onchange="window.saveSettings()"></div>
                <hr>
                <h4>إدارة لوازم التسجيل</h4>
                <div style="display:flex; gap:10px; margin-bottom:10px;">
                    <input type="text" id="newSupply" placeholder="اسم اللازمة">
                    <button class="btn-primary" style="width:auto" onclick="window.addSupplyItem()">+</button>
                </div>
                <ul id="settingsSupplies" style="list-style:none; padding:0"></ul>
                <hr>
                <button class="btn-primary" style="background:var(--danger-color)" onclick="window.clearAllData()">تهيئة المصنع (حذف الكل)</button>
            </div>
        </div>

    </div>

    <!-- القائمة السفلية / الجانبية -->
    <div class="bottom-nav">
        <button class="nav-item active" onclick="window.navTo('home', this)">
            <i class="fas fa-home"></i><span>الرئيسية</span>
        </button>
        <button class="nav-item" onclick="window.navTo('students', this)">
            <i class="fas fa-users"></i><span>المتدربون</span>
        </button>
        <button class="nav-item" onclick="window.navTo('expenses', this)">
            <i class="fas fa-wallet"></i><span>المصاريف</span>
        </button>
        <button class="nav-item" onclick="window.navTo('settings', this)">
            <i class="fas fa-cog"></i><span>الإعدادات</span>
        </button>
    </div>

    <!-- Modals -->
    <div id="balanceModal" class="modal modal-small">
        <div class="modal-content" style="text-align:center;">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:20px;">
                <h3 style="margin:0; color:var(--primary-color)">تفاصيل الرصيد</h3>
                <span onclick="window.closeModal('balanceModal')" style="font-size:24px; cursor:pointer">&times;</span>
            </div>
            <div style="margin:20px 0; font-size:1.1rem;">
                <div style="display:flex; justify-content:space-between; border-bottom:1px solid #eee; padding:15px 0;">
                    <span>مجموع المداخيل:</span>
                    <span style="color:var(--success-color); font-weight:bold;" id="modalTotalIncome">0</span>
                </div>
                <div style="display:flex; justify-content:space-between; border-bottom:1px solid #eee; padding:15px 0;">
                    <span>مجموع المصاريف:</span>
                    <span style="color:var(--danger-color); font-weight:bold;" id="modalTotalExpenses">0</span>
                </div>
                <div style="display:flex; justify-content:space-between; padding:15px; font-weight:bold; background:#f9f9f9; margin-top:15px; border-radius:12px;">
                    <span>الرصيد الصافي:</span>
                    <span style="color:var(--primary-color)" id="modalNetBalance">0</span>
                </div>
            </div>
        </div>
    </div>

    <div id="profileModal" class="modal">
        <div class="modal-content">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
                <h3 style="margin:0; color:var(--primary-color)">ملف المتدرب</h3>
                <span onclick="window.closeModal('profileModal')" style="font-size:24px; cursor:pointer">&times;</span>
            </div>
            <div id="profileContent"></div>
            <h4 style="margin-bottom:10px; border-bottom:1px solid #eee; padding-bottom:5px;">لوازم التسجيل:</h4>
            <div id="profileSuppliesList"></div>
            <button class="btn-primary" style="margin-top:20px" onclick="window.goToPayFromProfile()">سجل الأداءات</button>
        </div>
    </div>

    <div id="paymentModal" class="modal">
        <div class="modal-content">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:15px;">
                <h3 id="payModalName" style="margin:0; color:var(--primary-color)"></h3>
                <span onclick="window.closeModal('paymentModal')" style="font-size:24px; cursor:pointer">&times;</span>
            </div>
            <div class="table-responsive">
                <table><thead><tr><th>البيان</th><th>تاريخ</th><th>الحالة</th><th></th></tr></thead><tbody id="payTableBody"></tbody></table>
            </div>
        </div>
    </div>

    <script>
        // --- Firebase Init ---
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

        let allStudents = [], allExpenses = [], appSettings = { trainFee: 300, supplies: ['صورة'] }, currentStudentId = null;
        const arabicMonths = ["يناير", "فبراير", "مارس", "أبريل", "مايو", "يونيو", "يوليو", "غشت", "شتنبر", "أكتوبر", "نونبر", "دجنبر"];

        window.onload = function() {
            const today = new Date().toISOString().split('T')[0];
            if(document.getElementById('sRegDate')) document.getElementById('sRegDate').value = today;
            if(document.getElementById('exDate')) document.getElementById('exDate').value = today;

            db.collection("settings").doc("config").onSnapshot(doc => {
                if(doc.exists) { appSettings = doc.data(); window.updateSettingsUI(); }
                else db.collection("settings").doc("config").set(appSettings);
            });

            db.collection("students").onSnapshot(snap => {
                allStudents = snap.docs.map(d => ({id: d.id, ...d.data()}));
                document.getElementById('loading-overlay').style.display = 'none';
                window.renderStudents();
                window.updateDashboard();
                if(currentStudentId && document.getElementById('paymentModal').style.display === 'flex') window.openPayModal(currentStudentId);
                if(currentStudentId && document.getElementById('profileModal').style.display === 'flex') window.openProfile(currentStudentId);
            });

            db.collection("expenses").onSnapshot(snap => {
                allExpenses = snap.docs.map(d => ({id: d.id, ...d.data()}));
                window.renderExpenses();
                window.updateDashboard();
            });
        };

        // --- Functions ---
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
                    container.innerHTML += `<label class="supply-checkbox-item"><input type="checkbox" value="${s}"> ${s}</label>`;
                });
            } else { el.style.display = 'none'; }
        }

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
                const dueDate = new Date(d); dueDate.setMonth(d.getMonth() + i);
                payments.push({ type: 'train', label: `تدريب شهر ${i+1}`, amount: trainFee, paid: 0, due: dueDate.toISOString() });
            }

            if(equipFee > 0) {
                const monthlyEquip = equipFee / equipDuration;
                for(let i=0; i<equipDuration; i++) {
                    const dueDate = new Date(d); dueDate.setMonth(d.getMonth() + i);
                    payments.push({ type: 'equip', label: `معدات قسط ${i+1}/${equipDuration}`, amount: monthlyEquip, paid: 0, due: dueDate.toISOString() });
                }
            }

            if(initPay > 0) {
                for(let p of payments) {
                    if(initPay <= 0.1) break;
                    let rem = p.amount - p.paid;
                    let pay = Math.min(rem, initPay);
                    p.paid += pay;
                    initPay -= pay;
                }
            }

            db.collection("students").add({
                name, dob, phone, cnie, regDate, trainFee, duration, equipFee, equipDuration, supplies, payments
            }).then(() => {
                e.target.reset(); window.toggleForm('studentForm'); alert("تمت الإضافة");
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
                            <td><span class="clickable-name" onclick="window.openProfile('${s.id}')">${s.name}</span><br><small style="color:#777">${s.phone}</small></td>
                            <td><div style="font-size:0.7em">${paidCount}/${trainParams.length} أشهر</div>
                            <div style="height:6px; background:#eee; border-radius:3px; width:100%"><div style="height:100%; background:var(--success-color); border-radius:3px; width:${percent}%"></div></div></td>
                            <td>
                                <button class="btn-icon btn-blue" onclick="window.openPayModal('${s.id}')"><i class="fas fa-money-bill-wave"></i></button>
                                <button class="btn-icon btn-red" onclick="window.deleteStudent('${s.id}')"><i class="fas fa-trash"></i></button>
                            </td>
                        </tr>`;
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

            document.getElementById('profileContent').innerHTML = `
                <div style="text-align:center; margin-bottom:15px;"><div style="width:60px; height:60px; background:#eee; border-radius:50%; margin:0 auto; display:flex; align-items:center; justify-content:center; font-size:24px; color:var(--primary-color)">${s.name.charAt(0)}</div><h3 style="margin:5px 0">${s.name}</h3></div>
                <p><strong>الهاتف:</strong> ${s.phone}</p>
                <p><strong>السن:</strong> ${age} سنة</p>
                <div style="display:grid; grid-template-columns:1fr 1fr; gap:10px; margin:15px 0;">
                    <div style="background:#e8f6f3; padding:10px; border-radius:10px; text-align:center"><small>المدفوع</small><div style="color:var(--success-color); font-weight:bold">${Math.round(totalPaid)}</div></div>
                    <div style="background:#fcebeb; padding:10px; border-radius:10px; text-align:center"><small>المتبقي</small><div style="color:var(--danger-color); font-weight:bold">${Math.round(totalReq - totalPaid)}</div></div>
                </div>
            `;

            const suppliesListDiv = document.getElementById('profileSuppliesList');
            suppliesListDiv.innerHTML = '';
            if(s.supplies) {
                s.supplies.forEach((sup, index) => {
                    const itemDiv = document.createElement('div');
                    itemDiv.className = 'supply-checkbox-item';
                    itemDiv.style.background = sup.checked ? '#d4edda' : '#f8d7da';
                    itemDiv.innerHTML = `<span style="flex-grow:1">${sup.name}</span><input type="checkbox" ${sup.checked ? 'checked' : ''} onchange="window.toggleSupply('${s.id}', ${index}, this.checked)">`;
                    suppliesListDiv.appendChild(itemDiv);
                });
            }
            const m = document.getElementById('profileModal');
            m.style.display = 'flex';
            setTimeout(() => m.style.opacity = 1, 10);
        }

        window.toggleSupply = function(studentId, supplyIndex, isChecked) {
            const s = allStudents.find(x => x.id === studentId);
            s.supplies[supplyIndex].checked = isChecked;
            db.collection("students").doc(studentId).update({ supplies: s.supplies });
        }

        window.goToPayFromProfile = function() {
            window.closeModal('profileModal');
            window.openPayModal(currentStudentId);
        }

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

                let rowClass = '', statusText = '', actionBtn = '';

                if (remaining <= 0.5) { 
                    rowClass = 'background-color: #d4edda'; 
                    statusText = 'مدفوع';
                    actionBtn = `<i class="fas fa-check-circle" style="color:green; font-size:1.5rem; cursor:pointer" onclick="window.cancelPayment('${s.id}', ${idx})"></i>`;
                } else if (p.paid > 0) { 
                    rowClass = 'background-color: #fff3cd'; 
                    statusText = `باقي: ${Math.round(remaining)}`;
                    actionBtn = `<button class="btn-primary" style="background:#d35400; font-size:0.8rem; padding:5px" onclick="window.processPayment('${s.id}', ${idx})">الباقي: ${Math.round(remaining)}</button>`;
                } else {
                    statusText = 'غير مدفوع';
                    actionBtn = `<button class="btn-primary" style="font-size:0.8rem; padding:5px" onclick="window.processPayment('${s.id}', ${idx})">استخلاص</button>`;
                }

                tbody.innerHTML += `
                    <tr style="${rowClass}">
                        <td><b>${p.label}</b><br><small style="color:#555">${Math.round(p.paid)} / ${Math.round(p.amount)}</small></td>
                        <td style="font-size:0.8rem">${new Date(p.due).toLocaleDateString('ar-MA')}</td>
                        <td>${statusText}</td>
                        <td>${actionBtn}</td>
                    </tr>
                `;
            });
            const m = document.getElementById('paymentModal');
            m.style.display = 'flex';
            setTimeout(() => m.style.opacity = 1, 10);
        }

        window.processPayment = function(studentId, idx) {
            const s = allStudents.find(x => x.id === studentId);
            const p = s.payments[idx];
            const remaining = p.amount - p.paid;

            let input = prompt(`المبلغ المتبقي: ${Math.round(remaining)} درهم.\nأدخل المبلغ المدفوع الآن:`, Math.round(remaining));
            if(input !== null) {
                let amount = parseFloat(input);
                if(isNaN(amount) || amount < 0) return alert("مبلغ غير صحيح");
                if(amount > remaining) amount = remaining;
                p.paid += amount;
                db.collection("students").doc(studentId).update({ payments: s.payments });
            }
        }

        window.cancelPayment = function(studentId, idx) {
            if(confirm("إلغاء الدفع؟")) {
                const s = allStudents.find(x => x.id === studentId);
                s.payments[idx].paid = 0;
                db.collection("students").doc(studentId).update({ payments: s.payments });
            }
        }

        window.addExpense = function(e) {
            e.preventDefault();
            db.collection("expenses").add({
                desc: document.getElementById('exDesc').value,
                amount: parseFloat(document.getElementById('exAmount').value),
                date: document.getElementById('exDate').value
            }).then(() => e.target.reset());
        }

        window.renderExpenses = function() {
            const tbody = document.querySelector('#expensesTable tbody');
            tbody.innerHTML = '';
            allExpenses.sort((a,b) => new Date(b.date) - new Date(a.date)).forEach(ex => {
                tbody.innerHTML += `<tr><td>${ex.date}</td><td>${ex.desc}</td><td style="color:red">-${ex.amount}</td><td><button onclick="db.collection('expenses').doc('${ex.id}').delete()" style="background:none; border:none; color:red; cursor:pointer">x</button></td></tr>`;
            });
        }

        window.updateSettingsUI = function() {
            document.getElementById('defTrainFee').value = appSettings.trainFee;
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

        window.showBalanceDetails = function() {
            let inc = 0; allStudents.forEach(s => s.payments.forEach(p => inc += p.paid));
            let exp = 0; allExpenses.forEach(e => exp += e.amount);
            document.getElementById('modalTotalIncome').innerText = Math.round(inc) + ' درهم';
            document.getElementById('modalTotalExpenses').innerText = Math.round(exp) + ' درهم';
            document.getElementById('modalNetBalance').innerText = Math.round(inc - exp) + ' درهم';
            const m = document.getElementById('balanceModal');
            m.style.display = 'flex';
            setTimeout(() => m.style.opacity = 1, 10);
        }

        window.closeModal = function(id) {
            const m = document.getElementById(id);
            m.style.opacity = 0;
            setTimeout(() => m.style.display = 'none', 300);
        }

        window.updateDashboard = function() {
            document.getElementById('totalStudents').innerText = allStudents.length;
            let inc = 0; allStudents.forEach(s => s.payments.forEach(p => inc += p.paid));
            let exp = 0; allExpenses.forEach(e => exp += e.amount);
            document.getElementById('totalIncome').innerText = Math.round(inc);
            document.getElementById('netBalance').innerText = Math.round(inc - exp);

            const now = new Date();
            let currentMonthExp = 0;
            allExpenses.forEach(e => {
                const d = new Date(e.date);
                if(d.getMonth() === now.getMonth() && d.getFullYear() === now.getFullYear()) {
                    currentMonthExp += e.amount;
                }
            });
            document.getElementById('currentMonthExpenses').innerText = Math.round(currentMonthExp);

            const missingTbody = document.querySelector('#missingTable tbody');
            const overdueTbody = document.querySelector('#overdueTable tbody');
            const upcomingTbody = document.querySelector('#upcomingTable tbody');
            missingTbody.innerHTML = ''; overdueTbody.innerHTML = ''; upcomingTbody.innerHTML = '';

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
                        const row = `<tr class="clickable-row" onclick="window.openPayModal('${s.id}')"><td>${s.name}</td><td>${p.label}</td><td style="color:${diff<=0?'red':'orange'}">${Math.round(p.amount-p.paid)}</td></tr>`;
                        if(diff <= 0) overdueTbody.innerHTML += row;
                        else if(diff <= 3) upcomingTbody.innerHTML += row;
                    }
                });
            });
            
            if(!missingTbody.innerHTML) missingTbody.innerHTML = '<tr><td colspan="2" style="text-align:center">لا توجد نواقص</td></tr>';
            if(!overdueTbody.innerHTML) overdueTbody.innerHTML = '<tr><td colspan="3" style="text-align:center">لا توجد متأخرات</td></tr>';
            if(!upcomingTbody.innerHTML) upcomingTbody.innerHTML = '<tr><td colspan="3" style="text-align:center">لا توجد استحقاقات قريبة</td></tr>';

            window.renderDashboardReports();
        }

        window.renderDashboardReports = function() {
            const years = new Set();
            allStudents.forEach(s => s.payments.forEach(p => { if(p.paid > 0) years.add(new Date(p.due).getFullYear()); }));
            allExpenses.forEach(e => years.add(new Date(e.date).getFullYear()));
            years.add(new Date().getFullYear());

            const filter = document.getElementById('yearFilter');
            const currentYear = filter.value || new Date().getFullYear();
            filter.innerHTML = '';
            Array.from(years).sort().reverse().forEach(y => {
                filter.innerHTML += `<option value="${y}" ${y==currentYear?'selected':''}>${y}</option>`;
            });

            const container = document.getElementById('monthlyReportContainer');
            const stats = {};
            
            allStudents.forEach(s => s.payments.forEach(p => {
                if(p.paid > 0) {
                    const d = new Date(p.due);
                    if(d.getFullYear() == currentYear) {
                        const m = d.getMonth();
                        if(!stats[m]) stats[m] = {inc:0, exp:0, incomeDetails: [], expenseDetails: []};
                        stats[m].inc += p.paid;
                        stats[m].incomeDetails.push({ name: s.name, type: p.label, amount: p.paid });
                    }
                }
            }));

            allExpenses.forEach(e => {
                const d = new Date(e.date);
                if(d.getFullYear() == currentYear) {
                    const m = d.getMonth();
                    if(!stats[m]) stats[m] = {inc:0, exp:0, incomeDetails: [], expenseDetails: []};
                    stats[m].exp += e.amount;
                    stats[m].expenseDetails.push({ desc: e.desc, amount: e.amount, date: e.date });
                }
            });

            let html = '<ul class="report-list">';
            Object.keys(stats).sort((a,b)=>b-a).forEach(m => {
                const st = stats[m];
                const bal = st.inc - st.exp;
                let detailsHtml = '';
                if (st.incomeDetails.length > 0) {
                    detailsHtml += `<div class="detail-block"><div class="detail-title inc-title">تفاصيل المداخيل</div>`;
                    st.incomeDetails.forEach(d => { detailsHtml += `<div class="detail-row"><span class="detail-name">${d.name} (${d.type})</span><span class="detail-amount" style="color:var(--success-color)">+${Math.round(d.amount)}</span></div>`; });
                    detailsHtml += `</div>`;
                }
                if (st.expenseDetails.length > 0) {
                    detailsHtml += `<div class="detail-block"><div class="detail-title exp-title">تفاصيل المصاريف</div>`;
                    st.expenseDetails.forEach(d => { detailsHtml += `<div class="detail-row"><span class="detail-name">${d.desc} (${d.date})</span><span class="detail-amount" style="color:var(--danger-color)">-${Math.round(d.amount)}</span></div>`; });
                    detailsHtml += `</div>`;
                }
                html += `<li class="report-item"><div class="report-header" onclick="this.nextElementSibling.classList.toggle('show')"><span>${arabicMonths[m]}</span><span>${Math.round(bal)} dh</span></div><div class="report-details"><div class="report-row" style="background:#f9f9f9; padding:5px; font-weight:bold"><span>مجموع المداخيل: <span class="val-inc">${Math.round(st.inc)}</span></span><span>مجموع المصاريف: <span class="val-exp">${Math.round(st.exp)}</span></span></div><hr style="margin:10px 0; border:0; border-top:1px solid #eee">${detailsHtml}</div></li>`;
            });
            html += '</ul>';
            container.innerHTML = html;
        }
    </script>
</body>
</html>
