<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplikasi Keuangan Pesantren Terintegrasi Supabase</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700;800;900&display=swap" rel="stylesheet">
    <!-- Supabase JS SDK -->
    <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
    <!-- jsPDF for PDF Exports -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.28/jspdf.plugin.autotable.min.js"></script>
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
<body class="bg-slate-100 text-slate-900 font-medium antialiased min-h-screen flex flex-col selection:bg-emerald-700 selection:text-white">

    <div id="app" class="flex-1 flex flex-col">
    </div>

    <!-- Notification Modal -->
    <div id="modal-container" class="fixed inset-0 z-50 flex items-center justify-center bg-black/70 hidden backdrop-blur-sm p-4 transition-all duration-300">
        <div class="bg-white rounded-3xl p-6 sm:p-8 max-w-md w-full mx-auto shadow-2xl transform transition-all scale-95 opacity-0 duration-300 border-2 border-slate-300" id="modal-box">
            <div id="modal-icon" class="w-14 h-14 rounded-2xl flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner"></div>
            <h3 id="modal-title" class="text-xl font-extrabold text-center text-slate-900 mb-2"></h3>
            <p id="modal-message" class="text-sm font-bold text-center text-slate-800 mb-6 leading-relaxed"></p>
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
                { id: "T001", date: "2025-07-10", type: "Pemasukan", category: "Daftar Ulang", amount: 900000, desc: "Pembayaran Daftar Ulang Santri" },
                { id: "T002", date: "2025-07-15", type: "Pengeluaran", category: "Operasional", amount: 500000, desc: "Servis Mobil Operasional Pesantren" },
                { id: "T003", date: "2025-08-01", type: "Pemasukan", category: "Donasi / Hibah", amount: 1000000, desc: "Dana Hibah Yayasan" }
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
        let currentActiveFolderMonth = null;

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
                <div class="min-h-screen bg-slate-950 flex items-center justify-center p-4 sm:p-6">
                    <div class="max-w-md w-full bg-white rounded-3xl shadow-2xl p-6 sm:p-8 border-4 border-slate-700">
                        <div class="text-center mb-6 sm:mb-8">
                            <div class="w-16 h-16 sm:w-20 sm:h-20 bg-emerald-700 rounded-3xl mx-auto flex items-center justify-center text-white text-2xl sm:text-3xl shadow-xl shadow-emerald-700/50 mb-4">
                                <i class="fa-solid fa-mosque"></i>
                            </div>
                            <h1 class="text-xl sm:text-2xl font-black text-slate-900 tracking-tight">Keuangan Pesantren</h1>
                            <p class="text-[11px] sm:text-xs font-black text-emerald-800 mt-1 uppercase tracking-wider truncate px-2">${dbState.profile?.name || 'Pesantren Darul Ulum'}</p>
                        </div>

                        <form onsubmit="handleLogin(event)" class="space-y-4 sm:space-y-5">
                            <div>
                                <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-2">Pilih Ruangan Akses</label>
                                <div class="relative">
                                    <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-emerald-700 font-bold pointer-events-none"><i class="fa-solid fa-door-open"></i></span>
                                    <select id="role-select" class="w-full pl-11 pr-4 py-3 bg-slate-100 border-2 border-slate-400 rounded-2xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition shadow-sm">
                                        <option value="admin">Administrator Utama</option>
                                        <option value="pesantren">Admin Pesantren</option>
                                        <option value="treasurer">Bendahara Pusat</option>
                                    </select>
                                </div>
                            </div>

                            <div>
                                <label class="block text-xs font-black uppercase tracking-wider text-slate-900 mb-2">Kata Sandi Ruangan</label>
                                <div class="relative">
                                    <span class="absolute inset-y-0 left-0 pl-4 flex items-center text-emerald-700 font-bold pointer-events-none"><i class="fa-solid fa-lock"></i></span>
                                    <input type="password" id="role-pass" placeholder="Masukkan sandi..." required class="w-full pl-11 pr-4 py-3 bg-slate-100 border-2 border-slate-400 rounded-2xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition shadow-sm">
                                </div>
                            </div>

                            <div class="flex gap-2 pt-1">
                                <button type="submit" class="flex-1 py-3.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black rounded-2xl shadow-lg shadow-emerald-700/40 transition transform active:scale-95 flex items-center justify-center gap-2 text-xs sm:text-sm border-2 border-emerald-900">
                                    <i class="fa-solid fa-right-to-bracket text-base"></i> Masuk Ruangan
                                </button>
                                <button type="button" onclick="resetDefaultCredentials()" title="Reset sandi ke default" class="px-4 py-3.5 bg-slate-300 hover:bg-slate-400 text-slate-900 font-black rounded-2xl transition active:scale-95 flex items-center justify-center border-2 border-slate-500">
                                    <i class="fa-solid fa-rotate-right"></i>
                                </button>
                            </div>
                        </form>
                    </div>
                </div>
            `;
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
                if (role === 'admin') {
                    currentTab = 'cycles'; 
                } else if (role === 'treasurer') {
                    currentTab = 'transactions';
                } else {
                    currentTab = 'dashboard';
                }
                renderDashboard();
            } else {
                showModal('Akses Ditolak', 'Kata sandi ruangan yang Anda masukkan salah.', 'error');
            }
        }

        function renderDashboard() {
            const app = document.getElementById('app');
            if (!app || !currentUser) return;
            const roleName = currentUser.name;

            let tabs = [];
            if (currentUser.role === 'admin') {
                tabs = [
                    { id: 'cycles', label: 'Siklus Keuangan', icon: 'fa-chart-pie' },
                    { id: 'unpaid_admin', label: 'Santri Belum Bayar', icon: 'fa-triangle-exclamation' },
                    { id: 'dashboard', label: 'Beranda & Ringkasan', icon: 'fa-house' },
                    { id: 'profile', label: 'Profil Pesantren', icon: 'fa-school' },
                    { id: 'credentials', label: 'Sandi Ruangan', icon: 'fa-key' },
                    { id: 'reset_data', label: 'Kosongkan Data', icon: 'fa-trash-arrow-up' },
                    { id: 'sql_setup', label: 'Setup SQL Supabase', icon: 'fa-database' }
                ];
            } else if (currentUser.role === 'pesantren') {
                tabs = [
                    { id: 'dashboard', label: 'Beranda & Ringkasan', icon: 'fa-house' },
                    { id: 'santri', label: 'Data Santri', icon: 'fa-users' },
                    { id: 'spp_setting', label: 'Pengaturan SPP & Beasiswa', icon: 'fa-sliders' },
                    { id: 'payments', label: 'Catat Pembayaran', icon: 'fa-receipt' },
                    { id: 'arrears', label: 'Santri Belum Bayar', icon: 'fa-triangle-exclamation' }
                ];
            } else if (currentUser.role === 'treasurer') {
                tabs = [
                    { id: 'transactions', label: 'Buku Kas & Histori Transaksi', icon: 'fa-book' },
                    { id: 'spp_monitor', label: 'Monitoring SPP', icon: 'fa-money-bill-wave' },
                    { id: 'unpaid_treasurer', label: 'Santri Belum Bayar', icon: 'fa-triangle-exclamation' },
                    { id: 'reports', label: 'Santri & Beasiswa', icon: 'fa-user-graduate' }
                ];
            }

            const currentTabObj = tabs.find(t => t.id === currentTab) || tabs[0];

            app.innerHTML = `
                <div class="min-h-screen flex flex-col md:flex-row bg-slate-200">
                    <!-- Mobile Drawer Overlay -->
                    <div id="mobile-sidebar-overlay" onclick="toggleMobileSidebar(false)" class="fixed inset-0 bg-black/70 z-40 hidden md:hidden backdrop-blur-sm transition-opacity"></div>

                    <!-- Sidebar -->
                    <aside id="mobile-sidebar" class="w-72 bg-slate-900 text-white flex flex-col justify-between p-6 shadow-2xl fixed inset-y-0 left-0 z-50 transform -translate-x-full md:translate-x-0 md:static transition-transform duration-300 flex-shrink-0 border-r-2 border-slate-700">
                        <div>
                            <div class="mb-6 pb-6 border-b-2 border-slate-800">
                                <div class="flex items-center justify-between mb-4">
                                    <div class="flex items-center gap-3.5 min-w-0">
                                        <div class="w-12 h-12 bg-emerald-700 rounded-2xl flex items-center justify-center text-white text-xl shadow-md flex-shrink-0 border border-emerald-400">
                                            <i class="fa-solid fa-mosque"></i>
                                        </div>
                                        <div class="min-w-0">
                                            <h2 class="text-sm font-black tracking-tight truncate text-white">Keuangan Pesantren</h2>
                                            <p class="text-[11px] text-emerald-400 font-black truncate">${roleName}</p>
                                        </div>
                                    </div>
                                    <button onclick="toggleMobileSidebar(false)" class="md:hidden text-slate-300 hover:text-white p-2">
                                        <i class="fa-solid fa-xmark text-xl"></i>
                                    </button>
                                </div>
                                <button onclick="logout()" class="w-full flex items-center justify-center gap-2 px-4 py-3 bg-red-700 hover:bg-red-800 text-white rounded-2xl font-black text-xs transition shadow-md active:scale-95 border-2 border-red-900">
                                    <i class="fa-solid fa-arrow-right-from-bracket"></i> Keluar Ruangan
                                </button>
                            </div>

                            <nav class="space-y-1.5">
                                ${tabs.map(t => `
                                    <button onclick="switchTab('${t.id}')" class="w-full flex items-center gap-3 px-3.5 py-3 rounded-2xl font-black text-xs sm:text-sm transition text-left ${currentTab === t.id ? 'bg-emerald-700 text-white shadow-lg border-2 border-emerald-500' : 'text-slate-300 hover:text-white hover:bg-slate-800 border-2 border-transparent'}">
                                        <i class="fa-solid ${t.icon} text-sm w-5 text-center"></i>
                                        <span class="truncate">${t.label}</span>
                                    </button>
                                `).join('')}
                            </nav>
                        </div>

                        <div class="pt-6 mt-6 border-t-2 border-slate-800 space-y-3">
                            <button onclick="downloadPdfReport()" class="w-full flex items-center justify-center gap-2 px-4 py-3 bg-emerald-900/60 hover:bg-emerald-900 text-emerald-300 rounded-2xl font-black text-xs transition border-2 border-emerald-600">
                                <i class="fa-solid fa-file-pdf"></i> Unduh Laporan PDF
                            </button>
                        </div>
                    </aside>

                    <!-- Main Content Area -->
                    <div class="flex-1 flex flex-col min-w-0 overflow-y-auto">
                        <header class="bg-white border-b-2 border-slate-300 px-6 py-4 sticky top-0 z-20 flex items-center justify-between gap-4 shadow-sm">
                            <div class="flex items-center gap-3 min-w-0">
                                <button onclick="toggleMobileSidebar(true)" class="md:hidden w-10 h-10 rounded-xl bg-slate-200 hover:bg-slate-300 text-slate-900 flex items-center justify-center flex-shrink-0 transition border border-slate-400">
                                    <i class="fa-solid fa-bars text-lg"></i>
                                </button>
                                <div class="min-w-0">
                                    <h1 class="text-base sm:text-xl font-black text-slate-900 tracking-tight truncate flex items-center gap-2">
                                        <span class="truncate">${currentTabObj.label}</span>
                                    </h1>
                                    <p class="text-[11px] sm:text-xs font-bold text-slate-700 truncate mt-0.5">${dbState.profile?.name || 'Rangkuman aktivitas keuangan.'}</p>
                                </div>
                            </div>
                            <div class="hidden sm:flex items-center gap-3 flex-shrink-0">
                                <div class="px-3.5 py-2 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs font-black text-slate-900 flex items-center gap-2 shadow-xs">
                                    <span class="text-slate-600 font-black">Tahun Ajaran:</span>
                                    <span class="text-emerald-700 font-black">${dbState.profile?.currentYear || '2025/2026'}</span>
                                </div>
                            </div>
                        </header>

                        <main class="p-4 sm:p-8 space-y-6 flex-1 max-w-7xl w-full mx-auto">
                            ${renderTabContent()}
                        </main>
                    </div>
                </div>
            `;
        }

        function toggleMobileSidebar(open) {
            const sidebar = document.getElementById('mobile-sidebar');
            const overlay = document.getElementById('mobile-sidebar-overlay');
            if (!sidebar || !overlay) return;
            if (open) {
                sidebar.classList.remove('-translate-x-full');
                overlay.classList.remove('hidden');
            } else {
                sidebar.classList.add('-translate-x-full');
                overlay.classList.add('hidden');
            }
        }

        function switchTab(tabId) {
            currentTab = tabId;
            currentActiveFolderMonth = null;
            toggleMobileSidebar(false);
            renderDashboard();
        }

        function openMonthFolder(monthName) {
            currentActiveFolderMonth = monthName;
            renderDashboard();
        }

        function backToMainFolders() {
            currentActiveFolderMonth = null;
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
            const currentMonth = 'Agustus 2025';

            const renderUnpaidSantriTable = () => {
                const paidThisMonthSantriIds = paymentList.filter(p => p.type === 'SPP' && p.month === currentMonth).map(p => p.santriId);
                const regularSantri = santriList.filter(s => s.scholarship !== 'Ya' && s.status === 'Aktif');
                const unpaidSantri = regularSantri.filter(s => !paidThisMonthSantriIds.includes(s.id));

                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base flex items-center gap-2">
                                    <i class="fa-solid fa-triangle-exclamation text-red-700 text-lg"></i>
                                    Daftar Santri Belum Bayar SPP (${currentMonth})
                                </h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Berikut adalah daftar santri reguler yang belum melakukan setoran SPP periode ini.</p>
                            </div>
                            <div class="px-3 py-2 bg-red-700 text-white font-black text-[11px] sm:text-xs rounded-xl border-2 border-red-950 shadow-md text-center">
                                Total Belum Bayar: ${unpaidSantri.length} Santri
                            </div>
                        </div>

                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm">
                                <thead class="bg-red-800 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 rounded-l-2xl">ID & Nama Santri</th>
                                        <th class="p-3">Kelas</th>
                                        <th class="p-3">Tagihan SPP</th>
                                        <th class="p-3">No Telepon Wali</th>
                                        <th class="p-3 rounded-r-2xl text-center">Status</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${unpaidSantri.length === 0 ? `
                                        <tr>
                                            <td colspan="5" class="p-6 text-center font-black text-emerald-800 bg-emerald-50 text-xs sm:text-sm">
                                                <i class="fa-solid fa-circle-check text-lg mr-2"></i> Luar biasa! Seluruh santri telah melunasi SPP bulan ${currentMonth}.
                                            </td>
                                        </tr>
                                    ` : unpaidSantri.map(s => `
                                        <tr class="hover:bg-red-50/50 transition">
                                            <td class="p-3">
                                                <div class="font-black text-slate-900 text-xs sm:text-sm">${s.name}</div>
                                                <div class="text-[10px] sm:text-[11px] font-bold text-slate-700">ID: ${s.id}</div>
                                            </td>
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${s.class}</td>
                                            <td class="p-3 text-red-700 font-black whitespace-nowrap text-xs">Rp ${(s.customSpp !== undefined ? s.customSpp : (dbState.profile?.defaultSpp || 250000)).toLocaleString('id-ID')}</td>
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${s.phone}</td>
                                            <td class="p-3 text-center whitespace-nowrap">
                                                <span class="px-2.5 py-1 bg-red-700 text-white font-black text-[10px] sm:text-xs rounded-xl border border-red-950 shadow-xs inline-flex items-center gap-1">
                                                    <i class="fa-solid fa-circle-xmark"></i> Belum Bayar
                                                </span>
                                            </td>
                                        </tr>
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            };

            if (currentUser && currentUser.role === 'admin' && currentTab === 'unpaid_admin') return renderUnpaidSantriTable();
            if (currentUser && currentUser.role === 'pesantren' && currentTab === 'arrears') return renderUnpaidSantriTable();
            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'unpaid_treasurer') return renderUnpaidSantriTable();

            if (currentUser && currentUser.role === 'admin' && currentTab === 'cycles') {
                // Urutkan strictly kronologis dari tanggal terlama ke terbaru (atas ke bawah)
                let sortedChronological = [...txList].sort((a, b) => new Date(a.date) - new Date(b.date));
                let running = 0;
                let txWithRunningBalance = sortedChronological.map(t => {
                    if (t.type === 'Pemasukan') {
                        running += t.amount;
                    } else if (t.type === 'Pengeluaran') {
                        running -= t.amount;
                    }
                    return { ...t, saldoKas: running };
                });

                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base">Siklus & Rekapitulasi Keuangan Pesantren</h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Riwayat dan saldo kas mengalir dari atas (terlama) ke bawah (terbaru).</p>
                            </div>
                        </div>

                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 sm:gap-6 mb-8">
                            <div class="p-4 sm:p-5 bg-emerald-700 rounded-3xl border-2 border-emerald-900 shadow-lg text-white">
                                <div class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-emerald-100 mb-1 flex items-center gap-1.5">
                                    <i class="fa-solid fa-arrow-trend-up"></i> Total Pemasukan
                                </div>
                                <div class="text-xl sm:text-2xl font-black truncate">Rp ${totalIncome.toLocaleString('id-ID')}</div>
                            </div>
                            <div class="p-4 sm:p-5 bg-red-700 rounded-3xl border-2 border-red-900 shadow-lg text-white">
                                <div class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-red-100 mb-1 flex items-center gap-1.5">
                                    <i class="fa-solid fa-arrow-trend-down"></i> Total Pengeluaran
                                </div>
                                <div class="text-xl sm:text-2xl font-black truncate">Rp ${totalExpense.toLocaleString('id-ID')}</div>
                            </div>
                            <div class="p-4 sm:p-5 bg-indigo-700 rounded-3xl border-2 border-indigo-900 shadow-lg text-white">
                                <div class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-indigo-100 mb-1 flex items-center gap-1.5">
                                    <i class="fa-solid fa-wallet"></i> Arus Kas Bersih
                                </div>
                                <div class="text-xl sm:text-2xl font-black truncate">Rp ${balance.toLocaleString('id-ID')}</div>
                            </div>
                        </div>

                        <h4 class="font-black text-slate-900 mb-4 flex items-center gap-2 text-xs sm:text-sm"><i class="fa-solid fa-list-check text-emerald-700"></i> Riwayat Transaksi & Saldo Kas Kumulatif (Urutan Tanggal ke Bawah)</h4>
                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm">
                                <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 rounded-l-2xl">Tanggal</th>
                                        <th class="p-3">Jenis</th>
                                        <th class="p-3">Kategori</th>
                                        <th class="p-3">Keterangan</th>
                                        <th class="p-3 text-right">Nominal</th>
                                        <th class="p-3 rounded-r-2xl text-right">Saldo Kas Buku</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${txWithRunningBalance.map(t => `
                                        <tr class="hover:bg-slate-100 transition">
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${t.date}</td>
                                            <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 rounded-full text-[10px] font-black ${t.type === 'Pemasukan' ? 'bg-emerald-700 text-white border border-emerald-900' : 'bg-red-700 text-white border border-red-900'}">${t.type}</span></td>
                                            <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${t.category}</td>
                                            <td class="p-3 font-black text-slate-800 text-xs">${t.desc}</td>
                                            <td class="p-3 text-right font-black whitespace-nowrap text-xs ${t.type === 'Pemasukan' ? 'text-emerald-800' : 'text-red-700'}">${t.type === 'Pemasukan' ? '+ ' : '- '} Rp ${t.amount.toLocaleString('id-ID')}</td>
                                            <td class="p-3 text-right font-black text-indigo-900 whitespace-nowrap text-xs">Rp ${t.saldoKas.toLocaleString('id-ID')}</td>
                                        </tr>
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'transactions') {
                // Urutkan strictly kronologis dari tanggal terlama ke terbaru (atas ke bawah) untuk Bendahara Pusat
                let sortedChronological = [...txList].sort((a, b) => new Date(a.date) - new Date(b.date));
                let running = 0;
                let txWithRunningBalance = sortedChronological.map(t => {
                    if (t.type === 'Pemasukan') {
                        running += t.amount;
                    } else if (t.type === 'Pengeluaran') {
                        running -= t.amount;
                    }
                    return { ...t, saldoKas: running };
                });

                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md space-y-6">
                        <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between gap-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base flex items-center gap-2">
                                    <i class="fa-solid fa-book text-amber-700"></i> Buku Kas & Histori Transaksi Keuangan
                                </h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Urutan transaksi dan saldo kas mengalir ke bawah dari tanggal terlama ke terbaru.</p>
                            </div>

                            <div class="flex flex-wrap items-center gap-3">
                                <button onclick="openTransactionModal()" class="px-4 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs sm:text-sm rounded-xl shadow-md transition flex items-center gap-2 active:scale-95 border-2 border-emerald-950">
                                    <i class="fa-solid fa-plus-circle"></i> Tambah Transaksi Kas
                                </button>
                                <button onclick="downloadTransactionPdf()" class="px-4 py-2.5 bg-indigo-700 hover:bg-indigo-800 text-white font-black text-xs sm:text-sm rounded-xl shadow-md transition flex items-center gap-2 active:scale-95 border-2 border-indigo-950">
                                    <i class="fa-solid fa-file-pdf"></i> Download PDF Buku Kas
                                </button>
                            </div>
                        </div>

                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-3">
                            <div class="bg-emerald-700 p-3.5 px-4 rounded-2xl shadow-md text-white flex flex-col justify-center border-2 border-emerald-950">
                                <span class="text-[10px] font-black uppercase tracking-wider text-emerald-100">Total Pemasukan Kas</span>
                                <span class="text-xs sm:text-sm font-black truncate">Rp ${totalIncome.toLocaleString('id-ID')}</span>
                            </div>
                            <div class="bg-red-700 p-3.5 px-4 rounded-2xl shadow-md text-white flex flex-col justify-center border-2 border-red-950">
                                <span class="text-[10px] font-black uppercase tracking-wider text-red-100">Total Pengeluaran Kas</span>
                                <span class="text-xs sm:text-sm font-black truncate">Rp ${totalExpense.toLocaleString('id-ID')}</span>
                            </div>
                            <div class="bg-indigo-700 p-3.5 px-4 rounded-2xl shadow-md text-white flex flex-col justify-center border-2 border-indigo-950">
                                <span class="text-[10px] font-black uppercase tracking-wider text-indigo-100">Saldo Kas Bersih</span>
                                <span class="text-xs sm:text-sm font-black truncate">Rp ${balance.toLocaleString('id-ID')}</span>
                            </div>
                        </div>

                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm border-collapse">
                                <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 rounded-l-xl">Tanggal</th>
                                        <th class="p-3">Jenis</th>
                                        <th class="p-3">Kategori</th>
                                        <th class="p-3">Uraian / Keterangan</th>
                                        <th class="p-3 text-right">Nominal (Masuk/Keluar)</th>
                                        <th class="p-3 rounded-r-xl text-right">Saldo Kas Buku</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${txWithRunningBalance.length === 0 ? `
                                        <tr>
                                            <td colspan="6" class="p-6 text-center text-slate-500 font-bold">Belum ada catatan transaksi buku kas.</td>
                                        </tr>
                                    ` : txWithRunningBalance.map(t => `
                                        <tr class="hover:bg-slate-100 transition">
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${t.date}</td>
                                            <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 rounded-full text-[10px] font-black ${t.type === 'Pemasukan' ? 'bg-emerald-700 text-white border border-emerald-950' : 'bg-red-700 text-white border border-emerald-950'}">${t.type}</span></td>
                                            <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${t.category}</td>
                                            <td class="p-3 font-black text-slate-800 text-xs">${t.desc}</td>
                                            <td class="p-3 text-right font-black whitespace-nowrap text-xs ${t.type === 'Pemasukan' ? 'text-emerald-800' : 'text-red-700'}">${t.type === 'Pemasukan' ? '+ ' : '- '} Rp ${t.amount.toLocaleString('id-ID')}</td>
                                            <td class="p-3 text-right font-black text-indigo-900 whitespace-nowrap text-xs">Rp ${t.saldoKas.toLocaleString('id-ID')}</td>
                                        </tr>
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            }

            if (currentTab === 'dashboard') {
                if (currentUser && currentUser.role === 'pesantren') {
                    const paymentsByMonth = {};
                    paymentList.forEach(p => {
                        const mKey = p.month || 'Lainnya';
                        if (!paymentsByMonth[mKey]) {
                            paymentsByMonth[mKey] = [];
                        }
                        paymentsByMonth[mKey].push(p);
                    });

                    const monthKeys = Object.keys(paymentsByMonth);
                    const paidThisMonthSantriIds = paymentList.filter(p => p.type === 'SPP' && p.month === currentMonth).map(p => p.santriId);
                    const regularSantri = santriList.filter(s => s.scholarship !== 'Ya' && s.status === 'Aktif');
                    const unpaidSantriCount = regularSantri.filter(s => !paidThisMonthSantriIds.includes(s.id)).length;

                    let paymentsHtml = '';
                    if (currentActiveFolderMonth === null) {
                        paymentsHtml = `
                            <div class="mb-6">
                                <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center justify-between">
                                    <span class="flex items-center gap-2"><i class="fa-solid fa-clock-rotate-left text-emerald-700"></i> Aktivitas Pembayaran Terbaru</span>
                                    <span class="text-[11px] sm:text-xs font-black text-slate-800">Total: ${paymentList.length}</span>
                                </h4>
                                <div class="space-y-3">
                                    ${paymentList.map(p => `
                                        <div class="flex items-center justify-between p-3.5 bg-slate-50 rounded-2xl border-2 border-slate-300 text-xs shadow-xs">
                                            <div class="flex items-center gap-3 min-w-0">
                                                <div class="w-9 h-9 rounded-xl bg-emerald-700 text-white flex-shrink-0 flex items-center justify-center font-bold shadow-md border border-emerald-950"><i class="fa-solid fa-receipt"></i></div>
                                                <div class="min-w-0">
                                                    <div class="font-black text-slate-900 truncate text-xs sm:text-sm">${p.santriName} - ${p.type}</div>
                                                    <div class="text-[10px] sm:text-[11px] font-bold text-slate-800 truncate">Periode: <span class="text-emerald-800 font-black">${p.month}</span> • <span>${p.date}</span></div>
                                                </div>
                                            </div>
                                            <div class="font-black text-white bg-emerald-700 px-2.5 py-1.5 rounded-xl border border-emerald-950 text-xs flex-shrink-0 ml-2">Rp ${p.amount.toLocaleString('id-ID')}</div>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>

                            <div class="mt-8">
                                <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center gap-2">
                                    <i class="fa-solid fa-folder-tree text-amber-700"></i> Arsip Folder Riwayat Pembayaran Per Bulan
                                </h4>
                                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3">
                                    ${monthKeys.map(m => `
                                        <div onclick="openMonthFolder('${m}')" class="bg-amber-100 p-4 rounded-3xl border-2 border-amber-400 shadow-md hover:shadow-lg transition cursor-pointer flex items-center justify-between group">
                                            <div class="flex items-center gap-3 min-w-0">
                                                <div class="w-10 h-10 rounded-2xl bg-amber-700 text-white flex items-center justify-center text-lg shadow-md border border-amber-950 group-hover:scale-105 transition flex-shrink-0">
                                                    <i class="fa-solid fa-folder-open"></i>
                                                </div>
                                                <div class="min-w-0">
                                                    <h5 class="font-black text-slate-900 text-xs sm:text-sm truncate">${m}</h5>
                                                    <p class="text-[10px] sm:text-xs font-black text-slate-800 mt-0.5 truncate">${paymentsByMonth[m].length} Riwayat</p>
                                                </div>
                                            </div>
                                            <div class="w-7 h-7 rounded-full bg-amber-700 text-white flex items-center justify-center text-xs font-black group-hover:scale-110 transition flex-shrink-0 ml-2">
                                                <i class="fa-solid fa-chevron-right"></i>
                                            </div>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>
                        `;
                    } else {
                        const folderPayments = paymentsByMonth[currentActiveFolderMonth] || [];
                        paymentsHtml = `
                            <div class="mb-4 flex items-center justify-between">
                                <button onclick="backToMainFolders()" class="px-3.5 py-2 bg-slate-300 hover:bg-slate-400 text-slate-900 font-black text-xs rounded-xl transition flex items-center gap-2 active:scale-95 border-2 border-slate-500">
                                    <i class="fa-solid fa-arrow-left"></i> Kembali
                                </button>
                                <span class="px-3 py-1 bg-amber-700 text-white font-black text-xs rounded-xl border border-amber-950 truncate max-w-[200px]">
                                    <i class="fa-solid fa-folder-open mr-1"></i> ${currentActiveFolderMonth}
                                </span>
                            </div>

                            <div class="bg-amber-50 p-4 sm:p-5 rounded-3xl border-2 border-amber-300">
                                <h4 class="font-black text-slate-900 mb-4 text-xs sm:text-sm flex items-center gap-2 truncate">
                                    <i class="fa-solid fa-folder-open text-amber-700"></i> Arsip Bulan: ${currentActiveFolderMonth}
                                </h4>
                                <div class="space-y-3">
                                    ${folderPayments.length === 0 ? '<p class="text-xs font-black text-slate-700 italic">Tidak ada pembayaran di folder ini.</p>' : folderPayments.map(p => `
                                        <div class="flex items-center justify-between p-3 bg-white rounded-2xl border-2 border-slate-300 text-xs shadow-xs">
                                            <div class="flex items-center gap-3 min-w-0">
                                                <div class="w-9 h-9 rounded-xl bg-amber-700 text-white flex-shrink-0 flex items-center justify-center font-bold shadow-md border border-emerald-950"><i class="fa-solid fa-receipt"></i></div>
                                                <div class="min-w-0">
                                                    <div class="font-black text-slate-900 truncate text-xs">${p.santriName} - ${p.type}</div>
                                                    <div class="text-[10px] font-bold text-slate-800 truncate">Tanggal: <span class="text-emerald-800 font-black">${p.date}</span></div>
                                                </div>
                                            </div>
                                            <div class="font-black text-white bg-amber-700 px-2.5 py-1.5 rounded-xl border border-amber-950 text-xs flex-shrink-0 ml-2">Rp ${p.amount.toLocaleString('id-ID')}</div>
                                        </div>
                                    `).join('')}
                                </div>
                            </div>
                        `;
                    }

                    return `
                        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                            <div class="bg-emerald-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-emerald-900">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-emerald-100 truncate">Total Santri Aktif</span>
                                    <div class="w-10 h-10 rounded-2xl bg-emerald-900 text-white flex items-center justify-center text-base border border-emerald-500 flex-shrink-0"><i class="fa-solid fa-users"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-xl sm:text-2xl font-black truncate">${totalSantri} <span class="text-xs font-bold text-emerald-100">Santri</span></div>
                                    <div class="text-[11px] font-black text-emerald-100 mt-0.5 flex items-center gap-1 truncate"><i class="fa-solid fa-circle-check"></i> ${activeSantri} Aktif</div>
                                </div>
                            </div>

                            <div class="bg-blue-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-blue-900">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-blue-100 truncate">Sudah Bayar</span>
                                    <div class="w-10 h-10 rounded-2xl bg-blue-900 text-white flex items-center justify-center text-base border border-blue-400 flex-shrink-0"><i class="fa-solid fa-circle-check"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-xl sm:text-2xl font-black truncate">${paidThisMonthSantriIds.length} <span class="text-xs font-bold text-blue-100">Santri</span></div>
                                    <div class="text-[11px] font-black text-blue-100 mt-0.5 truncate">Setoran SPP Lunas</div>
                                </div>
                            </div>

                            <div class="bg-purple-800 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-purple-950">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-purple-100 truncate">Santri Beasiswa</span>
                                    <div class="w-10 h-10 rounded-2xl bg-purple-950 text-white flex items-center justify-center text-base border border-purple-500 flex-shrink-0"><i class="fa-solid fa-user-graduate"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-xl sm:text-2xl font-black truncate">${scholarshipSantri} <span class="text-xs font-bold text-purple-100">Santri</span></div>
                                    <div class="text-[11px] font-black text-purple-100 mt-0.5 truncate">Bebas Biaya SPP</div>
                                </div>
                            </div>

                            <div class="bg-red-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-red-950">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-red-100 truncate">Belum Bayar</span>
                                    <div class="w-10 h-10 rounded-2xl bg-red-950 text-white flex items-center justify-center text-base border border-red-400 flex-shrink-0"><i class="fa-solid fa-triangle-exclamation"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-xl sm:text-2xl font-black truncate">${unpaidSantriCount} <span class="text-xs font-bold text-red-100">Santri</span></div>
                                    <div class="text-[11px] font-black text-red-100 mt-0.5 truncate">Tunggakan Periode Ini</div>
                                </div>
                            </div>
                        </div>

                        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                            <div class="lg:col-span-2 bg-white p-5 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                                ${paymentsHtml}
                            </div>

                            <div class="bg-slate-900 p-5 sm:p-6 rounded-3xl text-white shadow-xl flex flex-col justify-between relative overflow-hidden border-2 border-slate-700">
                                <div class="absolute right-0 bottom-0 opacity-10 transform translate-x-8 translate-y-8 text-9xl pointer-events-none">
                                    <i class="fa-solid fa-mosque"></i>
                                </div>
                                <div>
                                    <h3 class="text-lg sm:text-xl font-black mb-2">Selamat Bertugas, Pengurus Pesantren! 👋</h3>
                                    <p class="text-xs text-slate-300 font-bold leading-relaxed mb-6">
                                        Gunakan menu di sebelah kiri untuk mengelola data santri, mencatat setoran SPP bulanan, dan memantau daftar santri yang belum bayar secara terstruktur.
                                    </p>
                                </div>
                                <div class="grid grid-cols-2 gap-3 pt-4 border-t-2 border-slate-800">
                                    <div class="bg-slate-800 p-3 rounded-2xl border border-slate-700 min-w-0">
                                        <div class="text-base sm:text-lg font-black text-white truncate">${totalSantri}</div>
                                        <div class="text-[10px] font-bold text-slate-400 truncate">Total Santri</div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    `;
                } else if (currentUser && currentUser.role === 'admin') {
                    return `
                        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                            <div class="bg-emerald-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-emerald-900">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-emerald-100 truncate">Total Santri Aktif</span>
                                    <div class="w-10 h-10 rounded-2xl bg-emerald-900 text-white flex items-center justify-center text-base border border-emerald-500 flex-shrink-0"><i class="fa-solid fa-users"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-xl sm:text-2xl font-black truncate">${totalSantri} <span class="text-xs font-bold text-emerald-100">Santri</span></div>
                                    <div class="text-[11px] font-black text-emerald-100 mt-0.5 flex items-center gap-1 truncate"><i class="fa-solid fa-circle-check"></i> ${activeSantri} Aktif</div>
                                </div>
                            </div>

                            <div class="bg-blue-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-blue-900">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-blue-100 truncate">Total Pemasukan</span>
                                    <div class="w-10 h-10 rounded-2xl bg-blue-900 text-white flex items-center justify-center text-base border border-blue-400 flex-shrink-0"><i class="fa-solid fa-arrow-trend-up"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-lg sm:text-xl font-black truncate">Rp ${totalIncome.toLocaleString('id-ID')}</div>
                                    <div class="text-[11px] font-black text-blue-100 mt-0.5 truncate">Akumulasi Kas Masuk</div>
                                </div>
                            </div>

                            <div class="bg-purple-800 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-purple-950">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-purple-100 truncate">Total Pengeluaran</span>
                                    <div class="w-10 h-10 rounded-2xl bg-purple-950 text-white flex items-center justify-center text-base border border-purple-500 flex-shrink-0"><i class="fa-solid fa-arrow-trend-down"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-lg sm:text-xl font-black truncate">Rp ${totalExpense.toLocaleString('id-ID')}</div>
                                    <div class="text-[11px] font-black text-purple-100 mt-0.5 truncate">Akumulasi Kas Keluar</div>
                                </div>
                            </div>

                            <div class="bg-indigo-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white flex flex-col justify-between border-2 border-indigo-900">
                                <div class="flex items-center justify-between">
                                    <span class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-indigo-100 truncate">Saldo Kas Bersih</span>
                                    <div class="w-10 h-10 rounded-2xl bg-indigo-900 text-white flex items-center justify-center text-base border border-indigo-500 flex-shrink-0"><i class="fa-solid fa-wallet"></i></div>
                                </div>
                                <div class="mt-3">
                                    <div class="text-lg sm:text-xl font-black truncate">Rp ${balance.toLocaleString('id-ID')}</div>
                                    <div class="text-[11px] font-black text-indigo-100 mt-0.5 truncate">Sisa Saldo Kas Utama</div>
                                </div>
                            </div>
                        </div>

                        <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                            <div class="lg:col-span-2 bg-white p-5 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                                <h3 class="font-black text-slate-900 text-sm sm:text-base mb-4 flex items-center gap-2">
                                    <i class="fa-solid fa-school text-emerald-700"></i> Informasi & Profil Pesantren Utama
                                </h3>
                                <div class="space-y-3 text-xs sm:text-sm font-black text-slate-800">
                                    <div class="p-3.5 bg-slate-50 rounded-2xl border-2 border-slate-200 flex justify-between items-center">
                                        <span class="text-slate-600">Nama Pesantren:</span>
                                        <span class="text-slate-900">${dbState.profile?.name || '-'}</span>
                                    </div>
                                    <div class="p-3.5 bg-slate-50 rounded-2xl border-2 border-slate-200 flex justify-between items-center">
                                        <span class="text-slate-600">Yayasan Pengelola:</span>
                                        <span class="text-slate-900">${dbState.profile?.foundation || '-'}</span>
                                    </div>
                                    <div class="p-3.5 bg-slate-50 rounded-2xl border-2 border-slate-200 flex justify-between items-center">
                                        <span class="text-slate-600">Tahun Ajaran Aktif:</span>
                                        <span class="text-emerald-700">${dbState.profile?.currentYear || '-'}</span>
                                    </div>
                                    <div class="p-3.5 bg-slate-50 rounded-2xl border-2 border-slate-200 flex justify-between items-center">
                                        <span class="text-slate-600">Alamat:</span>
                                        <span class="text-slate-900 text-right">${dbState.profile?.address || '-'}</span>
                                    </div>
                                </div>
                            </div>

                            <div class="bg-slate-900 p-5 sm:p-6 rounded-3xl text-white shadow-xl flex flex-col justify-between relative overflow-hidden border-2 border-slate-700">
                                <div class="absolute right-0 bottom-0 opacity-10 transform translate-x-8 translate-y-8 text-9xl pointer-events-none">
                                    <i class="fa-solid fa-shield-halved"></i>
                                </div>
                                <div>
                                    <h3 class="text-lg sm:text-xl font-black mb-2">Administrator Utama 👋</h3>
                                    <p class="text-xs text-slate-300 font-bold leading-relaxed mb-6">
                                        Anda memiliki hak akses penuh untuk memantau siklus keuangan, mengelola sandi ruangan, dan memperbarui profil pesantren.
                                    </p>
                                </div>
                                <div class="grid grid-cols-2 gap-3 pt-4 border-t-2 border-slate-800">
                                    <div class="bg-slate-800 p-3 rounded-2xl border border-slate-700 min-w-0">
                                        <div class="text-base sm:text-lg font-black text-white truncate">${totalSantri}</div>
                                        <div class="text-[10px] font-bold text-slate-400 truncate">Total Santri</div>
                                    </div>
                                    <div class="bg-slate-800 p-3 rounded-2xl border border-slate-700 min-w-0">
                                        <div class="text-base sm:text-lg font-black text-white truncate">3</div>
                                        <div class="text-[10px] font-bold text-slate-400 truncate">Ruangan Akses</div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    `;
                }
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'sql_setup') {
                const sqlScript = `
-- ==========================================
-- SKRIP SQL LENGKAP SUPABASE
-- APLIKASI KEUANGAN PESANTREN TERINTEGRASI
-- ==========================================

CREATE TABLE IF NOT EXISTS pesantren_sync (
    id INT PRIMARY KEY,
    payload JSONB NOT NULL,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE pesantren_sync ENABLE ROW LEVEL SECURITY;

DROP POLICY IF EXISTS "Akses publik pesantren_sync" ON pesantren_sync;
CREATE POLICY "Akses publik pesantren_sync" 
ON pesantren_sync 
FOR ALL 
USING (true) 
WITH CHECK (true);
                `.trim();

                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md max-w-4xl mx-auto">
                        <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 mb-4">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base flex items-center gap-2"><i class="fa-solid fa-database text-emerald-700"></i> Setup Skrip SQL Lengkap Supabase</h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Salin skrip SQL di bawah ini dan jalankan pada <strong class="text-slate-900">Supabase SQL Editor</strong> Anda.</p>
                            </div>
                            <button onclick="copySqlScript()" class="px-4 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl shadow-md transition flex items-center gap-2 active:scale-95 flex-shrink-0 border border-emerald-900">
                                <i class="fa-solid fa-copy"></i> Salin Skrip SQL
                            </button>
                        </div>
                        <div class="relative">
                            <textarea id="sql-textarea" rows="10" readonly class="w-full p-4 bg-slate-900 text-emerald-400 font-mono text-xs rounded-2xl border-2 border-slate-700 focus:outline-none">${sqlScript}</textarea>
                        </div>
                        <div class="mt-4 p-3.5 rounded-2xl bg-emerald-100 border-2 border-emerald-400 text-xs font-black text-emerald-950 flex items-center gap-2">
                            <i class="fa-solid fa-circle-check text-emerald-700 text-base"></i>
                            <span class="truncate">Supabase URL: <strong>${SUPABASE_URL}</strong></span>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'reset_data') {
                return `
                    <div class="bg-white p-6 sm:p-8 rounded-3xl border-2 border-red-300 shadow-md max-w-2xl mx-auto text-center">
                        <div class="w-16 h-16 bg-red-700 text-white rounded-3xl flex items-center justify-center text-2xl mx-auto mb-4 shadow-md border border-red-900">
                            <i class="fa-solid fa-triangle-exclamation"></i>
                        </div>
                        <h3 class="font-black text-slate-900 text-base sm:text-lg mb-2">Kosongkan Semua Data & Mulai Dari Awal</h3>
                        <p class="text-xs font-bold text-slate-800 mb-6 leading-relaxed">
                            Fitur ini akan menghapus seluruh data santri, riwayat pembayaran SPP, dan catatan transaksi kas, namun <strong class="text-slate-900">tetap mempertahankan</strong> Profil Pesantren dan Sandi Akses Ruangan. Tindakan ini tidak dapat dibatalkan!
                        </p>
                        <button onclick="confirmResetAllData()" class="px-6 py-3.5 bg-red-700 hover:bg-red-800 text-white font-black rounded-2xl shadow-xl shadow-red-700/40 transition active:scale-95 text-xs sm:text-sm inline-flex items-center gap-2 border-2 border-red-950">
                            <i class="fa-solid fa-trash-arrow-up"></i> Kosongkan & Mulai Dari Awal
                        </button>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'credentials') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md max-w-xl mx-auto">
                        <h3 class="font-black text-slate-900 text-sm sm:text-base mb-2 flex items-center gap-2"><i class="fa-solid fa-key text-amber-700"></i> Kelola Sandi Ruangan</h3>
                        <p class="text-xs font-bold text-slate-800 mb-6">Ubah kata sandi untuk masing-masing ruangan akses sistem.</p>

                        <form onsubmit="updateCredentials(event)" class="space-y-4">
                            <div>
                                <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Sandi Administrator Utama</label>
                                <input type="text" id="pass-admin" value="${dbState.credentials?.admin?.pass || 'admin123'}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                            </div>
                            <div>
                                <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Sandi Admin Pesantren</label>
                                <input type="text" id="pass-pesantren" value="${dbState.credentials?.pesantren?.pass || 'santri123'}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                            </div>
                            <div>
                                <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Sandi Bendahara Pusat</label>
                                <input type="text" id="pass-treasurer" value="${dbState.credentials?.treasurer?.pass || 'pusat123'}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                            </div>
                            <button type="submit" class="w-full py-3.5 bg-amber-700 hover:bg-amber-800 text-white font-black rounded-xl shadow-lg shadow-amber-700/40 transition flex items-center justify-center gap-2 mt-2 text-xs sm:text-sm active:scale-95 border-2 border-amber-950">
                                <i class="fa-solid fa-floppy-disk"></i> Simpan Perubahan Sandi
                            </button>
                        </form>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'profile') {
                return `
                    <div class="bg-white p-5 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md max-w-2xl mx-auto">
                        <h3 class="font-black text-slate-900 text-sm sm:text-base mb-2 flex items-center gap-2"><i class="fa-solid fa-school text-emerald-700"></i> Pengaturan Profil Pesantren & Yayasan</h3>
                        <p class="text-xs font-bold text-slate-800 mb-6">Perbarui informasi nama pesantren, yayasan, alamat, dan nomor kontak resmi.</p>

                        <form onsubmit="updateProfile(event)" class="space-y-4">
                            <div>
                                <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Nama Pesantren</label>
                                <input type="text" id="prof-name" value="${dbState.profile?.name || ''}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                            </div>
                            <div>
                                <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Nama Yayasan</label>
                                <input type="text" id="prof-foundation" value="${dbState.profile?.foundation || ''}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                            </div>
                            <div>
                                <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Alamat Lengkap</label>
                                <textarea id="prof-address" rows="3" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">${dbState.profile?.address || ''}</textarea>
                            </div>
                            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4">
                                <div>
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Nomor Telepon / HP</label>
                                    <input type="text" id="prof-phone" value="${dbState.profile?.phone || ''}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                                </div>
                                <div>
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Tahun Ajaran Aktif</label>
                                    <input type="text" id="prof-year" value="${dbState.profile?.currentYear || ''}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                                </div>
                            </div>
                            <button type="submit" class="w-full py-3.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black rounded-xl shadow-lg shadow-emerald-700/40 transition flex items-center justify-center gap-2 mt-4 text-xs sm:text-sm active:scale-95 border-2 border-emerald-950">
                                <i class="fa-solid fa-floppy-disk"></i> Simpan Profil Pesantren
                            </button>
                        </form>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'pesantren' && currentTab === 'santri') {
                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base">Manajemen Data Santri Pesantren</h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Tambah data santri baru, atur status, dan beasiswa.</p>
                            </div>
                            <button onclick="openAddSantriModal()" class="w-full sm:w-auto px-4 py-3 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs sm:text-sm rounded-xl shadow-md shadow-emerald-700/40 transition flex items-center justify-center gap-2 active:scale-95 border-2 border-emerald-950">
                                <i class="fa-solid fa-user-plus"></i> Tambah Santri Baru
                            </button>
                        </div>

                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm">
                                <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 rounded-l-2xl">ID & Nama Santri</th>
                                        <th class="p-3">Kelas</th>
                                        <th class="p-3">Nominal SPP</th>
                                        <th class="p-3">Status Beasiswa</th>
                                        <th class="p-3 rounded-r-2xl text-center">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${santriList.map(s => `
                                        <tr class="hover:bg-slate-100 transition">
                                            <td class="p-3">
                                                <div class="font-black text-slate-900 text-xs sm:text-sm">${s.name}</div>
                                                <div class="text-[10px] font-bold text-slate-700">ID: ${s.id} | Telp: ${s.phone}</div>
                                            </td>
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${s.class}</td>
                                            <td class="p-3 text-emerald-800 font-black whitespace-nowrap text-xs">Rp ${(s.customSpp !== undefined ? s.customSpp : (dbState.profile?.defaultSpp || 250000)).toLocaleString('id-ID')}</td>
                                            <td class="p-3 whitespace-nowrap">
                                                <span class="px-2.5 py-1 rounded-full text-[10px] sm:text-xs font-black ${s.scholarship === 'Ya' ? 'bg-purple-800 text-white border border-purple-950' : 'bg-slate-800 text-white border border-slate-950'}">
                                                    ${s.scholarship === 'Ya' ? 'Beasiswa (Gratis)' : 'Reguler'}
                                                </span>
                                            </td>
                                            <td class="p-3 text-center whitespace-nowrap">
                                                <button onclick="deleteSantri('${s.id}')" class="px-3 py-1.5 bg-red-700 hover:bg-red-800 text-white text-xs font-black rounded-xl transition active:scale-95 border border-red-950 shadow-xs">
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
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md max-w-3xl mx-auto">
                        <h3 class="font-black text-slate-900 text-sm sm:text-base mb-2 flex items-center gap-2"><i class="fa-solid fa-sliders text-emerald-700"></i> Pengaturan Nominal SPP & Status Beasiswa Santri</h3>
                        <p class="text-[11px] sm:text-xs font-bold text-slate-800 mb-6">Atur nominal SPP bulanan dan ubah status beasiswa santri kapan saja.</p>

                        <div class="space-y-4">
                            ${santriList.map(s => `
                                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 p-4 bg-slate-100 rounded-2xl border-2 border-slate-300 shadow-xs">
                                    <div class="min-w-0">
                                        <div class="font-black text-slate-900 text-xs sm:text-sm truncate">${s.name}</div>
                                        <div class="text-[10px] sm:text-xs font-bold text-slate-700 truncate">${s.class} | Telp: ${s.phone}</div>
                                    </div>
                                    <div class="flex flex-wrap items-center gap-2 flex-shrink-0">
                                        <div class="flex items-center gap-1">
                                            <select id="spp-scholarship-${s.id}" class="px-2.5 py-2 bg-white border-2 border-slate-400 rounded-xl text-xs font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                                                <option value="Tidak" ${s.scholarship === 'Tidak' ? 'selected' : ''}>Reguler</option>
                                                <option value="Ya" ${s.scholarship === 'Ya' ? 'selected' : ''}>Beasiswa</option>
                                            </select>
                                        </div>
                                        <div class="flex items-center gap-1">
                                            <span class="text-xs font-black text-slate-800">Rp</span>
                                            <input type="number" id="spp-santri-${s.id}" value="${s.customSpp !== undefined ? s.customSpp : (dbState.profile?.defaultSpp || 250000)}" class="w-28 sm:w-32 px-3 py-2 bg-white border-2 border-slate-400 rounded-xl text-xs sm:text-sm font-black text-emerald-800 focus:ring-2 focus:ring-emerald-700">
                                        </div>
                                        <button onclick="saveSantriSppAndScholarship('${s.id}')" class="px-3.5 py-2 bg-emerald-700 hover:bg-emerald-800 text-white text-xs font-black rounded-xl transition flex items-center justify-center gap-1 active:scale-95 border-2 border-emerald-950">
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
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base">Pencatatan Pembayaran & Rekam Jejak SPP</h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Catat pembayaran Daftar Ulang & SPP santri.</p>
                            </div>
                            <div class="flex items-center gap-3">
                                <button onclick="openPaymentModal()" class="w-full sm:w-auto px-4 py-3 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs sm:text-sm rounded-xl shadow-md shadow-emerald-700/40 transition flex items-center justify-center gap-2 active:scale-95 border-2 border-emerald-950">
                                    <i class="fa-solid fa-receipt"></i> Catat Pembayaran Baru
                                </button>
                            </div>
                        </div>

                        <div class="mb-8">
                            <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center gap-2"><i class="fa-solid fa-clock-rotate-left text-emerald-700"></i> Riwayat Pembayaran Terbaru</h4>
                            <div class="overflow-x-auto">
                                <table class="w-full text-left text-xs sm:text-sm">
                                    <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                        <tr>
                                            <th class="p-3 rounded-l-2xl">Tanggal</th>
                                            <th class="p-3">Nama Santri</th>
                                            <th class="p-3">Jenis Pembayaran</th>
                                            <th class="p-3">Periode / Bulan</th>
                                            <th class="p-3">Status</th>
                                            <th class="p-3 rounded-r-2xl text-right">Nominal</th>
                                        </tr>
                                    </thead>
                                    <tbody class="divide-y-2 divide-slate-200">
                                        ${paymentList.map(p => `
                                            <tr class="hover:bg-slate-100 transition">
                                                <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${p.date}</td>
                                                <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${p.santriName}</td>
                                                <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 bg-teal-800 text-white rounded-full text-[10px] sm:text-xs font-black border border-teal-950">${p.type}</span></td>
                                                <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${p.month}</td>
                                                <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 bg-emerald-700 text-white rounded-full text-[10px] sm:text-xs font-black border border-emerald-950">${p.status}</span></td>
                                                <td class="p-3 text-right font-black text-emerald-800 whitespace-nowrap text-xs">Rp ${p.amount.toLocaleString('id-ID')}</td>
                                            </tr>
                                        `).join('')}
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'spp_monitor') {
                const paidThisMonthSantriIds = paymentList.filter(p => p.type === 'SPP' && p.month === currentMonth).map(p => p.santriId);
                const sppPayments = paymentList.filter(p => p.type === 'SPP');
                const totalSpp = sppPayments.reduce((acc, curr) => acc + curr.amount, 0);

                return `
                    <div class="space-y-6">
                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 sm:gap-6 mb-6">
                            <div class="bg-emerald-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white border-2 border-emerald-900">
                                <div class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-emerald-100 mb-1">Total Pemasukan Kas</div>
                                <div class="text-lg sm:text-2xl font-black truncate">Rp ${totalIncome.toLocaleString('id-ID')}</div>
                            </div>
                            <div class="bg-red-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white border-2 border-red-900">
                                <div class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-red-100 mb-1">Total Pengeluaran Kas</div>
                                <div class="text-lg sm:text-2xl font-black truncate">Rp ${totalExpense.toLocaleString('id-ID')}</div>
                            </div>
                            <div class="bg-indigo-700 p-4 sm:p-5 rounded-3xl shadow-lg text-white border-2 border-indigo-900">
                                <div class="text-[10px] sm:text-xs font-black uppercase tracking-wider text-indigo-100 mb-1">Saldo Kas Bersih</div>
                                <div class="text-lg sm:text-2xl font-black truncate">Rp ${balance.toLocaleString('id-ID')}</div>
                            </div>
                        </div>

                        <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                            <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                                <div>
                                    <h3 class="font-black text-slate-900 text-sm sm:text-base">Monitoring SPP Masuk & Aktivitas Pembayaran</h3>
                                    <p class="text-[11px] sm:text-xs font-bold text-slate-800">Pemantauan setoran SPP bulanan dan daftar pembayaran terbaru.</p>
                                </div>
                                <div class="p-3 bg-emerald-700 text-white rounded-2xl shadow-md text-left sm:text-right w-full sm:w-auto border-2 border-emerald-950">
                                    <div class="text-[10px] font-black uppercase text-emerald-100">Total SPP Masuk</div>
                                    <div class="text-base sm:text-lg font-black truncate">Rp ${totalSpp.toLocaleString('id-ID')}</div>
                                </div>
                            </div>

                            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-6">
                                <div class="p-4 bg-blue-700 text-white rounded-2xl shadow-md border-2 border-blue-950">
                                    <div class="text-[11px] font-black uppercase text-blue-100">Sudah Bayar (${currentMonth})</div>
                                    <div class="text-xl sm:text-2xl font-black mt-1">${paidThisMonthSantriIds.length} Santri</div>
                                </div>
                                <div class="p-4 bg-purple-800 text-white rounded-2xl shadow-md border-2 border-purple-950">
                                    <div class="text-[11px] font-black uppercase text-purple-100">Total Transaksi SPP Tercatat</div>
                                    <div class="text-xl sm:text-2xl font-black mt-1">${sppPayments.length} Transaksi</div>
                                </div>
                            </div>

                            <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm"><i class="fa-solid fa-receipt text-emerald-700 mr-2"></i> Riwayat Pembayaran SPP Terbaru</h4>
                            <div class="overflow-x-auto">
                                <table class="w-full text-left text-xs sm:text-sm">
                                    <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                        <tr>
                                            <th class="p-3 rounded-l-xl">Tanggal</th>
                                            <th class="p-3">Nama Santri</th>
                                            <th class="p-3">Periode</th>
                                            <th class="p-3">Status</th>
                                            <th class="p-3 rounded-r-xl text-right">Nominal</th>
                                        </tr>
                                    </thead>
                                    <tbody class="divide-y-2 divide-slate-200">
                                        ${sppPayments.map(p => `
                                            <tr class="hover:bg-slate-100 transition">
                                                <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${p.date}</td>
                                                <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${p.santriName}</td>
                                                <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${p.month}</td>
                                                <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 bg-emerald-700 text-white font-black text-[10px] rounded-full border border-emerald-950">${p.status}</span></td>
                                                <td class="p-3 text-right font-black text-emerald-800 whitespace-nowrap text-xs">Rp ${p.amount.toLocaleString('id-ID')}</td>
                                            </tr>
                                        `).join('')}
                                    </tbody>
                                </table>
                            </div>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'reports') {
                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base">Data Santri Terdaftar & Nominal SPP</h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Daftar nama santri beserta besaran SPP dan status beasiswa untuk keperluan audit.</p>
                            </div>
                            <button onclick="downloadPdfReport()" class="px-4 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl shadow-md transition flex items-center gap-2 active:scale-95 border-2 border-emerald-950">
                                <i class="fa-solid fa-file-pdf"></i> Unduh PDF Laporan Santri
                            </button>
                        </div>

                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm">
                                <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 rounded-l-2xl">ID & Nama Santri</th>
                                        <th class="p-3">Kelas</th>
                                        <th class="p-3">Nominal SPP</th>
                                        <th class="p-3">Status Beasiswa</th>
                                        <th class="p-3 rounded-r-2xl text-right">No Telepon</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${santriList.map(s => `
                                        <tr class="hover:bg-slate-100 transition">
                                            <td class="p-3">
                                                <div class="font-black text-slate-900 text-xs sm:text-sm">${s.name}</div>
                                                <div class="text-[10px] font-bold text-slate-700">ID: ${s.id}</div>
                                            </td>
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${s.class}</td>
                                            <td class="p-3 text-emerald-800 font-black whitespace-nowrap text-xs">Rp ${(s.customSpp !== undefined ? s.customSpp : (dbState.profile?.defaultSpp || 250000)).toLocaleString('id-ID')}</td>
                                            <td class="p-3 whitespace-nowrap">
                                                <span class="px-2.5 py-1 rounded-full text-[10px] sm:text-xs font-black ${s.scholarship === 'Ya' ? 'bg-purple-800 text-white border border-purple-950' : 'bg-slate-800 text-white border border-slate-950'}">
                                                    ${s.scholarship === 'Ya' ? 'Beasiswa (Gratis)' : 'Reguler'}
                                                </span>
                                            </td>
                                            <td class="p-3 text-right text-slate-900 font-black whitespace-nowrap text-xs">${s.phone}</td>
                                        </tr>
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            }

            return '';
        }

        function confirmResetAllData() {
            showModal('Konfirmasi Kosongkan Data', 'Apakah Anda yakin ingin mengosongkan seluruh data santri, pembayaran, dan transaksi kas? Tindakan ini tidak dapat dibatalkan.', 'error', [
                { text: 'Batal', class: 'bg-slate-300 text-slate-900 hover:bg-slate-400 flex-1 py-3 font-black border-2 border-slate-500 text-xs sm:text-sm', onClick: closeModal },
                { text: 'Ya, Kosongkan', class: 'bg-red-700 text-white hover:bg-red-800 flex-1 py-3 shadow-md shadow-red-700/40 font-black border-2 border-red-950 text-xs sm:text-sm', onClick: () => {
                    dbState.santri = [];
                    dbState.payments = [];
                    dbState.transactions = [];
                    saveDb();
                    closeModal();
                    renderDashboard();
                    showModal('Berhasil Dikosongkan', 'Seluruh data santri, pembayaran, dan transaksi telah dibersihkan. Sistem siap digunakan dari awal.', 'success');
                }}
            ]);
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
                { text: 'Batal', class: 'bg-slate-300 text-slate-900 hover:bg-slate-400 flex-1 py-3 font-black border-2 border-slate-500 text-xs sm:text-sm', onClick: closeModal },
                { text: 'Simpan', class: 'bg-emerald-700 text-white hover:bg-emerald-800 flex-1 py-3 shadow-md shadow-emerald-700/40 font-black border-2 border-emerald-950 text-xs sm:text-sm', onClick: () => {
                    const name = document.getElementById('modal-santri-name').value;
                    const santriClass = document.getElementById('modal-santri-class').value;
                    const customSpp = parseInt(document.getElementById('modal-santri-spp').value) || defaultSpp;
                    const scholarship = document.getElementById('modal-santri-scholarship').value;
                    const phone = document.getElementById('modal-santri-phone').value;

                    if (!name) return;

                    if (!dbState.santri) dbState.santri = [];
                    const newId = 'S00' + (dbState.santri.length + 1);
                    dbState.santri.push({ id: newId, name, class: santriClass, customSpp, status: 'Aktif', scholarship, phone });
                    saveDb();
                    closeModal();
                    renderDashboard();
                    showModal('Berhasil', 'Santri baru berhasil ditambahkan.', 'success');
                }}
            ]);

            document.getElementById('modal-message').innerHTML = `
                <div class="space-y-3 text-left mt-2">
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Nama Lengkap Santri</label>
                        <input type="text" id="modal-santri-name" placeholder="cth: Ahmad Dani" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Kelas</label>
                        <input type="text" id="modal-santri-class" placeholder="cth: VII-A (Tsanawiyah)" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Nominal SPP Bulanan (Rp)</label>
                        <input type="number" id="modal-santri-spp" value="${defaultSpp}" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-emerald-800 focus:ring-2 focus:ring-emerald-700">
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Status Beasiswa</label>
                        <select id="modal-santri-scholarship" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                            <option value="Tidak">Reguler (Tidak Beasiswa)</option>
                            <option value="Ya">Penerima Beasiswa (Gratis SPP)</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Nomor Telepon Wali</label>
                        <input type="text" id="modal-santri-phone" placeholder="cth: 08123456789" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
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
                { text: 'Batal', class: 'bg-slate-300 text-slate-900 hover:bg-slate-400 flex-1 py-3 font-black border-2 border-slate-500 text-xs sm:text-sm', onClick: closeModal },
                { text: 'Catat Pembayaran', class: 'bg-emerald-700 text-white hover:bg-emerald-800 flex-1 py-3 shadow-md shadow-emerald-700/40 font-black border-2 border-emerald-950 text-xs sm:text-sm', onClick: () => {
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

                    dbState.payments.push(newPayment);

                    if (newPayment.amount > 0) {
                        dbState.transactions.push({
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
                    showModal('Berhasil', 'Pembayaran berhasil dicatat dan masuk ke buku kas umum.', 'success');
                }}
            ]);

            document.getElementById('modal-message').innerHTML = `
                <div class="space-y-3 text-left mt-2">
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Tanggal Pembayaran</label>
                        <input type="date" id="pay-date" value="${todayStr}" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Pilih Santri</label>
                        <select id="pay-santri" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                            ${santriOptions}
                        </select>
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Jenis Pembayaran</label>
                        <select id="pay-type" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                            <option value="SPP">SPP Bulanan</option>
                            <option value="Daftar Ulang">Daftar Ulang</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Bulan / Keterangan Periode</label>
                        <input type="text" id="pay-month" value="Agustus 2025" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Nominal (Rp)</label>
                        <input type="number" id="pay-amount" value="${defaultSppVal}" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-emerald-800 focus:ring-2 focus:ring-emerald-700">
                    </div>
                </div>
            `;
        }

        function openTransactionModal() {
            const todayStr = new Date().toISOString().split('T')[0];
            showModal('Catat Kas Pemasukan / Pengeluaran', 'Formulir transaksi kas:', 'info', [
                { text: 'Batal', class: 'bg-slate-300 text-slate-900 hover:bg-slate-400 flex-1 py-3 font-black border-2 border-slate-500 text-xs sm:text-sm', onClick: closeModal },
                { text: 'Simpan Transaksi', class: 'bg-emerald-700 text-white hover:bg-emerald-800 flex-1 py-3 shadow-md shadow-emerald-700/40 font-black border-2 border-emerald-950 text-xs sm:text-sm', onClick: () => {
                    const type = document.getElementById('trx-type').value;
                    const category = document.getElementById('trx-cat').value;
                    const amount = parseInt(document.getElementById('trx-amount').value) || 0;
                    const desc = document.getElementById('trx-desc').value;

                    if (!category || !amount) return;

                    if (!dbState.transactions) dbState.transactions = [];
                    dbState.transactions.push({
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
                <div class="space-y-3 text-left mt-2">
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Jenis Arus Kas</label>
                        <select id="trx-type" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                            <option value="Pemasukan">Pemasukan Kas</option>
                            <option value="Pengeluaran">Pengeluaran Kas</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Kategori Transaksi</label>
                        <input type="text" id="trx-cat" placeholder="cth: Operasional / Donasi" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Nominal (Rp)</label>
                        <input type="number" id="trx-amount" placeholder="500000" class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700">
                    </div>
                    <div>
                        <label class="block text-[11px] font-black uppercase text-slate-900 mb-1">Keterangan / Deskripsi</label>
                        <textarea id="trx-desc" rows="2" placeholder="Catatan atau rincian transaksi..." class="w-full px-3 py-2.5 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700"></textarea>
                    </div>
                </div>
            `;
        }

        function downloadPdfReport() {
            try {
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF();

                doc.setFont("helvetica", "bold");
                doc.setFontSize(16);
                doc.text(dbState.profile?.name || "Pesantren Darul Ulum", 14, 18);
                
                doc.setFontSize(10);
                doc.setFont("helvetica", "normal");
                doc.text(dbState.profile?.foundation || "", 14, 24);
                doc.text(dbState.profile?.address || "", 14, 30);
                doc.text("Laporan Resmi Keuangan & Data Santri", 14, 38);

                const santriRows = (dbState.santri || []).map((s, idx) => [
                    idx + 1,
                    s.name,
                    s.class,
                    s.scholarship === 'Ya' ? 'Beasiswa' : 'Reguler',
                    'Rp ' + (s.customSpp !== undefined ? s.customSpp : 250000).toLocaleString('id-ID'),
                    s.phone
                ]);

                doc.autoTable({
                    startY: 45,
                    head: [['No', 'Nama Santri', 'Kelas', 'Status', 'Nominal SPP', 'Telepon']],
                    body: santriRows,
                    theme: 'grid',
                    headStyles: { fillColor: [4, 120, 87] }
                });

                doc.save("Laporan-Keuangan-Pesantren.pdf");
                showModal('Berhasil Unduh PDF', 'File laporan PDF berhasil diunduh.', 'success');
            } catch (err) {
                console.error("Gagal export PDF:", err);
                showModal('Gagal', 'Terjadi kesalahan saat menghasilkan PDF.', 'error');
            }
        }

        function downloadTransactionPdf() {
            try {
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF();

                doc.setFont("helvetica", "bold");
                doc.setFontSize(16);
                doc.text(dbState.profile?.name || "Pesantren Darul Ulum", 14, 18);
                
                doc.setFontSize(10);
                doc.setFont("helvetica", "normal");
                doc.text(dbState.profile?.foundation || "", 14, 24);
                doc.text("Buku Kas Umum & Histori Transaksi Keuangan", 14, 32);

                let sortedChronological = [...(dbState.transactions || [])].sort((a, b) => new Date(a.date) - new Date(b.date));
                let running = 0;
                const txRows = sortedChronological.map((t, idx) => {
                    if (t.type === 'Pemasukan') {
                        running += t.amount;
                    } else if (t.type === 'Pengeluaran') {
                        running -= t.amount;
                    }
                    return [
                        idx + 1,
                        t.date,
                        t.type,
                        t.category,
                        t.desc,
                        (t.type === 'Pemasukan' ? '+ ' : '- ') + 'Rp ' + t.amount.toLocaleString('id-ID'),
                        'Rp ' + running.toLocaleString('id-ID')
                    ];
                });

                doc.autoTable({
                    startY: 38,
                    head: [['No', 'Tanggal', 'Jenis', 'Kategori', 'Keterangan', 'Nominal', 'Saldo Kas Buku']],
                    body: txRows,
                    theme: 'grid',
                    headStyles: { fillColor: [4, 120, 87] }
                });

                doc.save("Buku-Kas-Umum.pdf");
                showModal('Berhasil Unduh PDF', 'File buku kas PDF berhasil diunduh.', 'success');
            } catch (err) {
                console.error("Gagal export PDF transaksi:", err);
                showModal('Gagal', 'Terjadi kesalahan saat menghasilkan PDF buku kas.', 'error');
            }
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
                icon.className = 'w-14 h-14 rounded-2xl bg-emerald-700 text-white flex items-center justify-center text-2xl mb-4 mx-auto shadow-md border border-emerald-950';
                icon.innerHTML = '<i class="fa-solid fa-check"></i>';
            } else if (type === 'error') {
                icon.className = 'w-14 h-14 rounded-2xl bg-red-700 text-white flex items-center justify-center text-2xl mb-4 mx-auto shadow-md border border-red-950';
                icon.innerHTML = '<i class="fa-solid fa-triangle-exclamation"></i>';
            } else {
                icon.className = 'w-14 h-14 rounded-2xl bg-emerald-700 text-white flex items-center justify-center text-2xl mb-4 mx-auto shadow-md border border-emerald-950';
                icon.innerHTML = '<i class="fa-solid fa-info"></i>';
            }

            if (actions.length === 0) {
                actEl.innerHTML = `<button onclick="closeModal()" class="w-full py-3.5 bg-emerald-700 text-white font-black rounded-xl text-xs sm:text-sm hover:bg-emerald-800 transition shadow-lg shadow-emerald-700/40 active:scale-95 border-2 border-emerald-950">Tutup</button>`;
            } else {
                actEl.innerHTML = actions.map((a, idx) => `
                    <button id="modal-act-${idx}" class="px-4 py-3 font-black rounded-xl text-xs sm:text-sm transition active:scale-95 ${a.class}">${a.text}</button>
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

        window.onload = function() {
            try {
                fetchCloudData().catch(err => console.warn("Cloud sync warning:", err));
                initRealtimeUpdates();
            } catch (err) {
                console.warn("Startup initialization warning:", err);
            }
            renderAuthPortal();
        };
    </script>
</body>
</html>
