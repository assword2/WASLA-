<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>NEXUS POS — Cafe & Restaurant Operating System</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/lucide@latest"></script>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&family=JetBrains+Mono:wght@500;700&display=swap" rel="stylesheet">
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          fontFamily: {
            sans: ['Inter', 'sans-serif'],
            mono: ['JetBrains Mono', 'monospace'],
          },
          colors: {
            void: '#07080c',
            obsidian: '#0d1017',
            slateCard: 'rgba(19, 24, 36, 0.72)',
            neonOrange: '#ff5e1a',
            neonPink: '#ff1e7a',
            neonRed: '#ff2a4b',
            neonCyan: '#00e5ff',
            neonEmerald: '#10b981',
          },
          boxShadow: {
            'neon-orange': '0 0 20px rgba(255, 94, 26, 0.35)',
            'neon-pink': '0 0 20px rgba(255, 30, 122, 0.35)',
          }
        }
      }
    }
  </script>
  <style>
    body {
      background-color: #07080c;
      background-image: 
        radial-gradient(circle at 12% 15%, rgba(255, 94, 26, 0.08) 0%, transparent 38%),
        radial-gradient(circle at 88% 82%, rgba(255, 30, 122, 0.08) 0%, transparent 40%);
      color: #f1f5f9;
      overflow-x: hidden;
    }

    /* Glassmorphism Surface */
    .glass {
      background: rgba(16, 20, 30, 0.72);
      backdrop-filter: blur(16px);
      -webkit-backdrop-filter: blur(16px);
      border: 1px solid rgba(255, 255, 255, 0.07);
    }

    .glass-hover {
      transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
    }
    .glass-hover:hover {
      border-color: rgba(255, 94, 26, 0.45);
      transform: translateY(-2px);
      box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.6), 0 0 15px rgba(255, 94, 26, 0.15);
    }

    /* Custom Scrollbars */
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: #090b10; }
    ::-webkit-scrollbar-thumb { background: #232a3b; border-radius: 99px; }
    ::-webkit-scrollbar-thumb:hover { background: #ff5e1a; }

    /* Print Thermal Receipt Styles */
    @media print {
      body * { visibility: hidden; }
      #receiptPrintArea, #receiptPrintArea * { visibility: visible; }
      #receiptPrintArea {
        position: fixed;
        left: 0;
        top: 0;
        width: 80mm;
        background: #fff !important;
        color: #000 !important;
        padding: 12px;
        font-family: 'Courier New', monospace;
        font-size: 12px;
        box-shadow: none !important;
        border: none !important;
      }
      .no-print { display: none !important; }
    }

    .nav-btn.active {
      background: linear-gradient(90deg, rgba(255, 94, 26, 0.18), rgba(255, 30, 122, 0.12));
      border-left: 3px solid #ff5e1a;
      color: #fff;
    }
    .nav-btn.active i { color: #ff5e1a; }
  </style>
</head>
<body class="min-h-screen flex flex-col lg:flex-row selection:bg-neonOrange selection:text-white">

  <!-- TOAST NOTIFICATION CONTAINER -->
  <div id="toastContainer" class="fixed top-4 right-4 z-[100] flex flex-col gap-2 pointer-events-none"></div>

  <!-- SIDEBAR NAVIGATION -->
  <aside class="w-full lg:w-64 glass border-b lg:border-b-0 lg:border-r border-white/10 flex flex-col justify-between shrink-0 z-30">
    <div>
      <!-- Brand Header -->
      <div class="p-5 flex items-center justify-between border-b border-white/10">
        <div class="flex items-center gap-3">
          <div class="w-10 h-10 rounded-xl bg-gradient-to-br from-neonOrange to-neonPink flex items-center justify-center shadow-neon-orange font-extrabold text-white text-lg">
            N
          </div>
          <div>
            <h1 id="sidebarCafeName" class="font-bold text-base tracking-tight text-white leading-tight">NEXUS LOUNGE</h1>
            <span class="text-[11px] font-mono text-neonOrange uppercase tracking-widest">POS & ERP v3.4</span>
          </div>
        </div>
        <button id="mobileMenuToggle" class="lg:hidden p-2 rounded-lg bg-white/5 text-slate-300">
          <i data-lucide="menu" class="w-5 h-5"></i>
        </button>
      </div>

      <!-- Nav Links -->
      <nav id="sidebarNav" class="hidden lg:flex flex-col p-3 gap-1">
        <button onclick="navigate('dashboard')" data-route="dashboard" class="nav-btn active flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-400 hover:text-white hover:bg-white/5 transition">
          <i data-lucide="layout-dashboard" class="w-4 h-4"></i>
          <span>Overview & KPIs</span>
        </button>
        <button onclick="navigate('pos')" data-route="pos" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-400 hover:text-white hover:bg-white/5 transition">
          <i data-lucide="monitor-smartphone" class="w-4 h-4"></i>
          <span>POS Terminal</span>
          <span id="navActiveOrdersBadge" class="ml-auto text-[11px] font-mono px-2 py-0.5 rounded-full bg-neonOrange/20 text-neonOrange border border-neonOrange/30">0</span>
        </button>
        <button onclick="navigate('inventory')" data-route="inventory" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-400 hover:text-white hover:bg-white/5 transition">
          <i data-lucide="boxes" class="w-4 h-4"></i>
          <span>Inventory & Stock</span>
          <span id="navLowStockBadge" class="hidden ml-auto text-[11px] font-mono px-2 py-0.5 rounded-full bg-neonRed/20 text-neonRed border border-neonRed/30">!</span>
        </button>
        <button onclick="navigate('finance')" data-route="finance" class="nav-btn flex items-center justify-between px-4 py-3 rounded-xl text-sm font-medium text-slate-400 hover:text-white hover:bg-white/5 transition">
          <div class="flex items-center gap-3">
            <i data-lucide="wallet" class="w-4 h-4"></i>
            <span>Accounting & P&L</span>
          </div>
          <i data-lucide="lock" class="w-3.5 h-3.5 text-slate-500 pin-lock-icon"></i>
        </button>
        <button onclick="navigate('staff')" data-route="staff" class="nav-btn flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-slate-400 hover:text-white hover:bg-white/5 transition">
          <i data-lucide="users" class="w-4 h-4"></i>
          <span>Staff & Shifts</span>
        </button>
        <button onclick="navigate('settings')" data-route="settings" class="nav-btn flex items-center justify-between px-4 py-3 rounded-xl text-sm font-medium text-slate-400 hover:text-white hover:bg-white/5 transition">
          <div class="flex items-center gap-3">
            <i data-lucide="shield-check" class="w-4 h-4"></i>
            <span>Security & Settings</span>
          </div>
          <i data-lucide="lock" class="w-3.5 h-3.5 text-slate-500 pin-lock-icon"></i>
        </button>
      </nav>
    </div>

    <!-- Bottom Shift & Lock Status -->
    <div class="hidden lg:block p-4 border-t border-white/10">
      <div class="glass rounded-xl p-3 mb-3">
        <div class="flex items-center justify-between text-xs mb-1">
          <span class="text-slate-400">Active Cashier</span>
          <span class="flex items-center gap-1 text-emerald-400 font-medium"><span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span> Online</span>
        </div>
        <p id="activeCashierLabel" class="text-sm font-semibold text-white">Omar Tarek (Shift A)</p>
        <p id="liveClock" class="text-[11px] font-mono text-slate-400 mt-1">--:--:--</p>
      </div>
      <button id="lockToggleBtn" onclick="toggleAdminLock()" class="w-full py-2 px-3 rounded-xl bg-white/5 hover:bg-white/10 border border-white/10 text-xs font-medium text-slate-300 flex items-center justify-center gap-2 transition">
        <i data-lucide="lock" class="w-3.5 h-3.5 text-neonPink"></i>
        <span id="lockToggleText">Admin Locked (PIN: 1234)</span>
      </button>
    </div>
  </aside>

  <!-- MAIN CONTENT WORKSPACE -->
  <main class="flex-1 flex flex-col min-w-0">
    <!-- Top Header Bar -->
    <header class="glass border-b border-white/10 px-6 py-3.5 flex flex-wrap items-center justify-between gap-4 sticky top-0 z-20">
      <div>
        <h2 id="pageTitle" class="text-lg font-bold text-white tracking-tight">Executive Overview</h2>
        <p id="pageSubtitle" class="text-xs text-slate-400">Real-time floor operations, revenue pulse & inventory health</p>
      </div>
      <div class="flex items-center gap-3">
        <button onclick="navigate('pos')" class="px-4 py-2 rounded-xl bg-gradient-to-r from-neonOrange to-neonPink text-white text-xs font-bold tracking-wide shadow-neon-orange hover:opacity-95 transition flex items-center gap-2">
