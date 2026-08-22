<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, viewport-fit=cover">
<title>Dairy Ledger</title>
<meta name="theme-color" content="#24382a">
<link rel="manifest" href="manifest.json">
<link rel="icon" href="icon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="icon.svg">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Ledger">
<style>
  :root{
    --paper:#fdf9ee;
    --cream:#faf2df;
    --cream-2:#f2e6c8;
    --ink:#2a2317;
    --ink-soft:#5a4f3c;
    --ink-faint:#8c7f66;
    --forest:#24382a;
    --forest-2:#182a1f;
    --forest-3:#2f4a37;
    --gold:#dc9f2c;
    --gold-light:#f2c869;
    --terracotta:#bd5a2e;
    --terracotta-dark:#8f431f;
    --good:#3f7a4e;
    --bad:#b0432c;
    --line:rgba(42,35,23,0.12);
    --shadow:0 14px 34px -16px rgba(42,35,23,0.28);
    --shadow-lg:0 24px 60px -20px rgba(24,42,31,0.45);
    --ease:cubic-bezier(.16,.84,.44,1);
    --safe-top:env(safe-area-inset-top,0px);
    --safe-bottom:env(safe-area-inset-bottom,0px);
  }
  *{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent;}
  html,body{height:100%;}
  body{
    background:var(--forest-2);
    color:var(--ink);
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,sans-serif;
    font-size:15px;
    line-height:1.45;
    overscroll-behavior-y:none;
    -webkit-font-smoothing:antialiased;
  }
  button{font-family:inherit;cursor:pointer;border:none;background:none;color:inherit;}
  input,select,textarea{font-family:inherit;font-size:16px;color:var(--ink);}
  h1,h2,h3,h4{font-family:Georgia,"Iowan Old Style","Palatino Linotype",Palatino,serif;font-weight:600;letter-spacing:-0.01em;color:var(--ink);}
  ::selection{background:var(--gold);color:var(--forest-2);}
  .num{font-variant-numeric:tabular-nums;}

  .app{
    max-width:480px;
    margin:0 auto;
    min-height:100dvh;
    background:var(--paper);
    position:relative;
    box-shadow:var(--shadow-lg);
    display:flex;
    flex-direction:column;
  }

  /* ---------- top bar ---------- */
  .topbar{
    position:sticky; top:0; z-index:40;
    background:var(--forest);
    color:var(--cream);
    padding-top:calc(var(--safe-top) + 14px);
    box-shadow:0 2px 0 rgba(0,0,0,.08);
  }
  .topbar-inner{
    display:flex; align-items:center; justify-content:space-between;
    padding:2px 20px 14px;
  }
  .topbar h1{
    font-size:21px; color:var(--cream); font-weight:600; letter-spacing:-0.01em;
  }
  .brand-row{display:flex;align-items:center;gap:10px;}
  .brand-mark{
    width:26px;height:26px;border-radius:8px;background:var(--gold);
    display:flex;align-items:center;justify-content:center;flex-shrink:0;
  }
  .brand-mark svg{width:15px;height:15px;}
  .month-chip{
    display:flex; align-items:center; gap:6px;
    background:rgba(250,242,223,0.12); color:var(--cream);
    padding:8px 12px; border-radius:100px; font-weight:700; font-size:13px;
    border:1px solid rgba(250,242,223,0.18);
    transition:background .25s var(--ease);
  }
  .month-chip:active{background:rgba(250,242,223,0.22);}
  .month-chip svg{width:13px;height:13px;opacity:.85;}
  .offline-badge{
    display:none; align-items:center; gap:5px;
    background:rgba(220,159,44,0.22); color:var(--gold-light);
    padding:5px 10px 5px 8px; border-radius:100px; font-size:11px; font-weight:700;
    letter-spacing:.02em; margin-right:8px; border:1px solid rgba(242,200,105,0.3);
  }
  .offline-badge.show{display:inline-flex;}
  .offline-badge svg{width:11px;height:11px;}

  /* ---------- main view ---------- */
  .view{
    flex:1;
    padding:18px 16px calc(120px + var(--safe-bottom));
    overflow-x:hidden;
  }
  .section-title{
    font-size:12.5px; font-weight:700; letter-spacing:.1em; text-transform:uppercase;
    color:var(--ink-faint); margin:22px 2px 10px;
  }
  .section-title:first-child{margin-top:2px;}

  /* ---------- KPI cards ---------- */
  .kpi-grid{display:grid; grid-template-columns:1fr 1fr; gap:10px;}
  .kpi-card{
    background:var(--forest); color:var(--cream);
    border-radius:18px; padding:16px 16px 14px;
    position:relative; overflow:hidden;
    box-shadow:var(--shadow);
  }
  .kpi-card.gold{background:linear-gradient(155deg,var(--gold) 0%, #c98a1f 100%); color:var(--forest-2);}
  .kpi-card.terracotta{background:linear-gradient(155deg,var(--terracotta) 0%, var(--terracotta-dark) 100%); color:var(--paper);}
  .kpi-card.wide{grid-column:1 / -1;}
  .kpi-label{font-size:11.5px; font-weight:700; letter-spacing:.06em; text-transform:uppercase; opacity:.75;}
  .kpi-value{font-family:Georgia,"Iowan Old Style","Palatino Linotype",Palatino,serif; font-size:26px; font-weight:600; margin-top:6px; line-height:1.1;}
  .kpi-sub{font-size:12px; opacity:.7; margin-top:4px;}

  /* ---------- generic cards / list rows ---------- */
  .card{
    background:var(--cream); border-radius:16px; padding:14px 16px;
    box-shadow:0 1px 0 var(--line);
    border:1px solid var(--line);
  }
  .list{display:flex; flex-direction:column; gap:8px;}
  .row{
    display:flex; align-items:center; justify-content:space-between; gap:12px;
    background:var(--cream); border:1px solid var(--line);
    border-radius:14px; padding:12px 14px;
    transition:transform .18s var(--ease), background .18s var(--ease);
  }
  .row:active{transform:scale(.98); background:var(--cream-2);}
  .row-main{display:flex; flex-direction:column; gap:2px; min-width:0;}
  .row-title{font-weight:700; font-size:14.5px; color:var(--ink); white-space:nowrap; overflow:hidden; text-overflow:ellipsis;}
  .row-sub{font-size:12.5px; color:var(--ink-soft);}
  .row-end{display:flex; flex-direction:column; align-items:flex-end; gap:3px; flex-shrink:0;}
  .row-amt{font-weight:800; font-size:15px; color:var(--ink);}
  .badge{
    font-size:10.5px; font-weight:700; letter-spacing:.03em; padding:3px 8px; border-radius:100px;
    text-transform:uppercase;
  }
  .badge.paid{background:rgba(63,122,78,.14); color:var(--good);}
  .badge.partial{background:rgba(220,159,44,.18); color:#93691b;}
  .badge.unpaid{background:rgba(176,67,44,.14); color:var(--bad);}
  .badge.feed{background:rgba(220,159,44,.18); color:#93691b;}
  .badge.labour{background:rgba(189,90,46,.14); color:var(--terracotta-dark);}
  .badge.other{background:rgba(90,79,60,.14); color:var(--ink-soft);}

  .date-heading{
    font-size:12px; font-weight:700; color:var(--ink-faint); text-transform:uppercase; letter-spacing:.06em;
    margin:16px 2px 8px;
  }
  .date-heading:first-child{margin-top:0;}

  .empty{
    text-align:center; padding:44px 20px; color:var(--ink-faint);
  }
  .empty svg{width:40px;height:40px;opacity:.35;margin-bottom:10px;}
  .empty p{font-size:13.5px;}

  /* ---------- dues ---------- */
  .due-row{
    display:flex; align-items:center; justify-content:space-between; gap:10px;
    padding:11px 4px; border-bottom:1px solid var(--line);
  }
  .due-row:last-child{border-bottom:none;}
  .avatar{
    width:34px;height:34px;border-radius:50%;background:var(--forest-3);color:var(--cream);
    display:flex;align-items:center;justify-content:center;font-weight:700;font-size:13px;flex-shrink:0;
  }
  .due-name{font-weight:700;font-size:14px;}
  .due-amt{font-weight:800;font-size:14.5px;color:var(--bad);}
  .due-amt.zero{color:var(--good);}
  .settle-btn{
    font-size:11.5px; font-weight:700; color:var(--forest); background:var(--gold-light);
    padding:6px 10px; border-radius:100px; margin-left:8px; flex-shrink:0;
  }

  /* ---------- buttons ---------- */
  .btn{
    display:inline-flex; align-items:center; justify-content:center; gap:8px;
    padding:14px 20px; border-radius:14px; font-weight:700; font-size:14.5px;
    transition:transform .2s var(--ease), box-shadow .2s var(--ease);
    width:100%;
  }
  .btn-primary{background:var(--forest); color:var(--cream); box-shadow:0 12px 26px -12px rgba(36,56,42,.55);}
  .btn-primary:active{transform:scale(.97);}
  .btn-ghost{background:var(--cream-2); color:var(--ink);}
  .btn-danger{background:rgba(176,67,44,.1); color:var(--bad);}
  .btn-row{display:flex; gap:10px;}
  .btn-row .btn{flex:1;}

  /* ---------- FAB ---------- */
  .fab{
    position:fixed; z-index:45;
    right:calc((100vw - min(480px,100vw))/2 + 18px);
    bottom:calc(88px + var(--safe-bottom));
    width:56px; height:56px; border-radius:50%;
    background:var(--terracotta); color:var(--paper);
    display:flex; align-items:center; justify-content:center;
    box-shadow:0 16px 32px -10px rgba(189,90,46,.6);
    transition:transform .25s var(--ease), opacity .25s var(--ease);
  }
  .fab:active{transform:scale(.92);}
  .fab svg{width:24px;height:24px;}
  .fab.hidden{opacity:0; pointer-events:none; transform:scale(.6);}

  /* ---------- tab bar ---------- */
  .tabbar{
    position:fixed; left:0; right:0; bottom:0; z-index:44;
    max-width:480px; margin:0 auto;
    background:rgba(253,249,238,0.92); backdrop-filter:blur(16px);
    border-top:1px solid var(--line);
    display:flex; padding:8px 4px calc(8px + var(--safe-bottom));
  }
  .tab{
    flex:1; display:flex; flex-direction:column; align-items:center; gap:4px;
    padding:6px 2px; color:var(--ink-faint); border-radius:12px;
    transition:color .2s var(--ease);
  }
  .tab svg{width:21px;height:21px;transition:transform .2s var(--ease);}
  .tab span{font-size:10.5px; font-weight:700; letter-spacing:.01em;}
  .tab.active{color:var(--forest);}
  .tab.active svg{transform:translateY(-1px);}

  /* ---------- bottom sheet ---------- */
  .sheet-overlay{
    position:fixed; inset:0; background:rgba(24,20,12,0.45); z-index:60;
    opacity:0; pointer-events:none; transition:opacity .3s var(--ease);
  }
  .sheet-overlay.open{opacity:1; pointer-events:auto;}
  .sheet{
    position:fixed; left:0; right:0; bottom:0; z-index:61;
    max-width:480px; margin:0 auto;
    background:var(--paper); border-radius:22px 22px 0 0;
    box-shadow:0 -20px 60px -20px rgba(0,0,0,.4);
    padding:10px 20px calc(24px + var(--safe-bottom));
    max-height:88vh; overflow-y:auto;
    transform:translateY(105%);
    transition:transform .38s var(--ease);
  }
  .sheet.open{transform:translateY(0);}
  .sheet-handle{width:38px;height:4px;border-radius:100px;background:var(--line);margin:6px auto 14px;}
  .sheet h2{font-size:19px;margin-bottom:16px;}

  .field{margin-bottom:14px;}
  .field label{display:block; font-size:12px; font-weight:700; color:var(--ink-soft); text-transform:uppercase; letter-spacing:.04em; margin-bottom:6px;}
  .field input, .field select, .field textarea{
    width:100%; padding:12px 14px; border-radius:12px; border:1.5px solid var(--line);
    background:var(--cream); color:var(--ink); outline:none;
    transition:border-color .2s var(--ease);
  }
  .field input:focus, .field select:focus, .field textarea:focus{border-color:var(--forest);}
  .field-row{display:flex; gap:10px;}
  .field-row .field{flex:1;}
  .segmented{display:flex; background:var(--cream); border-radius:12px; padding:4px; border:1.5px solid var(--line);}
  .segmented button{flex:1; padding:9px 6px; border-radius:9px; font-weight:700; font-size:13px; color:var(--ink-soft);}
  .segmented button.active{background:var(--forest); color:var(--cream);}
  .hint{font-size:11.5px; color:var(--ink-faint); margin-top:4px;}

  .mini-cal{background:var(--cream); border:1.5px solid var(--line); border-radius:14px; padding:12px;}
  .mini-cal-head{display:flex; align-items:center; justify-content:space-between; margin-bottom:8px;}
  .mini-cal-title{font-weight:700; font-size:13.5px; color:var(--ink);}
  .cal-nav-btn{width:28px;height:28px;border-radius:9px;background:var(--cream-2);display:flex;align-items:center;justify-content:center;font-size:17px;font-weight:700;color:var(--ink-soft);flex-shrink:0;}
  .cal-nav-btn:active{background:var(--line);}
  .mini-cal-grid{display:grid; grid-template-columns:repeat(7,1fr); gap:3px;}
  .mini-cal-dow{text-align:center; font-size:10px; font-weight:700; color:var(--ink-faint); text-transform:uppercase; padding:2px 0 6px;}
  .cal-day{aspect-ratio:1/1; display:flex; align-items:center; justify-content:center; border-radius:9px; font-size:12.5px; font-weight:700; color:var(--ink); transition:background .15s var(--ease), color .15s var(--ease);}
  .cal-day.other-month{opacity:.28; pointer-events:none;}
  .cal-day.today{box-shadow:inset 0 0 0 1.5px var(--forest-3);}
  .cal-day.selected{background:var(--forest); color:var(--cream);}
  .cal-day.selected.today{box-shadow:inset 0 0 0 1.5px var(--gold-light);}
  .mini-cal-foot{display:flex; align-items:center; justify-content:space-between; margin-top:10px; font-size:12px; color:var(--ink-soft);}
  .mini-cal-foot button{font-weight:700; color:var(--terracotta-dark);}

  .banner{
    display:flex; gap:10px; align-items:flex-start;
    background:rgba(220,159,44,.14); border:1px solid rgba(220,159,44,.35);
    color:#7a5511; border-radius:14px; padding:12px 14px; font-size:12.5px; line-height:1.45;
    margin-bottom:16px;
  }
  .banner svg{width:17px;height:17px;flex-shrink:0;margin-top:1px;}
  .banner b{display:block;font-size:13px;margin-bottom:2px;}

  .toast{
    position:fixed; left:50%; bottom:calc(96px + var(--safe-bottom)); z-index:80;
    transform:translate(-50%,20px); opacity:0; pointer-events:none;
    background:var(--forest-2); color:var(--cream); padding:11px 20px; border-radius:100px;
    font-size:13px; font-weight:700; box-shadow:var(--shadow-lg);
    transition:opacity .25s var(--ease), transform .25s var(--ease);
    white-space:nowrap;
  }
  .toast.show{opacity:1; transform:translate(-50%,0);}

  .chip-row{display:flex; gap:8px; flex-wrap:wrap; margin-bottom:16px;}
  .chip{
    padding:8px 14px; border-radius:100px; background:var(--cream-2); font-size:12.5px; font-weight:700;
    color:var(--ink-soft); border:1px solid transparent;
  }
  .chip.active{background:var(--forest); color:var(--cream);}

  .icon-btn{width:34px;height:34px;border-radius:10px;background:var(--cream-2);display:flex;align-items:center;justify-content:center;flex-shrink:0;}
  .icon-btn svg{width:16px;height:16px;}

  .mini-stat{display:flex; justify-content:space-between; padding:9px 0; border-bottom:1px solid var(--line); font-size:13.5px;}
  .mini-stat:last-child{border-bottom:none;}
  .mini-stat b{font-weight:800;}

  a.link-plain{color:var(--forest); font-weight:700; text-decoration:underline; text-underline-offset:2px;}

  /* ---------- printable statement ---------- */
  #printArea{ display:none; }
  #printArea h1{ font-size:20px; margin-bottom:2px; }
  #printArea .stmt-sub{ font-size:12.5px; color:#444; margin-bottom:18px; }
  #printArea table{ width:100%; border-collapse:collapse; font-size:12.5px; }
  #printArea th, #printArea td{ text-align:left; padding:6px 8px; border-bottom:1px solid #ccc; }
  #printArea th{ text-transform:uppercase; font-size:10.5px; letter-spacing:.04em; color:#555; }
  #printArea td.num, #printArea th.num{ text-align:right; font-variant-numeric:tabular-nums; }
  #printArea .stmt-totals{ margin-top:16px; width:260px; margin-left:auto; }
  #printArea .stmt-totals div{ display:flex; justify-content:space-between; padding:4px 0; font-size:13px; }
  #printArea .stmt-totals .due{ font-weight:800; font-size:15px; border-top:2px solid #000; margin-top:4px; padding-top:8px; }
  #printArea .stmt-gen{ margin-top:24px; font-size:10.5px; color:#777; }
  @media print{
    body{ background:#fff; }
    .app, .tabbar, .fab, .sheet, .sheet-overlay, .toast{ display:none !important; }
    #printArea{ display:block !important; color:#000; background:#fff; max-width:720px; margin:0 auto; padding:20px; }
  }
</style>
</head>
<body>
<div class="app">
  <header class="topbar">
    <div class="topbar-inner">
      <div class="brand-row">
        <div class="brand-mark"><svg viewBox="0 0 24 24" fill="none"><path d="M12 3c4.5 5.6 7 9 7 12.2A7 7 0 1 1 5 15.2C5 12 7.5 8.6 12 3z" fill="#182a1f"/></svg></div>
        <h1 id="pageTitle">Dashboard</h1>
      </div>
      <div style="display:flex;align-items:center;">
        <span class="offline-badge" id="offlineBadge">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><path d="M2 8.5a17 17 0 0 1 20 0M5.5 12.2a12 12 0 0 1 13 0M9 15.8a7 7 0 0 1 6 0"/><path d="M3 3l18 18"/></svg>
          Offline
        </span>
        <button class="month-chip" id="monthBtn">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round"><rect x="3" y="5" width="18" height="16" rx="3"/><path d="M3 10h18M8 3v4M16 3v4"/></svg>
          <span id="monthLabel">Aug 2026</span>
        </button>
      </div>
    </div>
  </header>

  <main class="view" id="view"></main>

  <button class="fab" id="fab">
    <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4" stroke-linecap="round"><path d="M12 5v14M5 12h14"/></svg>
  </button>

  <nav class="tabbar">
    <button class="tab active" data-tab="home">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M4 11.5 12 4l8 7.5"/><path stroke-linecap="round" stroke-linejoin="round" d="M6 10v10h4.5v-6h3v6H18V10"/></svg>
      <span>Home</span>
    </button>
    <button class="tab" data-tab="supply">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 3.5c3.8 4.7 5.8 7.8 5.8 10.3A5.8 5.8 0 1 1 6.2 13.8c0-2.5 2-5.6 5.8-10.3z"/></svg>
      <span>Supply</span>
    </button>
    <button class="tab" data-tab="production">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M4 20.5V10L12 4l8 6v10.5H4z"/><path stroke-linecap="round" stroke-linejoin="round" d="M4 10 12 4l8 6M9.5 20.5V14h5v6.5"/></svg>
      <span>Herd</span>
    </button>
    <button class="tab" data-tab="expenses">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3.5" y="7" width="17" height="12.5" rx="2.5"/><path stroke-linecap="round" d="M3.5 10.5h17M14.5 15h3"/></svg>
      <span>Expenses</span>
    </button>
    <button class="tab" data-tab="more">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.2" stroke-linecap="round"><circle cx="5.5" cy="12" r="1.1" fill="currentColor" stroke="none"/><circle cx="12" cy="12" r="1.1" fill="currentColor" stroke="none"/><circle cx="18.5" cy="12" r="1.1" fill="currentColor" stroke="none"/></svg>
      <span>More</span>
    </button>
  </nav>
</div>

<div class="sheet-overlay" id="sheetOverlay"></div>
<div class="sheet" id="sheet">
  <div class="sheet-handle"></div>
  <div id="sheetContent"></div>
</div>

<div class="toast" id="toast"></div>

<div id="printArea"></div>

<script>
(function(){
  'use strict';

  /* ================= storage ================= */
  var DB_KEY = 'dairyLedgerDB_v1';
  var BACKUP_KEY = 'dairyLedgerLastBackup';
  var db = load();

  function defaultDB(){
    return { clients:[], cattle:[], supply:[], production:[], expenses:[] };
  }
  function load(){
    try{
      var raw = localStorage.getItem(DB_KEY);
      if(!raw) return defaultDB();
      var parsed = JSON.parse(raw);
      var d = defaultDB();
      for(var k in d){ if(Array.isArray(parsed[k])) d[k] = parsed[k]; }
      return d;
    }catch(e){ return defaultDB(); }
  }
  function save(){
    try{
      localStorage.setItem(DB_KEY, JSON.stringify(db));
      return true;
    }catch(e){
      toast('Save failed — storage full or blocked. Export a backup!');
      return false;
    }
  }
  function uid(){ return Date.now().toString(36) + Math.random().toString(36).slice(2,7); }

  /* ================= cloud sync (optional account) =================
     Guest mode (authUser === null) is byte-for-byte the same as before this
     feature existed: every mutation still just calls save(). Signing in is an
     opt-in upgrade layered on top via the syncAfter* wrappers below — nothing
     here ever runs, or loads the Firebase SDK, unless the user has actually
     used the Account feature (see AUTH_FLAG_KEY / restoreSession). */
  var AUTH_FLAG_KEY = 'dairyLedgerAuthUsed';
  var FIREBASE_CONFIG = {
    apiKey: "AIzaSyC4mBfQDO5cZXOWX6vNIshmg4NyJxP2yJI",
    authDomain: "dairy-ledger-8d2e4.firebaseapp.com",
    projectId: "dairy-ledger-8d2e4",
    storageBucket: "dairy-ledger-8d2e4.firebasestorage.app",
    messagingSenderId: "557857723065",
    appId: "1:557857723065:web:b42e234a90a8224d31c168"
  };
  var CLOUD_COLLECTIONS = ['clients','cattle','supply','production','expenses'];
  var FIREBASE_SDK_VERSION = '10.14.1';

  var authUser = null;        // null = guest; Firebase User object once signed in
  var cloudUnsubs = [];       // active onSnapshot() unsubscribe fns
  var fb = null;               // { appMod, authMod, fsMod, app, auth, fs } once loaded
  var firebaseLoadPromise = null;

  function firebaseConfigured(){
    return !!(FIREBASE_CONFIG.apiKey && FIREBASE_CONFIG.projectId);
  }

  function loadFirebase(){
    if(firebaseLoadPromise) return firebaseLoadPromise;
    if(!firebaseConfigured()){
      return Promise.reject(new Error('Cloud sync isn\'t set up yet.'));
    }
    var V = FIREBASE_SDK_VERSION;
    firebaseLoadPromise = Promise.all([
      import('https://www.gstatic.com/firebasejs/'+V+'/firebase-app.js'),
      import('https://www.gstatic.com/firebasejs/'+V+'/firebase-auth.js'),
      import('https://www.gstatic.com/firebasejs/'+V+'/firebase-firestore.js')
    ]).then(function(mods){
      var appMod = mods[0], authMod = mods[1], fsMod = mods[2];
      var app = appMod.initializeApp(FIREBASE_CONFIG);
      var auth = authMod.getAuth(app);
      var fs;
      try{
        fs = fsMod.initializeFirestore(app, {
          localCache: fsMod.persistentLocalCache({ tabManager: fsMod.persistentSingleTabManager({}) })
        });
      }catch(e){
        fs = fsMod.getFirestore(app);
      }
      fb = { appMod:appMod, authMod:authMod, fsMod:fsMod, app:app, auth:auth, fs:fs };
      return fb;
    }).catch(function(err){
      firebaseLoadPromise = null; // allow retrying later (e.g. was offline)
      throw err;
    });
    return firebaseLoadPromise;
  }

  function friendlyAuthError(code){
    var map = {
      'auth/email-already-in-use': 'That email already has an account — try logging in instead.',
      'auth/invalid-email': 'That doesn\'t look like a valid email address.',
      'auth/weak-password': 'Password should be at least 6 characters.',
      'auth/wrong-password': 'Wrong password. Try again or reset it.',
      'auth/invalid-credential': 'Wrong email or password.',
      'auth/user-not-found': 'No account found with that email.',
      'auth/network-request-failed': 'No internet connection — try again once you\'re online.',
      'auth/too-many-requests': 'Too many attempts. Wait a bit and try again.'
    };
    return (code && map[code]) || 'Something went wrong. Please try again.';
  }
  function friendlyAuthErrorEx(err){
    if(err && !err.code) return err.message || 'Something went wrong. Please try again.';
    return friendlyAuthError(err && err.code);
  }

  function setAuthUsedFlag(v){
    if(v) localStorage.setItem(AUTH_FLAG_KEY, '1'); else localStorage.removeItem(AUTH_FLAG_KEY);
  }

  function isDbEmpty(d){
    return CLOUD_COLLECTIONS.every(function(c){ return !(d[c]||[]).length; });
  }

  function detachCloudListeners(){
    cloudUnsubs.forEach(function(u){ u(); });
    cloudUnsubs = [];
  }

  function attachCloudListeners(uidStr){
    detachCloudListeners();
    CLOUD_COLLECTIONS.forEach(function(coll){
      var ref = fb.fsMod.collection(fb.fs, 'users', uidStr, coll);
      var unsub = fb.fsMod.onSnapshot(ref, function(snap){
        db[coll] = snap.docs.map(function(d){ return d.data(); });
        save();
        render();
      }, function(){ toast('Sync error — check your connection'); });
      cloudUnsubs.push(unsub);
    });
  }

  function cloudDocRef(coll, id){
    return fb.fsMod.doc(fb.fs, 'users', authUser.uid, coll, id);
  }
  function cloudUpsert(coll, rec){
    if(!authUser || !fb) return;
    fb.fsMod.setDoc(cloudDocRef(coll, rec.id), rec).catch(function(){});
  }
  function cloudDelete(coll, id){
    if(!authUser || !fb) return;
    fb.fsMod.deleteDoc(cloudDocRef(coll, id)).catch(function(){});
  }
  function commitInChunks(colRef, deleteRefs, putRecs){
    var CHUNK = 400;
    var ops = deleteRefs.map(function(ref){ return {type:'delete', ref:ref}; })
      .concat(putRecs.map(function(rec){ return {type:'set', ref:fb.fsMod.doc(colRef, rec.id), rec:rec}; }));
    var chain = Promise.resolve();
    for(var i=0;i<ops.length;i+=CHUNK){
      (function(slice){
        chain = chain.then(function(){
          var batch = fb.fsMod.writeBatch(fb.fs);
          slice.forEach(function(op){
            if(op.type==='delete') batch.delete(op.ref); else batch.set(op.ref, op.rec);
          });
          return batch.commit();
        });
      })(ops.slice(i, i+CHUNK));
    }
    return chain;
  }
  function cloudReplaceAll(newDb){
    if(!authUser || !fb) return;
    CLOUD_COLLECTIONS.forEach(function(coll){
      var colRef = fb.fsMod.collection(fb.fs, 'users', authUser.uid, coll);
      fb.fsMod.getDocs(colRef).then(function(snap){
        var deleteRefs = []; snap.forEach(function(d){ deleteRefs.push(d.ref); });
        return commitInChunks(colRef, deleteRefs, newDb[coll]||[]);
      }).catch(function(){ toast('Could not sync all changes — try again online'); });
    });
  }
  function migrateLocalToCloud(uidStr, localDb){
    var colRefs = {};
    CLOUD_COLLECTIONS.forEach(function(coll){ colRefs[coll] = fb.fsMod.collection(fb.fs, 'users', uidStr, coll); });
    return Promise.all(CLOUD_COLLECTIONS.map(function(coll){
      return commitInChunks(colRefs[coll], [], localDb[coll]||[]);
    }));
  }

  /* Every mutation site in the app calls one of these three instead of a bare
     save() — while signed out they're a no-op beyond today's save(). */
  function syncAfterWrite(coll, recs){
    save();
    if(authUser) recs.forEach(function(r){ cloudUpsert(coll, r); });
  }
  function syncAfterDelete(coll, id){
    save();
    if(authUser) cloudDelete(coll, id);
  }
  function syncAfterReplaceAll(newDb){
    save();
    if(authUser) cloudReplaceAll(newDb);
  }

  function checkCloudEmpty(uidStr){
    return Promise.all(CLOUD_COLLECTIONS.map(function(coll){
      return fb.fsMod.getDocs(fb.fsMod.collection(fb.fs,'users',uidStr,coll)).then(function(snap){ return snap.empty; });
    })).then(function(results){ return results.every(Boolean); });
  }

  function finishSignIn(user){
    authUser = user;
    setAuthUsedFlag(true);
    attachCloudListeners(user.uid);
    closeSheet();
    toast('Signed in as ' + user.email);
    if(state.tab==='more') render();
  }

  function promptMigration(mode, priorGuestDb, user){
    var counts = CLOUD_COLLECTIONS.map(function(c){ return (priorGuestDb[c]||[]).length + ' ' + c; }).join(', ');
    var body, buttons;
    if(mode==='upload'){
      body = 'Upload your existing data to your account? ('+counts+')';
      buttons = '<div class="btn-row"><button class="btn btn-ghost" id="migCancel">Cancel sign-in</button>' +
        '<button class="btn btn-primary" id="migGo">Upload</button></div>';
    } else {
      body = 'This device has local data that isn\'t part of your account yet. ('+counts+')';
      buttons = '<div class="btn-row"><button class="btn btn-ghost" id="migKeep">Keep account data only</button>' +
        '<button class="btn btn-primary" id="migGo">Merge into account</button></div>' +
        '<button class="btn btn-danger" id="migCancel" style="margin-top:10px;">Cancel sign-in</button>';
    }
    openSheet(
      '<h2>Existing Data Found</h2><p style="font-size:13.5px;color:var(--ink-soft);margin-bottom:18px;">'+esc(body)+'</p>' + buttons,
      function(root){
        var goBtn = root.querySelector('#migGo');
        guardOnce(goBtn, function(){
          migrateLocalToCloud(user.uid, priorGuestDb).then(function(){
            finishSignIn(user);
          }).catch(function(){
            toast('Could not upload your data — try again once online');
            goBtn.disabled = false;
          });
        });
        var keepBtn = root.querySelector('#migKeep');
        if(keepBtn) keepBtn.addEventListener('click', function(){ finishSignIn(user); });
        root.querySelector('#migCancel').addEventListener('click', function(){
          fb.authMod.signOut(fb.auth).then(function(){
            authUser = null; toast('Sign-in cancelled — your local data is unchanged'); closeSheet();
          });
        });
      }
    );
  }

  function performSignIn(isSignUp, email, password){
    var priorGuestDb = { clients:db.clients, cattle:db.cattle, supply:db.supply, production:db.production, expenses:db.expenses };
    return loadFirebase().then(function(f){
      var authCall = isSignUp
        ? f.authMod.createUserWithEmailAndPassword(f.auth, email, password)
        : f.authMod.signInWithEmailAndPassword(f.auth, email, password);
      return authCall.then(function(cred){
        var user = cred.user;
        return checkCloudEmpty(user.uid).then(function(cloudEmpty){
          var localEmpty = isDbEmpty(priorGuestDb);
          if(cloudEmpty && !localEmpty) promptMigration('upload', priorGuestDb, user);
          else if(!cloudEmpty && !localEmpty) promptMigration('merge', priorGuestDb, user);
          else finishSignIn(user);
        });
      });
    }).catch(function(err){
      toast(friendlyAuthErrorEx(err));
    });
  }

  function performSignOut(){
    if(!fb){ return; }
    fb.authMod.signOut(fb.auth).then(function(){
      authUser = null;
      detachCloudListeners();
      setAuthUsedFlag(false);
      db = load();
      toast('Signed out');
      render();
    }).catch(function(){ toast('Could not sign out — try again'); });
  }

  function performPasswordReset(email){
    return loadFirebase().then(function(f){
      return f.authMod.sendPasswordResetEmail(f.auth, email);
    }).then(function(){
      toast('Reset link sent — check your inbox');
    }).catch(function(err){
      toast(friendlyAuthErrorEx(err));
    });
  }

  function restoreSession(){
    if(!firebaseConfigured() || localStorage.getItem(AUTH_FLAG_KEY) !== '1') return;
    loadFirebase().then(function(f){
      var unsub = f.authMod.onAuthStateChanged(f.auth, function(user){
        unsub();
        if(user){
          authUser = user;
          attachCloudListeners(user.uid);
          if(state.tab==='more') render();
        } else {
          setAuthUsedFlag(false);
        }
      });
    }).catch(function(){ /* offline or SDK failed to load — stay on cached local view */ });
  }

  /* ================= state ================= */
  var state = { tab:'home', month: todayISO().slice(0,7) };

  function todayISO(){
    var d = new Date();
    var tz = d.getTimezoneOffset()*60000;
    return new Date(d - tz).toISOString().slice(0,10);
  }

  /* ================= formatting ================= */
  function money(n){
    n = Number(n)||0;
    var neg = n < 0; n = Math.abs(n);
    var s = n.toLocaleString('en-IN', {maximumFractionDigits:2, minimumFractionDigits: (n%1===0?0:2)});
    return (neg?'-':'') + '₹' + s;
  }
  function liters(n){
    n = Number(n)||0;
    return n.toLocaleString('en-IN', {maximumFractionDigits:1}) + ' L';
  }
  function monthLabel(ym){
    var parts = ym.split('-'); var d = new Date(Number(parts[0]), Number(parts[1])-1, 1);
    return d.toLocaleDateString('en-IN', {month:'short', year:'numeric'});
  }
  function dateLabel(iso){
    var d = new Date(iso + 'T00:00:00');
    return d.toLocaleDateString('en-IN', {day:'2-digit', month:'short', year:'numeric', weekday:'short'});
  }
  function dateShort(iso){
    var d = new Date(iso + 'T00:00:00');
    return d.toLocaleDateString('en-IN', {day:'2-digit', month:'short'});
  }
  function esc(s){
    return String(s==null?'':s).replace(/[&<>"']/g, function(c){
      return {'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c];
    });
  }
  function initials(name){
    var parts = (name||'?').trim().split(/\s+/);
    return ((parts[0]||'')[0]||'?').toUpperCase() + ((parts[1]||'')[0]||'').toUpperCase();
  }

  /* ================= data helpers ================= */
  function clientById(id){ return db.clients.find(function(c){return c.id===id;}); }
  function cattleById(id){ return db.cattle.find(function(c){return c.id===id;}); }
  function inMonth(iso, ym){ return iso && iso.slice(0,7) === ym; }

  function supplyDue(entry){ return Math.max(0, (Number(entry.amount)||0) - (Number(entry.paidAmt)||0)); }

  function clientDues(){
    var map = {};
    db.supply.forEach(function(e){
      var due = supplyDue(e);
      if(!map[e.clientId]) map[e.clientId] = 0;
      map[e.clientId] += due;
    });
    return Object.keys(map).map(function(id){
      return { clientId:id, due: Math.round(map[id]*100)/100 };
    }).filter(function(x){return x.due > 0.004;})
      .sort(function(a,b){return b.due - a.due;});
  }

  function totalDue(){
    return db.supply.reduce(function(s,e){return s + supplyDue(e);}, 0);
  }

  function monthSupply(ym){ return db.supply.filter(function(e){return inMonth(e.date, ym);}); }
  function monthProduction(ym){ return db.production.filter(function(e){return inMonth(e.date, ym);}); }
  function monthExpenses(ym){ return db.expenses.filter(function(e){return inMonth(e.date, ym);}); }

  function settlePayment(clientId, amount){
    var remaining = amount;
    var entries = db.supply.filter(function(e){return e.clientId===clientId && supplyDue(e) > 0.004;})
      .sort(function(a,b){ return a.date < b.date ? -1 : (a.date > b.date ? 1 : 0); });
    var touched = [];
    for(var i=0;i<entries.length && remaining > 0.004;i++){
      var e = entries[i];
      var due = supplyDue(e);
      var pay = Math.min(due, remaining);
      e.paidAmt = Math.round(((Number(e.paidAmt)||0) + pay)*100)/100;
      e.paid = supplyDue(e) <= 0.004;
      remaining -= pay;
      touched.push(e);
    }
    syncAfterWrite('supply', touched);
  }

  /* ================= dom refs ================= */
  var $view = document.getElementById('view');
  var $pageTitle = document.getElementById('pageTitle');
  var $monthLabel = document.getElementById('monthLabel');
  var $fab = document.getElementById('fab');
  var $sheet = document.getElementById('sheet');
  var $sheetOverlay = document.getElementById('sheetOverlay');
  var $sheetContent = document.getElementById('sheetContent');
  var $toast = document.getElementById('toast');
  var $tabs = document.querySelectorAll('.tab');
  var $offlineBadge = document.getElementById('offlineBadge');

  function updateOnlineStatus(refresh){
    $offlineBadge.classList.toggle('show', !navigator.onLine);
    if(refresh && state.tab === 'more') render();
  }
  window.addEventListener('online', function(){ updateOnlineStatus(true); });
  window.addEventListener('offline', function(){ updateOnlineStatus(true); });
  updateOnlineStatus(false);

  var TAB_TITLES = { home:'Dashboard', supply:'Milk Supply', production:'Herd Production', expenses:'Expenses', more:'More' };
  var FAB_HIDDEN_TABS = { home:true, more:true };

  function toast(msg){
    $toast.textContent = msg;
    $toast.classList.add('show');
    clearTimeout(toast._t);
    toast._t = setTimeout(function(){ $toast.classList.remove('show'); }, 1800);
  }

  /* ================= sheet ================= */
  function openSheet(html, onMount){
    $sheetContent.innerHTML = html;
    $sheet.classList.add('open');
    $sheetOverlay.classList.add('open');
    if(onMount) onMount($sheetContent);
  }
  function closeSheet(){
    $sheet.classList.remove('open');
    $sheetOverlay.classList.remove('open');
  }
  $sheetOverlay.addEventListener('click', closeSheet);

  /* Enter key on a field moves to the next field in `getOrder()` (an id array
     computed fresh each time, since fields can show/hide); Enter on the last
     field in the order clicks the save button instead of submitting/reloading. */
  function wireEnterChain(root, fieldIds, getOrder, saveBtnId){
    fieldIds.forEach(function(id){
      var el = root.querySelector('#'+id);
      if(!el) return;
      el.addEventListener('keydown', function(ev){
        if(ev.key !== 'Enter') return;
        ev.preventDefault();
        var order = getOrder();
        var idx = order.indexOf(id);
        if(idx === -1) return;
        var nextId = order[idx+1];
        if(nextId){
          var nextEl = root.querySelector('#'+nextId);
          if(nextEl){ nextEl.focus(); if(nextEl.select) nextEl.select(); }
        } else {
          var btn = root.querySelector('#'+saveBtnId);
          if(btn && !btn.disabled) btn.click();
        }
      });
    });
  }

  /* Prevents double-submit from a fast double-tap: disables the button on first
     click so a second tap during the close animation can't create a duplicate entry. */
  function guardOnce(btn, fn){
    btn.addEventListener('click', function(){
      if(btn.disabled) return;
      btn.disabled = true;
      fn();
    });
  }

  function confirmDeleteThen(title, message, onConfirm){
    openSheet(
      '<h2>'+esc(title)+'</h2><p style="font-size:13.5px;color:var(--ink-soft);margin-bottom:18px;">'+esc(message)+'</p>' +
      '<div class="btn-row"><button class="btn btn-ghost" id="cdCancel">Cancel</button><button class="btn btn-danger" id="cdConfirm">Delete</button></div>',
      function(root){
        root.querySelector('#cdCancel').addEventListener('click', closeSheet);
        root.querySelector('#cdConfirm').addEventListener('click', function(){
          closeSheet(); onConfirm();
        });
      }
    );
  }

  /* ================= tab switching ================= */
  $tabs.forEach(function(btn){
    btn.addEventListener('click', function(){
      state.tab = btn.getAttribute('data-tab');
      $tabs.forEach(function(b){ b.classList.toggle('active', b===btn); });
      $pageTitle.textContent = TAB_TITLES[state.tab];
      $fab.classList.toggle('hidden', !!FAB_HIDDEN_TABS[state.tab]);
      render();
    });
  });

  $fab.addEventListener('click', function(){
    if(state.tab === 'supply') openSupplyForm();
    else if(state.tab === 'production') openProductionForm();
    else if(state.tab === 'expenses') openExpenseForm();
  });

  document.getElementById('monthBtn').addEventListener('click', function(){
    openSheet(
      '<h2>Select Month</h2>' +
      '<div class="field"><label>Month</label><input type="month" id="monthPick" value="'+state.month+'"></div>' +
      '<button class="btn btn-primary" id="monthOk">Apply</button>',
      function(root){
        root.querySelector('#monthOk').addEventListener('click', function(){
          var v = root.querySelector('#monthPick').value;
          if(v){ state.month = v; $monthLabel.textContent = monthLabel(v); render(); }
          closeSheet();
        });
      }
    );
  });

  /* ================= HOME ================= */
  function renderHome(){
    var ym = state.month;
    var sup = monthSupply(ym);
    var prod = monthProduction(ym);
    var exp = monthExpenses(ym);

    var supQty = sup.reduce(function(s,e){return s+(Number(e.qty)||0);},0);
    var supAmt = sup.reduce(function(s,e){return s+(Number(e.amount)||0);},0);
    var collected = sup.reduce(function(s,e){return s+(Number(e.paidAmt)||0);},0);
    var prodQty = prod.reduce(function(s,e){return s+(Number(e.morning)||0)+(Number(e.evening)||0);},0);
    var expAmt = exp.reduce(function(s,e){return s+(Number(e.amount)||0);},0);
    var net = supAmt - expAmt;
    var dues = clientDues();
    var allDue = totalDue();

    var html = '';
    html += '<div class="kpi-grid">';
    html += kpiCard('Milk Sold', liters(supQty), money(supAmt)+' billed', '');
    html += kpiCard('Milk Produced', liters(prodQty), db.cattle.length + ' animals', '');
    html += kpiCard('Expenses', money(expAmt), 'feed + labour + other', 'terracotta');
    html += kpiCard('Net (this month)', money(net), 'sales − expenses', 'gold');
    html += '</div>';

    html += '<div class="section-title">Outstanding Dues</div>';
    html += '<div class="card">';
    if(allDue < 0.01){
      html += '<div style="text-align:center;padding:10px 0;color:var(--ink-faint);font-size:13px;">All clients settled up ✨</div>';
    } else {
      html += '<div class="mini-stat"><span>Total pending</span><b class="num" style="color:var(--bad)">'+money(allDue)+'</b></div>';
      var top = dues.slice(0,4);
      top.forEach(function(d){
        var c = clientById(d.clientId);
        html += '<div class="due-row"><div style="display:flex;align-items:center;gap:10px;min-width:0;">' +
          '<div class="avatar">'+esc(initials(c?c.name:'?'))+'</div>' +
          '<div class="due-name" style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap;">'+esc(c?c.name:'Unknown')+'</div></div>' +
          '<div style="display:flex;align-items:center;">' +
          '<span class="due-amt num">'+money(d.due)+'</span>' +
          '<button class="settle-btn" data-settle="'+d.clientId+'">Settle</button>' +
          '</div></div>';
      });
      if(dues.length > 4){
        html += '<div style="text-align:center;margin-top:8px;"><span style="font-size:12px;color:var(--ink-faint);">+'+(dues.length-4)+' more — see Supply tab</span></div>';
      }
    }
    html += '</div>';

    html += '<div class="section-title">This Month Snapshot</div>';
    html += '<div class="card">';
    html += '<div class="mini-stat"><span>Cash collected</span><b class="num">'+money(collected)+'</b></div>';
    html += '<div class="mini-stat"><span>Feed cost</span><b class="num">'+money(sumByCat(exp,'feed'))+'</b></div>';
    html += '<div class="mini-stat"><span>Labour cost</span><b class="num">'+money(sumByCat(exp,'labour'))+'</b></div>';
    html += '<div class="mini-stat"><span>Other expenses</span><b class="num">'+money(sumByCat(exp,'other'))+'</b></div>';
    html += '<div class="mini-stat"><span>Avg. herd yield / day</span><b class="num">'+liters(avgYieldPerDay(prod, ym))+'</b></div>';
    html += '</div>';

    $view.innerHTML = html;

    $view.querySelectorAll('[data-settle]').forEach(function(btn){
      btn.addEventListener('click', function(){ openSettleForm(btn.getAttribute('data-settle')); });
    });
  }
  function sumByCat(list, cat){
    return list.filter(function(e){return e.category===cat;}).reduce(function(s,e){return s+(Number(e.amount)||0);},0);
  }
  function avgYieldPerDay(prodList, ym){
    if(!prodList.length) return 0;
    var days = {};
    prodList.forEach(function(e){ days[e.date] = (days[e.date]||0) + (Number(e.morning)||0) + (Number(e.evening)||0); });
    var total = 0, n = 0;
    for(var k in days){ total += days[k]; n++; }
    return n ? total/n : 0;
  }
  function kpiCard(label, value, sub, variant){
    return '<div class="kpi-card '+(variant||'')+'"><div class="kpi-label">'+esc(label)+'</div>' +
      '<div class="kpi-value num">'+value+'</div>' +
      '<div class="kpi-sub">'+esc(sub)+'</div></div>';
  }

  /* ================= SUPPLY ================= */
  function renderSupply(){
    var ym = state.month;
    var entries = monthSupply(ym).sort(function(a,b){ return a.date < b.date ? 1 : (a.date > b.date ? -1 : 0); });
    var dues = clientDues();

    var html = '';
    if(!db.clients.length){
      html += emptyState('No clients yet', 'Add your first client in the More tab to start logging supply.');
      $view.innerHTML = html;
      return;
    }

    if(dues.length){
      html += '<div class="section-title">Client Dues</div><div class="card"><div class="list" style="gap:0;">';
      dues.forEach(function(d){
        var c = clientById(d.clientId);
        html += '<div class="due-row"><div style="display:flex;align-items:center;gap:10px;min-width:0;">' +
          '<div class="avatar">'+esc(initials(c?c.name:'?'))+'</div>' +
          '<div class="due-name" style="overflow:hidden;text-overflow:ellipsis;white-space:nowrap;">'+esc(c?c.name:'Unknown')+'</div></div>' +
          '<div style="display:flex;align-items:center;">' +
          '<span class="due-amt num">'+money(d.due)+'</span>' +
          '<button class="settle-btn" data-settle="'+d.clientId+'">Settle</button>' +
          '</div></div>';
      });
      html += '</div></div>';
    }

    html += '<div class="section-title">Entries — '+esc(monthLabel(ym))+'</div>';
    if(!entries.length){
      html += emptyState('No entries this month', 'Tap the + button to log today’s milk supply.');
    } else {
      html += '<div class="list">';
      var lastDate = null;
      entries.forEach(function(e){
        if(e.date !== lastDate){ html += '<div class="date-heading">'+dateLabel(e.date)+'</div>'; lastDate = e.date; }
        var c = clientById(e.clientId);
        var status = e.paid ? 'paid' : (Number(e.paidAmt)>0 ? 'partial' : 'unpaid');
        html += '<div class="row" data-edit-supply="'+e.id+'">' +
          '<div class="row-main"><div class="row-title">'+esc(c?c.name:'Unknown')+'</div>' +
          '<div class="row-sub">'+liters(e.qty)+' × '+money(e.rate)+'/L</div></div>' +
          '<div class="row-end"><div class="row-amt num">'+money(e.amount)+'</div>' +
          '<span class="badge '+status+'">'+status+'</span></div></div>';
      });
      html += '</div>';
    }

    $view.innerHTML = html;
    $view.querySelectorAll('[data-settle]').forEach(function(btn){
      btn.addEventListener('click', function(){ openSettleForm(btn.getAttribute('data-settle')); });
    });
    $view.querySelectorAll('[data-edit-supply]').forEach(function(row){
      row.addEventListener('click', function(){ openSupplyForm(row.getAttribute('data-edit-supply')); });
    });
  }

  function openSettleForm(clientId){
    var c = clientById(clientId);
    var due = clientDues().find(function(d){return d.clientId===clientId;});
    var dueAmt = due ? due.due : 0;
    openSheet(
      '<h2>Settle — '+esc(c?c.name:'Client')+'</h2>' +
      '<div class="card" style="margin-bottom:16px;"><div class="mini-stat"><span>Pending amount</span><b class="num" style="color:var(--bad)">'+money(dueAmt)+'</b></div></div>' +
      '<div class="field"><label>Payment received</label><input type="number" id="stAmt" inputmode="decimal" step="0.01" enterkeyhint="done" value="'+dueAmt+'"></div>' +
      '<button class="btn btn-primary" id="stOk">Record Payment</button>',
      function(root){
        var btn = root.querySelector('#stOk');
        guardOnce(btn, function(){
          var amt = parseFloat(root.querySelector('#stAmt').value);
          if(!amt || amt <= 0){ toast('Enter a valid amount'); btn.disabled = false; return; }
          settlePayment(clientId, amt);
          closeSheet(); toast('Payment recorded'); render();
        });
        wireEnterChain(root, ['stAmt'], function(){ return ['stAmt']; }, 'stOk');
        root.querySelector('#stAmt').focus();
      }
    );
  }

  function openSupplyForm(entryId){
    var editing = entryId ? db.supply.find(function(e){return e.id===entryId;}) : null;
    if(!db.clients.length){ toast('Add a client first (More tab)'); return; }
    var clientOptions = db.clients.map(function(c){
      return '<option value="'+c.id+'"'+((editing?editing.clientId:db.clients[0].id)===c.id?' selected':'')+'>'+esc(c.name)+'</option>';
    }).join('');
    var d = editing ? editing.date : todayISO();
    var qty = editing ? editing.qty : '';
    var rate = editing ? editing.rate : (db.clients[0] ? db.clients[0].rate : '');
    var status = editing ? (editing.paid ? 'paid' : (Number(editing.paidAmt)>0 ? 'partial' : 'unpaid')) : 'paid';
    var paidAmt = editing ? editing.paidAmt : '';

    var html = '<h2>'+(editing?'Edit':'Log')+' Milk Supply</h2>';
    if(!editing){
      html += '<div class="field"><label>Date mode</label><div class="segmented" id="fDateMode">' +
        '<button type="button" data-v="single" class="active">Single date</button>' +
        '<button type="button" data-v="multi">Multiple dates</button>' +
        '</div></div>';
    }
    html += '<div class="field" id="fDateSingleWrap"><label>Date</label><input type="date" id="fDate" value="'+d+'"></div>';
    html += '<div class="field" id="fDateMultiWrap" style="display:none;"><label>Dates</label><div class="mini-cal" id="fCal"></div>' +
      '<div class="hint">Tap days to add or remove them. Same client, quantity, rate and status will be logged for every date picked.</div></div>';
    html += '<div class="field"><label>Client</label><select id="fClient">'+clientOptions+'</select></div>';
    html += '<div class="field-row">' +
      '<div class="field"><label>Quantity (L)</label><input type="number" id="fQty" inputmode="decimal" step="0.1" enterkeyhint="next" value="'+qty+'"></div>' +
      '<div class="field"><label>Rate (₹/L)</label><input type="number" id="fRate" inputmode="decimal" step="0.5" enterkeyhint="done" value="'+rate+'"></div>' +
      '</div>';
    html += '<div class="field"><label>Amount</label><input type="number" id="fAmount" inputmode="decimal" step="0.01" readonly style="opacity:.8"></div>';
    html += '<div class="field"><label>Payment status</label><div class="segmented" id="fStatus">' +
      '<button type="button" data-v="paid" class="'+(status==='paid'?'active':'')+'">Paid</button>' +
      '<button type="button" data-v="partial" class="'+(status==='partial'?'active':'')+'">Partial</button>' +
      '<button type="button" data-v="unpaid" class="'+(status==='unpaid'?'active':'')+'">Unpaid</button>' +
      '</div></div>';
    html += '<div class="field" id="fPaidAmtWrap"><label>Amount paid</label><input type="number" id="fPaidAmt" inputmode="decimal" step="0.01" enterkeyhint="done" value="'+paidAmt+'"></div>';
    html += '<div class="btn-row" style="margin-top:6px;">';
    if(editing) html += '<button class="btn btn-danger" id="fDelete">Delete</button>';
    html += '<button class="btn btn-primary" id="fSave">Save</button>';
    html += '</div>';

    openSheet(html, function(root){
      var qtyEl = root.querySelector('#fQty'), rateEl = root.querySelector('#fRate'), amtEl = root.querySelector('#fAmount');
      var statusWrap = root.querySelector('#fStatus'), paidWrap = root.querySelector('#fPaidAmtWrap'), paidEl = root.querySelector('#fPaidAmt');
      var curStatus = status;
      var dateMode = 'single';
      var calYm = state.month;
      var selectedDates = new Set();
      var singleWrap = root.querySelector('#fDateSingleWrap');
      var multiWrap = root.querySelector('#fDateMultiWrap');
      var calEl = root.querySelector('#fCal');

      function pad2(n){ return n<10 ? '0'+n : ''+n; }
      function isoOf(y,m,day){ return y+'-'+pad2(m+1)+'-'+pad2(day); }
      function shiftMonth(delta){
        var parts = calYm.split('-'); var y = Number(parts[0]), m = Number(parts[1])-1;
        m += delta;
        if(m<0){ m=11; y--; } else if(m>11){ m=0; y++; }
        calYm = y+'-'+pad2(m+1);
        renderCal();
      }
      function renderCal(){
        var parts = calYm.split('-'); var y = Number(parts[0]), m = Number(parts[1])-1;
        var startDow = new Date(y,m,1).getDay();
        var daysInMonth = new Date(y,m+1,0).getDate();
        var daysInPrevMonth = new Date(y,m,0).getDate();
        var todayIso = todayISO();
        var h = '<div class="mini-cal-head">' +
          '<button type="button" class="cal-nav-btn" id="calPrev">&lsaquo;</button>' +
          '<div class="mini-cal-title">'+monthLabel(calYm)+'</div>' +
          '<button type="button" class="cal-nav-btn" id="calNext">&rsaquo;</button>' +
          '</div><div class="mini-cal-grid">';
        ['S','M','T','W','T','F','S'].forEach(function(dw){ h += '<div class="mini-cal-dow">'+dw+'</div>'; });
        for(var i=0;i<startDow;i++){
          h += '<div class="cal-day other-month">'+(daysInPrevMonth-startDow+1+i)+'</div>';
        }
        for(var dnum=1; dnum<=daysInMonth; dnum++){
          var iso = isoOf(y,m,dnum);
          var cls = 'cal-day' + (selectedDates.has(iso)?' selected':'') + (iso===todayIso?' today':'');
          h += '<div class="'+cls+'" data-date="'+iso+'">'+dnum+'</div>';
        }
        var trailing = (7 - ((startDow+daysInMonth) % 7)) % 7;
        for(var t=1;t<=trailing;t++){ h += '<div class="cal-day other-month">'+t+'</div>'; }
        h += '</div><div class="mini-cal-foot"><span>'+selectedDates.size+' date'+(selectedDates.size===1?'':'s')+' selected</span><button type="button" id="calClear">Clear</button></div>';
        calEl.innerHTML = h;
        calEl.querySelector('#calPrev').addEventListener('click', function(){ shiftMonth(-1); });
        calEl.querySelector('#calNext').addEventListener('click', function(){ shiftMonth(1); });
        calEl.querySelector('#calClear').addEventListener('click', function(){ selectedDates.clear(); renderCal(); });
        calEl.querySelectorAll('.cal-day[data-date]').forEach(function(el){
          el.addEventListener('click', function(){
            var iso = el.getAttribute('data-date');
            if(selectedDates.has(iso)) selectedDates.delete(iso); else selectedDates.add(iso);
            renderCal();
          });
        });
      }
      renderCal();

      var modeWrap = root.querySelector('#fDateMode');
      if(modeWrap){
        modeWrap.querySelectorAll('button').forEach(function(b){
          b.addEventListener('click', function(){
            modeWrap.querySelectorAll('button').forEach(function(x){x.classList.remove('active');});
            b.classList.add('active');
            dateMode = b.getAttribute('data-v');
            singleWrap.style.display = dateMode==='single' ? '' : 'none';
            multiWrap.style.display = dateMode==='multi' ? '' : 'none';
          });
        });
      }

      function recalcAmount(){
        var amt = (parseFloat(qtyEl.value)||0) * (parseFloat(rateEl.value)||0);
        amtEl.value = Math.round(amt*100)/100;
        syncPaidWithStatus();
      }
      function syncPaidWithStatus(){
        var amt = parseFloat(amtEl.value)||0;
        paidWrap.style.display = curStatus==='partial' ? '' : 'none';
        if(curStatus==='paid') paidEl.value = amt;
        if(curStatus==='unpaid') paidEl.value = 0;
      }
      qtyEl.addEventListener('input', recalcAmount);
      rateEl.addEventListener('input', recalcAmount);
      root.querySelector('#fClient').addEventListener('change', function(e){
        if(!editing){ var c = clientById(e.target.value); if(c) rateEl.value = c.rate; recalcAmount(); }
      });
      statusWrap.querySelectorAll('button').forEach(function(b){
        b.addEventListener('click', function(){
          statusWrap.querySelectorAll('button').forEach(function(x){x.classList.remove('active');});
          b.classList.add('active'); curStatus = b.getAttribute('data-v'); syncPaidWithStatus();
        });
      });
      recalcAmount();
      if(editing) paidEl.value = paidAmt;

      var saveBtn = root.querySelector('#fSave');
      guardOnce(saveBtn, function(){
        var qtyV = parseFloat(qtyEl.value);
        if(!qtyV || qtyV <= 0){ toast('Enter a valid quantity'); saveBtn.disabled = false; return; }
        var rateV = parseFloat(rateEl.value)||0;
        var amount = Math.round(qtyV*rateV*100)/100;
        var pAmt = curStatus==='paid' ? amount : (curStatus==='unpaid' ? 0 : (parseFloat(paidEl.value)||0));
        var pAmtFinal = Math.min(pAmt, amount);
        var clientIdV = root.querySelector('#fClient').value;

        if(!editing && dateMode==='multi'){
          if(!selectedDates.size){ toast('Select at least one date'); saveBtn.disabled = false; return; }
          var dates = Array.from(selectedDates);
          var newRecs = dates.map(function(iso){
            return {
              id: uid(), date: iso, clientId: clientIdV,
              qty: qtyV, rate: rateV, amount: amount,
              paidAmt: pAmtFinal, paid: pAmtFinal >= amount - 0.004
            };
          });
          newRecs.forEach(function(r){ db.supply.push(r); });
          syncAfterWrite('supply', newRecs);
          closeSheet(); toast(dates.length + ' entries added'); render();
          return;
        }

        var rec = editing || {id:uid()};
        rec.date = root.querySelector('#fDate').value || todayISO();
        rec.clientId = clientIdV;
        rec.qty = qtyV; rec.rate = rateV; rec.amount = amount;
        rec.paidAmt = pAmtFinal;
        rec.paid = rec.paidAmt >= amount - 0.004;
        if(!editing) db.supply.push(rec);
        syncAfterWrite('supply', [rec]); closeSheet(); toast('Saved'); render();
      });

      wireEnterChain(root, ['fQty','fRate','fPaidAmt'], function(){
        var order = ['fQty','fRate'];
        if(paidWrap.style.display !== 'none') order.push('fPaidAmt');
        return order;
      }, 'fSave');

      var delBtn = root.querySelector('#fDelete');
      if(delBtn) delBtn.addEventListener('click', function(){
        confirmDeleteThen('Delete Entry?', 'This supply entry will be permanently removed.', function(){
          db.supply = db.supply.filter(function(e){return e.id!==editing.id;});
          syncAfterDelete('supply', editing.id); toast('Deleted'); render();
        });
      });
    });
  }

  /* ================= PRODUCTION ================= */
  function renderProduction(){
    var ym = state.month;
    var entries = monthProduction(ym).sort(function(a,b){ return a.date < b.date ? 1 : (a.date > b.date ? -1 : 0); });

    var html = '';
    if(!db.cattle.length){
      html += emptyState('No cattle yet', 'Add your animals in the More tab to start logging production.');
      $view.innerHTML = html;
      return;
    }

    html += '<div class="section-title">Per Animal — '+esc(monthLabel(ym))+'</div><div class="card"><div class="list" style="gap:0;">';
    db.cattle.forEach(function(c){
      var mine = entries.filter(function(e){return e.cattleId===c.id;});
      var total = mine.reduce(function(s,e){return s+(Number(e.morning)||0)+(Number(e.evening)||0);},0);
      var avg = mine.length ? total/mine.length : 0;
      html += '<div class="mini-stat"><span>'+esc(c.name)+(c.tag?' <span style="color:var(--ink-faint)">('+esc(c.tag)+')</span>':'')+'</span>' +
        '<b class="num">'+liters(total)+' <span style="font-weight:600;color:var(--ink-faint);font-size:11.5px;">avg '+liters(avg)+'/day</span></b></div>';
    });
    html += '</div></div>';

    html += '<div class="section-title">Daily Log</div>';
    if(!entries.length){
      html += emptyState('No entries this month', 'Tap the + button to log today’s yield.');
    } else {
      html += '<div class="list">';
      var lastDate = null;
      entries.forEach(function(e){
        if(e.date !== lastDate){ html += '<div class="date-heading">'+dateLabel(e.date)+'</div>'; lastDate = e.date; }
        var c = cattleById(e.cattleId);
        var tot = (Number(e.morning)||0) + (Number(e.evening)||0);
        html += '<div class="row" data-edit-prod="'+e.id+'">' +
          '<div class="row-main"><div class="row-title">'+esc(c?c.name:'Unknown')+'</div>' +
          '<div class="row-sub">AM '+liters(e.morning)+' &middot; PM '+liters(e.evening)+'</div></div>' +
          '<div class="row-end"><div class="row-amt num">'+liters(tot)+'</div></div></div>';
      });
      html += '</div>';
    }

    $view.innerHTML = html;
    $view.querySelectorAll('[data-edit-prod]').forEach(function(row){
      row.addEventListener('click', function(){ openProductionForm(row.getAttribute('data-edit-prod')); });
    });
  }

  function openProductionForm(entryId){
    var editing = entryId ? db.production.find(function(e){return e.id===entryId;}) : null;
    if(!db.cattle.length){ toast('Add an animal first (More tab)'); return; }
    var opts = db.cattle.map(function(c){
      return '<option value="'+c.id+'"'+((editing?editing.cattleId:db.cattle[0].id)===c.id?' selected':'')+'>'+esc(c.name)+'</option>';
    }).join('');
    var d = editing ? editing.date : todayISO();

    var html = '<h2>'+(editing?'Edit':'Log')+' Production</h2>';
    html += '<div class="field"><label>Date</label><input type="date" id="pDate" value="'+d+'"></div>';
    html += '<div class="field"><label>Animal</label><select id="pCattle">'+opts+'</select></div>';
    html += '<div class="field-row">' +
      '<div class="field"><label>Morning (L)</label><input type="number" id="pMorn" inputmode="decimal" step="0.1" enterkeyhint="next" value="'+(editing?editing.morning:'')+'"></div>' +
      '<div class="field"><label>Evening (L)</label><input type="number" id="pEve" inputmode="decimal" step="0.1" enterkeyhint="done" value="'+(editing?editing.evening:'')+'"></div>' +
      '</div>';
    html += '<div class="btn-row" style="margin-top:6px;">';
    if(editing) html += '<button class="btn btn-danger" id="pDelete">Delete</button>';
    html += '<button class="btn btn-primary" id="pSave">Save</button></div>';

    openSheet(html, function(root){
      var saveBtn = root.querySelector('#pSave');
      guardOnce(saveBtn, function(){
        var morn = parseFloat(root.querySelector('#pMorn').value)||0;
        var eve = parseFloat(root.querySelector('#pEve').value)||0;
        if(morn<=0 && eve<=0){ toast('Enter at least one value'); saveBtn.disabled = false; return; }
        var rec = editing || {id:uid()};
        rec.date = root.querySelector('#pDate').value || todayISO();
        rec.cattleId = root.querySelector('#pCattle').value;
        rec.morning = morn; rec.evening = eve;
        if(!editing) db.production.push(rec);
        syncAfterWrite('production', [rec]); closeSheet(); toast('Saved'); render();
      });
      wireEnterChain(root, ['pMorn','pEve'], function(){ return ['pMorn','pEve']; }, 'pSave');
      var delBtn = root.querySelector('#pDelete');
      if(delBtn) delBtn.addEventListener('click', function(){
        confirmDeleteThen('Delete Entry?', 'This production entry will be permanently removed.', function(){
          db.production = db.production.filter(function(e){return e.id!==editing.id;});
          syncAfterDelete('production', editing.id); toast('Deleted'); render();
        });
      });
    });
  }

  /* ================= EXPENSES ================= */
  var CAT_LABEL = {feed:'Feed', labour:'Labour', other:'Other'};

  function renderExpenses(){
    var ym = state.month;
    var entries = monthExpenses(ym).sort(function(a,b){ return a.date < b.date ? 1 : (a.date > b.date ? -1 : 0); });
    var total = entries.reduce(function(s,e){return s+(Number(e.amount)||0);},0);

    var html = '';
    html += '<div class="kpi-grid">';
    html += kpiCard('Feed', money(sumByCat(entries,'feed')), monthLabel(ym), 'gold');
    html += kpiCard('Labour', money(sumByCat(entries,'labour')), monthLabel(ym), 'terracotta');
    html += kpiCard('Other', money(sumByCat(entries,'other')), monthLabel(ym), '');
    html += kpiCard('Total', money(total), monthLabel(ym), '');
    html += '</div>';

    html += '<div class="section-title">Entries — '+esc(monthLabel(ym))+'</div>';
    if(!entries.length){
      html += emptyState('No expenses logged', 'Tap the + button to add feed, labour or other costs.');
    } else {
      html += '<div class="list">';
      var lastDate = null;
      entries.forEach(function(e){
        if(e.date !== lastDate){ html += '<div class="date-heading">'+dateLabel(e.date)+'</div>'; lastDate = e.date; }
        html += '<div class="row" data-edit-exp="'+e.id+'">' +
          '<div class="row-main"><div class="row-title">'+esc(CAT_LABEL[e.category]||e.category)+'</div>' +
          '<div class="row-sub">'+esc(e.note||'—')+'</div></div>' +
          '<div class="row-end"><div class="row-amt num">'+money(e.amount)+'</div>' +
          '<span class="badge '+esc(e.category)+'">'+esc(CAT_LABEL[e.category]||e.category)+'</span></div></div>';
      });
      html += '</div>';
    }

    $view.innerHTML = html;
    $view.querySelectorAll('[data-edit-exp]').forEach(function(row){
      row.addEventListener('click', function(){ openExpenseForm(row.getAttribute('data-edit-exp')); });
    });
  }

  function openExpenseForm(entryId){
    var editing = entryId ? db.expenses.find(function(e){return e.id===entryId;}) : null;
    var cat = editing ? editing.category : 'feed';
    var d = editing ? editing.date : todayISO();

    var html = '<h2>'+(editing?'Edit':'Add')+' Expense</h2>';
    html += '<div class="field"><label>Date</label><input type="date" id="eDate" value="'+d+'"></div>';
    html += '<div class="field"><label>Category</label><div class="segmented" id="eCat">' +
      '<button type="button" data-v="feed" class="'+(cat==='feed'?'active':'')+'">Feed</button>' +
      '<button type="button" data-v="labour" class="'+(cat==='labour'?'active':'')+'">Labour</button>' +
      '<button type="button" data-v="other" class="'+(cat==='other'?'active':'')+'">Other</button>' +
      '</div></div>';
    html += '<div class="field"><label>Amount (₹)</label><input type="number" id="eAmt" inputmode="decimal" step="0.01" enterkeyhint="next" value="'+(editing?editing.amount:'')+'"></div>';
    html += '<div class="field"><label>Note (optional)</label><input type="text" id="eNote" placeholder="e.g. Cattle feed - 2 bags" enterkeyhint="done" value="'+(editing?esc(editing.note||''):'')+'"></div>';
    html += '<div class="btn-row" style="margin-top:6px;">';
    if(editing) html += '<button class="btn btn-danger" id="eDelete">Delete</button>';
    html += '<button class="btn btn-primary" id="eSave">Save</button></div>';

    openSheet(html, function(root){
      var curCat = cat;
      var catWrap = root.querySelector('#eCat');
      catWrap.querySelectorAll('button').forEach(function(b){
        b.addEventListener('click', function(){
          catWrap.querySelectorAll('button').forEach(function(x){x.classList.remove('active');});
          b.classList.add('active'); curCat = b.getAttribute('data-v');
        });
      });
      var saveBtn = root.querySelector('#eSave');
      guardOnce(saveBtn, function(){
        var amt = parseFloat(root.querySelector('#eAmt').value);
        if(!amt || amt<=0){ toast('Enter a valid amount'); saveBtn.disabled = false; return; }
        var rec = editing || {id:uid()};
        rec.date = root.querySelector('#eDate').value || todayISO();
        rec.category = curCat; rec.amount = amt;
        rec.note = root.querySelector('#eNote').value.trim();
        if(!editing) db.expenses.push(rec);
        syncAfterWrite('expenses', [rec]); closeSheet(); toast('Saved'); render();
      });
      wireEnterChain(root, ['eAmt','eNote'], function(){ return ['eAmt','eNote']; }, 'eSave');
      var delBtn = root.querySelector('#eDelete');
      if(delBtn) delBtn.addEventListener('click', function(){
        confirmDeleteThen('Delete Expense?', 'This expense entry will be permanently removed.', function(){
          db.expenses = db.expenses.filter(function(e){return e.id!==editing.id;});
          syncAfterDelete('expenses', editing.id); toast('Deleted'); render();
        });
      });
    });
  }

  /* ================= MORE ================= */
  function renderMore(){
    var html = '';

    html += '<div class="section-title">Clients ('+db.clients.length+')</div>';
    html += '<div class="list">';
    db.clients.forEach(function(c){
      html += '<div class="row" data-edit-client="'+c.id+'"><div class="row-main">' +
        '<div class="row-title">'+esc(c.name)+'</div><div class="row-sub">'+esc(c.phone||'No phone')+'</div></div>' +
        '<div class="row-end"><div class="row-amt num">'+money(c.rate)+'/L</div></div></div>';
    });
    html += '<button class="row" id="addClient" style="justify-content:center;border-style:dashed;color:var(--forest);font-weight:700;">+ Add Client</button>';
    html += '</div>';

    html += '<div class="section-title">Cattle ('+db.cattle.length+')</div>';
    html += '<div class="list">';
    db.cattle.forEach(function(c){
      html += '<div class="row" data-edit-cattle="'+c.id+'"><div class="row-main">' +
        '<div class="row-title">'+esc(c.name)+'</div><div class="row-sub">'+esc(c.tag||'No tag')+'</div></div></div>';
    });
    html += '<button class="row" id="addCattle" style="justify-content:center;border-style:dashed;color:var(--forest);font-weight:700;">+ Add Animal</button>';
    html += '</div>';

    if(location.protocol === 'file:'){
      html += '<div class="banner"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 9v4M12 16.5h.01M10.3 3.9 1.8 18a2 2 0 0 0 1.7 3h17a2 2 0 0 0 1.7-3L13.7 3.9a2 2 0 0 0-3.4 0Z"/></svg>' +
        '<div><b>Running as a local file</b>Data entry and saving still work fine offline like this. But the app can\'t install as a home-screen app or auto-update this way. For that, host it online once (e.g. GitHub Pages) and open that link — after the first visit it keeps working fully offline, no network needed at all.</div></div>';
    } else if(!navigator.onLine){
      html += '<div class="banner"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 9v4M12 16.5h.01M10.3 3.9 1.8 18a2 2 0 0 0 1.7 3h17a2 2 0 0 0 1.7-3L13.7 3.9a2 2 0 0 0-3.4 0Z"/></svg>' +
        '<div><b>You\'re offline right now</b>That\'s fine — everything you enter is saved on this device immediately and nothing is lost. No internet connection is required to use this app.</div></div>';
    }

    html += '<div class="section-title">Account</div>';
    if(authUser){
      html += '<div class="card"><div class="mini-stat"><span>Signed in as</span><b class="num" style="font-weight:700;">'+esc(authUser.email)+'</b></div>' +
        '<div class="mini-stat"><span>Sync status</span><span class="badge paid">Synced</span></div>' +
        '<button class="btn btn-ghost" id="signOutBtn" style="margin-top:12px;">Sign Out</button></div>';
    } else {
      html += '<div class="card"><p style="font-size:12.5px;color:var(--ink-soft);margin-bottom:12px;">Not signed in — data stays on this device only. Sign in to keep the same data across your devices.</p>' +
        '<div class="btn-row"><button class="btn btn-ghost" id="signInBtn">Sign In</button><button class="btn btn-primary" id="signUpBtn">Sign Up</button></div></div>';
    }

    html += '<div class="section-title">Backup</div>';
    html += '<div class="card"><p style="font-size:12.5px;color:var(--ink-soft);margin-bottom:12px;">All data lives only on this device’s browser. Export a backup regularly, especially before clearing browser data.</p>';
    html += '<div class="mini-stat"><span>Last backup</span><b class="num" style="'+(lastBackupOverdue()?'color:var(--bad)':'')+'">'+esc(lastBackupLabel())+'</b></div>';
    html += '<div class="btn-row" style="margin-top:10px;"><button class="btn btn-ghost" id="exportBtn">Export</button><button class="btn btn-ghost" id="importBtn">Import</button></div>';
    html += '<input type="file" id="importFile" accept="application/json" style="display:none;">';
    html += '</div>';

    html += '<div class="section-title">Danger Zone</div>';
    html += '<div class="card"><button class="btn btn-danger" id="clearBtn">Erase All Data</button></div>';

    $view.innerHTML = html;

    $view.querySelector('#addClient').addEventListener('click', function(){ openClientForm(); });
    $view.querySelector('#addCattle').addEventListener('click', function(){ openCattleForm(); });
    $view.querySelectorAll('[data-edit-client]').forEach(function(row){
      row.addEventListener('click', function(){ openClientForm(row.getAttribute('data-edit-client')); });
    });
    $view.querySelectorAll('[data-edit-cattle]').forEach(function(row){
      row.addEventListener('click', function(){ openCattleForm(row.getAttribute('data-edit-cattle')); });
    });
    var signInBtn = $view.querySelector('#signInBtn');
    var signUpBtn = $view.querySelector('#signUpBtn');
    var signOutBtn = $view.querySelector('#signOutBtn');
    if(signInBtn) signInBtn.addEventListener('click', function(){ openAuthSheet('login'); });
    if(signUpBtn) signUpBtn.addEventListener('click', function(){ openAuthSheet('signup'); });
    if(signOutBtn) signOutBtn.addEventListener('click', performSignOut);
    $view.querySelector('#exportBtn').addEventListener('click', exportData);
    $view.querySelector('#importBtn').addEventListener('click', function(){ $view.querySelector('#importFile').click(); });
    $view.querySelector('#importFile').addEventListener('change', importData);
    $view.querySelector('#clearBtn').addEventListener('click', function(){
      var scope = authUser ? 'from this device and your account' : 'from this device';
      openSheet(
        '<h2>Erase All Data?</h2><p style="font-size:13.5px;color:var(--ink-soft);margin-bottom:18px;">This deletes all clients, cattle, supply, production and expense records '+scope+'. This cannot be undone — export a backup first.</p>' +
        '<div class="btn-row"><button class="btn btn-ghost" id="cancelClear">Cancel</button><button class="btn btn-danger" id="confirmClear">Erase Everything</button></div>',
        function(root){
          root.querySelector('#cancelClear').addEventListener('click', closeSheet);
          root.querySelector('#confirmClear').addEventListener('click', function(){
            var wiped = defaultDB(); db = wiped; syncAfterReplaceAll(wiped); closeSheet(); toast('All data erased'); render();
          });
        }
      );
    });
  }

  function openClientForm(id){
    var editing = id ? clientById(id) : null;
    var html = '<h2>'+(editing?'Edit':'Add')+' Client</h2>';
    html += '<div class="field"><label>Name</label><input type="text" id="cName" enterkeyhint="next" value="'+(editing?esc(editing.name):'')+'" placeholder="e.g. Sharma Sir"></div>';
    html += '<div class="field"><label>Phone (optional)</label><input type="tel" id="cPhone" enterkeyhint="next" value="'+(editing?esc(editing.phone||''):'')+'"></div>';
    html += '<div class="field"><label>Default rate (₹/L)</label><input type="number" id="cRate" inputmode="decimal" step="0.5" enterkeyhint="done" value="'+(editing?editing.rate:'60')+'"></div>';
    if(editing) html += '<button class="btn btn-ghost" id="cPrintStatement" style="margin-bottom:14px;">Print Statement</button>';
    html += '<div class="btn-row" style="margin-top:6px;">';
    if(editing) html += '<button class="btn btn-danger" id="cDelete">Delete</button>';
    html += '<button class="btn btn-primary" id="cSave">Save</button></div>';

    openSheet(html, function(root){
      var saveBtn = root.querySelector('#cSave');
      guardOnce(saveBtn, function(){
        var name = root.querySelector('#cName').value.trim();
        if(!name){ toast('Enter a name'); saveBtn.disabled = false; return; }
        var rec = editing || {id:uid()};
        rec.name = name;
        rec.phone = root.querySelector('#cPhone').value.trim();
        rec.rate = parseFloat(root.querySelector('#cRate').value)||0;
        if(!editing) db.clients.push(rec);
        syncAfterWrite('clients', [rec]); closeSheet(); toast('Saved'); render();
      });
      wireEnterChain(root, ['cName','cPhone','cRate'], function(){ return ['cName','cPhone','cRate']; }, 'cSave');
      root.querySelector('#cName').focus();
      var printBtn = root.querySelector('#cPrintStatement');
      if(printBtn) printBtn.addEventListener('click', function(){ openStatementForm(editing.id); });
      var delBtn = root.querySelector('#cDelete');
      if(delBtn) delBtn.addEventListener('click', function(){
        confirmDeleteThen('Delete Client?', 'This removes the client. Their existing supply entries will stay in your records but show as "Unknown".', function(){
          db.clients = db.clients.filter(function(c){return c.id!==editing.id;});
          syncAfterDelete('clients', editing.id); toast('Deleted'); render();
        });
      });
    });
  }

  function openCattleForm(id){
    var editing = id ? cattleById(id) : null;
    var html = '<h2>'+(editing?'Edit':'Add')+' Animal</h2>';
    html += '<div class="field"><label>Name</label><input type="text" id="aName" enterkeyhint="next" value="'+(editing?esc(editing.name):'')+'" placeholder="e.g. Ganga"></div>';
    html += '<div class="field"><label>Tag / ID (optional)</label><input type="text" id="aTag" enterkeyhint="done" value="'+(editing?esc(editing.tag||''):'')+'"></div>';
    html += '<div class="btn-row" style="margin-top:6px;">';
    if(editing) html += '<button class="btn btn-danger" id="aDelete">Delete</button>';
    html += '<button class="btn btn-primary" id="aSave">Save</button></div>';

    openSheet(html, function(root){
      var saveBtn = root.querySelector('#aSave');
      guardOnce(saveBtn, function(){
        var name = root.querySelector('#aName').value.trim();
        if(!name){ toast('Enter a name'); saveBtn.disabled = false; return; }
        var rec = editing || {id:uid()};
        rec.name = name;
        rec.tag = root.querySelector('#aTag').value.trim();
        if(!editing) db.cattle.push(rec);
        syncAfterWrite('cattle', [rec]); closeSheet(); toast('Saved'); render();
      });
      wireEnterChain(root, ['aName','aTag'], function(){ return ['aName','aTag']; }, 'aSave');
      root.querySelector('#aName').focus();
      var delBtn = root.querySelector('#aDelete');
      if(delBtn) delBtn.addEventListener('click', function(){
        confirmDeleteThen('Delete Animal?', 'This removes the animal. Its existing production entries will stay in your records but show as "Unknown".', function(){
          db.cattle = db.cattle.filter(function(c){return c.id!==editing.id;});
          syncAfterDelete('cattle', editing.id); toast('Deleted'); render();
        });
      });
    });
  }

  function openAuthSheet(mode){
    var html = '<h2>Account</h2>';
    html += '<div class="segmented" id="authMode">' +
      '<button type="button" data-v="login" class="'+(mode==='login'?'active':'')+'">Log In</button>' +
      '<button type="button" data-v="signup" class="'+(mode==='signup'?'active':'')+'">Sign Up</button>' +
      '</div>';
    html += '<div class="field" style="margin-top:14px;"><label>Email</label><input type="email" id="authEmail" inputmode="email" enterkeyhint="next" autocomplete="email"></div>';
    html += '<div class="field"><label>Password</label><input type="password" id="authPass" enterkeyhint="next" autocomplete="current-password"></div>';
    html += '<div class="field" id="authConfirmWrap" style="display:'+(mode==='signup'?'':'none')+';"><label>Confirm Password</label><input type="password" id="authConfirm" enterkeyhint="done" autocomplete="new-password"></div>';
    html += '<button class="btn btn-primary" id="authGo" style="margin-top:6px;">'+(mode==='signup'?'Sign Up':'Log In')+'</button>';
    html += '<button class="btn btn-ghost" id="authForgot" style="margin-top:10px;">Forgot password?</button>';

    openSheet(html, function(root){
      var curMode = mode;
      var modeWrap = root.querySelector('#authMode');
      var confirmWrap = root.querySelector('#authConfirmWrap');
      var goBtn = root.querySelector('#authGo');
      modeWrap.querySelectorAll('button').forEach(function(b){
        b.addEventListener('click', function(){
          modeWrap.querySelectorAll('button').forEach(function(x){x.classList.remove('active');});
          b.classList.add('active');
          curMode = b.getAttribute('data-v');
          confirmWrap.style.display = curMode==='signup' ? '' : 'none';
          goBtn.textContent = curMode==='signup' ? 'Sign Up' : 'Log In';
        });
      });
      guardOnce(goBtn, function(){
        var email = root.querySelector('#authEmail').value.trim();
        var pass = root.querySelector('#authPass').value;
        if(!email || !pass){ toast('Enter your email and password'); goBtn.disabled = false; return; }
        if(curMode==='signup'){
          var confirm = root.querySelector('#authConfirm').value;
          if(pass !== confirm){ toast('Passwords don\'t match'); goBtn.disabled = false; return; }
        }
        goBtn.textContent = 'Please wait…';
        performSignIn(curMode==='signup', email, pass).finally(function(){ goBtn.disabled = false; goBtn.textContent = curMode==='signup' ? 'Sign Up' : 'Log In'; });
      });
      wireEnterChain(root, ['authEmail','authPass','authConfirm'], function(){
        var order = ['authEmail','authPass'];
        if(confirmWrap.style.display !== 'none') order.push('authConfirm');
        return order;
      }, 'authGo');
      root.querySelector('#authForgot').addEventListener('click', function(){
        var email = root.querySelector('#authEmail').value.trim();
        if(!email){ toast('Enter your email above first'); return; }
        performPasswordReset(email);
      });
    });
  }

  function openStatementForm(clientId){
    var c = clientById(clientId);
    if(!c) return;
    var ym = state.month;
    var parts = ym.split('-'); var y = Number(parts[0]), m = Number(parts[1])-1;
    var firstDay = ym + '-01';
    var lastDayNum = new Date(y, m+1, 0).getDate();
    var lastDay = ym + '-' + (lastDayNum<10?'0'+lastDayNum:lastDayNum);

    var html = '<h2>Print Statement</h2>';
    html += '<div class="field-row">' +
      '<div class="field"><label>From</label><input type="date" id="stmtFrom" value="'+firstDay+'"></div>' +
      '<div class="field"><label>To</label><input type="date" id="stmtTo" value="'+lastDay+'"></div>' +
      '</div>';
    html += '<button class="btn btn-primary" id="stmtGo">Generate &amp; Print</button>';

    openSheet(html, function(root){
      var goBtn = root.querySelector('#stmtGo');
      guardOnce(goBtn, function(){
        var from = root.querySelector('#stmtFrom').value;
        var to = root.querySelector('#stmtTo').value;
        if(!from || !to || from > to){ toast('Pick a valid date range'); goBtn.disabled = false; return; }
        buildAndPrintStatement(c, from, to);
        goBtn.disabled = false;
        closeSheet();
      });
    });
  }

  function buildAndPrintStatement(c, from, to){
    var entries = db.supply.filter(function(e){ return e.clientId===c.id && e.date >= from && e.date <= to; })
      .sort(function(a,b){ return a.date < b.date ? -1 : (a.date > b.date ? 1 : 0); });
    var totalBilled = 0, totalPaid = 0;
    var rows = entries.map(function(e){
      totalBilled += Number(e.amount)||0;
      totalPaid += Number(e.paidAmt)||0;
      return '<tr><td>'+dateShort(e.date)+'</td><td class="num">'+liters(e.qty)+'</td><td class="num">'+money(e.rate)+'</td>' +
        '<td class="num">'+money(e.amount)+'</td><td class="num">'+money(e.paidAmt)+'</td><td class="num">'+money(supplyDue(e))+'</td></tr>';
    }).join('');
    var due = Math.round((totalBilled-totalPaid)*100)/100;

    var html = '<h1>'+esc(c.name)+'</h1>';
    html += '<div class="stmt-sub">'+(c.phone?esc(c.phone)+' &middot; ':'')+'Statement: '+dateShort(from)+' &ndash; '+dateShort(to)+'</div>';
    if(!entries.length){
      html += '<p style="font-size:13px;color:#555;">No supply entries in this date range.</p>';
    } else {
      html += '<table><thead><tr><th>Date</th><th class="num">Qty</th><th class="num">Rate</th><th class="num">Amount</th><th class="num">Paid</th><th class="num">Due</th></tr></thead><tbody>'+rows+'</tbody></table>';
    }
    html += '<div class="stmt-totals">' +
      '<div><span>Total billed</span><b class="num">'+money(totalBilled)+'</b></div>' +
      '<div><span>Total paid</span><b class="num">'+money(totalPaid)+'</b></div>' +
      '<div class="due"><span>Balance due</span><b class="num">'+money(due)+'</b></div>' +
      '</div>';
    html += '<div class="stmt-gen">Generated on '+dateLabel(todayISO())+'</div>';

    document.getElementById('printArea').innerHTML = html;
    window.print();
  }

  function lastBackupLabel(){
    var iso = localStorage.getItem(BACKUP_KEY);
    return iso ? dateLabel(iso) : 'Never backed up';
  }
  function lastBackupOverdue(){
    var iso = localStorage.getItem(BACKUP_KEY);
    if(!iso) return true;
    var days = (Date.parse(todayISO()) - Date.parse(iso)) / 86400000;
    return days > 7;
  }
  function exportData(){
    var blob = new Blob([JSON.stringify(db, null, 2)], {type:'application/json'});
    var url = URL.createObjectURL(blob);
    var a = document.createElement('a');
    a.href = url;
    a.download = 'dairy-ledger-backup-' + todayISO() + '.json';
    document.body.appendChild(a); a.click(); document.body.removeChild(a);
    URL.revokeObjectURL(url);
    localStorage.setItem(BACKUP_KEY, todayISO());
    toast('Backup downloaded');
    render();
  }
  function importData(ev){
    var file = ev.target.files[0];
    if(!file) return;
    var reader = new FileReader();
    reader.onload = function(){
      try{
        var parsed = JSON.parse(reader.result);
        var d = defaultDB();
        for(var k in d){ if(Array.isArray(parsed[k])) d[k] = parsed[k]; }
        db = d; syncAfterReplaceAll(d); toast('Backup restored'); render();
      }catch(e){ toast('Invalid backup file'); }
      ev.target.value = '';
    };
    reader.readAsText(file);
  }

  function emptyState(title, sub){
    return '<div class="empty">' +
      '<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6"><circle cx="12" cy="12" r="9"/><path stroke-linecap="round" d="M9 12h6M12 9v6"/></svg>' +
      '<h3 style="font-size:15px;margin-bottom:4px;">'+esc(title)+'</h3><p>'+esc(sub)+'</p></div>';
  }

  /* ================= render dispatch ================= */
  function render(){
    $monthLabel.textContent = monthLabel(state.month);
    if(state.tab==='home') renderHome();
    else if(state.tab==='supply') renderSupply();
    else if(state.tab==='production') renderProduction();
    else if(state.tab==='expenses') renderExpenses();
    else if(state.tab==='more') renderMore();
  }

  $fab.classList.toggle('hidden', !!FAB_HIDDEN_TABS[state.tab]);
  render();
  restoreSession();

  if('serviceWorker' in navigator && (location.protocol==='http:' || location.protocol==='https:')){
    window.addEventListener('load', function(){
      navigator.serviceWorker.register('sw.js').catch(function(){});
    });
  }
})();
</script>
</body>
</html>
