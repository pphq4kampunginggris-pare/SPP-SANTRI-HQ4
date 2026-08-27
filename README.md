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

    <!-- Notification & Dynamic Form Modal -->
    <div id="modal-container" class="fixed inset-0 z-50 flex items-center justify-center bg-black/70 hidden backdrop-blur-sm p-4 transition-all duration-300">
        <div class="bg-white rounded-3xl p-6 sm:p-8 max-w-lg w-full mx-auto shadow-2xl transform transition-all scale-95 opacity-0 duration-300 border-2 border-slate-300 overflow-y-auto max-h-[90vh]" id="modal-box">
            <div id="modal-icon" class="w-14 h-14 rounded-2xl flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner"></div>
            <h3 id="modal-title" class="text-xl font-extrabold text-center text-slate-900 mb-2"></h3>
            <div id="modal-message" class="text-sm font-bold text-slate-800 mb-6 leading-relaxed"></div>
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

        const MONTH_OPTIONS = [
            "Juli 2026", "Agustus 2026", "September 2026", "Oktober 2026", 
            "November 2026", "Desember 2026", "Januari 2027", "Februari 2027", 
            "Maret 2027", "April 2027", "Mei 2027", "Juni 2027"
        ];

        // Automatically set current active month based on system date (August 2026)
        const CURRENT_ACTIVE_MONTH = "Agustus 2026";

        const DEFAULT_STATE = {
            profile: {
                name: "Pesantren Darul Ulum Al-Islamy",
                foundation: "Yayasan Pendidikan Islam Darul Ulum",
                address: "Jl. Pesantren No. 45, Kediri, Jawa Timur",
                phone: "(0354) 555123",
                adminPesantrenPhone: "08123456789",
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
                { id: "P001", santriId: "S001", santriName: "Ahmad Fauzi", type: "SPP", month: "Agustus 2026", amount: 250000, date: "2026-08-05", status: "Lunas" },
                { id: "P002", santriId: "S002", santriName: "Siti Aminah", type: "SPP", month: "Agustus 2026", amount: 0, date: "2026-08-06", status: "Beasiswa (Gratis)" },
                { id: "P003", santriId: "S003", santriName: "Muhammad Alif", type: "Daftar Ulang", month: "Juli 2026", amount: 750000, date: "2026-07-10", status: "Lunas" }
            ],
            transactions: [
                { id: "T001", date: "2026-07-10", type: "Pemasukan", category: "Daftar Ulang", amount: 900000, desc: "Pembayaran Daftar Ulang Santri" },
                { id: "T002", date: "2026-07-15", type: "Pengeluaran", category: "Operasional", amount: 500000, desc: "Servis Mobil Operasional Pesantren" },
                { id: "T003", date: "2026-08-01", type: "Pemasukan", category: "Donasi / Hibah", amount: 1000000, desc: "Dana Hibah Yayasan ke Rekening Bendahara Pusat" }
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

        function showModal(title, message, type = 'info', actionsHtml = '') {
            const container = document.getElementById('modal-container');
            const box = document.getElementById('modal-box');
            const iconEl = document.getElementById('modal-icon');
            const titleEl = document.getElementById('modal-title');
            const msgEl = document.getElementById('modal-message');
            const actionsEl = document.getElementById('modal-actions');

            if (!container || !box) return;

            titleEl.textContent = title;
            if (typeof message === 'string') {
                msgEl.innerHTML = message;
            } else {
                msgEl.innerHTML = '';
                msgEl.appendChild(message);
            }

            if (type === 'error') {
                iconEl.className = 'w-14 h-14 rounded-2xl flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner bg-red-100 text-red-700 border-2 border-red-400';
                iconEl.innerHTML = '<i class="fa-solid fa-triangle-exclamation"></i>';
            } else if (type === 'success') {
                iconEl.className = 'w-14 h-14 rounded-2xl flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner bg-emerald-100 text-emerald-700 border-2 border-emerald-400';
                iconEl.innerHTML = '<i class="fa-solid fa-circle-check"></i>';
            } else {
                iconEl.className = 'w-14 h-14 rounded-2xl flex items-center justify-center text-2xl mb-4 mx-auto shadow-inner bg-indigo-100 text-indigo-700 border-2 border-indigo-400';
                iconEl.innerHTML = '<i class="fa-solid fa-circle-info"></i>';
            }

            if (actionsHtml) {
                actionsEl.innerHTML = actionsHtml;
            } else {
                actionsEl.innerHTML = `
                    <button onclick="closeModal()" class="w-full sm:w-auto px-6 py-3 bg-slate-900 hover:bg-slate-800 text-white font-black text-xs rounded-xl shadow-md transition active:scale-95 border border-slate-700">
                        Tutup
                    </button>
                `;
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
                    { id: 'spp_monitor_admin', label: 'Monitoring SPP & Folder', icon: 'fa-folder-tree' },
                    { id: 'santri', label: 'Data Santri', icon: 'fa-users' },
                    { id: 'unpaid_admin', label: 'Santri Belum Bayar', icon: 'fa-triangle-exclamation' },
                    { id: 'wa_invoice_admin', label: 'Kirim WA & Invoice Kas', icon: 'fa-paper-plane' },
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
            const currentMonth = CURRENT_ACTIVE_MONTH;

            const renderUnpaidSantriTable = () => {
                const paidThisMonthSantriIds = paymentList.filter(p => p.type === 'SPP' && (p.month === currentMonth || (p.month && p.month.includes(currentMonth)))).map(p => p.santriId);
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
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Berikut adalah daftar santri reguler yang belum melakukan setoran SPP periode ini. Nama santri yang sudah dibayar akan otomatis hilang dari daftar ini.</p>
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

            if (currentUser && currentUser.role === 'admin' && currentTab === 'spp_monitor_admin') {
                const paymentsByMonth = {};
                const sortedPayments = [...paymentList].sort((a, b) => new Date(a.date) - new Date(b.date));
                sortedPayments.forEach(p => {
                    const mKey = p.month || 'Lainnya';
                    if (!paymentsByMonth[mKey]) {
                        paymentsByMonth[mKey] = [];
                    }
                    paymentsByMonth[mKey].push(p);
                });
                const monthKeys = Object.keys(paymentsByMonth);

                let contentHtml = '';
                if (currentActiveFolderMonth === null) {
                    contentHtml = `
                        <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                            <div class="mb-6">
                                <h3 class="font-black text-slate-900 text-sm sm:text-base flex items-center gap-2">
                                    <i class="fa-solid fa-money-bill-wave text-emerald-700"></i> Monitoring SPP & Folder Bulanan (Admin Utama)
                                </h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Pantau seluruh catatan pembayaran SPP dan klik folder bulanan untuk melihat rincian pembayaran pada periode tersebut.</p>
                            </div>

                            <div class="mb-8">
                                <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center gap-2">
                                    <i class="fa-solid fa-folder-tree text-amber-700"></i> Arsip Folder Riwayat Pembayaran Bulanan
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

                            <div>
                                <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center gap-2">
                                    <i class="fa-solid fa-receipt text-emerald-700"></i> Keseluruhan Riwayat Pembayaran (Urutan Kronologis)
                                </h4>
                                <div class="overflow-x-auto">
                                    <table class="w-full text-left text-xs sm:text-sm">
                                        <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                            <tr>
                                                <th class="p-3 rounded-l-2xl">Tanggal</th>
                                                <th class="p-3">Nama Santri</th>
                                                <th class="p-3">Jenis</th>
                                                <th class="p-3">Periode Bulan</th>
                                                <th class="p-3">Status</th>
                                                <th class="p-3 rounded-r-2xl text-right">Nominal</th>
                                            </tr>
                                        </thead>
                                        <tbody class="divide-y-2 divide-slate-200">
                                            ${sortedPayments.length === 0 ? `
                                                <tr>
                                                    <td colspan="6" class="p-6 text-center text-slate-500 font-bold">Belum ada catatan pembayaran.</td>
                                                </tr>
                                            ` : sortedPayments.map(p => `
                                                <tr class="hover:bg-slate-100 transition">
                                                    <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs"><i class="fa-regular fa-calendar text-emerald-700 mr-1.5"></i> ${p.date}</td>
                                                    <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${p.santriName}</td>
                                                    <td class="p-3 whitespace-nowrap"><span class="px-2 py-0.5 bg-teal-800 text-white rounded-full text-[10px] font-black">${p.type}</span></td>
                                                    <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${p.month}</td>
                                                    <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 bg-emerald-700 text-white rounded-full text-[10px] font-black">${p.status}</span></td>
                                                    <td class="p-3 text-right font-black text-emerald-800 whitespace-nowrap text-xs">Rp ${p.amount.toLocaleString('id-ID')}</td>
                                                </tr>
                                            `).join('')}
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    `;
                } else {
                    const folderPayments = (paymentsByMonth[currentActiveFolderMonth] || []).sort((a, b) => new Date(a.date) - new Date(b.date));
                    contentHtml = `
                        <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                            <div class="mb-4 flex items-center justify-between">
                                <button onclick="backToMainFolders()" class="px-3.5 py-2 bg-slate-300 hover:bg-slate-400 text-slate-900 font-black text-xs rounded-xl transition flex items-center gap-2 active:scale-95 border-2 border-slate-500">
                                    <i class="fa-solid fa-arrow-left"></i> Kembali ke Folder Utama
                                </button>
                                <span class="px-3 py-1 bg-amber-700 text-white font-black text-xs rounded-xl border border-amber-950 truncate max-w-[250px]">
                                    <i class="fa-solid fa-folder-open mr-1"></i> ${currentActiveFolderMonth}
                                </span>
                            </div>

                            <div class="bg-amber-50 p-4 sm:p-5 rounded-3xl border-2 border-amber-300">
                                <h4 class="font-black text-slate-900 mb-4 text-xs sm:text-sm flex items-center gap-2 truncate">
                                    <i class="fa-solid fa-folder-open text-amber-700"></i> Arsip Folder Periode: ${currentActiveFolderMonth} (Urutan Kronologis)
                                </h4>
                                <div class="overflow-x-auto">
                                    <table class="w-full text-left text-xs sm:text-sm">
                                        <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                            <tr>
                                                <th class="p-3 rounded-l-xl">Tanggal</th>
                                                <th class="p-3">Nama Santri</th>
                                                <th class="p-3">Jenis</th>
                                                <th class="p-3">Periode Bulan</th>
                                                <th class="p-3">Status</th>
                                                <th class="p-3 rounded-r-xl text-right">Nominal</th>
                                            </tr>
                                        </thead>
                                        <tbody class="divide-y-2 divide-slate-200">
                                            ${folderPayments.length === 0 ? '<tr><td colspan="6" class="p-4 text-xs font-black text-slate-700 italic text-center">Tidak ada pembayaran di folder ini.</td></tr>' : folderPayments.map(p => `
                                                <tr class="hover:bg-white transition">
                                                    <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs"><i class="fa-regular fa-calendar text-amber-700 mr-1.5"></i> ${p.date}</td>
                                                    <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${p.santriName}</td>
                                                    <td class="p-3 whitespace-nowrap"><span class="px-2 py-0.5 bg-teal-800 text-white rounded-full text-[10px] font-black">${p.type}</span></td>
                                                    <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${p.month}</td>
                                                    <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 bg-amber-700 text-white rounded-full text-[10px] font-black">${p.status}</span></td>
                                                    <td class="p-3 text-right font-black text-amber-800 whitespace-nowrap text-xs">Rp ${p.amount.toLocaleString('id-ID')}</td>
                                                </tr>
                                            `).join('')}
                                        </tbody>
                                    </table>
                                </div>
                            </div>
                        </div>
                    `;
                }
                return contentHtml;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'cycles') {
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

                let groupedTxs = {};
                txWithRunningBalance.forEach(t => {
                    let mLabel = "Lainnya";
                    MONTH_OPTIONS.forEach(m => {
                        if (t.date && t.date.includes("2026-08") && m.includes("Agustus")) mLabel = m;
                        else if (t.date && t.date.includes("2026-07") && m.includes("Juli")) mLabel = m;
                        else if (t.date && t.date.includes("2026-09") && m.includes("September")) mLabel = m;
                        else if (t.date && t.date.includes("2026-10") && m.includes("Oktober")) mLabel = m;
                        else if (t.date && t.date.includes("2026-11") && m.includes("November")) mLabel = m;
                        else if (t.date && t.date.includes("2026-12") && m.includes("Desember")) mLabel = m;
                        else if (t.date && t.date.includes("2027-01") && m.includes("Januari")) mLabel = m;
                        else if (t.date && t.date.includes("2027-02") && m.includes("Februari")) mLabel = m;
                        else if (t.date && t.date.includes("2027-03") && m.includes("Maret")) mLabel = m;
                        else if (t.date && t.date.includes("2027-04") && m.includes("April")) mLabel = m;
                        else if (t.date && t.date.includes("2027-05") && m.includes("Mei")) mLabel = m;
                        else if (t.date && t.date.includes("2027-06") && m.includes("Juni")) mLabel = m;
                    });
                    if (!groupedTxs[mLabel]) groupedTxs[mLabel] = [];
                    groupedTxs[mLabel].push(t);
                });

                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 mb-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base">Siklus & Rekapitulasi Keuangan Pesantren</h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Riwayat rekam jejak tanggal transaksi dan saldo kas mengalir dari atas (terlama) ke bawah (terbaru).</p>
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

                        <div class="flex items-center justify-between mb-4">
                            <h4 class="font-black text-slate-900 flex items-center gap-2 text-xs sm:text-sm"><i class="fa-solid fa-list-check text-emerald-700"></i> Rekam Jejak Transaksi Berdasarkan Tanggal (Urutan Kronologis)</h4>
                        </div>
                        <div class="overflow-x-auto">
                            <table class="w-full text-left text-xs sm:text-sm">
                                <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                    <tr>
                                        <th class="p-3 rounded-l-2xl">Tanggal Transaksi</th>
                                        <th class="p-3">Jenis</th>
                                        <th class="p-3">Kategori</th>
                                        <th class="p-3">Keterangan</th>
                                        <th class="p-3 text-right">Nominal</th>
                                        <th class="p-3 text-right">Saldo Kas Buku</th>
                                        <th class="p-3 rounded-r-2xl text-center">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${Object.keys(groupedTxs).length === 0 ? `
                                        <tr>
                                            <td colspan="7" class="p-6 text-center text-slate-500 font-bold">Belum ada catatan rekam jejak transaksi.</td>
                                        </tr>
                                    ` : Object.keys(groupedTxs).map(monthName => `
                                        <tr class="bg-emerald-600 text-white">
                                            <td colspan="7" class="p-3.5 font-black text-xs tracking-wider uppercase">
                                                <i class="fa-solid fa-calendar-days mr-2"></i> Periode Bulan & Tahun: ${monthName}
                                            </td>
                                        </tr>
                                        ${groupedTxs[monthName].map(t => `
                                            <tr class="hover:bg-slate-100 transition">
                                                <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs"><i class="fa-regular fa-calendar text-emerald-700 mr-1.5"></i> ${t.date}</td>
                                                <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 rounded-full text-[10px] font-black ${t.type === 'Pemasukan' ? 'bg-emerald-700 text-white border border-emerald-900' : 'bg-red-700 text-white border border-red-900'}">${t.type}</span></td>
                                                <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${t.category}</td>
                                                <td class="p-3 font-black text-slate-800 text-xs">${t.desc}</td>
                                                <td class="p-3 text-right font-black whitespace-nowrap text-xs ${t.type === 'Pemasukan' ? 'text-emerald-800' : 'text-red-700'}">${t.type === 'Pemasukan' ? '+ ' : '- '} Rp ${t.amount.toLocaleString('id-ID')}</td>
                                                <td class="p-3 text-right font-black text-indigo-900 whitespace-nowrap text-xs">Rp ${t.saldoKas.toLocaleString('id-ID')}</td>
                                                <td class="p-3 text-center whitespace-nowrap space-x-1">
                                                    <button onclick="openEditTransactionModal('${t.id}')" class="px-3 py-1.5 bg-amber-600 hover:bg-amber-700 text-white text-xs font-black rounded-xl transition active:scale-95 border border-amber-800 shadow-xs">
                                                        <i class="fa-solid fa-pen-to-square mr-1"></i> Edit
                                                    </button>
                                                    <button onclick="deleteTransaction('${t.id}')" class="px-3 py-1.5 bg-red-700 hover:bg-red-800 text-white text-xs font-black rounded-xl transition active:scale-95 border border-red-950 shadow-xs">
                                                        <i class="fa-solid fa-trash-can mr-1"></i> Hapus
                                                    </button>
                                                </td>
                                            </tr>
                                        `).join('')}
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'admin' && currentTab === 'wa_invoice_admin') {
                const incomeTxs = txList.filter(t => t.type === 'Pemasukan');
                const defaultPhone = dbState.profile?.adminPesantrenPhone || "08123456789";

                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md space-y-6">
                        <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-4 pb-4 border-b-2 border-slate-200">
                            <div>
                                <h3 class="font-black text-slate-900 text-base sm:text-lg flex items-center gap-2">
                                    <i class="fa-solid fa-paper-plane text-emerald-700"></i> Kirim WA & Invoice Kas Bendahara Pusat
                                </h3>
                                <p class="text-xs font-bold text-slate-700 mt-1">
                                    Fitur untuk mengirimkan format pesan WhatsApp bergaya invoice resmi kepada <strong>Admin Pesantren</strong> apabila ada uang masuk ke rekening bendahara pusat, lengkap dengan instruksi penginputan & konfirmasi pertanggungjawaban.
                                </p>
                            </div>
                            <div class="flex-shrink-0">
                                <button onclick="openTransactionModal()" class="px-4 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs sm:text-sm rounded-xl shadow-md transition flex items-center gap-2 active:scale-95 border-2 border-emerald-950">
                                    <i class="fa-solid fa-plus-circle"></i> Input Cash In Baru
                                </button>
                            </div>
                        </div>

                        <!-- Phone Configuration Box -->
                        <div class="bg-emerald-50 border-2 border-emerald-300 p-4 sm:p-5 rounded-2xl flex flex-col md:flex-row md:items-center justify-between gap-4">
                            <div class="flex items-center gap-3">
                                <div class="w-12 h-12 bg-emerald-700 text-white rounded-2xl flex items-center justify-center text-xl flex-shrink-0 shadow-md border border-emerald-900">
                                    <i class="fa-brands fa-whatsapp"></i>
                                </div>
                                <div>
                                    <h4 class="font-black text-slate-900 text-xs sm:text-sm">Nomor WhatsApp Tujuan (Admin Pesantren)</h4>
                                    <p class="text-[11px] font-bold text-slate-700">Notifikasi invoice kas masuk akan dikirimkan ke nomor ini.</p>
                                </div>
                            </div>
                            <div class="flex items-center gap-2 w-full md:w-auto">
                                <input type="text" id="wa-target-phone" value="${defaultPhone}" placeholder="Contoh: 08123456789" class="w-full md:w-64 px-3.5 py-2.5 bg-white border-2 border-slate-400 rounded-xl text-xs font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 transition">
                                <button onclick="saveAdminPesantrenPhone()" class="px-4 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl transition border border-emerald-900 flex-shrink-0 active:scale-95 shadow-sm">
                                    <i class="fa-solid fa-floppy-disk mr-1"></i> Simpan
                                </button>
                            </div>
                        </div>

                        <!-- Income Transactions Table with WA Action -->
                        <div>
                            <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center gap-2">
                                <i class="fa-solid fa-receipt text-emerald-700"></i> Daftar Transaksi Kas Masuk Rekening Bendahara Pusat
                            </h4>
                            <div class="overflow-x-auto">
                                <table class="w-full text-left text-xs sm:text-sm border-collapse">
                                    <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                        <tr>
                                            <th class="p-3 rounded-l-xl">ID Bukti</th>
                                            <th class="p-3">Tanggal</th>
                                            <th class="p-3">Kategori Pemasukan</th>
                                            <th class="p-3">Uraian / Sumber Dana</th>
                                            <th class="p-3 text-right">Nominal</th>
                                            <th class="p-3 rounded-r-xl text-center">Tindakan Kirim WA</th>
                                        </tr>
                                    </thead>
                                    <tbody class="divide-y-2 divide-slate-200">
                                        ${incomeTxs.length === 0 ? `
                                            <tr>
                                                <td colspan="6" class="p-6 text-center text-slate-500 font-bold">Belum ada transaksi uang masuk ke rekening bendahara pusat.</td>
                                            </tr>
                                        ` : incomeTxs.map(t => `
                                            <tr class="hover:bg-slate-100 transition">
                                                <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">
                                                    <span class="px-2 py-0.5 bg-slate-200 text-slate-900 rounded-md font-mono border border-slate-400">INV-${t.id}</span>
                                                </td>
                                                <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs"><i class="fa-regular fa-calendar text-emerald-700 mr-1.5"></i> ${t.date}</td>
                                                <td class="p-3 font-black text-emerald-800 whitespace-nowrap text-xs">${t.category}</td>
                                                <td class="p-3 font-black text-slate-800 text-xs">${t.desc}</td>
                                                <td class="p-3 text-right font-black text-emerald-800 whitespace-nowrap text-xs">Rp ${t.amount.toLocaleString('id-ID')}</td>
                                                <td class="p-3 text-center whitespace-nowrap">
                                                    <div class="flex items-center justify-center gap-1.5">
                                                        <button onclick="previewWaInvoiceModal('${t.id}')" class="px-3 py-1.5 bg-indigo-700 hover:bg-indigo-800 text-white text-xs font-black rounded-xl transition active:scale-95 border border-indigo-950 shadow-xs" title="Pratinjau Format Invoice">
                                                            <i class="fa-solid fa-eye mr-1"></i> Preview Invoice
                                                        </button>
                                                        <button onclick="sendWaInvoice('${t.id}')" class="px-3 py-1.5 bg-emerald-600 hover:bg-emerald-700 text-white text-xs font-black rounded-xl transition active:scale-95 border border-emerald-950 shadow-xs" title="Kirim Langsung via WhatsApp">
                                                            <i class="fa-brands fa-whatsapp mr-1 text-sm"></i> Kirim WA
                                                        </button>
                                                        <button onclick="copyWaInvoiceText('${t.id}')" class="px-3 py-1.5 bg-slate-700 hover:bg-slate-800 text-white text-xs font-black rounded-xl transition active:scale-95 border border-slate-900 shadow-xs" title="Salin Teks Invoice">
                                                            <i class="fa-solid fa-copy"></i>
                                                        </button>
                                                    </div>
                                                </td>
                                            </tr>
                                        `).join('')}
                                    </tbody>
                                </table>
                            </div>
                        </div>

                        <!-- Quick Manual Custom Message Box -->
                        <div class="bg-slate-50 border-2 border-slate-300 p-5 rounded-3xl space-y-4">
                            <h4 class="font-black text-slate-900 text-xs sm:text-sm flex items-center gap-2">
                                <i class="fa-solid fa-pen-ruler text-amber-700"></i> Buat Notifikasi Uang Masuk Manual (Custom)
                            </h4>
                            <p class="text-xs font-bold text-slate-700">Gunakan form di bawah ini jika ingin membuat notifikasi invoice manual secara cepat tanpa memilih dari daftar transaksi.</p>
                            
                            <form onsubmit="handleCustomWaInvoiceSubmit(event)" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4">
                                <div>
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1">Tanggal Terima</label>
                                    <input type="date" id="cust-date" value="${new Date().toISOString().split('T')[0]}" required class="w-full px-3 py-2 bg-white border-2 border-slate-300 rounded-xl text-xs font-black text-slate-900">
                                </div>
                                <div>
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1">Nominal Pemasukan (Rp)</label>
                                    <input type="number" id="cust-amount" placeholder="Contoh: 1500000" required class="w-full px-3 py-2 bg-white border-2 border-slate-300 rounded-xl text-xs font-black text-slate-900">
                                </div>
                                <div>
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1">Kategori / Sumber</label>
                                    <input type="text" id="cust-category" placeholder="Contoh: Donasi Alumni / Hibah" required class="w-full px-3 py-2 bg-white border-2 border-slate-300 rounded-xl text-xs font-black text-slate-900">
                                </div>
                                <div>
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1">Keterangan Transfer</label>
                                    <input type="text" id="cust-desc" placeholder="Contoh: Transfer BRI a.n Bendahara Pusat" required class="w-full px-3 py-2 bg-white border-2 border-slate-300 rounded-xl text-xs font-black text-slate-900">
                                </div>
                                <div class="sm:col-span-2 lg:col-span-4 flex justify-end gap-2 pt-2">
                                    <button type="submit" class="px-5 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl transition border-2 border-emerald-950 shadow-md active:scale-95 flex items-center gap-2">
                                        <i class="fa-brands fa-whatsapp text-sm"></i> Generate & Kirim WA Invoice
                                    </button>
                                </div>
                            </form>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'transactions') {
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

                let groupedTxs = {};
                txWithRunningBalance.forEach(t => {
                    let mLabel = "Lainnya";
                    MONTH_OPTIONS.forEach(m => {
                        if (t.date && t.date.includes("2026-08") && m.includes("Agustus")) mLabel = m;
                        else if (t.date && t.date.includes("2026-07") && m.includes("Juli")) mLabel = m;
                        else if (t.date && t.date.includes("2026-09") && m.includes("September")) mLabel = m;
                        else if (t.date && t.date.includes("2026-10") && m.includes("Oktober")) mLabel = m;
                        else if (t.date && t.date.includes("2026-11") && m.includes("November")) mLabel = m;
                        else if (t.date && t.date.includes("2026-12") && m.includes("Desember")) mLabel = m;
                        else if (t.date && t.date.includes("2027-01") && m.includes("Januari")) mLabel = m;
                        else if (t.date && t.date.includes("2027-02") && m.includes("Februari")) mLabel = m;
                        else if (t.date && t.date.includes("2027-03") && m.includes("Maret")) mLabel = m;
                        else if (t.date && t.date.includes("2027-04") && m.includes("April")) mLabel = m;
                        else if (t.date && t.date.includes("2027-05") && m.includes("Mei")) mLabel = m;
                        else if (t.date && t.date.includes("2027-06") && m.includes("Juni")) mLabel = m;
                    });
                    if (!groupedTxs[mLabel]) groupedTxs[mLabel] = [];
                    groupedTxs[mLabel].push(t);
                });

                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md space-y-6">
                        <div class="flex flex-col lg:flex-row lg:items-center lg:justify-between gap-6">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base flex items-center gap-2">
                                    <i class="fa-solid fa-book text-amber-700"></i> Buku Kas & Rekam Jejak Tanggal Transaksi
                                </h3>
                                <p class="text-[11px] sm:text-xs font-bold text-slate-800">Setiap transaksi dilengkapi form tanggal untuk memantau rekam jejak kronologis secara akurat.</p>
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
                                        <th class="p-3 rounded-l-xl">Tanggal Transaksi</th>
                                        <th class="p-3">Jenis</th>
                                        <th class="p-3">Kategori</th>
                                        <th class="p-3">Uraian / Keterangan</th>
                                        <th class="p-3 text-right">Nominal</th>
                                        <th class="p-3 text-right">Saldo Kas Buku</th>
                                        <th class="p-3 rounded-r-xl text-center">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${Object.keys(groupedTxs).length === 0 ? `
                                        <tr>
                                            <td colspan="7" class="p-6 text-center text-slate-500 font-bold">Belum ada catatan transaksi buku kas.</td>
                                        </tr>
                                    ` : Object.keys(groupedTxs).map(monthName => `
                                        <tr class="bg-emerald-600 text-white">
                                            <td colspan="7" class="p-3.5 font-black text-xs tracking-wider uppercase">
                                                <i class="fa-solid fa-calendar-days mr-2"></i> Periode Bulan & Tahun: ${monthName}
                                            </td>
                                        </tr>
                                        ${groupedTxs[monthName].map(t => `
                                            <tr class="hover:bg-slate-100 transition">
                                                <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs"><i class="fa-regular fa-calendar text-amber-700 mr-1.5"></i> ${t.date}</td>
                                                <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 rounded-full text-[10px] font-black ${t.type === 'Pemasukan' ? 'bg-emerald-700 text-white border border-emerald-950' : 'bg-red-700 text-white border border-emerald-950'}">${t.type}</span></td>
                                                <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${t.category}</td>
                                                <td class="p-3 font-black text-slate-800 text-xs">${t.desc}</td>
                                                <td class="p-3 text-right font-black whitespace-nowrap text-xs ${t.type === 'Pemasukan' ? 'text-emerald-800' : 'text-red-700'}">${t.type === 'Pemasukan' ? '+ ' : '- '} Rp ${t.amount.toLocaleString('id-ID')}</td>
                                                <td class="p-3 text-right font-black text-indigo-900 whitespace-nowrap text-xs">Rp ${t.saldoKas.toLocaleString('id-ID')}</td>
                                                <td class="p-3 text-center whitespace-nowrap space-x-1">
                                                    <button onclick="openEditTransactionModal('${t.id}')" class="px-3 py-1.5 bg-amber-600 hover:bg-amber-700 text-white text-xs font-black rounded-xl transition active:scale-95 border border-amber-800 shadow-xs">
                                                        <i class="fa-solid fa-pen-to-square mr-1"></i> Edit
                                                    </button>
                                                    <button onclick="deleteTransaction('${t.id}')" class="px-3 py-1.5 bg-red-700 hover:bg-red-800 text-white text-xs font-black rounded-xl transition active:scale-95 border border-red-950 shadow-xs">
                                                        <i class="fa-solid fa-trash-can mr-1"></i> Hapus
                                                    </button>
                                                </td>
                                            </tr>
                                        `).join('')}
                                    `).join('')}
                                </tbody>
                            </table>
                        </div>
                    </div>
                `;
            }

            if ((currentUser && currentUser.role === 'pesantren' && currentTab === 'santri') || (currentUser && currentUser.role === 'admin' && currentTab === 'santri')) {
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
                                        <th class="p-3 rounded-l-2xl">Nama Santri</th>
                                        <th class="p-3">Kelas</th>
                                        <th class="p-3">No HP Ortu</th>
                                        <th class="p-3">Nominal SPP</th>
                                        <th class="p-3">Status Beasiswa</th>
                                        <th class="p-3 rounded-r-2xl text-center">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody class="divide-y-2 divide-slate-200">
                                    ${santriList.map(s => `
                                        <tr class="hover:bg-slate-100 transition">
                                            <td class="p-3 whitespace-nowrap">
                                                <div class="font-black text-slate-900 text-xs sm:text-sm">${s.name}</div>
                                                <div class="text-[10px] font-bold text-slate-700">ID: ${s.id}</div>
                                            </td>
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${s.class}</td>
                                            <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${s.phone}</td>
                                            <td class="p-3 text-emerald-800 font-black whitespace-nowrap text-xs">Rp ${(s.customSpp !== undefined ? s.customSpp : (dbState.profile?.defaultSpp || 250000)).toLocaleString('id-ID')}</td>
                                            <td class="p-3 whitespace-nowrap">
                                                <span class="px-2.5 py-1 rounded-full text-[10px] sm:text-xs font-black ${s.scholarship === 'Ya' ? 'bg-purple-800 text-white border border-purple-950' : 'bg-slate-800 text-white border border-slate-950'}">
                                                    ${s.scholarship === 'Ya' ? 'Beasiswa (Gratis)' : 'Reguler'}
                                                </span>
                                            </td>
                                            <td class="p-3 text-center whitespace-nowrap space-x-1">
                                                <button onclick="openEditSantriModal('${s.id}')" class="px-3 py-1.5 bg-amber-600 hover:bg-amber-700 text-white text-xs font-black rounded-xl transition active:scale-95 border border-amber-800 shadow-xs">
                                                    <i class="fa-solid fa-pen-to-square mr-1"></i> Edit
                                                </button>
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

            if (currentTab === 'dashboard') {
                if (currentUser && currentUser.role === 'pesantren') {
                    const paymentsByMonth = {};
                    const sortedPayments = [...paymentList].sort((a, b) => new Date(a.date) - new Date(b.date));
                    sortedPayments.forEach(p => {
                        const mKey = p.month || 'Lainnya';
                        if (!paymentsByMonth[mKey]) {
                            paymentsByMonth[mKey] = [];
                        }
                        paymentsByMonth[mKey].push(p);
                    });

                    const monthKeys = Object.keys(paymentsByMonth);
                    const paidThisMonthSantriIds = paymentList.filter(p => p.type === 'SPP' && (p.month === currentMonth || (p.month && p.month.includes(currentMonth)))).map(p => p.santriId);
                    const regularSantri = santriList.filter(s => s.scholarship !== 'Ya' && s.status === 'Aktif');
                    const unpaidSantriCount = regularSantri.filter(s => !paidThisMonthSantriIds.includes(s.id)).length;

                    let paymentsHtml = '';
                    if (currentActiveFolderMonth === null) {
                        paymentsHtml = `
                            <div class="mb-6">
                                <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center justify-between">
                                    <span class="flex items-center gap-2"><i class="fa-solid fa-clock-rotate-left text-emerald-700"></i> Aktivitas Pembayaran & Rekam Jejak Tanggal</span>
                                    <span class="text-[11px] sm:text-xs font-black text-slate-800">Total: ${sortedPayments.length}</span>
                                </h4>
                                <div class="overflow-x-auto">
                                    <table class="w-full text-left text-xs sm:text-sm">
                                        <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                            <tr>
                                                <th class="p-3 rounded-l-2xl">Tanggal</th>
                                                <th class="p-3">Nama Santri</th>
                                                <th class="p-3">Periode Bulan</th>
                                                <th class="p-3">Uang / Nominal</th>
                                                <th class="p-3">Status</th>
                                                <th class="p-3 rounded-r-2xl text-center">Aksi</th>
                                            </tr>
                                        </thead>
                                        <tbody class="divide-y-2 divide-slate-200">
                                            ${sortedPayments.length === 0 ? `
                                                <tr>
                                                    <td colspan="6" class="p-6 text-center text-slate-500 font-bold">Belum ada catatan pembayaran.</td>
                                                </tr>
                                            ` : sortedPayments.map(p => `
                                                <tr class="hover:bg-slate-100 transition">
                                                    <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs"><i class="fa-regular fa-calendar text-emerald-700 mr-1.5"></i> ${p.date}</td>
                                                    <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${p.santriName}</td>
                                                    <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs">${p.month}</td>
                                                    <td class="p-3 font-black text-emerald-800 whitespace-nowrap text-xs">Rp ${p.amount.toLocaleString('id-ID')}</td>
                                                    <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 bg-emerald-700 text-white rounded-full text-[10px] sm:text-xs font-black border border-emerald-950">${p.status}</span></td>
                                                    <td class="p-3 text-center whitespace-nowrap space-x-1">
                                                        <button onclick="openEditPaymentModal('${p.id}')" class="px-3 py-1.5 bg-amber-600 hover:bg-amber-700 text-white text-xs font-black rounded-xl transition active:scale-95 border border-amber-800 shadow-xs">
                                                            <i class="fa-solid fa-pen-to-square mr-1"></i> Edit
                                                        </button>
                                                        <button onclick="deletePayment('${p.id}')" class="px-3 py-1.5 bg-red-700 hover:bg-red-800 text-white text-xs font-black rounded-xl transition active:scale-95 border border-red-950 shadow-xs">
                                                            <i class="fa-solid fa-trash-can mr-1"></i> Hapus
                                                        </button>
                                                    </td>
                                                </tr>
                                            `).join('')}
                                        </tbody>
                                    </table>
                                </div>
                            </div>

                            <div class="mt-8">
                                <h4 class="font-black text-slate-900 mb-3 text-xs sm:text-sm flex items-center gap-2">
                                    <i class="fa-solid fa-folder-tree text-amber-700"></i> Arsip Folder Riwayat Pembayaran (Termasuk Periode Multi-Bulan)
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
                        const folderPayments = (paymentsByMonth[currentActiveFolderMonth] || []).sort((a, b) => new Date(a.date) - new Date(b.date));
                        paymentsHtml = `
                            <div class="mb-4 flex items-center justify-between">
                                <button onclick="backToMainFolders()" class="px-3.5 py-2 bg-slate-300 hover:bg-slate-400 text-slate-900 font-black text-xs rounded-xl transition flex items-center gap-2 active:scale-95 border-2 border-slate-500">
                                    <i class="fa-solid fa-arrow-left"></i> Kembali ke Folder Utama
                                </button>
                                <span class="px-3 py-1 bg-amber-700 text-white font-black text-xs rounded-xl border border-amber-950 truncate max-w-[250px]">
                                    <i class="fa-solid fa-folder-open mr-1"></i> ${currentActiveFolderMonth}
                                </span>
                            </div>

                            <div class="bg-amber-50 p-4 sm:p-5 rounded-3xl border-2 border-amber-300">
                                <h4 class="font-black text-slate-900 mb-4 text-xs sm:text-sm flex items-center gap-2 truncate">
                                    <i class="fa-solid fa-folder-open text-amber-700"></i> Arsip Folder Periode: ${currentActiveFolderMonth}
                                </h4>
                                <div class="overflow-x-auto">
                                    <table class="w-full text-left text-xs sm:text-sm">
                                        <thead class="bg-slate-900 text-white uppercase text-[10px] sm:text-xs font-black tracking-wider">
                                            <tr>
                                                <th class="p-3 rounded-l-xl">Tanggal</th>
                                                <th class="p-3">Nama Santri</th>
                                                <th class="p-3">Periode Bulan</th>
                                                <th class="p-3">Uang / Nominal</th>
                                                <th class="p-3">Status</th>
                                                <th class="p-3 rounded-r-xl text-center">Aksi</th>
                                            </tr>
                                        </thead>
                                        <tbody class="divide-y-2 divide-slate-200">
                                            ${folderPayments.length === 0 ? '<tr><td colspan="6" class="p-4 text-xs font-black text-slate-700 italic text-center">Tidak ada pembayaran di folder ini.</td></tr>' : folderPayments.map(p => `
                                                <tr class="hover:bg-white transition">
                                                    <td class="p-3 text-slate-900 font-black whitespace-nowrap text-xs"><i class="fa-regular fa-calendar text-amber-700 mr-1.5"></i> ${p.date}</td>
                                                    <td class="p-3 font-black text-slate-900 whitespace-nowrap text-xs">${p.santriName}</td>
                                                    <td class="p-3 text-slate-900 font-black text-xs">${p.month}</td>
                                                    <td class="p-3 font-black text-amber-800 whitespace-nowrap text-xs">Rp ${p.amount.toLocaleString('id-ID')}</td>
                                                    <td class="p-3 whitespace-nowrap"><span class="px-2.5 py-1 bg-amber-700 text-white rounded-full text-[10px] font-black border border-amber-950">${p.status}</span></td>
                                                    <td class="p-3 text-center whitespace-nowrap space-x-1">
                                                        <button onclick="openEditPaymentModal('${p.id}')" class="px-3 py-1.5 bg-amber-600 hover:bg-amber-700 text-white text-xs font-black rounded-xl transition active:scale-95 border border-amber-800 shadow-xs">
                                                            <i class="fa-solid fa-pen-to-square mr-1"></i> Edit
                                                        </button>
                                                        <button onclick="deletePayment('${p.id}')" class="px-3 py-1.5 bg-red-700 hover:bg-red-800 text-white text-xs font-black rounded-xl transition active:scale-95 border border-red-950 shadow-xs">
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
                                        Gunakan menu di sebelah kiri untuk mengelola data santri, mencatat setoran SPP, dan memantau rekam jejak keuangan.
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
                                        Anda memiliki hak akses penuh untuk memantau siklus keuangan dengan rekam jejak tanggal, mengelola sandi ruangan, dan memperbarui profil pesantren.
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
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Nomor Telepon / HP Pesantren</label>
                                    <input type="text" id="prof-phone" value="${dbState.profile?.phone || ''}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                                </div>
                                <div>
                                    <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">HP WA Admin Pesantren</label>
                                    <input type="text" id="prof-wa-admin" value="${dbState.profile?.adminPesantrenPhone || '08123456789'}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                                </div>
                            </div>
                            <div>
                                <label class="block text-[11px] font-black uppercase tracking-wider text-slate-900 mb-1.5">Tahun Ajaran Aktif</label>
                                <input type="text" id="prof-year" value="${dbState.profile?.currentYear || ''}" required class="w-full px-4 py-3 bg-slate-100 border-2 border-slate-300 rounded-xl text-xs sm:text-sm font-black text-slate-900 focus:ring-2 focus:ring-emerald-700 focus:bg-white transition">
                            </div>
                            <button type="submit" class="w-full py-3.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black rounded-xl shadow-lg shadow-emerald-700/40 transition flex items-center justify-center gap-2 mt-4 text-xs sm:text-sm active:scale-95 border-2 border-emerald-950">
                                <i class="fa-solid fa-floppy-disk"></i> Simpan Profil Pesantren
                            </button>
                        </form>
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
                                <div class="flex flex-col sm:flex-row sm:items-center justify-between gap-4 p-4 bg-slate-100 rounded-2xl border-2 border-slate-300">
                                    <div>
                                        <div class="font-black text-slate-900 text-xs sm:text-sm">${s.name} (${s.class})</div>
                                        <div class="text-[10px] font-bold text-slate-600">ID: ${s.id} | HP: ${s.phone}</div>
                                    </div>
                                    <div class="flex items-center gap-2">
                                        <button onclick="openEditSantriModal('${s.id}')" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-900 active:scale-95 transition">
                                            <i class="fa-solid fa-pen-to-square mr-1"></i> Ubah SPP / Beasiswa
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
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md max-w-2xl mx-auto space-y-4">
                        <div class="flex items-center justify-between">
                            <div>
                                <h3 class="font-black text-slate-900 text-sm sm:text-base flex items-center gap-2"><i class="fa-solid fa-receipt text-emerald-700"></i> Catat Pembayaran SPP / Biaya Santri</h3>
                                <p class="text-xs font-bold text-slate-700">Input setoran tunai atau transfer dari wali santri.</p>
                            </div>
                            <button onclick="openAddPaymentModal()" class="px-4 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl shadow-md transition border-2 border-emerald-950 active:scale-95 flex items-center gap-2">
                                <i class="fa-solid fa-plus-circle"></i> Catat Pembayaran
                            </button>
                        </div>
                    </div>
                `;
            }

            if (currentUser && currentUser.role === 'treasurer' && currentTab === 'reports') {
                return `
                    <div class="bg-white p-4 sm:p-6 rounded-3xl border-2 border-slate-300 shadow-md">
                        <h3 class="font-black text-slate-900 text-sm sm:text-base mb-4 flex items-center gap-2"><i class="fa-solid fa-user-graduate text-emerald-700"></i> Rekapitulasi Santri & Penerima Beasiswa</h3>
                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 mb-6">
                            <div class="bg-indigo-700 p-4 rounded-2xl text-white font-black border-2 border-indigo-950">
                                <div class="text-xs uppercase text-indigo-100">Total Santri</div>
                                <div class="text-2xl mt-1">${totalSantri} Santri</div>
                            </div>
                            <div class="bg-emerald-700 p-4 rounded-2xl text-white font-black border-2 border-emerald-950">
                                <div class="text-xs uppercase text-emerald-100">Santri Reguler</div>
                                <div class="text-2xl mt-1">${totalSantri - scholarshipSantri} Santri</div>
                            </div>
                            <div class="bg-purple-800 p-4 rounded-2xl text-white font-black border-2 border-purple-950">
                                <div class="text-xs uppercase text-purple-100">Santri Beasiswa</div>
                                <div class="text-2xl mt-1">${scholarshipSantri} Santri</div>
                            </div>
                        </div>
                    </div>
                `;
            }

            return `<div class="bg-white p-6 rounded-3xl text-center font-black text-slate-700">Ruangan aktif dapat diakses melalui menu navigasi samping.</div>`;
        }

        function formatWaNumber(phoneStr) {
            let cleaned = (phoneStr || "").replace(/\D/g, "");
            if (cleaned.startsWith("0")) {
                cleaned = "62" + cleaned.substring(1);
            }
            return cleaned;
        }

        function saveAdminPesantrenPhone() {
            const phoneEl = document.getElementById('wa-target-phone');
            if (phoneEl) {
                const val = phoneEl.value.trim();
                dbState.profile.adminPesantrenPhone = val;
                saveDb();
                showModal('Berhasil Disimpan', `Nomor WhatsApp Admin Pesantren berhasil diperbarui menjadi: <strong>${val}</strong>`, 'success');
            }
        }

        function buildOfficialWaInvoiceText(invId, date, amount, category, desc) {
            const pesName = dbState.profile?.name || "Pesantren Darul Ulum Al-Islamy";
            const formattedAmount = "Rp " + Number(amount).toLocaleString('id-ID');

            return `🧾 *INVOICE & NOTIFIKASI KAS MASUK*
*${pesName.toUpperCase()}*
-----------------------------------------
📌 *No. Bukti Invoice:* INV-${invId}
📅 *Tanggal Terima:* ${date}
💰 *Nominal Masuk:* ${formattedAmount}
🏷️ *Kategori Kas:* ${category}
📝 *Uraian / Sumber:* ${desc}
🏦 *Rekening Tujuan:* Kas Rekening Bendahara Pusat
-----------------------------------------
⚠️ *INSTRUKSI UTAMA KEPADA ADMIN PESANTREN:*
Yth. Admin Pesantren, dimohon untuk segera:
1. Menginput transaksi uang masuk ini ke sistem laporan keuangan internal pesantren.
2. Memberikan konfirmasi pertanggungjawaban kepada Administrator / Bendahara Utama setelah selesai diinput.

_Syukron wa Jazakumullahu Khairan._
-----------------------------------------`;
        }

        function generateWaInvoiceTextForTx(txId) {
            const tx = (dbState.transactions || []).find(t => t.id === txId);
            if (!tx) return "";
            return buildOfficialWaInvoiceText(tx.id, tx.date, tx.amount, tx.category, tx.desc);
        }

        function sendWaInvoice(txId) {
            const text = generateWaInvoiceTextForTx(txId);
            const targetPhone = formatWaNumber(dbState.profile?.adminPesantrenPhone || "08123456789");
            const url = `https://wa.me/${targetPhone}?text=${encodeURIComponent(text)}`;
            window.open(url, '_blank');
        }

        function copyWaInvoiceText(txId) {
            const text = generateWaInvoiceTextForTx(txId);
            const dummy = document.createElement("textarea");
            document.body.appendChild(dummy);
            dummy.value = text;
            dummy.select();
            document.execCommand("copy");
            document.body.removeChild(dummy);
            showModal('Teks Dycetak & Disalin', 'Format pesan WA bergaya invoice resmi berhasil disalin ke clipboard!', 'success');
        }

        function previewWaInvoiceModal(txId) {
            const tx = (dbState.transactions || []).find(t => t.id === txId);
            if (!tx) return;

            const text = buildOfficialWaInvoiceText(tx.id, tx.date, tx.amount, tx.category, tx.desc);

            const contentHtml = `
                <div class="space-y-4 text-left">
                    <div class="p-4 bg-emerald-950 text-emerald-100 rounded-2xl font-mono text-xs leading-relaxed whitespace-pre-wrap border-2 border-emerald-600 shadow-inner">
                        ${text.replace(/\*(.*?)\*/g, '<strong>$1</strong>')}
                    </div>
                </div>
            `;

            const actionsHtml = `
                <button onclick="sendWaInvoice('${tx.id}'); closeModal();" class="w-full sm:w-auto px-5 py-2.5 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl transition border-2 border-emerald-950 flex items-center justify-center gap-2 active:scale-95">
                    <i class="fa-brands fa-whatsapp text-sm"></i> Kirim WhatsApp Now
                </button>
                <button onclick="copyWaInvoiceText('${tx.id}')" class="w-full sm:w-auto px-5 py-2.5 bg-indigo-700 hover:bg-indigo-800 text-white font-black text-xs rounded-xl transition border-2 border-indigo-950 flex items-center justify-center gap-2 active:scale-95">
                    <i class="fa-solid fa-copy"></i> Salin Teks
                </button>
                <button onclick="closeModal()" class="w-full sm:w-auto px-5 py-2.5 bg-slate-300 hover:bg-slate-400 text-slate-900 font-black text-xs rounded-xl transition border-2 border-slate-500 flex items-center justify-center">
                    Batal
                </button>
            `;

            showModal('Pratinjau Format WA Invoice', contentHtml, 'info', actionsHtml);
        }

        function handleCustomWaInvoiceSubmit(e) {
            e.preventDefault();
            const date = document.getElementById('cust-date').value;
            const amount = parseFloat(document.getElementById('cust-amount').value || 0);
            const category = document.getElementById('cust-category').value.trim();
            const desc = document.getElementById('cust-desc').value.trim();

            const customId = "CUST-" + Math.floor(1000 + Math.random() * 9000);
            const text = buildOfficialWaInvoiceText(customId, date, amount, category, desc);

            const targetPhone = formatWaNumber(dbState.profile?.adminPesantrenPhone || "08123456789");
            const url = `https://wa.me/${targetPhone}?text=${encodeURIComponent(text)}`;
            window.open(url, '_blank');
        }

        function openAddSantriModal() {
            const html = `
                <form onsubmit="saveSantri(event)" class="space-y-3 text-left">
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">ID Santri</label>
                        <input type="text" id="santri-id" value="S00${(dbState.santri?.length || 0) + 1}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nama Santri</label>
                        <input type="text" id="santri-name" placeholder="Nama Lengkap..." required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Kelas</label>
                        <input type="text" id="santri-class" placeholder="Contoh: VII-A (Tsanawiyah)" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">No HP Orang Tua / Wali</label>
                        <input type="text" id="santri-phone" placeholder="Contoh: 081234567890" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nominal SPP (Rp)</label>
                        <input type="number" id="santri-spp" value="${dbState.profile?.defaultSpp || 250000}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Status Beasiswa</label>
                        <select id="santri-scholarship" class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                            <option value="Tidak">Tidak (Reguler)</option>
                            <option value="Ya">Ya (Bebas SPP / Gratis)</option>
                        </select>
                    </div>
                    <div class="flex justify-end gap-2 pt-3">
                        <button type="submit" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-950">Simpan Santri</button>
                        <button type="button" onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">Batal</button>
                    </div>
                </form>
            `;
            showModal('Tambah Data Santri', html, 'info', ' ');
        }

        function saveSantri(e) {
            e.preventDefault();
            const id = document.getElementById('santri-id').value.trim();
            const name = document.getElementById('santri-name').value.trim();
            const cls = document.getElementById('santri-class').value.trim();
            const phone = document.getElementById('santri-phone').value.trim();
            const spp = parseFloat(document.getElementById('santri-spp').value || 0);
            const scholarship = document.getElementById('santri-scholarship').value;

            const existingIdx = (dbState.santri || []).findIndex(s => s.id === id);
            const santriObj = { id, name, class: cls, phone, customSpp: spp, status: 'Aktif', scholarship };

            if (existingIdx >= 0) {
                dbState.santri[existingIdx] = santriObj;
            } else {
                dbState.santri.push(santriObj);
            }

            saveDb();
            closeModal();
            renderDashboard();
        }

        function openEditSantriModal(id) {
            const s = (dbState.santri || []).find(item => item.id === id);
            if (!s) return;

            const html = `
                <form onsubmit="saveSantri(event)" class="space-y-3 text-left">
                    <input type="hidden" id="santri-id" value="${s.id}">
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nama Santri</label>
                        <input type="text" id="santri-name" value="${s.name}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Kelas</label>
                        <input type="text" id="santri-class" value="${s.class}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">No HP Orang Tua / Wali</label>
                        <input type="text" id="santri-phone" value="${s.phone}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nominal SPP (Rp)</label>
                        <input type="number" id="santri-spp" value="${s.customSpp !== undefined ? s.customSpp : 250000}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Status Beasiswa</label>
                        <select id="santri-scholarship" class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                            <option value="Tidak" ${s.scholarship !== 'Ya' ? 'selected' : ''}>Tidak (Reguler)</option>
                            <option value="Ya" ${s.scholarship === 'Ya' ? 'selected' : ''}>Ya (Bebas SPP / Gratis)</option>
                        </select>
                    </div>
                    <div class="flex justify-end gap-2 pt-3">
                        <button type="submit" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-950">Update Data</button>
                        <button type="button" onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">Batal</button>
                    </div>
                </form>
            `;
            showModal('Edit Data Santri', html, 'info', ' ');
        }

        function deleteSantri(id) {
            dbState.santri = (dbState.santri || []).filter(s => s.id !== id);
            saveDb();
            renderDashboard();
        }

        function openTransactionModal() {
            const html = `
                <form onsubmit="saveTransaction(event)" class="space-y-3 text-left">
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Tanggal Transaksi</label>
                        <input type="date" id="tx-date" value="${new Date().toISOString().split('T')[0]}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Jenis Transaksi</label>
                        <select id="tx-type" class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                            <option value="Pemasukan">Pemasukan (Uang Masuk ke Rekening Bendahara)</option>
                            <option value="Pengeluaran">Pengeluaran (Kas Keluar)</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Kategori</label>
                        <input type="text" id="tx-cat" placeholder="Contoh: Donasi / Hibah / Operasional" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nominal (Rp)</label>
                        <input type="number" id="tx-amount" placeholder="Contoh: 1000000" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Uraian / Keterangan</label>
                        <input type="text" id="tx-desc" placeholder="Penjelasan transaksi..." required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div class="flex justify-end gap-2 pt-3">
                        <button type="submit" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-950">Simpan Transaksi</button>
                        <button type="button" onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">Batal</button>
                    </div>
                </form>
            `;
            showModal('Tambah Transaksi Kas', html, 'info', ' ');
        }

        function saveTransaction(e) {
            e.preventDefault();
            const date = document.getElementById('tx-date').value;
            const type = document.getElementById('tx-type').value;
            const category = document.getElementById('tx-cat').value.trim();
            const amount = parseFloat(document.getElementById('tx-amount').value || 0);
            const desc = document.getElementById('tx-desc').value.trim();

            const newId = "T" + String((dbState.transactions?.length || 0) + 1).padStart(3, '0');
            dbState.transactions.push({ id: newId, date, type, category, amount, desc });

            saveDb();
            closeModal();
            renderDashboard();

            if (type === 'Pemasukan' && currentUser?.role === 'admin') {
                showModal(
                    'Pemasukan Berhasil Dicatat!',
                    `Transaksi pemasukan sebesar <strong>Rp ${amount.toLocaleString('id-ID')}</strong> telah tersimpan.<br><br>Apakah Anda ingin langsung mengirimkan format <strong>WhatsApp Invoice</strong> kepada Admin Pesantren?`,
                    'success',
                    `
                        <button onclick="sendWaInvoice('${newId}'); closeModal();" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-950 flex items-center gap-1.5">
                            <i class="fa-brands fa-whatsapp text-sm"></i> Ya, Kirim WA Invoice Now
                        </button>
                        <button onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">
                            Nanti Saja
                        </button>
                    `
                );
            }
        }

        function openEditTransactionModal(id) {
            const t = (dbState.transactions || []).find(item => item.id === id);
            if (!t) return;

            const html = `
                <form onsubmit="updateTransaction(event, '${t.id}')" class="space-y-3 text-left">
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Tanggal Transaksi</label>
                        <input type="date" id="tx-edit-date" value="${t.date}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Jenis Transaksi</label>
                        <select id="tx-edit-type" class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                            <option value="Pemasukan" ${t.type === 'Pemasukan' ? 'selected' : ''}>Pemasukan</option>
                            <option value="Pengeluaran" ${t.type === 'Pengeluaran' ? 'selected' : ''}>Pengeluaran</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Kategori</label>
                        <input type="text" id="tx-edit-cat" value="${t.category}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nominal (Rp)</label>
                        <input type="number" id="tx-edit-amount" value="${t.amount}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Uraian / Keterangan</label>
                        <input type="text" id="tx-edit-desc" value="${t.desc}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div class="flex justify-end gap-2 pt-3">
                        <button type="submit" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-950">Update Transaksi</button>
                        <button type="button" onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">Batal</button>
                    </div>
                </form>
            `;
            showModal('Edit Transaksi Kas', html, 'info', ' ');
        }

        function updateTransaction(e, id) {
            e.preventDefault();
            const date = document.getElementById('tx-edit-date').value;
            const type = document.getElementById('tx-edit-type').value;
            const category = document.getElementById('tx-edit-cat').value.trim();
            const amount = parseFloat(document.getElementById('tx-edit-amount').value || 0);
            const desc = document.getElementById('tx-edit-desc').value.trim();

            const idx = (dbState.transactions || []).findIndex(t => t.id === id);
            if (idx >= 0) {
                dbState.transactions[idx] = { id, date, type, category, amount, desc };
                saveDb();
            }
            closeModal();
            renderDashboard();
        }

        function deleteTransaction(id) {
            dbState.transactions = (dbState.transactions || []).filter(t => t.id !== id);
            saveDb();
            renderDashboard();
        }

        function openAddPaymentModal() {
            const santriOpts = (dbState.santri || []).map(s => `<option value="${s.id}">${s.name} (${s.class})</option>`).join('');
            const monthOpts = MONTH_OPTIONS.map(m => `<option value="${m}">${m}</option>`).join('');

            const html = `
                <form onsubmit="savePayment(event)" class="space-y-3 text-left">
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Pilih Santri</label>
                        <select id="pay-santri" class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                            ${santriOpts}
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Periode Bulan</label>
                        <select id="pay-month" class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                            ${monthOpts}
                        </select>
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Jenis Pembayaran</label>
                        <input type="text" id="pay-type" value="SPP" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nominal Setoran (Rp)</label>
                        <input type="number" id="pay-amount" value="250000" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Tanggal Setor</label>
                        <input type="date" id="pay-date" value="${new Date().toISOString().split('T')[0]}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div class="flex justify-end gap-2 pt-3">
                        <button type="submit" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-950">Simpan Setoran</button>
                        <button type="button" onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">Batal</button>
                    </div>
                </form>
            `;
            showModal('Catat Pembayaran SPP', html, 'info', ' ');
        }

        function savePayment(e) {
            e.preventDefault();
            const santriId = document.getElementById('pay-santri').value;
            const santriObj = (dbState.santri || []).find(s => s.id === santriId);
            const month = document.getElementById('pay-month').value;
            const type = document.getElementById('pay-type').value.trim();
            const amount = parseFloat(document.getElementById('pay-amount').value || 0);
            const date = document.getElementById('pay-date').value;

            const newId = "P" + String((dbState.payments?.length || 0) + 1).padStart(3, '0');
            const newPay = {
                id: newId,
                santriId,
                santriName: santriObj ? santriObj.name : 'Santri',
                type,
                month,
                amount,
                date,
                status: 'Lunas'
            };

            dbState.payments.push(newPay);

            // Auto log entry to cashbook
            const newTxId = "T" + String((dbState.transactions?.length || 0) + 1).padStart(3, '0');
            dbState.transactions.push({
                id: newTxId,
                date,
                type: 'Pemasukan',
                category: type,
                amount,
                desc: `Setoran ${type} (${month}) a.n ${santriObj ? santriObj.name : 'Santri'}`
            });

            saveDb();
            closeModal();
            renderDashboard();
        }

        function openEditPaymentModal(id) {
            const p = (dbState.payments || []).find(item => item.id === id);
            if (!p) return;

            const html = `
                <form onsubmit="updatePayment(event, '${p.id}')" class="space-y-3 text-left">
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nama Santri</label>
                        <input type="text" value="${p.santriName}" readonly class="w-full p-2.5 bg-slate-200 border border-slate-400 rounded-xl text-xs font-black text-slate-700">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Periode Bulan</label>
                        <input type="text" id="pay-edit-month" value="${p.month}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Nominal Setoran (Rp)</label>
                        <input type="number" id="pay-edit-amount" value="${p.amount}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div>
                        <label class="block text-xs font-black uppercase text-slate-900 mb-1">Tanggal Setor</label>
                        <input type="date" id="pay-edit-date" value="${p.date}" required class="w-full p-2.5 bg-slate-100 border border-slate-400 rounded-xl text-xs font-black">
                    </div>
                    <div class="flex justify-end gap-2 pt-3">
                        <button type="submit" class="px-4 py-2 bg-emerald-700 hover:bg-emerald-800 text-white font-black text-xs rounded-xl border border-emerald-950">Update Setoran</button>
                        <button type="button" onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">Batal</button>
                    </div>
                </form>
            `;
            showModal('Edit Catatan Pembayaran', html, 'info', ' ');
        }

        function updatePayment(e, id) {
            e.preventDefault();
            const month = document.getElementById('pay-edit-month').value.trim();
            const amount = parseFloat(document.getElementById('pay-edit-amount').value || 0);
            const date = document.getElementById('pay-edit-date').value;

            const idx = (dbState.payments || []).findIndex(p => p.id === id);
            if (idx >= 0) {
                dbState.payments[idx].month = month;
                dbState.payments[idx].amount = amount;
                dbState.payments[idx].date = date;
                saveDb();
            }
            closeModal();
            renderDashboard();
        }

        function deletePayment(id) {
            dbState.payments = (dbState.payments || []).filter(p => p.id !== id);
            saveDb();
            renderDashboard();
        }

        function updateCredentials(e) {
            e.preventDefault();
            const passAdmin = document.getElementById('pass-admin').value.trim();
            const passPesantren = document.getElementById('pass-pesantren').value.trim();
            const passTreasurer = document.getElementById('pass-treasurer').value.trim();

            dbState.credentials.admin.pass = passAdmin;
            dbState.credentials.pesantren.pass = passPesantren;
            dbState.credentials.treasurer.pass = passTreasurer;

            saveDb();
            showModal('Berhasil Disimpan', 'Kata sandi seluruh ruangan telah diperbarui.', 'success');
        }

        function updateProfile(e) {
            e.preventDefault();
            const name = document.getElementById('prof-name').value.trim();
            const foundation = document.getElementById('prof-foundation').value.trim();
            const address = document.getElementById('prof-address').value.trim();
            const phone = document.getElementById('prof-phone').value.trim();
            const adminWa = document.getElementById('prof-wa-admin').value.trim();
            const year = document.getElementById('prof-year').value.trim();

            dbState.profile = {
                ...dbState.profile,
                name,
                foundation,
                address,
                phone,
                adminPesantrenPhone: adminWa,
                currentYear: year
            };

            saveDb();
            showModal('Berhasil Disimpan', 'Informasi profil pesantren telah diperbarui.', 'success');
            renderDashboard();
        }

        function confirmResetAllData() {
            showModal(
                'Konfirmasi Kosongkan Data',
                'Apakah Anda yakin ingin menghapus seluruh data santri, pembayaran SPP, dan transaksi kas?',
                'error',
                `
                    <button onclick="executeResetAllData()" class="px-4 py-2 bg-red-700 hover:bg-red-800 text-white font-black text-xs rounded-xl border border-red-950">Ya, Hapus Semua Data</button>
                    <button onclick="closeModal()" class="px-4 py-2 bg-slate-300 text-slate-900 font-black text-xs rounded-xl">Batal</button>
                `
            );
        }

        function executeResetAllData() {
            dbState.santri = [];
            dbState.payments = [];
            dbState.transactions = [];
            saveDb();
            closeModal();
            showModal('Data Dikosongkan', 'Seluruh data telah dikosongkan. Anda dapat memasukkan data baru dari awal.', 'success');
            renderDashboard();
        }

        function copySqlScript() {
            const textarea = document.getElementById('sql-textarea');
            if (textarea) {
                textarea.select();
                document.execCommand('copy');
                showModal('Berhasil Disalin', 'Skrip SQL Supabase berhasil disalin ke clipboard.', 'success');
            }
        }

        function downloadPdfReport() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();

            const pName = dbState.profile?.name || 'Pesantren Darul Ulum Al-Islamy';
            const fName = dbState.profile?.foundation || 'Yayasan Education';

            doc.setFontSize(14);
            doc.text(pName.toUpperCase(), 14, 15);
            doc.setFontSize(10);
            doc.text(fName, 14, 21);
            doc.text("Laporan Keuangan & Kas Pesantren", 14, 27);
            doc.line(14, 30, 196, 30);

            const txRows = (dbState.transactions || []).map(t => [t.date, t.type, t.category, t.desc, `Rp ${t.amount.toLocaleString('id-ID')}`]);

            doc.autoTable({
                startY: 35,
                head: [['Tanggal', 'Jenis', 'Kategori', 'Keterangan', 'Nominal']],
                body: txRows,
                headStyles: { fillColor: [6, 78, 59] }
            });

            doc.save(`Laporan_Keuangan_Pesantren_${new Date().toISOString().split('T')[0]}.pdf`);
        }

        function downloadTransactionPdf() {
            downloadPdfReport();
        }

        window.onload = function() {
            fetchCloudData();
            initRealtimeUpdates();
            renderAuthPortal();
        };
    </script>
</body>
</html>
