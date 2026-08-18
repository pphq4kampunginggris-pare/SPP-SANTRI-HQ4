<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Keuangan Pesantren Terintegrasi Supabase</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
    <!-- Supabase JS SDK -->
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    <!-- jsPDF for PDF Exports -->
    <script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspdf-autotable@3.5.28/dist/jspdf.plugin.autotable.min.js"></script>
    <style>
        body { 
            font-family: 'Inter', sans-serif; 
            text-rendering: optimizeLegibility; 
            -webkit-font-smoothing: antialiased; 
            -moz-osx-font-smoothing: grayscale;
        }
        ::-webkit-scrollbar {
            height: 4px;
            width: 4px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 9999px;
        }
    </style>
</head>
<body class="bg-slate-100 text-slate-900 font-medium antialiased min-h-screen flex flex-col selection:bg-emerald-500 selection:text-white">

    <div id="app" class="flex-1 flex flex-col">
    </div>

    <!-- Notification Modal -->
    <div id="modal-container" class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 hidden backdrop-blur-sm p-4 transition-all duration-300">
        <div class="bg-white rounded-3xl p-6 sm:p-8 max-w-md w-full mx-auto shadow-2xl transform transition-all scale-95 opacity-0 duration-300 border border-slate-100" id="modal-box">
            <div id="modal-icon" class="w-14 h-14 rounded-2xl flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner"></div>
            <h3 id="modal-title" class="text-xl font-extrabold text-center text-slate-900 mb-2"></h3>
            <p id="modal-message" class="text-sm font-semibold text-center text-slate-700 mb-6 leading-relaxed"></p>
            <div id="modal-actions" class="flex flex-col sm:flex-row gap-3 justify-center"></div>
        </div>
    </div>

    <script>
        const SUPABASE_URL = "https://ygsjtaputrcruxgqfcbb.supabase.co";
        const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Inlnc2p0YXB1dHJjcnV4Z3FmY2JiIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODU5NDI0MDAsImV4cCI6MjEwMTUxODQwMH0.UomgHIAYAkpGZj-B36NHwwhe0jF5BeJWjuJHObNTGuY";
        
        let supabaseClient = null;
        try {
            supabaseClient = supabase.createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
        } catch (e) {
            console.error("Gagal menginisialisasi Supabase client:", e);
        }

        const DEFAULT_STATE = {
            profile: {
                name: "Pesantren Darul Ulum Al-Islamy",
                foundation: "Yayasan Pendidikan Islam Darul Ulum",
                address: "Jl. Pesantren No. 45, Kediri, Jawa Timur",
                phone: "(0354) 555123",
                email: "info@darululum.sch.id",
                currentYear: "2025/2026",
                defaultSpp: 250000
            },
            credentials: {
                admin: { user: "admin", pass: "admin123", name: "Administrator Utama" },
                pesantren: { user: "admin_pesantren", pass: "santri123", name: "Admin Pesantren" },
                treasurer: { user: "bendahara", pass: "pusat123", name: "Bendahara Pusat" }
            },
            santri: [
                { id: "S001", name: "Ahmad Fauzi", class: "VII-A (Tsanawiyah)", customSpp: 250000, status: "Aktif", scholarship: "Tidak", phone: "081234567890" },
                { id: "S002", name: "Siti Aminah", class: "VIII-B (Tsanawiyah)", customSpp: 250000, status: "Aktif", scholarship: "Ya", phone: "081345678901" },
                { id: "S003", name: "Muhammad Alif", class: "X-IPA (Aliyah)", customSpp: 300000, status: "Aktif", scholarship: "Tidak", phone: "081567890123" }
            ],
            payments: [
                { id: "P001", santriId: "S001", santriName: "Ahmad Fauzi", type: "SPP", month: "Agustus 2025", amount: 250000, date: "2025-08-05", status: "Lunas" },
                { id: "P002", santriId: "S002", santriName: "Siti Aminah", type: "SPP", month: "Agustus 2025", amount: 0, date: "2025-08-06", status: "Beasiswa (Gratis)" },
                { id: "P003", santriId: "S003", santriName: "Muhammad Alif", type: "Daftar Ulang", month: "Tahun Ajaran Baru", amount: 750000, date: "2025-07-10", status: "Lunas" }
            ],
            transactions: [
                { id: "T001", date: "2025-08-06", type: "Pemasukan", category: "SPP Bulanan", amount: 250000, desc: "SPP Ahmad Fauzi (Agustus 2025)" },
                { id: "T002", date: "2025-07-10", type: "Pemasukan", category: "Daftar Ulang", amount: 750000, desc: "Daftar Ulang Muhammad Alif" },
                { id: "T003", date: "2025-08-01", type: "Pengeluaran", category: "Operasional", amount: 350000, desc: "Pembelian ATK dan Buku Administrasi" }
            ]
        };

        let dbState = DEFAULT_STATE;
        try {
            const saved = localStorage.getItem('pesantren_db');
            if (saved) {
                const parsed = JSON.parse(saved);
                if (parsed && parsed.credentials) {
                    dbState = parsed;
                }
            }
        } catch (err) {
            console.warn("Gagal membaca localStorage, menggunakan state default.");
        }

        let currentUser = null;
        let currentTab = 'dashboard';

        async function saveDb() {
            try {
                localStorage.setItem('pesantren_db', JSON.stringify(dbState));
                window.dispatchEvent(new Event('storage_updated'));
            } catch (err) {
                console.error("Gagal menyimpan ke localStorage:", err);
            }

            if (supabaseClient) {
                try {
                    await supabaseClient.from('pesantren_sync').upsert({ id: 1, payload: dbState });
                } catch (err) {
                    console.warn("Sinkronisasi cloud Supabase tertunda:", err);
                }
            }
        }

        async function fetchCloudData() {
            if (!supabaseClient) return;
            try {
                const { data, error } = await supabaseClient.from('pesantren_sync').select('payload').eq('id', 1).maybeSingle();
                if (!error && data && data.payload && data.payload.credentials) {
                    dbState = data.payload;
                    try {
                        localStorage.setItem('pesantren_db', JSON.stringify(dbState));
                    } catch (e) {}
                    if (currentUser) {
                        renderDashboard();
                    }
                }
            } catch (err) {
                console.log("Menggunakan cache lokal atau data default.");
            }
        }

        function initRealtimeUpdates() {
            window.addEventListener('storage', (e) => {
                if (e.key === 'pesantren_db' && e.newValue) {
                    try {
                        const parsed = JSON.parse(e.newValue);
                        if (parsed && parsed.credentials) {
                            dbState = parsed;
                            if (currentUser) {
                                renderDashboard();
                            }
                        }
                    } catch (err) {}
                }
            });

            window.addEventListener('storage_updated', () => {
                if (currentUser) {
                    renderDashboard();
                }
            });

            setInterval(async () => {
                if (supabaseClient) {
                    try {
                        const { data, error } = await supabaseClient.from('pesantren_sync').select('payload').eq('id', 1).maybeSingle();
                        if (!error && data && data.payload && data.payload.credentials) {
                            const stringifiedNew = JSON.stringify(data.payload);
                            const stringifiedCurrent = JSON.stringify(dbState);
                            if (stringifiedNew !== stringifiedCurrent) {
                                dbState = data.payload;
                                localStorage.setItem('pesantren_db', stringifiedNew);
                                if (currentUser) {
                                    renderDashboard();
                                }
                            }
                        }
                    } catch (e) {}
                }
            }, 5000);
        }

        function renderAuthPortal() {
            const app = document.getElementById('app');
            if (!app) return;
            
            app.innerHTML = `
                <div class="min-h-screen bg-gradient-to-br from-slate-900 via-emerald-950 to-slate-900 flex items-center justify-center p-4 sm:p-6">
                    <div class="max-w-md w-full bg-white backdrop-blur-2xl rounded-3xl shadow-2xl p-6 sm:p-8 border border-white/20">
                        <div class="text-center mb-6 sm:mb-8">
                            <div class="w-16 h-16 sm:w-20 sm:h-20 bg-gradient-to-tr from-emerald-600 to-teal-500 rounded-3xl mx-auto flex items-center justify-center text-white text-2xl sm:text-3xl shadow-xl shadow-emerald-500/30 mb-4 transform hover:scale-105 transition duration-300">
                                <i class="fa-solid fa-mosque"></i>
                            </div>
                            <h1 class="text-2xl sm:text-3xl font-black text-slate-900 tracking-tight">Keuangan Pesantren</h1>
                            <p class="text-xs font-bold text-slate-600 mt-1">${dbState.profile?.name || 'Pesantren Darul Ulum'}</p>
                            <div class="mt-3 inline-block bg-emerald-100 text-emerald-900 text-xs font-black px-3.5 py-1.5 rounded-full border border-emerald-300 shadow-xs">
                                <i class="fa-solid fa-cloud-arrow-up mr-1 text-emerald-700"></i> Supabase Cloud & Realtime Terintegrasi
                            </div>
                        </div>

                        <form onsubmit="handleLogin(event)" class="space-y-4 sm:space-y-5">
                            <div>
                                <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-2">Pilih Ruangan Akses</label>
                                <div class="relative">
                                    <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-slate-500 pointer-events-none"><i class="fa-solid fa-door-open"></i></span>
                                    <select id="role-select" onchange="onRoleChange()" class="w-full pl-11 pr-4 py-3.5 bg-slate-50 border border-slate-300 rounded-2xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition shadow-xs">
                                        <option value="admin">Administrator Utama</option>
                                        <option value="pesantren">Admin Pesantren</option>
                                        <option value="treasurer">Bendahara Pusat</option>
                                    </select>
                                </div>
                            </div>

                            <div>
                                <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-2">Kata Sandi Ruangan</label>
                                <div class="relative">
                                    <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-slate-500 pointer-events-none"><i class="fa-solid fa-lock"></i></span>
                                    <input type="password" id="role-pass" placeholder="Masukkan sandi..." required class="w-full pl-11 pr-4 py-3.5 bg-slate-50 border border-slate-300 rounded-2xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition shadow-xs">
                                </div>
                                
                            </div>

                            <div class="flex gap-2 pt-1">
                                <button type="submit" class="flex-1 py-4 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white font-black rounded-2xl shadow-xl shadow-emerald-600/30 transition transform active:scale-95 flex items-center justify-center gap-2 text-sm sm:text-base">
                                    <i class="fa-solid fa-right-to-bracket text-lg"></i> Masuk Ruangan
                                </button>
                                <button type="button" onclick="resetDefaultCredentials()" title="Reset sandi ke default" class="px-4 py-4 bg-slate-200 hover:bg-slate-300 text-slate-800 font-bold rounded-2xl transition active:scale-95 flex items-center justify-center">
                                    <i class="fa-solid fa-rotate-right"></i>
                                </button>
                            </div>
                        </form>
                    </div>
                </div>
            `;
        }

        function onRoleChange() {
            const roleEl = document.getElementById('role-select');
            const hint = document.getElementById('hint-pass');
            if (!roleEl || !hint) return;
            const role = roleEl.value;
            if (role === 'admin') hint.innerHTML = `<i class="fa-solid fa-circle-info text-emerald-700"></i> Sandi default: <span class="font-black text-emerald-900 bg-emerald-100 px-2 py-0.5 rounded-md border border-emerald-300">admin123</span>`;
            if (role === 'pesantren') hint.innerHTML = `<i class="fa-solid fa-circle-info text-emerald-700"></i> Sandi default: <span class="font-black text-emerald-900 bg-emerald-100 px-2 py-0.5 rounded-md border border-emerald-300">santri123</span>`;
            if (role === 'treasurer') hint.innerHTML = `<i class="fa-solid fa-circle-info text-emerald-700"></i> Sandi default: <span class="font-black text-emerald-900 bg-emerald-100 px-2 py-0.5 rounded-md border border-emerald-300">pusat123</span>`;
        }

        function resetDefaultCredentials() {
            dbState.credentials = DEFAULT_STATE.credentials;
            saveDb();
            showModal('Berhasil Reset', 'Sandi seluruh ruangan telah dikembalikan ke sandi default (admin123, santri123, pusat123).', 'success');
            renderAuthPortal();
        }

        function handleLogin(e) {
            e.preventDefault();
            const roleEl = document.getElementById('role-select');
            const passEl = document.getElementById('role-pass');
            if (!roleEl || !passEl) return;
            
            const role = roleEl.value;
            const pass = passEl.value;

            if (!dbState.credentials) {
                dbState.credentials = DEFAULT_STATE.credentials;
            }

            const creds = dbState.credentials[role] || DEFAULT_STATE.credentials[role];

            if (creds && creds.pass === pass) {
                currentUser = { role, name: creds.name };
                currentTab = 'dashboard';
                renderDashboard();
            } else {
                showModal('Akses Ditolak', 'Kata sandi ruangan yang Anda masukkan salah. Anda dapat menekan tombol putar (reset) di samping tombol masuk untuk mengembalikan sandi ke default.', 'error');
            }
        }

        function renderDashboard() {
            const app = document.getElementById('app');
            if (!app || !currentUser) return;
            const roleName = currentUser.name;

            let tabs = [];
        if (currentUser.role === 'admin') {
            tabs = [
                { id: 'dashboard', label: 'Beranda', icon: 'fa-house' },
                { id: 'profile', label: 'Profil Pesantren', icon: 'fa-school' },
                { id: 'cycles', label: 'Siklus Keuangan', icon: 'fa-chart-pie' },
                { id: 'credentials', label: 'Sandi Ruangan', icon: 'fa-key' },
                { id: 'reset_data', label: 'Kosongkan Data', icon: 'fa-trash-arrow-up' },
                { id: 'sql_setup', label: 'Setup SQL Supabase', icon: 'fa-database' }
            ];
        } else if (currentUser.role === 'pesantren') {
                tabs = [
                    { id: 'dashboard', label: 'Beranda', icon: 'fa-house' },
                    { id: 'santri', label: 'Data Santri', icon: 'fa-users' },
                    { id: 'spp_setting', label: 'Pengaturan SPP & Beasiswa', icon: 'fa-sliders' },
                    { id: 'payments', label: 'Catat Pembayaran', icon: 'fa-receipt' },
                    { id: 'arrears', label: 'Monitoring Tunggakan', icon: 'fa-triangle-exclamation' }
                ];
            } else if (currentUser.role === 'treasurer') {
                tabs = [
                    { id: 'dashboard', label: 'Beranda', icon: 'fa-house' },
                    { id: 'spp_monitor', label: 'Monitoring SPP', icon: 'fa-money-bill-wave' },
                    { id: 'transactions', label: 'Kas Masuk/Keluar', icon: 'fa-wallet' },
                    { id: 'reports', label: 'Santri & Beasiswa', icon: 'fa-user-graduate' }
                ];
            }

            app.innerHTML = `
                <!-- Top Navbar -->
                <header class="bg-white backdrop-blur-md border-b border-slate-300 sticky top-0 z-30 shadow-xs">
                    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 h-16 sm:h-20 flex items-center justify-between">
                        <div class="flex items-center gap-3 overflow-hidden">
                            <div class="w-10 h-10 sm:w-12 sm:h-12 bg-gradient-to-tr from-emerald-600 to-teal-500 rounded-2xl flex-shrink-0 flex items-center justify-center text-white text-base sm:text-xl shadow-md shadow-emerald-500/20">
                                <i class="fa-solid fa-mosque"></i>
                            </div>
                            <div class="min-w-0">
                                <h1 class="text-sm sm:text-base font-black text-slate-900 truncate">${dbState.profile?.name || ''}</h1>
                                <p class="text-[11px] sm:text-xs font-bold text-emerald-800 flex items-center gap-1.5 truncate"><i class="fa-solid fa-shield-halved"></i> ${roleName} <span class="text-slate-600 font-semibold">(Supabase Sinkron)</span></p>
                            </div>
                        </div>

                        <div class="flex items-center gap-2">
                            <button onclick="downloadPdfReport()" class="px-3.5 py-2 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs rounded-xl transition flex items-center gap-1.5 shadow-md active:scale-95 flex-shrink-0">
                                <i class="fa-solid fa-file-pdf"></i> <span class="hidden sm:inline">Download PDF Laporan</span>
                            </button>
                            <button onclick="logout()" class="px-3.5 py-2 bg-rose-100 hover:bg-rose-200 text-rose-900 font-bold text-xs rounded-xl transition flex items-center gap-1.5 border border-rose-300 flex-shrink-0 active:scale-95 shadow-xs">
                                <i class="fa-solid fa-arrow-right-from-bracket"></i> <span class="hidden sm:inline">Keluar</span>
                            </button>
                        </div>
                    </div>
                </header>

                <!-- Navigation Tabs -->
                <nav class="bg-slate-200 border-b border-slate-300 sticky top-16 sm:top-20 z-20 backdrop-blur-md">
                    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3">
                        <div class="grid grid-cols-2 sm:flex sm:flex-wrap gap-2">
                            ${tabs.map(t => `
                                <button onclick="switchTab('${t.id}')" class="px-3.5 py-2.5 rounded-2xl font-black text-xs sm:text-sm transition flex items-center justify-center sm:justify-start gap-2 shadow-xs ${currentTab === t.id ? 'bg-emerald-700 text-white shadow-lg shadow-emerald-700/30' : 'bg-white text-slate-900 hover:bg-slate-50 border border-slate-300'}">
                                    <i class="fa-solid ${t.icon}"></i> ${t.label}
                                </button>
                            `).join('')}
                        </div>
                    </div>
                </nav>

                <!-- Main Content Body -->
                <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-6 sm:py-8 flex-1 w-full space-y-6">
                    ${renderTabContent()}
                </main>
            `;
        }

        function switchTab(tabId) {
            currentTab = tabId;
            renderDashboard();
        }

        function logout() {
            currentUser = null;
            renderAuthPortal();
        }

        function renderTabContent() {
            const santriList = dbState.santri || [];
            const paymentList = dbState.payments || [];
            const txList = dbState.transactions || [];

            const totalSantri = santriList.length;
            const activeSantri = santriList.filter(s => s.status === 'Aktif').length;
            const scholarshipSantri = santriList.filter(s => s.scholarship === 'Ya').length;
            
            const totalIncome = txList.filter(t => t.type === 'Pemasukan').reduce((acc, curr) => acc + curr.amount, 0);
            const totalExpense = txList.filter(t => t.type === 'Pengeluaran').reduce((acc, curr) => acc + curr.amount, 0);
            const balance = totalIncome - totalExpense;

            if (currentTab === 'dashboard') {
                if (currentUser && currentUser.role === 'pesantren') {
                    const currentMonth = 'Agustus 2025';
                    const paidThisMonthSantriIds = paymentList.filter(p => p.type === 'SPP' && p.month === currentMonth).map(p => p.santriId);
                    
                    return `
                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 sm:gap-6">
                            <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm flex flex-col justify-between">
                                <div class="flex items-center justify-between">
                                    <span class="text-[11px] sm:text-xs font-black uppercase tracking-wider text-emerald-900 bg-emerald-100 px-2.5 py-1 rounded-full border border-emerald-300">Total Santri</span>
                                    <div class="w-10 h-10 sm:w-12 sm:h-12 rounded-2xl bg-emerald-600 text-white flex items-center justify-center text-base sm:text-lg shadow-md"><i class="fa-solid fa-users"></i></div>
                                </div>
                                <div class="mt-4">
                                    <div class="text-2xl sm:text-3xl font-black text-slate-900">${totalSantri} <span class="text-sm sm:text-base font-bold text-slate-700">Santri</span></div>
                                    <div class="text-xs font-bold text-emerald-900 mt-1 flex items-center gap-1"><i class="fa-solid fa-circle-check"></i> ${activeSantri} Aktif | ${scholarshipSantri} Beasiswa</div>
                                </div>
                            </div>

                            <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm flex flex-col justify-between">
                                <div class="flex items-center justify-between">
                                    <span class="text-[11px] sm:text-xs font-black uppercase tracking-wider text-teal-900 bg-teal-100 px-2.5 py-1 rounded-full border border-teal-300">Sudah Bayar SPP</span>
                                    <div class="w-10 h-10 sm:w-12 sm:h-12 rounded-2xl bg-teal-600 text-white flex items-center justify-center text-base sm:text-lg shadow-md"><i class="fa-solid fa-circle-check"></i></div>
                                </div>
                                <div class="mt-4">
                                    <div class="text-2xl sm:text-3xl font-black text-slate-900">${paidThisMonthSantriIds.length} <span class="text-sm sm:text-base font-bold text-slate-700">Santri</span></div>
                                    <div class="text-xs font-bold text-slate-700 mt-1">Periode ${currentMonth}</div>
                                </div>
                            </div>

                            <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm flex flex-col justify-between">
                                <div class="flex items-center justify-between">
                                    <span class="text-[11px] sm:text-xs font-black uppercase tracking-wider text-indigo-900 bg-indigo-100 px-2.5 py-1 rounded-full border border-indigo-300">Santri Beasiswa</span>
                                    <div class="w-10 h-10 sm:w-12 sm:h-12 rounded-2xl bg-indigo-600 text-white flex items-center justify-center text-base sm:text-lg shadow-md"><i class="fa-solid fa-user-graduate"></i></div>
                                </div>
                                <div class="mt-4">
                                    <div class="text-2xl sm:text-3xl font-black text-slate-900">${scholarshipSantri} <span class="text-sm sm:text-base font-bold text-slate-700">Santri</span></div>
                                    <div class="text-xs font-bold text-slate-700 mt-1">Bebas Biaya SPP</div>
                                </div>
                            </div>
                        </div>

                        <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm">
                            <h3 class="font-extrabold text-slate-900 mb-4 flex items-center gap-2 text-sm sm:text-base"><i class="fa-solid fa-clock-rotate-left text-emerald-600"></i> Aktivitas Pembayaran Terbaru</h3>
                            <div class="space-y-3">
                                ${paymentList.slice(0, 3).map(p => `
                                    <div class="flex items-center justify-between p-3.5 sm:p-4 bg-slate-50 rounded-2xl border border-slate-200 text-xs sm:text-sm shadow-xs">
                                        <div class="flex items-center gap-3">
                                            <div class="w-9 h-9 sm:w-10 sm:h-10 rounded-xl bg-emerald-600 text-white flex-shrink-0 flex items-center justify-center font-bold shadow-sm"><i class="fa-solid fa-receipt"></i></div>
                                            <div class="min-w-0">
                                                <div class="font-extrabold text-slate-900 truncate">${p.santriName} - ${p.type}</div>
                                                <div class="text-xs font-semibold text-slate-700 truncate">${p.month} • <span class="text-emerald-800 font-bold">${p.date}</span></div>
                                            </div>
                                        </div>
                                        <div class="font-black text-emerald-900 bg-emerald-100 px-3 py-1.5 rounded-xl border border-emerald-300 text-xs sm:text-sm flex-shrink-0">Rp ${p.amount.toLocaleString('id-ID')}</div>
                                    </div>
                                `).join('')}
                            </div>
                        </div>
                    `;
                }

                return `
                    <div class="grid grid-cols-2 lg:grid-cols-4 gap-3 sm:gap-4">
                        <div class="bg-white p-4 sm:p-6 rounded-3xl border border-slate-300 shadow-sm flex flex-col justify-between">
                            <div class="flex items-center justify-between">
                                <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-emerald-900 bg-emerald-100 px-2 py-0.5 sm:px-2.5 sm:py-1 rounded-full border border-emerald-300">Total Santri</span>
                                <div class="w-9 h-9 sm:w-12 sm:h-12 rounded-2xl bg-emerald-600 text-white flex items-center justify-center text-sm sm:text-lg shadow-md"><i class="fa-solid fa-users"></i></div>
                            </div>
                            <div class="mt-3 sm:mt-4">
                                <div class="text-xl sm:text-3xl font-black text-slate-900">${totalSantri} <span class="text-xs sm:text-base font-bold text-slate-700">Santri</span></div>
                                <div class="text-[10px] sm:text-xs font-bold text-emerald-900 mt-1 truncate"><i class="fa-solid fa-circle-check"></i> ${activeSantri} Aktif | ${scholarshipSantri} Beasiswa</div>
                            </div>
                        </div>

                        <div class="bg-white p-4 sm:p-6 rounded-3xl border border-slate-300 shadow-sm flex flex-col justify-between">
                            <div class="flex items-center justify-between">
                                <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-teal-900 bg-teal-100 px-2 py-0.5 sm:px-2.5 sm:py-1 rounded-full border border-teal-300">Total Pemasukan</span>
                                <div class="w-9 h-9 sm:w-12 sm:h-12 rounded-2xl bg-teal-600 text-white flex items-center justify-center text-sm sm:text-lg shadow-md"><i class="fa-solid fa-arrow-trend-up"></i></div>
                            </div>
                            <div class="mt-3 sm:mt-4">
                                <div class="text-lg sm:text-2xl font-black text-slate-900 truncate">Rp ${totalIncome.toLocaleString('id-ID')}</div>
                                <div class="text-[10px] sm:text-xs font-bold text-slate-700 mt-1">Akumulasi kas masuk</div>
                            </div>
                        </div>

                        <div class="bg-white p-4 sm:p-6 rounded-3xl border border-slate-300 shadow-sm flex flex-col justify-between">
                            <div class="flex items-center justify-between">
                                <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-rose-900 bg-rose-100 px-2 py-0.5 sm:px-2.5 sm:py-1 rounded-full border border-rose-300">Total Pengeluaran</span>
                                <div class="w-9 h-9 sm:w-12 sm:h-12 rounded-2xl bg-rose-600 text-white flex items-center justify-center text-sm sm:text-lg shadow-md"><i class="fa-solid fa-arrow-trend-down"></i></div>
                            </div>
                            <div class="mt-3 sm:mt-4">
                                <div class="text-lg sm:text-2xl font-black text-slate-900 truncate">Rp ${totalExpense.toLocaleString('id-ID')}</div>
                                <div class="text-[10px] sm:text-xs font-bold text-slate-700 mt-1">Akumulasi kas keluar</div>
                            </div>
                        </div>

                        <div class="bg-white p-4 sm:p-6 rounded-3xl border border-slate-300 shadow-sm flex flex-col justify-between">
                            <div class="flex items-center justify-between">
                                <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-indigo-900 bg-indigo-100 px-2 py-0.5 sm:px-2.5 sm:py-1 rounded-full border border-indigo-300">Saldo Kas Bersih</span>
                                <div class="w-9 h-9 sm:w-12 sm:h-12 rounded-2xl bg-indigo-600 text-white flex items-center justify-center text-sm sm:text-lg shadow-md"><i class="fa-solid fa-wallet"></i></div>
                            </div>
                            <div class="mt-3 sm:mt-4">
                                <div class="text-lg sm:text-2xl font-black text-slate-900 truncate">Rp ${balance.toLocaleString('id-ID')}</div>
                                <div class="text-[10px] sm:text-xs font-bold text-slate-700 mt-1">Posisi Kas Terkini</div>
                            </div>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'sql_setup') {
                const sqlScript = `
-- ==========================================
-- SKRIP SQL LENGKAP SUPABASE
-- APLIKASI KEUANGAN PESANTREN TERINTEGRASI
-- ==========================================

-- 1. Buat Tabel Utama Sinkronisasi
CREATE TABLE IF NOT EXISTS pesantren_sync (
    id INT PRIMARY KEY,
    payload JSONB NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 2. Aktifkan Row Level Security (RLS)
ALTER TABLE pesantren_sync ENABLE ROW LEVEL SECURITY;

-- 3. Berikan Akses Publik untuk Anon Role (Select, Insert, Update)
DROP POLICY IF EXISTS "Akses publik pesantren_sync" ON pesantren_sync;
CREATE POLICY "Akses publik pesantren_sync" 
ON pesantren_sync 
FOR ALL 
USING (true) 
WITH CHECK (true);
                `.trim();

                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm max-w-4xl mx-auto">
                        <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 mb-4">
                            <div>
                                <h3 class="font-extrabold text-slate-900 text-base sm:text-lg flex items-center gap-2"><i class="fa-solid fa-database text-emerald-600"></i> Setup Skrip SQL Lengkap Supabase</h3>
                                <p class="text-xs sm:text-sm font-semibold text-slate-700">Salin skrip SQL di bawah ini dan jalankan pada <strong class="text-slate-900">Supabase SQL Editor</strong> Anda.</p>
                            </div>
                            <button onclick="copySqlScript()" class="px-4 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs rounded-xl shadow-md transition flex items-center gap-2 active:scale-95 flex-shrink-0">
                                <i class="fa-solid fa-copy"></i> Salin Skrip SQL
                            </button>
                        </div>
                        <div class="relative">
                            <textarea id="sql-textarea" rows="12" readonly class="w-full p-4 bg-slate-900 text-emerald-300 font-mono text-xs rounded-2xl border border-slate-800 focus:outline-none">${sqlScript}</textarea>
                        </div>
                        <div class="mt-4 p-4 rounded-2xl bg-emerald-50 border border-emerald-300 text-xs font-bold text-emerald-900 flex items-center gap-2">
                            <i class="fa-solid fa-circle-check text-emerald-700 text-base"></i>
                            <span>Supabase URL Terhubung: <strong class="text-emerald-950">${SUPABASE_URL}</strong></span>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'credentials') {
                return `
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                        <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-200 shadow-sm">
                            <h3 class="font-bold text-slate-900 text-base sm:text-lg mb-2 flex items-center gap-2"><i class="fa-solid fa-key text-amber-600"></i> Kelola Sandi Ruangan</h3>
                            <p class="text-xs sm:text-sm text-slate-500 mb-6">Ubah kata sandi untuk masing-masing ruangan akses sistem.</p>

                            <form onsubmit="updateCredentials(event)" class="space-y-4">
                                <div>
                                    <label class="block text-xs font-extrabold uppercase tracking-wider text-slate-800 mb-1.5">Sandi Administrator Utama</label>
                                    <input type="text" id="pass-admin" value="${dbState.credentials?.admin?.pass || 'admin123'}" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">
                                </div>
                                <div>
                                    <label class="block text-xs font-extrabold uppercase tracking-wider text-slate-800 mb-1.5">Sandi Admin Pesantren</label>
                                    <input type="text" id="pass-pesantren" value="${dbState.credentials?.pesantren?.pass || 'santri123'}" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">
                                </div>
                                <div>
                                    <label class="block text-xs font-extrabold uppercase tracking-wider text-slate-800 mb-1.5">Sandi Bendahara Pusat</label>
                                    <input type="text" id="pass-treasurer" value="${dbState.credentials?.treasurer?.pass || 'pusat123'}" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">
                                </div>
                                <button type="submit" class="w-full py-3.5 bg-amber-600 hover:bg-amber-700 text-white font-bold rounded-xl shadow-lg shadow-amber-600/30 transition flex items-center justify-center gap-2 mt-2 text-sm sm:text-base active:scale-95">
                                    <i class="fa-solid fa-floppy-disk"></i> Simpan Perubahan Sandi
                                </button>
                            </form>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'reset_data') {
                return `
                    <div class="bg-white p-6 sm:p-8 rounded-3xl border border-rose-200 shadow-sm max-w-xl mx-auto text-center">
                        <div class="w-16 h-16 bg-rose-100 text-rose-600 rounded-3xl flex items-center justify-center text-2xl mx-auto mb-4 shadow-inner">
                            <i class="fa-solid fa-triangle-exclamation"></i>
                        </div>
                        <h3 class="text-lg sm:text-xl font-black text-slate-900 mb-2">Kosongkan Data & Mulai dari Awal</h3>
                        <p class="text-xs sm:text-sm font-medium text-slate-600 mb-6 leading-relaxed">
                            Fitur ini akan menghapus seluruh data santri, riwayat pembayaran SPP, dan catatan transaksi kas keuangan. Profil pesantren dan sandi ruangan <strong>tidak akan berubah</strong>.
                        </p>
                        <button onclick="confirmResetAllData()" class="w-full py-4 bg-rose-600 hover:bg-rose-700 text-white font-black rounded-2xl shadow-xl shadow-rose-600/30 transition transform active:scale-95 flex items-center justify-center gap-2 text-sm sm:text-base">
                            <i class="fa-solid fa-trash-can"></i> Kosongkan Seluruh Data Sekarang
                        </button>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'profile') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm max-w-2xl mx-auto">
                        <h3 class="font-extrabold text-slate-900 text-base sm:text-lg mb-2 flex items-center gap-2"><i class="fa-solid fa-school text-emerald-600"></i> Pengaturan Profil Pesantren & Yayasan</h3>
                        <p class="text-xs sm:text-sm font-semibold text-slate-700 mb-6">Perbarui informasi nama pesantren, yayasan, alamat, dan nomor kontak resmi.</p>

                        <form onsubmit="updateProfile(event)" class="space-y-4">
                            <div>
                                <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-1.5">Nama Pesantren</label>
                                <input type="text" id="prof-name" value="${dbState.profile?.name || ''}" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">
                            </div>
                            <div>
                                <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-1.5">Nama Yayasan</label>
                                <input type="text" id="prof-foundation" value="${dbState.profile?.foundation || ''}" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">
                            </div>
                            <div>
                                <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-1.5">Alamat Lengkap</label>
                                <textarea id="prof-address" rows="3" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">${dbState.profile?.address || ''}</textarea>
                            </div>
                            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                                <div>
                                    <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-1.5">Nomor Telepon / HP</label>
                                    <input type="text" id="prof-phone" value="${dbState.profile?.phone || ''}" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">
                                </div>
                                <div>
                                    <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-1.5">Tahun Ajaran Aktif</label>
                                    <input type="text" id="prof-year" value="${dbState.profile?.currentYear || ''}" required class="w-full px-4 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500 focus:bg-white transition">
                                </div>
                            </div>
                            <button type="submit" class="w-full py-3.5 bg-emerald-600 hover:bg-emerald-700 text-white font-black rounded-xl shadow-lg shadow-emerald-600/30 transition flex items-center justify-center gap-2 mt-4 text-sm sm:text-base active:scale-95">
                                <i class="fa-solid fa-floppy-disk"></i> Simpan Profil Pesantren
                            </button>
                        </form>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'cycles') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-extrabold text-slate-900 text-base sm:text-lg">Siklus & Rekapitulasi Keuangan Pesantren</h3>
                                <p class="text-xs sm:text-sm font-semibold text-slate-700">Analisis sirkulasi kas masuk dan keluar (Terbaru di atas).</p>
                            </div>
                            <button onclick="downloadPdfReport()" class="px-4 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs rounded-xl shadow-md transition flex items-center gap-2 active:scale-95">
                                <i class="fa-solid fa-file-pdf"></i> Unduh Laporan PDF
                            </button>
                        </div>

                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 sm:gap-6 mb-8">
                            <div class="p-4 sm:p-5 bg-emerald-50 rounded-3xl border border-emerald-300 shadow-sm">
                                <div class="text-xs font-black uppercase text-emerald-900 mb-1">Total Pemasukan</div>
                                <div class="text-xl sm:text-2xl font-black text-emerald-950">Rp ${totalIncome.toLocaleString('id-ID')}</div>
                            </div>
                            <div class="p-4 sm:p-5 bg-rose-50 rounded-3xl border border-rose-300 shadow-sm">
                                <div class="text-xs font-black uppercase text-rose-900 mb-1">Total Pengeluaran</div>
                                <div class="text-xl sm:text-2xl font-black text-rose-950">Rp ${totalExpense.toLocaleString('id-ID')}</div>
                            </div>
                            <div class="p-4 sm:p-5 bg-indigo-50 rounded-3xl border border-indigo-300 shadow-sm">
                                <div class="text-xs font-black uppercase text-indigo-900 mb-1">Arus Kas Bersih</div>
                                <div class="text-xl sm:text-2xl font-black text-indigo-950">Rp ${balance.toLocaleString('id-ID')}</div>
                            </div>
                        </div>

                        <h4 class="font-extrabold text-slate-900 mb-4 flex items-center gap-2 text-sm sm:text-base"><i class="fa-solid fa-list-check text-emerald-600"></i> Riwayat Seluruh Transaksi Siklus Keuangan</h4>
                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm">
                                <thead class="bg-slate-200 text-slate-900 uppercase text-[11px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 sm:p-3.5 rounded-l-2xl">Tanggal</th>
                                        <th class="p-3 sm:p-3.5">Jenis</th>
                                        <th class="p-3 sm:p-3.5">Kategori</th>
                                        <th class="p-3 sm:p-3.5">Keterangan</th>
                                        <th class="p-3 sm:p-3.5 rounded-r-2xl text-right">Nominal</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y divide-slate-200">
                                    ${txList.map(t => `
                                        <tr class="hover:bg-slate-50 transition">
                                            <td class="p-3 sm:p-3.5 text-slate-800 font-bold whitespace-nowrap">${t.date}</td>
                                            <td class="p-3 sm:p-3.5 whitespace-nowrap"><span class="px-2.5 py-1 rounded-full text-[10px] sm:text-xs font-black ${t.type === 'Pemasukan' ? 'bg-emerald-100 text-emerald-950 border border-emerald-300' : 'bg-rose-100 text-rose-950 border border-rose-300'}">${t.type}</span></td>
                                            <td class="p-3 sm:p-3.5 font-bold text-slate-900 whitespace-nowrap">${t.category}</td>
                                            <td class="p-3 sm:p-3.5 text-slate-800 font-medium">${t.desc}</td>
                                            <td class="p-3 sm:p-3.5 text-right font-black whitespace-nowrap ${t.type === 'Pemasukan' ? 'text-emerald-800' : 'text-rose-800'}">Rp ${t.amount.toLocaleString('id-ID')}</td>
                                        </tr>
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'pesantren' && currentTab === 'santri') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-extrabold text-slate-900 text-base sm:text-lg">Manajemen Data Santri Pesantren</h3>
                                <p class="text-xs sm:text-sm font-semibold text-slate-700">Tambah data santri baru, atur status, dan beasiswa.</p>
                            </div>
                            <div class="flex items-center gap-2">
                                <button onclick="downloadPdfReport()" class="px-4 py-3 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs sm:text-sm rounded-xl shadow-md transition flex items-center gap-2 active:scale-95">
                                    <i class="fa-solid fa-file-pdf"></i> PDF Santri
                                </button>
                                <button onclick="openAddSantriModal()" class="px-4 py-3 bg-teal-600 hover:bg-teal-700 text-white font-bold text-xs sm:text-sm rounded-xl shadow-md shadow-teal-600/30 transition flex items-center justify-center gap-2 active:scale-95">
                                    <i class="fa-solid fa-user-plus"></i> Tambah Santri
                                </button>
                            </div>
                        </div>

                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm">
                                <thead class="bg-slate-200 text-slate-900 uppercase text-[11px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 sm:p-3.5 rounded-l-2xl">ID & Nama Santri</th>
                                        <th class="p-3 sm:p-3.5">Kelas</th>
                                        <th class="p-3 sm:p-3.5">Nominal SPP</th>
                                        <th class="p-3 sm:p-3.5">Status Beasiswa</th>
                                        <th class="p-3 sm:p-3.5 rounded-r-2xl text-center">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y divide-slate-200">
                                    ${santriList.map(s => `
                                        <tr class="hover:bg-slate-50 transition">
                                            <td class="p-3 sm:p-3.5">
                                                <div class="font-black text-slate-900">${s.name}</div>
                                                <div class="text-xs font-bold text-slate-600">ID: ${s.id} | Telp: ${s.phone}</div>
                                            </td>
                                            <td class="p-3 sm:p-3.5 text-slate-900 font-bold whitespace-nowrap">${s.class}</td>
                                            <td class="p-3 sm:p-3.5 text-emerald-900 font-black whitespace-nowrap">Rp ${(s.customSpp !== undefined ? s.customSpp : (dbState.profile?.defaultSpp || 250000)).toLocaleString('id-ID')}</td>
                                            <td class="p-3 sm:p-3.5 whitespace-nowrap">
                                                <span class="px-2.5 py-1 rounded-full text-[10px] sm:text-xs font-black ${s.scholarship === 'Ya' ? 'bg-purple-100 text-purple-950 border border-purple-300' : 'bg-slate-200 text-slate-950 border border-slate-300'}">
                                                    ${s.scholarship === 'Ya' ? 'Beasiswa (Gratis)' : 'Reguler'}
                                                </span>
                                            </td>
                                            <td class="p-3 sm:p-3.5 text-center whitespace-nowrap">
                                                <button onclick="deleteSantri('${s.id}')" class="px-3 py-1.5 bg-rose-100 hover:bg-rose-200 text-rose-950 text-xs font-bold rounded-xl transition active:scale-95 border border-rose-300">
                                                    <i class="fa-solid fa-trash-can mr-1"></i> Hapus
                                                </button>
                                            </td>
                                        </tr>
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'pesantren' && currentTab === 'spp_setting') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm max-w-3xl mx-auto">
                        <h3 class="font-extrabold text-slate-900 text-base sm:text-lg mb-2 flex items-center gap-2"><i class="fa-solid fa-sliders text-emerald-600"></i> Pengaturan Nominal SPP & Status Beasiswa Santri</h3>
                        <p class="text-xs sm:text-sm font-semibold text-slate-700 mb-6">Atur nominal SPP bulanan dan ubah status beasiswa santri kapan saja.</p>

                        <div class="space-y-4">
                            ${santriList.map(s => `
                                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 p-4 bg-slate-50 rounded-2xl border border-slate-300">
                                    <div>
                                        <div class="font-black text-slate-900 text-sm sm:text-base">${s.name}</div>
                                        <div class="text-xs font-semibold text-slate-700">${s.class} | Telp: ${s.phone}</div>
                                    </div>
                                    <div class="flex flex-wrap items-center gap-2.5">
                                        <div class="flex items-center gap-1.5 flex-1 sm:flex-none">
                                            <select id="spp-scholarship-${s.id}" class="w-full sm:w-auto px-3 py-2 bg-white border border-slate-300 rounded-xl text-xs font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                                                <option value="Tidak" ${s.scholarship === 'Tidak' ? 'selected' : ''}>Reguler</option>
                                                <option value="Ya" ${s.scholarship === 'Ya' ? 'selected' : ''}>Beasiswa</option>
                                            </select>
                                        </div>
                                        <div class="flex items-center gap-1 flex-1 sm:flex-none">
                                            <span class="text-xs font-bold text-slate-600">Rp</span>
                                            <input type="number" id="spp-santri-${s.id}" value="${s.customSpp !== undefined ? s.customSpp : (dbState.profile?.defaultSpp || 250000)}" class="w-full sm:w-32 px-3 py-2 bg-white border border-slate-300 rounded-xl text-xs sm:text-sm font-black text-emerald-900 focus:ring-2 focus:ring-emerald-500">
                                        </div>
                                        <button onclick="saveSantriSppAndScholarship('${s.id}')" class="w-full sm:w-auto px-4 py-2 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-bold rounded-xl transition flex items-center justify-center gap-1 active:scale-95">
                                            <i class="fa-solid fa-floppy-disk"></i> Simpan
                                        </button>
                                    </div>
                                </div>
                            `).join('')}
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'pesantren' && currentTab === 'payments') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-extrabold text-slate-900 text-base sm:text-lg">Pencatatan Pembayaran & Rekam Jejak SPP</h3>
                                <p class="text-xs sm:text-sm font-semibold text-slate-700">Catat pembayaran Daftar Ulang & SPP santri.</p>
                            </div>
                            <div class="flex items-center gap-2">
                                <button onclick="downloadPdfReport()" class="px-4 py-3 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs sm:text-sm rounded-xl shadow-md transition flex items-center gap-2 active:scale-95">
                                    <i class="fa-solid fa-file-pdf"></i> Unduh PDF Pembayaran
                                </button>
                                <button onclick="openPaymentModal()" class="px-4 py-3 bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700 text-white font-bold text-xs sm:text-sm rounded-xl shadow-md shadow-emerald-600/30 transition flex items-center justify-center gap-2 active:scale-95">
                                    <i class="fa-solid fa-receipt"></i> Catat Pembayaran Baru
                                </button>
                            </div>
                        </div>

                        <div class="mb-8">
                            <h4 class="font-extrabold text-slate-900 mb-3 text-xs sm:text-sm flex items-center gap-2"><i class="fa-solid fa-clock-rotate-left text-emerald-600"></i> Riwayat Pembayaran Terbaru</h4>
                            <div class="overflow-x-auto">
                                <table class="w-full text-left text-xs sm:text-sm">
                                    <thead class="bg-slate-200 text-slate-900 uppercase text-[11px] sm:text-xs font-black tracking-wider">
                                        <tr>
                                            <th class="p-3 sm:p-3.5 rounded-l-2xl">Tanggal</th>
                                            <th class="p-3 sm:p-3.5">Nama Santri</th>
                                            <th class="p-3 sm:p-3.5">Jenis Pembayaran</th>
                                            <th class="p-3 sm:p-3.5">Periode / Bulan</th>
                                            <th class="p-3 sm:p-3.5">Status</th>
                                            <th class="p-3 sm:p-3.5 rounded-r-2xl text-right">Nominal</th>
                                        </tr>
                                    </thead>
                                    <tbody class="divide-y divide-slate-200">
                                        ${paymentList.map(p => `
                                            <tr class="hover:bg-slate-50 transition">
                                                <td class="p-3 sm:p-3.5 text-slate-800 font-bold whitespace-nowrap">${p.date}</td>
                                                <td class="p-3 sm:p-3.5 font-black text-slate-900 whitespace-nowrap">${p.santriName}</td>
                                                <td class="p-3 sm:p-3.5 whitespace-nowrap"><span class="px-2.5 py-1 bg-teal-100 text-teal-950 rounded-full text-[10px] sm:text-xs font-black border border-teal-300">${p.type}</span></td>
                                                <td class="p-3 sm:p-3.5 text-slate-800 font-bold whitespace-nowrap">${p.month}</td>
                                                <td class="p-3 sm:p-3.5 whitespace-nowrap"><span class="px-2.5 py-1 bg-emerald-100 text-emerald-950 rounded-full text-[10px] sm:text-xs font-black border border-emerald-300">${p.status}</span></td>
                                                <td class="p-3 sm:p-3.5 text-right font-black text-emerald-900 whitespace-nowrap">Rp ${p.amount.toLocaleString('id-ID')}</td>
                                            </tr>
                                        `).join('')}
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'pesantren' && currentTab === 'arrears') {
                const currentMonth = 'Agustus 2025';
                const paidThisMonthSantriIds = paymentList.filter(p => p.type === 'SPP' && p.month === currentMonth).map(p => p.santriId);
                const regularSantri = santriList.filter(s => s.scholarship !== 'Ya' && s.status === 'Aktif');
                const unpaidSantri = regularSantri.filter(s => !paidThisMonthSantriIds.includes(s.id));

                return `
                    <div class="space-y-6">
                        <div class="flex justify-end">
                            <button onclick="downloadPdfReport()" class="px-4 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs rounded-xl shadow-md transition flex items-center gap-2 active:scale-95">
                                <i class="fa-solid fa-file-pdf"></i> Unduh Laporan PDF Tunggakan
                            </button>
                        </div>
                        <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 sm:gap-6">
                            <div class="bg-emerald-50 p-5 sm:p-6 rounded-3xl border border-emerald-300 shadow-sm">
                                <div class="flex items-center justify-between mb-2">
                                    <span class="text-[11px] sm:text-xs font-black uppercase text-emerald-950 bg-emerald-200 px-2.5 py-1 rounded-full">Sudah Bayar Bulan Ini</span>
                                    <i class="fa-solid fa-circle-check text-emerald-700 text-lg sm:text-xl"></i>
                                </div>
                                <div class="text-2xl sm:text-3xl font-black text-emerald-950">${paidThisMonthSantriIds.length} Santri</div>
                                <p class="text-xs font-bold text-emerald-900 mt-1">Pembayaran SPP ${currentMonth} telah dilunasi.</p>
                            </div>

                            <div class="bg-rose-50 p-5 sm:p-6 rounded-3xl border border-rose-300 shadow-sm">
                                <div class="flex items-center justify-between mb-2">
                                    <span class="text-[11px] sm:text-xs font-black uppercase text-rose-950 bg-rose-200 px-2.5 py-1 rounded-full">Belum Bayar / Tunggakan</span>
                                    <i class="fa-solid fa-triangle-exclamation text-rose-700 text-lg sm:text-xl"></i>
                                </div>
                                <div class="text-2xl sm:text-3xl font-black text-rose-950">${unpaidSantri.length} Santri</div>
                                <p class="text-xs font-bold text-rose-900 mt-1">Belum melakukan setoran SPP untuk periode ${currentMonth}.</p>
                            </div>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'spp_monitor') {
                const sppPayments = paymentList.filter(p => p.type === 'SPP');
                const totalSpp = sppPayments.reduce((acc, curr) => acc + curr.amount, 0);

                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-extrabold text-slate-900 text-base sm:text-lg">Monitoring SPP Masuk</h3>
                                <p class="text-xs sm:text-sm font-semibold text-slate-700">Pemantauan setoran SPP bulanan seluruh santri.</p>
                            </div>
                            <div class="flex items-center gap-3">
                                <button onclick="downloadPdfReport()" class="px-4 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs rounded-xl shadow-md transition flex items-center gap-2 active:scale-95">
                                    <i class="fa-solid fa-file-pdf"></i> Unduh PDF SPP
                                </button>
                                <div class="p-3.5 bg-emerald-50 rounded-2xl border border-emerald-300 text-left sm:text-right shadow-xs">
                                    <div class="text-[10px] sm:text-xs font-black uppercase text-emerald-950">Total SPP Masuk</div>
                                    <div class="text-lg sm:text-xl font-black text-emerald-950">Rp ${totalSpp.toLocaleString('id-ID')}</div>
                                </div>
                            </div>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'transactions') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm">
                        <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between gap-6 mb-6">
                            <div>
                                <h3 class="font-extrabold text-slate-900 text-base sm:text-lg">Pencatatan Pemasukan & Pengeluaran Kas</h3>
                                <p class="text-xs sm:text-sm font-semibold text-slate-700">Kelola arus kas masuk dan pengeluaran operasional.</p>
                            </div>

                            <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                                <div class="bg-emerald-50 p-3 px-4 rounded-2xl border border-emerald-300 shadow-xs flex flex-col justify-center">
                                    <span class="text-[10px] font-black uppercase tracking-wider text-emerald-950">Seluruh Pemasukan</span>
                                    <span class="text-sm sm:text-base font-black text-emerald-950">Rp ${totalIncome.toLocaleString('id-ID')}</span>
                                </div>
                                <div class="bg-rose-50 p-3 px-4 rounded-2xl border border-rose-300 shadow-xs flex flex-col justify-center">
                                    <span class="text-[10px] font-black uppercase tracking-wider text-rose-950">Seluruh Pengeluaran</span>
                                    <span class="text-sm sm:text-base font-black text-rose-950">Rp ${totalExpense.toLocaleString('id-ID')}</span>
                                </div>
                                <div class="bg-indigo-50 p-3 px-4 rounded-2xl border border-indigo-300 shadow-xs flex flex-col justify-center">
                                    <span class="text-[10px] font-black uppercase tracking-wider text-indigo-950">Saldo Kas Bersih</span>
                                    <span class="text-sm sm:text-base font-black text-indigo-950">Rp ${balance.toLocaleString('id-ID')}</span>
                                </div>
                            </div>
                        </div>

                        <div class="flex items-center justify-between mb-4">
                            <div class="flex items-center gap-2">
                                <button onclick="downloadPdfReport()" class="px-4 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs sm:text-sm rounded-xl shadow-md transition flex items-center gap-2 active:scale-95">
                                    <i class="fa-solid fa-file-pdf"></i> PDF Kas Keuangan
                                </button>
                                <button onclick="openTransactionModal()" class="px-4 py-2.5 bg-teal-600 hover:bg-teal-700 text-white font-bold text-xs sm:text-sm rounded-xl shadow-md shadow-teal-600/30 transition flex items-center gap-2 active:scale-95">
                                    <i class="fa-solid fa-plus-circle"></i> Tambah Transaksi Kas
                                </button>
                            </div>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'reports') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border border-slate-300 shadow-sm">
                        <div class="flex justify-between items-center mb-4">
                            <div>
                                <h3 class="font-extrabold text-slate-900 text-base sm:text-lg mb-1">Data Santri Aktif & Penerima Beasiswa</h3>
                                <p class="text-xs sm:text-sm font-semibold text-slate-700">Daftar lengkap santri untuk keperluan audit.</p>
                            </div>
                            <button onclick="downloadPdfReport()" class="px-4 py-2.5 bg-emerald-600 hover:bg-emerald-700 text-white font-bold text-xs rounded-xl shadow-md transition flex items-center gap-2 active:scale-95">
                                <i class="fa-solid fa-file-pdf"></i> Unduh Laporan PDF
                            </button>
                        </div>
                    </div>
                `;
            }

            return '';
        }

        // PDF Export Function using jsPDF
        function downloadPdfReport() {
            try {
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF();
                
                const profile = dbState.profile || {};
                const santriList = dbState.santri || [];
                const txList = dbState.transactions || [];
                const paymentList = dbState.payments || [];

                // Header
                doc.setFont("inter", "bold");
                doc.setFontSize(16);
                doc.text(profile.name || "Pesantren Darul Ulum", 14, 20);
                
                doc.setFontSize(10);
                doc.setFont("inter", "normal");
                doc.text(`Yayasan: ${profile.foundation || '-'}`, 14, 27);
                doc.text(`Alamat: ${profile.address || '-'} | Telp: ${profile.phone || '-'}`, 14, 33);
                doc.text(`Tahun Ajaran: ${profile.currentYear || '-'}`, 14, 39);

                doc.setLineWidth(0.5);
                doc.line(14, 44, 196, 44);

                // Section 1: Data Santri
                doc.setFont("inter", "bold");
                doc.setFontSize(12);
                doc.text("DAFTAR SANTRI DAN STATUS BEASISWA", 14, 52);

                const santriRows = santriList.map(s => [
                    s.id,
                    s.name,
                    s.class,
                    `Rp ${(s.customSpp !== undefined ? s.customSpp : 250000).toLocaleString('id-ID')}`,
                    s.scholarship === 'Ya' ? 'Beasiswa (Gratis)' : 'Reguler',
                    s.phone
                ]);

                doc.autoTable({
                    startY: 56,
                    head: [['ID', 'Nama Lengkap', 'Kelas', 'Nominal SPP', 'Status Beasiswa', 'Telepon']],
                    body: santriRows,
                    theme: 'grid',
                    headStyles: { fillColor: [16, 138, 95] },
                    styles: { fontSize: 9, font: 'inter' }
                });

                // Section 2: Transaksi Keuangan
                let finalY = doc.lastAutoTable.finalY + 15;
                if (finalY > 250) {
                    doc.addPage();
                    finalY = 20;
                }

                doc.setFont("inter", "bold");
                doc.setFontSize(12);
                doc.text("REKAPITULASI ARUS KAS KEUANGAN", 14, finalY);

                const txRows = txList.map(t => [
                    t.date,
                    t.type,
                    t.category,
                    t.desc,
                    `Rp ${t.amount.toLocaleString('id-ID')}`
                ]);

                doc.autoTable({
                    startY: finalY + 4,
                    head: [['Tanggal', 'Jenis', 'Kategori', 'Keterangan', 'Nominal']],
                    body: txRows,
                    theme: 'grid',
                    headStyles: { fillColor: [16, 138, 95] },
                    styles: { fontSize: 9, font: 'inter' }
                });

                doc.save(`Laporan_Keuangan_Pesantren_${new Date().toISOString().split('T')[0]}.pdf`);
                showModal('Berhasil Unduh PDF', 'File laporan PDF berhasil diunduh ke perangkat Anda.', 'success');
            } catch (err) {
                console.error("Gagal membuat PDF:", err);
                showModal('Gagal PDF', 'Terjadi kesalahan saat mengunduh laporan PDF.', 'error');
            }
        }

        function updateCredentials(e) {
            e.preventDefault();
            if (!dbState.credentials) dbState.credentials = DEFAULT_STATE.credentials;
            dbState.credentials.admin.pass = document.getElementById('pass-admin').value;
            dbState.credentials.pesantren.pass = document.getElementById('pass-pesantren').value;
            dbState.credentials.treasurer.pass = document.getElementById('pass-treasurer').value;
            saveDb();
            showModal('Berhasil', 'Kata sandi seluruh ruangan berhasil diperbarui.', 'success');
        }

        function confirmResetAllData() {
            showModal('Konfirmasi Kosongkan Data', 'Apakah Anda yakin ingin mengosongkan seluruh data santri, pembayaran, dan transaksi kas? Tindakan ini tidak dapat dibatalkan.', 'info', [
                { text: 'Batal', class: 'bg-slate-200 text-slate-700 hover:bg-slate-300 flex-1 py-3 font-bold', onClick: closeModal },
                { text: 'Ya, Kosongkan', class: 'bg-rose-600 text-white hover:bg-rose-700 flex-1 py-3 shadow-md shadow-rose-600/30 font-bold', onClick: () => {
                    dbState.santri = [];
                    dbState.payments = [];
                    dbState.transactions = [];
                    saveDb();
                    closeModal();
                    renderDashboard();
                    showModal('Berhasil Dikosongkan', 'Seluruh data santri dan keuangan telah dikosongkan. Anda dapat mulai menginput data baru dari awal.', 'success');
                }}
            ]);
        }

        function updateProfile(e) {
            e.preventDefault();
            if (!dbState.profile) dbState.profile = DEFAULT_STATE.profile;
            dbState.profile.name = document.getElementById('prof-name').value;
            dbState.profile.foundation = document.getElementById('prof-foundation').value;
            dbState.profile.address = document.getElementById('prof-address').value;
            dbState.profile.phone = document.getElementById('prof-phone').value;
            dbState.profile.currentYear = document.getElementById('prof-year').value;
            saveDb();
            renderDashboard();
            showModal('Berhasil', 'Profil pesantren dan yayasan berhasil diperbarui.', 'success');
        }

        function saveSantriSppAndScholarship(santriId) {
            const inputVal = document.getElementById(`spp-santri-${santriId}`).value;
            const scholarshipVal = document.getElementById(`spp-scholarship-${santriId}`).value;
            const santri = dbState.santri.find(s => s.id === santriId);
            if (santri) {
                santri.customSpp = parseInt(inputVal) || 250000;
                santri.scholarship = scholarshipVal;
                saveDb();
                renderDashboard();
                showModal('Berhasil', `Pengaturan untuk ${santri.name} berhasil diperbarui.`, 'success');
            }
        }

        function openAddSantriModal() {
            const defaultSpp = dbState.profile?.defaultSpp || 250000;
            showModal('Tambah Santri Baru', 'Formulir input data santri:', 'info', [
                { text: 'Batal', class: 'bg-slate-200 text-slate-800 hover:bg-slate-300 flex-1 py-3 font-bold', onClick: closeModal },
                { text: 'Simpan', class: 'bg-emerald-600 text-white hover:bg-emerald-700 flex-1 py-3 shadow-md shadow-emerald-600/30 font-bold', onClick: () => {
                    const name = document.getElementById('modal-santri-name').value;
                    const santriClass = document.getElementById('modal-santri-class').value;
                    const customSpp = parseInt(document.getElementById('modal-santri-spp').value) || defaultSpp;
                    const scholarship = document.getElementById('modal-santri-scholarship').value;
                    const phone = document.getElementById('modal-santri-phone').value;

                    if (!name) return;

                    if (!dbState.santri) dbState.santri = [];
                    const newId = 'S00' + (dbState.santri.length + 1);
                    dbState.santri.unshift({ id: newId, name, class: santriClass, customSpp, status: 'Aktif', scholarship, phone });
                    saveDb();
                    closeModal();
                    renderDashboard();
                    showModal('Berhasil', 'Santri baru berhasil ditambahkan.', 'success');
                }}
            ]);

            document.getElementById('modal-message').innerHTML = `
                <div class="space-y-3.5 text-left mt-2">
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Nama Lengkap Santri</label>
                        <input type="text" id="modal-santri-name" placeholder="cth: Ahmad Dani" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Kelas</label>
                        <input type="text" id="modal-santri-class" placeholder="cth: VII-A (Tsanawiyah)" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Nominal SPP Bulanan (Rp)</label>
                        <input type="number" id="modal-santri-spp" value="${defaultSpp}" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-black text-emerald-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Status Beasiswa</label>
                        <select id="modal-santri-scholarship" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                            <option value="Tidak">Reguler (Tidak Beasiswa)</option>
                            <option value="Ya">Penerima Beasiswa (Gratis SPP)</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Nomor Telepon Wali</label>
                        <input type="text" id="modal-santri-phone" placeholder="cth: 08123456789" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                </div>
            `;
        }

        function deleteSantri(id) {
            if (!dbState.santri) return;
            dbState.santri = dbState.santri.filter(s => s.id !== id);
            saveDb();
            renderDashboard();
            showModal('Berhasil', 'Data santri berhasil dihapus.', 'success');
        }

        function openPaymentModal() {
            const santriList = dbState.santri || [];
            const santriOptions = santriList.map(s => `<option value="${s.id}">${s.name} (${s.class})</option>`).join('');
            const todayStr = new Date().toISOString().split('T')[0];
            const defaultSppVal = santriList[0]?.customSpp !== undefined ? santriList[0].customSpp : (dbState.profile?.defaultSpp || 250000);

            showModal('Catat Pembayaran Santri', 'Formulir transaksi pembayaran:', 'info', [
                { text: 'Batal', class: 'bg-slate-200 text-slate-800 hover:bg-slate-300 flex-1 py-3 font-bold', onClick: closeModal },
                { text: 'Catat Pembayaran', class: 'bg-emerald-600 text-white hover:bg-emerald-700 flex-1 py-3 shadow-md shadow-emerald-600/30 font-bold', onClick: () => {
                    const paymentDate = document.getElementById('pay-date').value || todayStr;
                    const santriId = document.getElementById('pay-santri').value;
                    const type = document.getElementById('pay-type').value;
                    const month = document.getElementById('pay-month').value;
                    const amount = parseInt(document.getElementById('pay-amount').value) || 0;

                    const santri = santriList.find(s => s.id === santriId);
                    if (!santri) return;

                    if (!dbState.payments) dbState.payments = [];
                    if (!dbState.transactions) dbState.transactions = [];

                    const newPayment = {
                        id: 'P00' + (dbState.payments.length + 1) + Math.floor(Math.random()*100),
                        santriId,
                        santriName: santri.name,
                        type,
                        month,
                        amount: santri.scholarship === 'Ya' && type === 'SPP' ? 0 : amount,
                        date: paymentDate,
                        status: santri.scholarship === 'Ya' && type === 'SPP' ? 'Beasiswa (Gratis)' : 'Lunas'
                    };

                    dbState.payments.unshift(newPayment);

                    if (newPayment.amount > 0) {
                        dbState.transactions.unshift({
                            id: 'T00' + (dbState.transactions.length + 1) + Math.floor(Math.random()*100),
                            date: paymentDate,
                            type: 'Pemasukan',
                            category: type === 'SPP' ? 'SPP Bulanan' : 'Daftar Ulang',
                            amount: newPayment.amount,
                            desc: `${type} ${santri.name} (${month})`
                        });
                    }

                    saveDb();
                    closeModal();
                    renderDashboard();
                    showModal('Berhasil', 'Pembayaran berhasil dicatat.', 'success');
                }}
            ]);

            document.getElementById('modal-message').innerHTML = `
                <div class="space-y-3.5 text-left mt-2">
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Tanggal Pembayaran</label>
                        <input type="date" id="pay-date" value="${todayStr}" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Pilih Santri</label>
                        <select id="pay-santri" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                            ${santriOptions}
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Jenis Pembayaran</label>
                        <select id="pay-type" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                            <option value="SPP">SPP Bulanan</option>
                            <option value="Daftar Ulang">Daftar Ulang</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Bulan / Keterangan Periode</label>
                        <input type="text" id="pay-month" value="Agustus 2025" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Nominal (Rp)</label>
                        <input type="number" id="pay-amount" value="${defaultSppVal}" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-black text-emerald-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                </div>
            `;
        }

        function openTransactionModal() {
            const todayStr = new Date().toISOString().split('T')[0];
            showModal('Catat Kas Pemasukan / Pengeluaran', 'Formulir transaksi kas:', 'info', [
                { text: 'Batal', class: 'bg-slate-200 text-slate-800 hover:bg-slate-300 flex-1 py-3 font-bold', onClick: closeModal },
                { text: 'Simpan Transaksi', class: 'bg-emerald-600 text-white hover:bg-emerald-700 flex-1 py-3 shadow-md shadow-emerald-600/30 font-bold', onClick: () => {
                    const type = document.getElementById('trx-type').value;
                    const category = document.getElementById('trx-cat').value;
                    const amount = parseInt(document.getElementById('trx-amount').value) || 0;
                    const desc = document.getElementById('trx-desc').value;

                    if (!category || !amount) return;

                    if (!dbState.transactions) dbState.transactions = [];
                    dbState.transactions.unshift({
                        id: 'T00' + (dbState.transactions.length + 1) + Math.floor(Math.random()*100),
                        date: todayStr,
                        type,
                        category,
                        amount,
                        desc
                    });

                    saveDb();
                    closeModal();
                    renderDashboard();
                    showModal('Berhasil', 'Transaksi kas berhasil dicatat.', 'success');
                }}
            ]);

            document.getElementById('modal-message').innerHTML = `
                <div class="space-y-3.5 text-left mt-2">
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Jenis Arus Kas</label>
                        <select id="trx-type" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                            <option value="Pemasukan">Pemasukan Kas</option>
                            <option value="Pengeluaran">Pengeluaran Kas</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Kategori Transaksi</label>
                        <input type="text" id="trx-cat" placeholder="cth: Operasional / Donasi / Honor" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Nominal (Rp)</label>
                        <input type="number" id="trx-amount" placeholder="500000" class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-500">
                    </div>
                    <div>
                        <label class="block text-xs font-black text-slate-900 mb-1">Keterangan / Deskripsi</label>
                        <textarea id="trx-desc" rows="2" placeholder="Catatan atau rincian transaksi..." class="w-full px-3.5 py-3 bg-slate-50 border border-slate-300 rounded-xl text-sm font-bold text-slate-900 focus:ring-2 focus:ring-emerald-500"></textarea>
                    </div>
                </div>
            `;
        }

        function copySqlScript() {
            const textarea = document.getElementById('sql-textarea');
            if (!textarea) return;
            textarea.select();
            document.execCommand('copy');
            showModal('Berhasil Disalin', 'Skrip SQL berhasil disalin ke clipboard Anda.', 'success');
        }

        function showModal(title, message, type, actions = []) {
            const container = document.getElementById('modal-container');
            const box = document.getElementById('modal-box');
            const icon = document.getElementById('modal-icon');
            const titleEl = document.getElementById('modal-title');
            const msgEl = document.getElementById('modal-message');
            const actEl = document.getElementById('modal-actions');
            if (!container || !box || !icon || !titleEl || !msgEl || !actEl) return;

            titleEl.innerText = title;
            msgEl.innerHTML = message;

            if (type === 'success') {
                icon.className = 'w-14 h-14 rounded-2xl bg-emerald-100 text-emerald-700 flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner';
                icon.innerHTML = '<i class="fa-solid fa-check"></i>';
            } else if (type === 'error') {
                icon.className = 'w-14 h-14 rounded-2xl bg-rose-100 text-rose-700 flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner';
                icon.innerHTML = '<i class="fa-solid fa-triangle-exclamation"></i>';
            } else {
                icon.className = 'w-14 h-14 rounded-2xl bg-emerald-100 text-emerald-700 flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner';
                icon.innerHTML = '<i class="fa-solid fa-info"></i>';
            }

            if (actions.length === 0) {
                actEl.innerHTML = `<button onclick="closeModal()" class="w-full py-3.5 bg-emerald-600 text-white font-black rounded-xl text-sm hover:bg-emerald-700 transition shadow-lg shadow-emerald-600/30 active:scale-95">Tutup</button>`;
            } else {
                actEl.innerHTML = actions.map((a, idx) => `
                    <button id="modal-act-${idx}" class="px-4 py-3 font-bold rounded-xl text-sm transition active:scale-95 ${a.class}">${a.text}</button>
                `).join('');

                actions.forEach((a, idx) => {
                    const btn = document.getElementById(`modal-act-${idx}`);
                    if (btn) btn.onclick = a.onClick;
                });
            }

            container.classList.remove('hidden');
            setTimeout(() => {
                box.classList.remove('scale-95', 'opacity-0');
                box.classList.add('scale-100', 'opacity-100');
            }, 10);
        }

        function closeModal() {
            const container = document.getElementById('modal-container');
            const box = document.getElementById('modal-box');
            if (!container || !box) return;
            box.classList.remove('scale-100', 'opacity-100');
            box.classList.add('scale-95', 'opacity-0');
            setTimeout(() => {
                container.classList.add('hidden');
            }, 300);
        }

        window.onload = async function() {
            try {
                await fetchCloudData();
                initRealtimeUpdates();
            } catch (err) {
                console.warn("Gagal fetch cloud data saat startup:", err);
            }
            renderAuthPortal();
        };
    </script>
</body>
</html>
