<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport"
  content="width=device-width,initial-scale=1,maximum-scale=1,viewport-fit=cover,user-scalable=no" />
<title>Pixel Valley Pro — Top HUD + Global Open/Close</title>
<style>
  :root {
    --dark-bg: #0e1320;
    --dark-wood: #4b3522;
    --dark-wood-2: #6b4a2f;
    --dark-wood-3: #8a613e;
    --dark-panel: #172235;
    --dark-text: #e8f0ff;
    --dark-muted: #9fb2ce;

    --grass-dark: #5e9a68;
    --grass-light: #a8d6af;
    --soil-dark: #4a3a2a;
    --soil-light: #c59a6f;

    --accent: #79c07a;
    --accent2: #d2a358;
    --danger: #c85b5b;

    --light-bg: #eaf1f9;
    --light-wood-2: #e0b88f;
    --light-panel: #f5f8ff;
    --light-text: #0a213d;

    --metal-1: #6d7684;
    --metal-2: #8791a3;
  }

  * { box-sizing: border-box; }
  html, body {
    margin:0; padding:0; height:100%;
    background: var(--dark-bg); color: var(--dark-text);
    font-family: system-ui, -apple-system, Segoe UI, Roboto, Helvetica, Arial, sans-serif;
    overscroll-behavior: none;
  }

  /* Layout: top 10% HUD, bottom 90% game */
  #app {
    position: fixed; inset: 0;
    display: grid;
    grid-template-rows: 10% 90%;
    gap: 0;
    padding: env(safe-area-inset-top) 0 env(safe-area-inset-bottom);
  }

  /* Top HUD 10% */
  .topbar {
    display:grid;
    grid-template-columns: 1fr auto;
    align-items:center; gap:8px;
    background: var(--dark-wood-2);
    border-bottom: 2px solid #2b1d12;
    padding: 6px 8px;
  }
  .hudLeft { display:flex; gap:8px; align-items:center; flex-wrap:wrap; }
  .hudRight { display:flex; gap:8px; align-items:center; flex-wrap:wrap; }
  .pill {
    background: var(--dark-wood-2); color: var(--dark-muted);
    border: 1px solid #2b1d12; border-radius: 999px;
    padding: 3px 8px; font-size: 12px; display: inline-flex; gap: 6px; align-items: center;
  }
  .btn {
    background: var(--accent); color: #052c12;
    border: none; border-radius: 12px; padding: 8px 12px; font-weight: 800; cursor: pointer;
    box-shadow: 0 2px 0 rgba(0,0,0,0.3);
  }
  .btn.dark { background: var(--dark-panel); color: var(--dark-text); border: 1px solid #0a1424; }
  .btn.secondary { background: var(--accent2); color: #3a2500; }
  .btn.danger { background: var(--danger); color: #fff; }
  .btn.tab { background: #2b3648; color: #cfe2ff; }
  .btn.enter { background: #2a6f3a; }
  .btn.enter.hidden { display: none; }

  /* Bottom 90% game area */
  .gameArea {
    position: relative;
    background: var(--grass-dark);
    overflow:hidden;
  }
  #gameCanvas {
    width: 100%; height: 100%;
    display: block; image-rendering: pixelated;
    background: var(--grass-dark);
    touch-action: manipulation;
  }

  /* Floating control dock */
  .dock {
    position: absolute; left: 8px; bottom: 8px;
    display:flex; gap:8px; align-items:flex-end; z-index: 20;
  }
  .pad {
    display:grid; grid-template-columns: repeat(3,48px); grid-template-rows: repeat(3,48px);
    gap:6px; padding:6px; background: rgba(30, 20, 12, 0.45);
    border: 2px solid #2b1d12; border-radius: 12px; backdrop-filter: blur(2px);
  }
  .pad .arrowBtn {
    width: 48px; height: 48px; border-radius: 10px;
    border: 1px solid #2b1d12; background: var(--dark-wood-3); color: var(--dark-text);
    font-weight: 900; font-size: 20px; cursor: pointer;
    box-shadow: inset 0 1px 0 rgba(255,255,255,0.06), 0 2px 0 #2b1d12;
    touch-action: manipulation;
  }
  .dockActions { display:flex; flex-direction:column; gap:8px; }

  /* Minimap */
  .minimap {
    position:absolute; right:8px; top:8px; z-index:20;
    width: 120px; height: 80px; border: 2px solid #2b1d12; border-radius:8px;
    background: #101826; box-shadow: 0 6px 18px rgba(0,0,0,0.35);
  }
  #miniCanvas { width:100%; height:100%; display:block; image-rendering:pixelated; }

  /* Toasts */
  .toast {
    position: fixed; left: 50%; bottom: 10%; transform: translateX(-50%);
    background: #0f1624; color: #cfe2ff; border: 1px solid #0a1424; padding: 8px 12px; border-radius: 8px;
    box-shadow: 0 6px 18px rgba(0,0,0,0.35); z-index: 50;
  }

  /* Drawer */
  .drawer {
    position: absolute; right: 8px; bottom: 8px;
    width: min(440px, 54vw); max-height: 72vh; overflow:auto;
    background: var(--dark-wood); border: 2px solid #2b1d12; border-radius: 12px;
    box-shadow: 0 12px 32px rgba(0,0,0,0.35);
    padding: 8px; z-index: 30; display: none;
  }
  .grid3 { display: grid; grid-template-columns: repeat(3, minmax(0,1fr)); gap: 8px; }
  .card {
    background: var(--dark-wood-3);
    border: 1px solid #2b1d12; border-radius: 8px; padding: 6px; text-align: left; cursor: pointer;
    box-shadow: inset 0 1px 0 rgba(255,255,255,0.06);
  }
  .cardTitle { font-weight:800; margin-bottom:4px; }
  .small { font-size: 11px; opacity: 0.85; }

  /* Title screen */
  #titleOverlay {
    position: fixed; inset: 0; z-index: 9999;
    display: grid; grid-template-rows: 1fr auto; place-items: center;
    background: #000;
  }
  .titleBox { width: min(860px, 92vw); text-align: center; color: #cfe2ff; }
  .titleLogo { font-size: clamp(32px, 8vw, 72px); font-weight: 900; letter-spacing: 1px; margin-bottom: 12px; text-shadow: 0 4px 0 #0a1424; }
  .titleSub { color: #9fb2ce; margin-bottom: 20px; }
  .titleControls { display: flex; flex-wrap: wrap; gap: 12px; justify-content: center; }
  .titleFooter { color: #8aa3c3; padding: 16px; font-size: 12px; }

  /* Veil */
  #veil {
    position: fixed; inset: 0; background: #000;
    z-index: 9998; pointer-events: none; opacity: 1; transition: opacity 1200ms ease;
  }

  /* Shop interior (reused for Seed/Workshop/Secret) */
  #shopInterior {
    position: fixed; inset: 0;
    display: none; grid-template-rows: auto 1fr auto;
    z-index: 40;
  }
  .shopHeader {
    display:flex; align-items:center; justify-content: space-between; gap:8px;
    padding:8px; background: var(--dark-wood-2); border-bottom: 2px solid #2b1d12;
  }
  .shopBody { position: relative; overflow:auto; padding:0; }
  .shopFooter { display:flex; justify-content:flex-end; gap:8px; padding:8px;
    background: var(--dark-wood-2); border-top: 2px solid #2b1d12; }
  .interiorWrap { position: relative; min-height: 100%; }
  .interiorWrap.seed {
    background:
      linear-gradient(0deg, rgba(0,0,0,0.05), rgba(0,0,0,0.08)),
      repeating-linear-gradient(90deg, #5c402b 0 12px, #5a3e2a 12px 24px);
  }
  .interiorWrap.workshop {
    background:
      linear-gradient(180deg, rgba(20,24,32,0.55), rgba(20,24,32,0.65)),
      repeating-linear-gradient(0deg, var(--metal-1) 0 10px, var(--metal-2) 10px 20px);
  }
  .interiorWrap.secret {
    background:
      radial-gradient(300px 120px at 10% 20%, rgba(255,255,255,0.05), transparent 65%),
      linear-gradient(180deg, rgba(10,10,14,0.85), rgba(10,10,14,0.95));
  }
  .shelf {
    position: relative; margin: 12px auto; width: min(760px, 94vw);
    display: grid; grid-template-columns: repeat(4, minmax(0,1fr)); gap: 8px;
    background: var(--dark-wood-3);
    border: 2px solid #2b1d12; border-radius: 12px; padding: 8px;
    box-shadow: inset 0 2px 0 rgba(255,255,255,0.06);
  }

  /* Overlays */
  .overlay {
    position: fixed; inset: 0; display: none; z-index: 60;
    grid-template-rows: auto 1fr auto;
    background: rgba(0,0,0,0.75); backdrop-filter: blur(2px);
  }
  .overlayHeader, .overlayFooter {
    background: var(--dark-wood-2); padding: 8px; border-bottom: 2px solid #2b1d12;
    display:flex; align-items:center; justify-content: space-between; gap:8px;
  }
  .overlayFooter { border-top: 2px solid #2b1d12; border-bottom: none; }
  .overlayBody { background: var(--dark-wood); padding: 8px; overflow:auto; }
</style>
</head>
<body>
<div id="app">
  <!-- Top 10% HUD -->
  <div class="topbar">
    <div class="hudLeft">
      <div class="pill" id="moneyPill">Money: $100</div>
      <div class="pill" id="succPill">Success: 65%</div>
      <div class="pill" id="statPill">Fail:0 • Harv:0 • Upg:0</div>
      <div class="pill" id="dayPill">Day 1 • Spring • ☀️ Clear</div>
      <div class="pill" id="timePill">06:00</div>
      <div class="pill" id="stamPill">Stamina: 100%</div>
    </div>
    <div class="hudRight">
      <!-- Saves -->
      <button class="btn dark" id="saveBtn">Save</button>
      <button class="btn dark" id="loadBtn">Load</button>
      <button class="btn dark" id="optionsBtn">Options</button>

      <!-- Global open/close -->
      <button class="btn tab" id="openSeedBtn">Open Seed Shop</button>
      <button class="btn tab" id="openWorkBtn">Open Workshop</button>
      <button class="btn tab" id="openSecretBtn">Open Secret Shop</button>
      <button class="btn tab" id="openQuestHallBtn">Open Quest Hall</button>
      <button class="btn secondary" id="closeAllBtn">Close All</button>

      <!-- Gameplay panels -->
      <button class="btn tab" id="btnInv">Inventory</button>
      <button class="btn tab" id="btnCraft">Craft</button>
      <button class="btn tab" id="btnCook">Cook</button>
      <button class="btn tab" id="btnFish">Fish</button>
      <button class="btn tab" id="btnAnimals">Animals</button>
      <button class="btn tab" id="btnStorage">Storage</button>
      <button class="btn tab" id="btnQuests">Quests</button>
      <button class="btn tab" id="btnAch">Achievements</button>
      <button class="btn enter hidden" id="btnEnter">Enter</button>
    </div>
  </div>

  <!-- Bottom 90% game canvas -->
  <div class="gameArea">
    <canvas id="gameCanvas"></canvas>
    <div class="minimap"><canvas id="miniCanvas"></canvas></div>

    <!-- Floating dock -->
    <div class="dock">
      <div class="pad">
        <button class="arrowBtn" aria-label="empty"></button>
        <button class="arrowBtn" id="btnUp" aria-label="move up">↑</button>
        <button class="arrowBtn" aria-label="empty"></button>
        <button class="arrowBtn" id="btnLeft" aria-label="move left">←</button>
        <button class="arrowBtn" id="btnInteract" aria-label="interact">◎</button>
        <button class="arrowBtn" id="btnRight" aria-label="move right">→</button>
        <button class="arrowBtn" aria-label="empty"></button>
        <button class="arrowBtn" id="btnDown" aria-label="move down">↓</button>
        <button class="arrowBtn" aria-label="empty"></button>
      </div>
      <div class="dockActions">
        <button class="btn" id="sellReadyBtn">Sell ready</button>
        <button class="btn dark" id="clearDeadBtn">Clear dead</button>
        <button class="btn secondary" id="resetBtn">Reset</button>
        <button class="btn dark" id="sleepBtn">Sleep</button>
      </div>
    </div>

    <!-- Drawer -->
    <div class="drawer" id="invDrawer">
      <div class="pill" style="margin-bottom:8px;">Panel</div>
      <div class="grid3" id="invGrid"></div>
    </div>
  </div>
</div>

<!-- Title overlay -->
<div id="titleOverlay">
  <div class="titleBox">
    <div class="titleLogo">🏡 Pixel Valley Pro</div>
    <div class="titleSub">Global shop open/close, quest hall, seasons, weather, day/night, stamina, autosave, minimap.</div>
    <div class="titleControls">
      <button class="btn" id="startBtn">Start</button>
      <button class="btn dark" id="optBtn">Options</button>
      <button class="btn dark" id="titleLoadBtn">Load save</button>
    </div>
    <div id="titleOptions" style="display:none; margin-top: 12px;">
      <div style="display:flex; gap:12px; flex-wrap:wrap; justify-content:center;">
        <div class="pill" style="gap:8px;">
          <strong>Theme:</strong>
          <button class="btn dark" id="optLight">Light</button>
          <button class="btn dark" id="optDark">Dark</button>
        </div>
        <div class="pill" style="gap:8px;">
          <strong>Orientation:</strong>
          <button class="btn dark" id="orientPortrait">Portrait</button>
          <button class="btn dark" id="orientLandscape">Landscape</button>
          <button class="btn dark" id="orientAuto">Auto</button>
        </div>
        <div class="pill" style="gap:8px;">
          <strong>Pixel scale:</strong>
          <button class="btn dark" id="pix1">1x</button>
          <button class="btn dark" id="pix2">2x</button>
          <button class="btn dark" id="pix3">3x</button>
        </div>
        <div class="pill" style="gap:8px;">
          <strong>Autosave:</strong>
          <button class="btn dark" id="autoOn">On</button>
          <button class="btn dark" id="autoOff">Off</button>
        </div>
      </div>
    </div>
    <div class="titleFooter">Tap Start to play. Use HUD buttons to open/close shops and quest hall anytime.</div>
  </div>
</div>

<!-- Veil -->
<div id="veil"></div>

<!-- Shop interior -->
<div id="shopInterior">
  <div class="shopHeader">
    <div style="display:flex; gap:8px; align-items:center;">
      <div class="pill" id="interiorTitle">Inside: Shop</div>
      <div class="pill" id="shopMoneyPill">Money: $100</div>
    </div>
    <div style="display:flex; gap:8px;">
      <button class="btn dark" id="leaveShopBtn">Close</button>
    </div>
  </div>
  <div class="shopBody">
    <div class="interiorWrap seed" id="interiorWrap">
      <div class="shelf" id="interiorShelf"></div>
      <div class="shelf"><div style="grid-column: 1/-1;"><div class="pill" id="interiorTip">Bundles of 5 get 5% off.</div></div></div>
    </div>
  </div>
  <div class="shopFooter">
    <button class="btn" id="bundleBtn">Buy 5 random (-5%)</button>
    <button class="btn danger" id="sellAllBtn">Sell all ready crops</button>
  </div>
</div>

<!-- Quest hall overlay -->
<div class="overlay" id="questsOverlay">
  <div class="overlayHeader">
    <div class="pill">Quest Hall</div>
    <div style="display:flex; gap:8px;">
      <div class="pill" id="questMoneyPill">Money: $100</div>
      <button class="btn dark" id="closeQuestsBtn">Close</button>
    </div>
  </div>
  <div class="overlayBody"><div class="grid3" id="questList"></div></div>
  <div class="overlayFooter">
    <div class="pill">Complete tasks for rewards.</div>
    <button class="btn" id="refreshQuestsBtn">Refresh</button>
  </div>
</div>

<!-- Crafting -->
<div class="overlay" id="craftOverlay">
  <div class="overlayHeader">
    <div class="pill">Crafting Bench</div>
    <button class="btn dark" id="closeCraftBtn">Close</button>
  </div>
  <div class="overlayBody"><div class="grid3" id="craftGrid"></div></div>
  <div class="overlayFooter"><div class="pill">Combine items for tools and boosts.</div></div>
</div>

<!-- Cooking -->
<div class="overlay" id="cookOverlay">
  <div class="overlayHeader">
    <div class="pill">Kitchen</div>
    <button class="btn dark" id="closeCookBtn">Close</button>
  </div>
  <div class="overlayBody"><div class="grid3" id="cookGrid"></div></div>
  <div class="overlayFooter"><div class="pill">Cook meals to restore stamina.</div></div>
</div>

<!-- Fishing -->
<div class="overlay" id="fishOverlay">
  <div class="overlayHeader">
    <div class="pill">Fishing</div>
    <button class="btn dark" id="closeFishBtn">Close</button>
  </div>
  <div class="overlayBody">
    <div id="fishArea" style="height:240px; display:grid; place-items:center; background:#0f2a3f; border:2px solid #0a1424; border-radius:12px;">
      <button class="btn" id="fishCastBtn">Cast</button>
    </div>
    <div class="grid3" id="fishGrid"></div>
  </div>
  <div class="overlayFooter"><div class="pill">Time your tap when the bobber glows.</div></div>
</div>

<!-- Animals -->
<div class="overlay" id="animalsOverlay">
  <div class="overlayHeader">
    <div class="pill">Barn & Coop</div>
    <button class="btn dark" id="closeAnimalsBtn">Close</button>
  </div>
  <div class="overlayBody"><div class="grid3" id="animalsGrid"></div></div>
  <div class="overlayFooter"><div class="pill">Feed animals for daily bonuses.</div></div>
</div>

<!-- Storage -->
<div class="overlay" id="storageOverlay">
  <div class="overlayHeader">
    <div class="pill">Storage Chest</div>
    <button class="btn dark" id="closeStorageBtn">Close</button>
  </div>
  <div class="overlayBody"><div class="grid3" id="storageGrid"></div></div>
  <div class="overlayFooter">
    <button class="btn dark" id="depositAllBtn">Deposit all</button>
    <button class="btn" id="withdrawAllBtn">Withdraw all</button>
  </div>
</div>

<!-- Achievements -->
<div class="overlay" id="achOverlay">
  <div class="overlayHeader">
    <div class="pill">Achievements</div>
    <button class="btn dark" id="closeAchBtn">Close</button>
  </div>
  <div class="overlayBody"><div class="grid3" id="achGrid"></div></div>
  <div class="overlayFooter"><div class="pill">Milestones grant cosmetics and boosts.</div></div>
</div>

<script>
(function(){
  'use strict';

  /* ===== Config ===== */
  const MAX_SUCCESS = 0.95, BASE_SUCCESS = 0.65;
  const PORTRAIT_W = 360, PORTRAIT_H = 640;
  const LANDSCAPE_W = 640, LANDSCAPE_H = 360;
  let orientationMode = 'auto';
  let canvasLogical = { w: PORTRAIT_W, h: PORTRAIT_H };
  let pixelScale = 2;
  const tileBase = 16;
  let tileSize = tileBase * pixelScale;
  const PLAYER_SPEED = 1.0;
  const INTERACT_RADIUS = 12;
  const AUTOSAVE_INTERVAL_MS = 20000;

  /* ===== Data ===== */
  const seeds = [
    {id:'Turnip', price:10, grow:6, sell:18, color:'#9ad47a'},
    {id:'Potato', price:18, grow:8, sell:32, color:'#a0894e'},
    {id:'Carrot', price:12, grow:7, sell:22, color:'#e07a3f'},
    {id:'Tomato', price:20, grow:9, sell:36, color:'#c84d4d'},
    {id:'Corn', price:24, grow:11, sell:44, color:'#e6d65f'},
    {id:'Wheat', price:8, grow:6, sell:16, color:'#d8b46a'},
    {id:'Strawberry', price:30, grow:12, sell:60, color:'#df3b53'},
    {id:'Blueberry', price:28, grow:12, sell:56, color:'#5f78d6'},
    {id:'Pumpkin', price:40, grow:15, sell:86, color:'#f09a3e'},
    {id:'Sunflower', price:22, grow:10, sell:42, color:'#f2c04e'},
    {id:'Grape', price:26, grow:12, sell:52, color:'#7a4bb3'},
    {id:'Beet', price:16, grow:7, sell:28, color:'#a03a55'},
    {id:'Cabbage', price:18, grow:9, sell:34, color:'#6fbf6f'},
    {id:'Onion', price:14, grow:8, sell:26, color:'#c8b58f'},
    {id:'Coffee', price:36, grow:14, sell:78, color:'#5a3a2a'}
  ];
  const industry = [
    {id:'Irrigation Kit', price:150, boost:0.05, desc:'Watering improves germination.'},
    {id:'Soil Test Pack', price:220, boost:0.05, desc:'Match pH for sprout rates.'},
    {id:'Compost Bin', price:300, boost:0.05, desc:'Microbes, nutrients, resilience.'},
    {id:'Greenhouse Cover', price:420, boost:0.07, desc:'Shield from weather failures.'},
    {id:'Seed Treater', price:600, boost:0.08, desc:'Pre-treated seeds reduce failure.'},
    {id:'Smart Sprinklers', price:800, boost:0.05, desc:'Consistent moisture control.'}
  ];
  const secrets = [
    {id:'Lucky Charm', price:750, boost:0.05, desc:'Whispers of the valley favor you.'},
    {id:'Ancient Soil', price:900, boost:0.05, desc:'Old fertility, new fortune.'},
    {id:'Moonlight Lamp', price:1200, boost:0.03, desc:'Night growth blessings.'}
  ];
  const crafts = [
    {id:'Sprinkler', req:{Wheat:3, Turnip:2}, effect:{boost:0.02}, desc:'Tiny moisture boost.'},
    {id:'Compost Tea', req:{Beet:2, Cabbage:1}, effect:{boost:0.02}, desc:'Nutrient splash.'},
    {id:'Seed Pouches', req:{Corn:2, Onion:2}, effect:{invSlots:6}, desc:'Carry more seeds.'}
  ];
  const recipes = [
    {id:'Hearty Stew', req:{Potato:2, Carrot:2, Onion:1}, stamina:40, desc:'Restores stamina.'},
    {id:'Fruit Jam', req:{Strawberry:2, Blueberry:2}, stamina:20, desc:'Sweet boost.'},
    {id:'Pumpkin Pie', req:{Pumpkin:1, Wheat:2}, stamina:50, desc:'Comfort food.'}
  ];
  const fishables = [
    {id:'Minnow', chance:0.5, price:8},
    {id:'Trout', chance:0.35, price:22},
    {id:'Salmon', chance:0.12, price:38},
    {id:'Golden Koi', chance:0.03, price:120},
  ];
  const animalsData = [
    {id:'Chicken', price:120, feed:'Wheat', bonus:{money:12}, desc:'Lays eggs for cash.'},
    {id:'Cow', price:320, feed:'Corn', bonus:{boost:0.01}, desc:'Milk morale bonus.'},
    {id:'Beehive', price:260, feed:'Sunflower', bonus:{sellBonus:0.05}, desc:'Honey sales +5%.'}
  ];

  const GRID_W = 5, GRID_H = 6;
  const seasons = ['Spring','Summer','Autumn','Winter'];
  const weatherTypes = [
    {id:'Clear', icon:'☀️', mod:0},
    {id:'Rain', icon:'🌧️', mod:+0.03},
    {id:'Wind', icon:'🌬️', mod:-0.02},
    {id:'Storm', icon:'⛈️', mod:-0.05},
    {id:'Fog', icon:'🌫️', mod:-0.01},
  ];
  const QUESTS_POOL = [
    {id:'harvest10', label:'Harvest 10 crops', goal:10, reward:{money:120, seed:'Pumpkin'}},
    {id:'plant15', label:'Plant 15 seeds', goal:15, reward:{money:90, seed:'Strawberry'}},
    {id:'collect7', label:'Collect 7 seed types', goal:7, useDistinct:true, reward:{money:140, seed:'Coffee'}},
    {id:'noFail5', label:'Harvest 5 with no failures', goal:5, noFail:true, reward:{money:160, seed:'Blueberry'}},
    {id:'sell500', label:'Earn $500 from sales', goal:500, reward:{money:200, seed:'Corn'}, trackMoney:true},
    {id:'fish5', label:'Catch 5 fish', goal:5, reward:{money:80, seed:'Tomato'}, trackFish:true},
  ];
  const achievements = [
    {id:'FirstHarvest', label:'First Harvest', cond: s=>s.stats.harvests>=1, reward:'+ Cosmetic hat'},
    {id:'GreenThumb', label:'50 Harvests', cond: s=>s.stats.harvests>=50, reward:'+2% success'},
    {id:'Collector', label:'10 Seed Types', cond: s=>s.stats.distinctSeeds.size>=10, reward:'Special banner'},
    {id:'Angler', label:'10 Fish', cond: s=>s.stats.fishCaught>=10, reward:'Fishing badge'}
  ];

  /* ===== State ===== */
  const state = {
    theme:'dark', ready:false, money:100, boosts:0,
    autosave:true,
    day:1, timeMin:360, // 06:00
    stamina:100, seasonIndex:0, weatherIndex:0,
    inv:{}, stats:{ failures:0, harvests:0, upgrades:0, distinctSeeds:new Set(), soldMoney:0, fishCaught:0 },
    plots:Array.from({length: GRID_W*GRID_H}, ()=>({seedId:null, plantedAt:null, grow:0, ready:false, dead:false, color:null, sell:0, fail:false})),
    purchasedUpgrades:new Set(), purchasedSecrets:new Set(),
    selectedSeed:null, inInterior:false, interiorType:null,
    quests:[], questProgress:{}, questsCompleted:new Set(),
    animals:[], storage:{},
    saveSlot:'slot1'
  };

  /* ===== DOM ===== */
  const canvas = document.getElementById('gameCanvas');
  const ctx = canvas.getContext('2d');
  const miniCanvas = document.getElementById('miniCanvas');
  const miniCtx = miniCanvas.getContext('2d');

  const moneyPill = document.getElementById('moneyPill');
  const succPill = document.getElementById('succPill');
  const statPill = document.getElementById('statPill');
  const dayPill = document.getElementById('dayPill');
  const timePill = document.getElementById('timePill');
  const stamPill = document.getElementById('stamPill');

  const saveBtn = document.getElementById('saveBtn');
  const loadBtn = document.getElementById('loadBtn');
  const optionsBtn = document.getElementById('optionsBtn');

  const openSeedBtn = document.getElementById('openSeedBtn');
  const openWorkBtn = document.getElementById('openWorkBtn');
  const openSecretBtn = document.getElementById('openSecretBtn');
  const openQuestHallBtn = document.getElementById('openQuestHallBtn');
  const closeAllBtn = document.getElementById('closeAllBtn');

  const btnUp = document.getElementById('btnUp');
  const btnDown = document.getElementById('btnDown');
  const btnLeft = document.getElementById('btnLeft');
  const btnRight = document.getElementById('btnRight');
  const btnInteract = document.getElementById('btnInteract');
  const btnEnter = document.getElementById('btnEnter');

  const btnInv = document.getElementById('btnInv');
  const btnCraft = document.getElementById('btnCraft');
  const btnCook = document.getElementById('btnCook');
  const btnFish = document.getElementById('btnFish');
  const btnAnimals = document.getElementById('btnAnimals');
  const btnStorage = document.getElementById('btnStorage');
  const btnQuests = document.getElementById('btnQuests');
  const btnAch = document.getElementById('btnAch');

  const invDrawer = document.getElementById('invDrawer');
  const invGrid = document.getElementById('invGrid');

  const titleOverlay = document.getElementById('titleOverlay');
  const startBtn = document.getElementById('startBtn');
  const optBtn = document.getElementById('optBtn');
  const titleOptions = document.getElementById('titleOptions');
  const optLight = document.getElementById('optLight');
  const optDark = document.getElementById('optDark');
  const orientPortrait = document.getElementById('orientPortrait');
  const orientLandscape = document.getElementById('orientLandscape');
  const orientAuto = document.getElementById('orientAuto');
  const pix1 = document.getElementById('pix1');
  const pix2 = document.getElementById('pix2');
  const pix3 = document.getElementById('pix3');
  const titleLoadBtn = document.getElementById('titleLoadBtn');
  const autoOn = document.getElementById('autoOn');
  const autoOff = document.getElementById('autoOff');

  const veil = document.getElementById('veil');

  const shopInterior = document.getElementById('shopInterior');
  const interiorTitle = document.getElementById('interiorTitle');
  const shopMoneyPill = document.getElementById('shopMoneyPill');
  const interiorShelf = document.getElementById('interiorShelf');
  const leaveShopBtn = document.getElementById('leaveShopBtn');
  const bundleBtn = document.getElementById('bundleBtn');
  const sellAllBtn = document.getElementById('sellAllBtn');
  const interiorWrap = document.getElementById('interiorWrap');
  const interiorTip = document.getElementById('interiorTip');

  const questsOverlay = document.getElementById('questsOverlay');
  const questList = document.getElementById('questList');
  const questMoneyPill = document.getElementById('questMoneyPill');
  const refreshQuestsBtn = document.getElementById('refreshQuestsBtn');
  const closeQuestsBtn = document.getElementById('closeQuestsBtn');

  const craftOverlay = document.getElementById('craftOverlay');
  const craftGrid = document.getElementById('craftGrid');
  const closeCraftBtn = document.getElementById('closeCraftBtn');

  const cookOverlay = document.getElementById('cookOverlay');
  const cookGrid = document.getElementById('cookGrid');
  const closeCookBtn = document.getElementById('closeCookBtn');

  const fishOverlay = document.getElementById('fishOverlay');
  const fishGrid = document.getElementById('fishGrid');
  const fishCastBtn = document.getElementById('fishCastBtn');
  const closeFishBtn = document.getElementById('closeFishBtn');

  const animalsOverlay = document.getElementById('animalsOverlay');
  const animalsGrid = document.getElementById('animalsGrid');
  const closeAnimalsBtn = document.getElementById('closeAnimalsBtn');

  const storageOverlay = document.getElementById('storageOverlay');
  const storageGrid = document.getElementById('storageGrid');
  const depositAllBtn = document.getElementById('depositAllBtn');
  const withdrawAllBtn = document.getElementById('withdrawAllBtn');
  const closeStorageBtn = document.getElementById('closeStorageBtn');

  const achOverlay = document.getElementById('achOverlay');
  const achGrid = document.getElementById('achGrid');
  const closeAchBtn = document.getElementById('closeAchBtn');

  /* ===== Utilities ===== */
  function nowSec(){ return Math.floor(Date.now()/1000); }
  function clamp(v,min,max){ return Math.max(min, Math.min(max, v)); }
  function dist(ax, ay, bx, by){ const dx=ax-bx, dy=ay-by; return Math.sqrt(dx*dx+dy*dy); }
  function hhmm(min){ const h = Math.floor(min/60), m = Math.floor(min%60); return `${String(h).padStart(2,'0')}:${String(m).padStart(2,'0')}`; }
  function toast(msg){
    const el = document.createElement('div'); el.className = 'toast'; el.textContent = msg;
    document.body.appendChild(el); setTimeout(()=> el.remove(), 1600);
  }

  /* ===== Orientation & Canvas ===== */
  function chooseLogicalSize(){
    if (orientationMode==='portrait') return { w: PORTRAIT_W, h: PORTRAIT_H };
    if (orientationMode==='landscape') return { w: LANDSCAPE_W, h: LANDSCAPE_H };
    const vw = window.innerWidth, vh = window.innerHeight;
    return (vh >= vw) ? { w: PORTRAIT_W, h: PORTRAIT_H } : { w: LANDSCAPE_W, h: LANDSCAPE_H };
  }
  function resizeCanvas(){
    canvasLogical = chooseLogicalSize();
    canvas.width = canvasLogical.w;
    canvas.height = canvasLogical.h;
    const areaRect = document.querySelector('.gameArea').getBoundingClientRect();
    const aspect = canvasLogical.w / canvasLogical.h;
    let w = areaRect.width, h = areaRect.height;
    const cur = w / h;
    if (cur > aspect) w = h * aspect; else h = w / aspect;
    canvas.style.width = Math.round(w) + 'px';
    canvas.style.height = Math.round(h) + 'px';
    miniCanvas.width = 120; miniCanvas.height = 80;
    positionWorld();
  }
  window.addEventListener('resize', resizeCanvas);

  /* ===== World ===== */
  let world = {
    seedShop:  { x: 24, y: 88, w: 72, h: 56, label: 'Seed Shop', color: '#cf7967' },
    indShop:   { x: 0,  y: 88, w: 72, h: 56, label: 'Workshop', color: '#6e91d1' },
    secShop:   { x: 0,  y: 0,  w: 72, h: 56, label: 'Back Room', color: '#e0c766' },
    farmhouse: { x: 24, y: 0,  w: 92, h: 68, label: 'Farmhouse', color: '#a6784f' },
    questHall: { x: 0,  y: 0,  w: 92, h: 62, label: 'Quest Hall', color: '#7a4bb3' },
    pond:      { x: 0,  y: 0,  w: 84, h: 54, label: 'Pond', color: '#2f6ea8' },
    barn:      { x: 0,  y: 0,  w: 86, h: 62, label: 'Barn', color: '#b24d4d' },
    chest:     { x: 0,  y: 0,  w: 52, h: 36, label: 'Chest', color: '#7b5a38' },
    garden:    { x: 0,  y: 0, cols: GRID_W, rows: GRID_H },
    doors:     [], trees:[]
  };
  const player = { x: PORTRAIT_W/2, y: PORTRAIT_H/2, w: tileBase, h: tileBase, speed: PLAYER_SPEED * pixelScale, vx:0, vy:0, walkTimer:0 };

  function positionWorld(){
    const W = canvasLogical.w, H = canvasLogical.h;
    world.seedShop.x = 24; world.seedShop.y = 88;
    world.indShop.x  = W - (world.indShop.w + 24); world.indShop.y = 88;
    world.secShop.x  = W/2 - world.secShop.w/2; world.secShop.y = H - (world.secShop.h + 84);
    world.farmhouse.x= 24; world.farmhouse.y = H - (world.farmhouse.h + 84);
    world.questHall.x= W/2 - world.questHall.w/2; world.questHall.y = 24;
    world.barn.x     = W - (world.barn.w + 24); world.barn.y = H - (world.barn.h + 84);
    world.pond.x     = W/2 - world.pond.w/2; world.pond.y = H/2 - world.pond.h/2 + 40;
    world.chest.x    = 24; world.chest.y = 28;
    world.garden.x   = W/2 - (tileSize*3); world.garden.y = H/2 - (tileSize*3);

    world.doors = [
      { id:'seed', rect:{ x: world.seedShop.x + world.seedShop.w/2 - 10, y: world.seedShop.y + world.seedShop.h, w: 20, h: 12 }, title:'Inside: Seed Shop' },
      { id:'industrial', rect:{ x: world.indShop.x + world.indShop.w/2 - 10, y: world.indShop.y + world.indShop.h, w: 20, h: 12 }, title:'Inside: Workshop' },
      { id:'secret', rect:{ x: world.secShop.x + world.secShop.w/2 - 10, y: world.secShop.y + world.secShop.h, w: 20, h: 12 }, title:'Inside: Back Room' },
      { id:'farmhouse', rect:{ x: world.farmhouse.x + world.farmhouse.w/2 - 10, y: world.farmhouse.y + world.farmhouse.h, w: 20, h: 12 }, title:'Inside: Farmhouse' },
      { id:'questhall', rect:{ x: world.questHall.x + world.questHall.w/2 - 10, y: world.questHall.y + world.questHall.h, w: 20, h: 12 }, title:'Quest Hall' },
      { id:'barn', rect:{ x: world.barn.x + world.barn.w/2 - 10, y: world.barn.y + world.barn.h, w: 20, h: 12 }, title:'Inside: Barn' }
    ];

    world.trees = [];
    for (let i=0;i<12;i++){
      const tx = Math.random() * (W - 32) + 16;
      const ty = Math.random() * (H - 32) + 16;
      const gx = W/2, gy = H/2;
      if (Math.hypot(tx-gx, ty-gy) < 120) continue;
      world.trees.push({x: Math.floor(tx), y: Math.floor(ty)});
    }

    player.x = Math.min(Math.max(player.x, tileBase), W - tileBase);
    player.y = Math.min(Math.max(player.y, tileBase), H - tileBase);
  }

  /* ===== Input ===== */
  const keys = {};
  window.addEventListener('keydown', e => { keys[e.key.toLowerCase()] = true; });
  window.addEventListener('keyup', e => { keys[e.key.toLowerCase()] = false; });

  function bindHold(btn, key){
    const start=(ev)=>{ ev.preventDefault(); keys[key]=true; };
    const stop=()=>{ keys[key]=false; };
    btn.addEventListener('pointerdown', start);
    btn.addEventListener('pointerup', stop);
    btn.addEventListener('pointerleave', stop);
    btn.addEventListener('pointercancel', stop);
  }
  bindHold(btnUp,'arrowup'); bindHold(btnDown,'arrowdown');
  bindHold(btnLeft,'arrowleft'); bindHold(btnRight,'arrowright');
  btnInteract.addEventListener('click', ()=> interact());
  btnEnter.addEventListener('click', ()=> enterInteriorAtDoor());

  canvas.addEventListener('click', (e)=>{
    const r = canvas.getBoundingClientRect();
    const x = (e.clientX - r.left) * (canvas.width / r.width);
    const y = (e.clientY - r.top) * (canvas.height / r.height);
    tryTap(x, y);
  });

  /* ===== HUD & Theme ===== */
  function successWeatherMod(){ return weatherTypes[state.weatherIndex].mod || 0; }
  function successChance(){ return Math.min(MAX_SUCCESS, BASE_SUCCESS + state.boosts + successWeatherMod()); }
  function updateHUD(){
    moneyPill.textContent = `Money: $${Math.floor(state.money)}`;
    succPill.textContent = `Success: ${Math.round(successChance()*100)}%`;
    statPill.textContent = `Fail:${state.stats.failures} • Harv:${state.stats.harvests} • Upg:${state.stats.upgrades}`;
    dayPill.textContent = `Day ${state.day} • ${seasons[state.seasonIndex]} • ${weatherTypes[state.weatherIndex].icon} ${weatherTypes[state.weatherIndex].id}`;
    timePill.textContent = hhmm(state.timeMin);
    stamPill.textContent = `Stamina: ${Math.round(state.stamina)}%`;
    shopMoneyPill.textContent = `Money: $${Math.floor(state.money)}`;
    questMoneyPill.textContent = `Money: $${Math.floor(state.money)}`;
  }
  function applyTheme(){ document.body.classList.toggle('light', state.theme === 'light'); }
  function setPixelScale(scale){
    pixelScale = scale; tileSize = tileBase * pixelScale;
    player.speed = PLAYER_SPEED * pixelScale; positionWorld();
  }

  /* ===== Autosave ===== */
  let autosaveTimer = null;
  function startAutosave(){
    if (autosaveTimer) clearInterval(autosaveTimer);
    if (!state.autosave) return;
    autosaveTimer = setInterval(()=>{ doSave(); }, AUTOSAVE_INTERVAL_MS);
  }

  /* ===== Drawer ===== */
  function openDrawer(populateFn){
    invDrawer.style.display='block';
    invGrid.innerHTML = '';
    populateFn();
  }
  btnInv.addEventListener('click', ()=>{
    invDrawer.style.display = invDrawer.style.display === 'none' ? 'block' : 'none';
    renderInventory();
  });

  /* ===== Shops: Global open/close ===== */
  function openSeedShop(){ openShop('seed', 'Inside: Seed Shop'); }
  function openWorkshop(){ openShop('industrial', 'Inside: Workshop'); }
  function openSecretShop(){ openShop('secret', 'Inside: Back Room'); }
  function openShop(type, title){
    state.inInterior = true; state.interiorType = type;
    interiorTitle.textContent = title;
    setInteriorTheme(type);
    shopInterior.style.display = 'grid';
    renderInteriorShelf();
  }
  function openQuestHall(){ questsOverlay.style.display = 'grid'; renderQuests(); }
  function closeAllOverlays(){
    shopInterior.style.display = 'none';
    questsOverlay.style.display = 'none';
    craftOverlay.style.display = 'none';
    cookOverlay.style.display = 'none';
    fishOverlay.style.display = 'none';
    animalsOverlay.style.display = 'none';
    storageOverlay.style.display = 'none';
    achOverlay.style.display = 'none';
    state.inInterior = false; state.interiorType = null;
  }
  openSeedBtn.addEventListener('click', openSeedShop);
  openWorkBtn.addEventListener('click', openWorkshop);
  openSecretBtn.addEventListener('click', openSecretShop);
  openQuestHallBtn.addEventListener('click', openQuestHall);
  closeAllBtn.addEventListener('click', closeAllOverlays);

  /* ===== Inventory & Panels ===== */
  function renderInventory(){
    invGrid.innerHTML = '';
    const items = Object.keys(state.inv).filter(k=>state.inv[k]>0);
    if (items.length===0){
      const d = document.createElement('div'); d.className='card';
      d.innerHTML = '<div class="cardTitle">Empty</div><div class="small">Buy seeds at the Seed Shop.</div>';
      invGrid.appendChild(d);
      return;
    }
    items.forEach(k=>{
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `<div class="cardTitle">${k}</div><div class="pill">x${state.inv[k]}</div>`;
      card.addEventListener('click', ()=>{ state.selectedSeed = k; renderInventory(); });
      invGrid.appendChild(card);
    });
  }

  btnCraft.addEventListener('click', ()=> openCraft());
  btnCook.addEventListener('click', ()=> openCook());
  btnFish.addEventListener('click', ()=> openFish());
  btnAnimals.addEventListener('click', ()=> openAnimals());
  btnStorage.addEventListener('click', ()=> openStorage());
  btnQuests.addEventListener('click', ()=> openQuestHall());
  btnAch.addEventListener('click', ()=> openAch());

  /* ===== Shop interior ===== */
  function setInteriorTheme(type){
    interiorWrap.className = 'interiorWrap ' + (type==='seed'?'seed':type==='industrial'?'workshop':'secret');
    bundleBtn.style.display = (type==='seed') ? 'inline-block' : 'none';
    interiorTip.textContent = (type==='seed') ? 'Bundles of 5 get 5% off.' :
      (type==='industrial') ? 'Upgrades boost success up to 95%.' :
      'Rare boosts (requires $1000+ to unlock items).';
  }
  function renderInteriorShelf(){
    interiorShelf.innerHTML = '';
    let items = [];
    if (state.interiorType === 'seed') items = seeds;
    else if (state.interiorType === 'industrial') items = industry;
    else items = secrets;

    items.forEach(it=>{
      const card = document.createElement('div'); card.className = 'card';
      const priceLabel = (state.interiorType==='secret' && state.money<1000) ? 'Locked' : ('$'+it.price);
      card.innerHTML = `
        <div class="cardTitle">${it.id}</div>
        <div class="small">${state.interiorType==='seed' ? `Grow ${it.grow}s • Sell $${it.sell}` : it.desc}</div>
        <div class="pill">${(state.interiorType==='industrial' && state.purchasedUpgrades.has(it.id)) ? 'Owned' :
                             (state.interiorType==='secret' && state.purchasedSecrets.has(it.id)) ? 'Owned' : priceLabel}</div>`;
      card.addEventListener('click', ()=>{
        if (state.interiorType==='seed'){
          if (state.money<it.price) return toast('Not enough.');
          state.money -= it.price;
          state.inv[it.id] = (state.inv[it.id]||0) + 1;
          state.stats.distinctSeeds.add(it.id);
          toast(`Bought ${it.id}.`);
        } else if (state.interiorType==='industrial'){
          if (state.purchasedUpgrades.has(it.id)) return;
          if (state.money<it.price) return toast('Not enough.');
          state.money -= it.price; state.purchasedUpgrades.add(it.id);
          state.boosts = Math.min(MAX_SUCCESS-BASE_SUCCESS, state.boosts + it.boost);
          state.stats.upgrades++; toast(`Upgrade: ${it.id}.`);
        } else {
          const unlocked = state.money>=1000;
          if (!unlocked || state.purchasedSecrets.has(it.id)) return;
          if (state.money<it.price) return toast('Not enough.');
          state.money -= it.price; state.purchasedSecrets.add(it.id);
          state.boosts = Math.min(MAX_SUCCESS-BASE_SUCCESS, state.boosts + it.boost);
          state.stats.upgrades++; toast(`Secret: ${it.id}.`);
        }
        updateHUD(); renderInteriorShelf();
      });
      interiorShelf.appendChild(card);
    });
  }
  leaveShopBtn.addEventListener('click', ()=>{ shopInterior.style.display='none'; state.inInterior=false; state.interiorType=null; });
  bundleBtn.addEventListener('click', ()=>{
    if (state.interiorType!=='seed') return toast('Bundles only in Seed Shop.');
    let picks=[]; for(let i=0;i<5;i++){ picks.push(seeds[Math.floor(Math.random()*seeds.length)]); }
    const total=picks.reduce((s,x)=>s+x.price,0), discounted=Math.round(total*0.95);
    if (state.money < discounted) return toast('Not enough money.');
    state.money -= discounted;
    picks.forEach(s=>{ state.inv[s.id]=(state.inv[s.id]||0)+1; state.stats.distinctSeeds.add(s.id); });
    updateHUD(); toast('Bundle purchased.');
  });
  sellAllBtn.addEventListener('click', ()=>{
    let sold=0, earned=0;
    state.plots.forEach((p,i)=>{
      if (p.seedId && p.ready && !p.dead){
        state.money += p.sell; earned += p.sell; state.stats.harvests++;
        state.plots[i] = {seedId:null, plantedAt:null, grow:0, ready:false, dead:false, color:null, sell:0, fail:false};
        sold++;
      }
    });
    state.stats.soldMoney += earned;
    updateHUD(); toast(sold?`Sold ${sold} (${earned}).`:'No ready crops.');
    checkQuests();
  });

  /* ===== Quest hall ===== */
  function openQuests(){ questsOverlay.style.display='grid'; renderQuests(); }
  function closeQuests(){ questsOverlay.style.display='none'; }
  closeQuestsBtn.addEventListener('click', closeQuests);
  refreshQuestsBtn.addEventListener('click', ()=>{ rollQuests(); renderQuests(); });

  function rollQuests(){
    const pool = [...QUESTS_POOL];
    const picks = [];
    while (picks.length < 3 && pool.length){
      const i = Math.floor(Math.random()*pool.length);
      picks.push(pool.splice(i,1)[0]);
    }
    state.quests = picks;
    state.quests.forEach(q=>{
      state.questProgress[q.id] = state.questProgress[q.id] || 0;
    });
  }
  function renderQuests(){
    questList.innerHTML = '';
    state.quests.forEach(q=>{
      const prog = state.questProgress[q.id] || 0;
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `<div class="cardTitle">${q.label}</div><div class="small">Progress: ${prog}/${q.goal}</div>`;
      const btn = document.createElement('button'); btn.className='btn';
      btn.textContent = state.questsCompleted.has(q.id) ? 'Claimed' : (prog>=q.goal ? 'Claim' : 'Locked');
      btn.disabled = !(prog>=q.goal) || state.questsCompleted.has(q.id);
      btn.addEventListener('click', ()=>{
        if (btn.disabled) return;
        state.questsCompleted.add(q.id);
        state.money += q.reward.money;
        state.inv[q.reward.seed] = (state.inv[q.reward.seed]||0) + 1;
        state.stats.distinctSeeds.add(q.reward.seed);
        updateHUD(); renderQuests(); toast(`Quest reward: $${q.reward.money} + ${q.reward.seed}`);
      });
      card.appendChild(btn);
      questList.appendChild(card);
    });
    questMoneyPill.textContent = `Money: $${Math.floor(state.money)}`;
  }
  function checkQuests(){
    state.quests.forEach(q=>{
      if (q.trackMoney){ state.questProgress[q.id] = Math.min(q.goal, state.stats.soldMoney); }
      else if (q.useDistinct){ state.questProgress[q.id] = Math.min(q.goal, state.stats.distinctSeeds.size); }
      else if (q.noFail){
        const safeCount = Math.max(0, state.stats.harvests - state.stats.failures);
        state.questProgress[q.id] = Math.min(q.goal, safeCount);
      } else if (q.id==='plant15'){ /* tracked on plant */ }
      else if (q.id==='harvest10'){ state.questProgress[q.id] = Math.min(q.goal, state.stats.harvests); }
      else if (q.trackFish){ state.questProgress[q.id] = Math.min(q.goal, state.stats.fishCaught); }
    });
  }

  /* ===== Crafting ===== */
  function openCraft(){ craftOverlay.style.display='grid'; renderCraft(); }
  function closeCraft(){ craftOverlay.style.display='none'; }
  closeCraftBtn.addEventListener('click', closeCraft);
  function renderCraft(){
    craftGrid.innerHTML='';
    crafts.forEach(c=>{
      const can = canAfford(c.req);
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `<div class="cardTitle">${c.id}</div><div class="small">${reqText(c.req)} • ${c.desc}</div>`;
      const btn = document.createElement('button'); btn.className='btn'; btn.textContent = can?'Craft':'Need items';
      btn.disabled = !can;
      btn.addEventListener('click', ()=>{
        if (!can) return;
        payReq(c.req);
        if (c.effect.boost){ state.boosts = Math.min(MAX_SUCCESS-BASE_SUCCESS, state.boosts + c.effect.boost); }
        toast(`Crafted ${c.id}.`);
        renderCraft(); updateHUD();
      });
      card.appendChild(btn); craftGrid.appendChild(card);
    });
  }

  /* ===== Cooking ===== */
  function openCook(){ cookOverlay.style.display='grid'; renderCook(); }
  function closeCook(){ cookOverlay.style.display='none'; }
  closeCookBtn.addEventListener('click', closeCook);
  function renderCook(){
    cookGrid.innerHTML='';
    recipes.forEach(r=>{
      const can = canAfford(r.req);
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `<div class="cardTitle">${r.id}</div><div class="small">${reqText(r.req)} • +${r.stamina}% stamina</div>`;
      const btn = document.createElement('button'); btn.className='btn'; btn.textContent = can?'Cook':'Need items';
      btn.disabled = !can;
      btn.addEventListener('click', ()=>{
        if (!can) return;
        payReq(r.req);
        state.stamina = clamp(state.stamina + r.stamina, 0, 100);
        toast(`Cooked ${r.id}.`);
        renderCook(); updateHUD();
      });
      card.appendChild(btn); cookGrid.appendChild(card);
    });
  }

  /* ===== Fishing ===== */
  function openFish(){ fishOverlay.style.display='grid'; renderFishInv(); }
  function closeFish(){ fishOverlay.style.display='none'; }
  closeFishBtn.addEventListener('click', closeFish);

  let fishCasting = false, bobberGlow = 0, fishTimer = null;
  const fishArea = document.getElementById('fishArea');
  document.getElementById('fishCastBtn').addEventListener('click', ()=>{
    if (fishCasting) return;
    fishCasting = true; bobberGlow = 0; fishCastBtn.textContent = 'Wait...';
    if (fishTimer) clearInterval(fishTimer);
    fishTimer = setInterval(()=>{
      bobberGlow = Math.random();
      fishArea.style.boxShadow = bobberGlow>0.85 ? '0 0 28px #52ffd180' : '0 0 18px rgba(0,0,0,0.35)';
    }, 700);
    setTimeout(()=>{ fishCastBtn.textContent = 'Tap to hook'; }, 1200);
  });
  fishArea.addEventListener('click', ()=>{
    if (!fishCasting) return;
    clearInterval(fishTimer); fishCasting = false; fishCastBtn.textContent = 'Cast';
    const success = bobberGlow > 0.75;
    fishArea.style.boxShadow = '0 0 18px rgba(0,0,0,0.35)';
    if (!success) return toast('The fish slipped away.');
    const catchItem = rollFish();
    const name = catchItem.id;
    state.inv[name] = (state.inv[name]||0) + 1;
    state.stats.fishCaught++;
    toast(`Caught ${name}!`);
    renderFishInv(); updateHUD(); checkQuests();
  });
  function renderFishInv(){
    fishGrid.innerHTML='';
    ['Minnow','Trout','Salmon','Golden Koi'].forEach(f=>{
      const card = document.createElement('div'); card.className='card';
      const count = state.inv[f]||0;
      const price = fishables.find(x=>x.id===f)?.price||0;
      card.innerHTML = `<div class="cardTitle">${f}</div><div class="small">x${count} • $${price}</div>`;
      const btn = document.createElement('button'); btn.className='btn'; btn.textContent = count?'Sell one':'Empty';
      btn.disabled = !count;
      btn.addEventListener('click', ()=>{
        if (!count) return;
        state.inv[f]--; state.money += price; updateHUD(); renderFishInv();
      });
      card.appendChild(btn); fishGrid.appendChild(card);
    });
  }
  function rollFish(){
    const r = Math.random(); let acc=0;
    for (const f of fishables){ acc += f.chance; if (r <= acc) return f; }
    return fishables[0];
  }

  /* ===== Animals ===== */
  function openAnimals(){ animalsOverlay.style.display='grid'; renderAnimals(); }
  function closeAnimals(){ animalsOverlay.style.display='none'; }
  closeAnimalsBtn.addEventListener('click', closeAnimals);
  function renderAnimals(){
    animalsGrid.innerHTML='';
    animalsData.forEach(a=>{
      const owned = state.animals.find(x=>x.id===a.id);
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `<div class="cardTitle">${a.id}</div><div class="small">${a.desc} • Feed ${a.feed}</div><div class="pill">${owned?'Owned':('$'+a.price)}</div>`;
      const buyBtn = document.createElement('button'); buyBtn.className='btn'; buyBtn.textContent = owned?'Feed':'Buy';
      buyBtn.addEventListener('click', ()=>{
        if (!owned){
          if (state.money < a.price) return toast('Not enough money.');
          state.money -= a.price; state.animals.push({id:a.id, lastFedDay:0}); updateHUD(); renderAnimals();
        } else {
          if ((state.inv[a.feed]||0) <= 0) return toast(`Need ${a.feed}.`);
          state.inv[a.feed]--; const pet = state.animals.find(x=>x.id===a.id);
          pet.lastFedDay = state.day; toast(`${a.id} fed.`);
        }
      });
      card.appendChild(buyBtn); animalsGrid.appendChild(card);
    });
  }
  function hasAnimalBonus(kind){
    let bonus = 0;
    for (const a of state.animals){
      const data = animalsData.find(x=>x.id===a.id);
      const fedToday = a.lastFedDay === state.day;
      if (!fedToday) continue;
      if (kind==='money' && data.bonus.money) bonus += data.bonus.money;
      if (kind==='boost' && data.bonus.boost) bonus += data.bonus.boost;
      if (kind==='sellBonus' && data.bonus.sellBonus) bonus += data.bonus.sellBonus;
    }
    return bonus;
  }

  /* ===== Storage ===== */
  function openStorage(){ storageOverlay.style.display='grid'; renderStorage(); }
  function closeStorage(){ storageOverlay.style.display='none'; }
  closeStorageBtn.addEventListener('click', closeStorage);
  depositAllBtn.addEventListener('click', ()=>{
    for (const k of Object.keys(state.inv)){
      if (state.inv[k]>0){
        state.storage[k] = (state.storage[k]||0) + state.inv[k];
        state.inv[k] = 0;
      }
    }
    renderInventory(); renderStorage();
  });
  withdrawAllBtn.addEventListener('click', ()=>{
    for (const k of Object.keys(state.storage)){
      if (state.storage[k]>0){
        state.inv[k] = (state.inv[k]||0) + state.storage[k];
        state.storage[k] = 0;
      }
    }
    renderInventory(); renderStorage();
  });
  function renderStorage(){
    storageGrid.innerHTML='';
    const keys = Object.keys(state.storage);
    if (keys.length===0){
      const card = document.createElement('div'); card.className='card'; card.innerHTML='<div class="cardTitle">Empty</div>';
      storageGrid.appendChild(card); return;
    }
    keys.forEach(k=>{
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `<div class="cardTitle">${k}</div><div class="small">x${state.storage[k]||0}</div>`;
      storageGrid.appendChild(card);
    });
  }

  /* ===== Achievements ===== */
  function openAch(){ achOverlay.style.display='grid'; renderAch(); }
  function closeAch(){ achOverlay.style.display='none'; }
  closeAchBtn.addEventListener('click', closeAch);
  function renderAch(){
    achGrid.innerHTML='';
    achievements.forEach(a=>{
      const done = a.cond(state);
      const card = document.createElement('div'); card.className='card';
      card.innerHTML = `<div class="cardTitle">${a.label}</div><div class="small">${done?'Completed':'Not yet'} • ${a.reward}</div>`;
      achGrid.appendChild(card);
    });
  }

  /* ===== Economy helpers ===== */
  function canAfford(req){ for (const k of Object.keys(req)){ if ((state.inv[k]||0) < req[k]) return false; } return true; }
  function reqText(req){ return Object.keys(req).map(k=>`${k} x${req[k]}`).join(', '); }
  function payReq(req){ for (const k of Object.keys(req)){ state.inv[k] -= req[k]; } }

  /* ===== Farming ===== */
  function nearestPlotIndex(px, py){
    const g = world.garden; let best=-1, bestD=Infinity;
    for (let r=0;r<g.rows;r++){
      for (let c=0;c<g.cols;c++){
        const idx=r*GRID_W+c;
        const cx=g.x+c*tileSize+tileSize/2; const cy=g.y+r*tileSize+tileSize/2;
        const d=dist(px,py,cx,cy); if (d<bestD){ bestD=d; best=idx; }
      }
    } return best;
  }
  function distToPlotCenter(px, py, idx){
    if (idx===-1) return Infinity;
    const g=world.garden; const r=Math.floor(idx/GRID_W), c=idx%GRID_W;
    const cx=g.x+c*tileSize+tileSize/2; const cy=g.y+r*tileSize+tileSize/2;
    return dist(px,py,cx,cy);
  }
  function pickAffordableSeed(){
    if (state.selectedSeed && (state.inv[state.selectedSeed]||0) > 0){
      const s = seeds.find(x=>x.id===state.selectedSeed);
      return s || null;
    }
    for (const k of Object.keys(state.inv)){ if (state.inv[k] > 0) return seeds.find(x=>x.id===k); }
    return null;
  }
  function plantAtPlot(idx){
    if (state.stamina <= 0) return toast('Too tired.');
    const p=state.plots[idx];
    if (p.seedId) return toast('Already planted.');
    const s = pickAffordableSeed();
    if (!s) return toast('Buy seeds inside.');
    state.inv[s.id] = (state.inv[s.id]||0) - 1;
    const willFail = Math.random() > successChance();
    state.plots[idx] = { seedId:s.id, plantedAt: nowSec(), grow:s.grow, ready:false, dead:false, color:s.color, sell:s.sell, fail:willFail };
    state.questProgress['plant15'] = Math.min(15, (state.questProgress['plant15']||0) + 1);
    state.stamina = clamp(state.stamina - 3, 0, 100);
    toast(`Planted ${s.id}.`); updateHUD(); checkQuests();
  }
  function harvestPlot(idx){
    const p=state.plots[idx];
    if (!p.seedId || p.dead || !p.ready) return;
    let sellVal = p.sell;
    sellVal = Math.round(sellVal * (1 + (hasAnimalBonus('sellBonus') || 0)));
    state.money += sellVal; state.stats.harvests++;
    state.stats.soldMoney += sellVal;
    state.plots[idx] = {seedId:null, plantedAt:null, grow:0, ready:false, dead:false, color:null, sell:0, fail:false};
    state.stamina = clamp(state.stamina - 2, 0, 100);
    updateHUD(); checkQuests();
  }
  function clearPlot(idx){
    const p=state.plots[idx];
    if (p.seedId && p.dead){
      state.plots[idx] = {seedId:null, plantedAt:null, grow:0, ready:false, dead:false, color:null, sell:0, fail:false};
      toast('Cleared.');
    }
  }
  document.getElementById('sellReadyBtn').addEventListener('click', ()=>{
    let sold=0, earned=0;
    state.plots.forEach((p,i)=>{
      if (p.seedId && p.ready && !p.dead){
        let sellVal = Math.round(p.sell * (1 + (hasAnimalBonus('sellBonus') || 0)));
        state.money += sellVal; earned += sellVal; state.stats.harvests++;
        state.plots[i] = {seedId:null, plantedAt:null, grow:0, ready:false, dead:false, color:null, sell:0, fail:false};
        sold++;
      }
    });
    state.stats.soldMoney += earned;
    updateHUD(); toast(sold?`Sold ${sold} (${earned}).`:'No ready crops.');
    checkQuests();
  });
  document.getElementById('clearDeadBtn').addEventListener('click', ()=>{
    let cleared=0;
    state.plots.forEach((p,i)=>{
      if (p.seedId && p.dead){
        state.plots[i] = {seedId:null, plantedAt:null, grow:0, ready:false, dead:false, color:null, sell:0, fail:false};
        cleared++;
      }
    });
    toast(cleared?`Cleared ${cleared}.`:'No dead.');
  });

  setInterval(()=>{
    let changed=false;
    state.plots.forEach(p=>{
      if (p.seedId && !p.ready && !p.dead){
        if (p.plantedAt + p.grow <= nowSec()){
          if (p.fail){ p.dead=true; state.stats.failures++; } else { p.ready=true; }
          changed=true;
        }
      }
    });
    if (changed) updateHUD();
  }, 500);

  /* ===== Interact ===== */
  function nearbyDoor(){
    for (const d of world.doors){
      const cx = d.rect.x + d.rect.w/2, cy = d.rect.y + d.rect.h/2;
      if (dist(player.x, player.y, cx, cy) <= INTERACT_RADIUS * pixelScale + 4) return d;
    } return null;
  }
  function rectHit(px, py, rect){ return px > rect.x && px < rect.x+rect.w && py > rect.y && py < rect.y+rect.h; }
  function interact(){
    if (state.inInterior) return;
    const idx = nearestPlotIndex(player.x, player.y);
    if (idx!==-1 && distToPlotCenter(player.x, player.y, idx) <= INTERACT_RADIUS * pixelScale){
      const p=state.plots[idx];
      if (!p.seedId) plantAtPlot(idx);
      else if (p.dead) clearPlot(idx);
      else if (p.ready) harvestPlot(idx);
      return;
    }
    if (rectHit(player.x, player.y, world.chest)) return openStorage();
    if (rectHit(player.x, player.y, world.pond)) return openFish();
    const d = nearbyDoor();
    if (d){
      if (d.id==='seed') return openSeedShop();
      if (d.id==='industrial') return openWorkshop();
      if (d.id==='secret') return openSecretShop();
      if (d.id==='questhall') return openQuestHall();
      if (d.id==='barn') return openAnimals();
      if (d.id==='farmhouse') return openCraft(); // repurpose farmhouse as crafting
    }
  }
  function tryTap(x, y){
    if (state.inInterior) return;
    const idx = nearestPlotIndex(x, y);
    if (idx!==-1 && distToPlotCenter(x, y, idx) < 24){
      const p=state.plots[idx];
      if (!p.seedId) plantAtPlot(idx);
      else if (p.dead) clearPlot(idx);
      else if (p.ready) harvestPlot(idx);
    }
  }

  /* ===== Day/Night, Weather, Seasons ===== */
  function advanceTime(dtMin){
    state.timeMin += dtMin;
    if (state.timeMin >= 24*60){
      state.timeMin = state.timeMin - 24*60;
      state.day++; if (state.day % 7 === 1) rollWeather(); // weekly change
      if (state.day % 20 === 1) advanceSeason();
      feedDayBonuses();
    }
    updateHUD();
  }
  function rollWeather(){
    const r = Math.random();
    if (r<0.55) state.weatherIndex = 0;
    else if (r<0.75) state.weatherIndex = 1;
    else if (r<0.87) state.weatherIndex = 2;
    else if (r<0.96) state.weatherIndex = 4;
    else state.weatherIndex = 3;
  }
  function advanceSeason(){ state.seasonIndex = (state.seasonIndex+1)%seasons.length; }
  function feedDayBonuses(){
    const cashBonus = hasAnimalBonus('money') || 0;
    const boostBonus = hasAnimalBonus('boost') || 0;
    if (cashBonus) { state.money += cashBonus; toast(`Animal bonus: +$${cashBonus}`); }
    if (boostBonus) { state.boosts = Math.min(MAX_SUCCESS-BASE_SUCCESS, state.boosts + boostBonus); toast(`Animal morale: +${Math.round(boostBonus*100)}%`); }
  }
  document.getElementById('sleepBtn').addEventListener('click', ()=>{
    if (state.timeMin < 20*60) return toast('Too early to sleep.');
    state.timeMin = 6*60; state.day++; state.stamina = 100;
    rollWeather(); feedDayBonuses(); updateHUD(); toast(`Day ${state.day}`); checkQuests();
  });

  /* ===== Drawing ===== */
  function shadeHex(hex, delta){
    const c = { r:parseInt(hex.slice(1,3),16), g:parseInt(hex.slice(3,5),16), b:parseInt(hex.slice(5,7),16) };
    const r = clamp(c.r + delta, 0, 255), g = clamp(c.g + delta, 0, 255), b = clamp(c.b + delta, 0, 255);
    return `rgb(${r},${g},${b})`;
  }
  function draw(){
    const W = canvasLogical.w, H = canvasLogical.h;

    // Ground
    ctx.fillStyle = (state.theme==='dark') ? '#5e9a68' : '#a8d6af';
    ctx.fillRect(0,0,W,H);

    // Trees
    const trunk = (state.theme==='dark') ? '#5b3e26' : '#a67649';
    const leavesBase = (state.theme==='dark') ? '#3f6a44' : '#77a57f';
    world.trees.forEach(t=>{
      ctx.fillStyle = trunk; ctx.fillRect(t.x, t.y, 6, 10);
      ctx.fillStyle = leavesBase; ctx.fillRect(t.x-6, t.y-10, 18, 12);
      ctx.fillStyle = shadeHex(leavesBase, 20); ctx.fillRect(t.x-4, t.y-16, 14, 8);
    });

    // Buildings & special areas
    [world.seedShop, world.indShop, world.secShop, world.farmhouse, world.questHall, world.barn].forEach(rect=>{
      const color = rect.color;
      ctx.fillStyle = color; ctx.fillRect(rect.x, rect.y, rect.w, rect.h);
      ctx.fillStyle = shadeHex(color, -40); ctx.fillRect(rect.x, rect.y, rect.w, 8);
      for (let i=0;i<rect.w;i+=8){ ctx.fillStyle = shadeHex(color, -52); ctx.fillRect(rect.x+i, rect.y+8, 6, 2); }
      ctx.fillStyle = shadeHex(color, -70); ctx.fillRect(rect.x + rect.w/2 - 6, rect.y + rect.h - 16, 12, 16);
      ctx.fillStyle = '#2b3648'; ctx.fillRect(rect.x + 6, rect.y + 10, 12, 10);
      ctx.fillRect(rect.x + rect.w - 18, rect.y + 10, 12, 10);
      ctx.fillStyle = '#e9d7b7'; ctx.font = '8px monospace'; ctx.textAlign='center';
      ctx.fillText(rect.label, rect.x + rect.w/2, rect.y + rect.h/2 + 3);
    });

    // Pond
    ctx.fillStyle = '#236090'; ctx.fillRect(world.pond.x, world.pond.y, world.pond.w, world.pond.h);
    ctx.fillStyle = '#1a4a72'; ctx.fillRect(world.pond.x, world.pond.y, world.pond.w, 6);

    // Chest
    const ch = world.chest;
    ctx.fillStyle = ch.color; ctx.fillRect(ch.x, ch.y, ch.w, ch.h);
    ctx.fillStyle = shadeHex(ch.color, -40); ctx.fillRect(ch.x, ch.y, ch.w, 6);
    ctx.fillStyle = '#e9d7b7'; ctx.font='8px monospace'; ctx.textAlign='center';
    ctx.fillText('Chest', ch.x + ch.w/2, ch.y + ch.h/2 + 3);

    // Doors
    world.doors.forEach(d=>{
      ctx.fillStyle = 'rgba(255,255,255,0.15)'; ctx.fillRect(d.rect.x, d.rect.y, d.rect.w, d.rect.h);
      ctx.fillStyle = '#e9d7b7'; ctx.font = '8px monospace'; ctx.textAlign='center';
      ctx.fillText('Door', d.rect.x + d.rect.w/2, d.rect.y + 8);
    });

    // Garden
    drawGarden();

    // Player
    drawPlayer();

    // Enter button visibility
    const near = nearbyDoor();
    if (near && !state.inInterior) btnEnter.classList.remove('hidden');
    else btnEnter.classList.add('hidden');

    // Minimap
    drawMinimap();
  }
  function drawGarden(){
    const g=world.garden; const margin=4*pixelScale;
    const w=tileSize-margin*2, h=tileSize-margin*2;
    for (let r=0;r<g.rows;r++){
      for (let c=0;c<g.cols;c++){
        const idx=r*GRID_W+c; const x=g.x+c*tileSize+margin; const y=g.y+r*tileSize+margin;
        ctx.fillStyle = (state.theme==='dark') ? '#4a3a2a' : '#c59a6f';
        ctx.fillRect(x,y,w,h);
        ctx.strokeStyle = (state.theme==='dark') ? '#2b1d12' : '#8a6a44';
        ctx.strokeRect(x,y,w,h);
        const p=state.plots[idx];
        if (p.seedId){
          ctx.fillStyle = p.color || '#7ec37e';
          const sprW=Math.floor(w*0.45), sprH=Math.floor(h*0.45);
          const sprX=x+Math.floor((w-sprW)/2), sprY=y+Math.floor((h-sprH)/2);
          ctx.fillRect(sprX,sprY,sprW,sprH);
          if (p.dead){ ctx.globalAlpha=0.5; ctx.fillStyle='#1f0f0f'; ctx.fillRect(x,y,w,h); ctx.globalAlpha=1; }
          else if (p.ready){ ctx.strokeStyle=(state.theme==='dark')?'#79c07a':'#2f8d3a'; ctx.lineWidth=2; ctx.strokeRect(x+1,y+1,w-2,h-2); }
          ctx.fillStyle = (state.theme==='dark') ? '#e9d7b7' : '#3a2a18';
          ctx.font='8px monospace'; ctx.textAlign='left';
          const t = p.dead ? 'Dead' : (p.ready ? `${p.seedId} ready` : `${p.seedId} ${Math.max(0,Math.ceil(p.plantedAt+p.grow-nowSec()))}s`);
          ctx.fillText(t, x+2, y+h-2);
        }
      }
    }
  }
  function drawPlayer(){
    const s=pixelScale, size=tileBase, px=player.x-(size*s)/2, py=player.y-(size*s)/2;
    ctx.globalAlpha=0.25; ctx.fillStyle='#000'; ctx.fillRect(px+2*s, py+size*s-5*s, (size-4)*s, 3*s); ctx.globalAlpha=1;
    ctx.fillStyle=(state.theme==='dark') ? '#2b3648' : '#2a4a7a'; ctx.fillRect(px+2*s, py+2*s, (size-4)*s, (size-4)*s);
    ctx.fillStyle=(state.theme==='dark') ? '#f1d2a3' : '#e0b88a'; ctx.fillRect(px+8*s, py+8*s, 6*s, 6*s);
    ctx.fillStyle='#000'; ctx.fillRect(px+9*s, py+10*s, 1*s, 1*s); ctx.fillRect(px+12*s, py+10*s, 1*s, 1*s);
    player.walkTimer=(player.walkTimer+1)%20; const step=Math.floor(player.walkTimer/10);
    ctx.fillStyle=(state.theme==='dark') ? '#1f2838' : '#365a8a';
    const legX= step===0 ? 6*s : 10*s; ctx.fillRect(px+legX, py+size*s-6*s, 4*s, 4*s);
  }
  function drawMinimap(){
    const W = miniCanvas.width, H = miniCanvas.height;
    miniCtx.fillStyle = '#0a1424'; miniCtx.fillRect(0,0,W,H);
    const sx = W/canvasLogical.w, sy = H/canvasLogical.h;
    function drawRect(rect, col){ miniCtx.fillStyle = col; miniCtx.fillRect(rect.x*sx, rect.y*sy, rect.w*sx, rect.h*sy); }
    [world.seedShop, world.indShop, world.secShop, world.farmhouse, world.questHall, world.barn, world.pond].forEach(r=> drawRect(r, '#5a8baf'));
    drawRect(world.garden, '#7a5a3a');
    miniCtx.fillStyle='#e9d7b7'; miniCtx.fillRect(player.x*sx-2, player.y*sy-2, 4, 4);
  }

  /* ===== Update & Loop ===== */
  function update(){
    if (state.inInterior) return;
    player.vx=0; player.vy=0;
    if (keys['arrowleft'] || keys['a'])  player.vx = -player.speed;
    if (keys['arrowright'] || keys['d']) player.vx =  player.speed;
    if (keys['arrowup'] || keys['w'])    player.vy = -player.speed;
    if (keys['arrowdown'] || keys['s'])  player.vy =  player.speed;
    player.x = Math.max(tileBase, Math.min(canvasLogical.w - tileBase, player.x + player.vx));
    player.y = Math.max(tileBase, Math.min(canvasLogical.h - tileBase, player.y + player.vy));
    if (player.vx||player.vy) state.stamina = clamp(state.stamina - 0.02, 0, 100);
    advanceTime(0.05);
  }
  function loop(){
    if (state.ready){ update(); draw(); }
    requestAnimationFrame(loop);
  }

  /* ===== Save/Load/Reset & Options ===== */
  function saveBlob(){
    return {
      theme: state.theme, orientationMode, pixelScale,
      autosave: state.autosave,
      money: state.money, boosts: state.boosts,
      day: state.day, timeMin: state.timeMin, stamina: state.stamina,
      seasonIndex: state.seasonIndex, weatherIndex: state.weatherIndex,
      inv: state.inv, plots: state.plots,
      stats: {
        failures: state.stats.failures, harvests: state.stats.harvests, upgrades: state.stats.upgrades,
        distinctSeeds:[...state.stats.distinctSeeds], soldMoney: state.stats.soldMoney, fishCaught: state.stats.fishCaught
      },
      purchasedUpgrades:[...state.purchasedUpgrades], purchasedSecrets:[...state.purchasedSecrets],
      selectedSeed: state.selectedSeed,
      quests: state.quests, questProgress: state.questProgress, questsCompleted: [...state.questsCompleted],
      animals: state.animals, storage: state.storage, saveSlot: state.saveSlot
    };
  }
  function hydrate(data){
    state.theme = data.theme || 'dark';
    orientationMode = data.orientationMode || 'auto';
    pixelScale = data.pixelScale || 2; tileSize = tileBase * pixelScale;

    state.autosave = data.autosave ?? true;

    state.money = data.money ?? 100;
    state.boosts = data.boosts || 0;

    state.day = data.day || 1;
    state.timeMin = data.timeMin || 360;
    state.stamina = data.stamina || 100;

    state.seasonIndex = data.seasonIndex || 0;
    state.weatherIndex = data.weatherIndex || 0;

    state.inv = data.inv || {};
    state.plots = Array.isArray(data.plots) ? data.plots : state.plots;

    state.stats.failures = data.stats?.failures || 0;
    state.stats.harvests = data.stats?.harvests || 0;
    state.stats.upgrades = data.stats?.upgrades || 0;
    state.stats.distinctSeeds = new Set(data.stats?.distinctSeeds || []);
    state.stats.soldMoney = data.stats?.soldMoney || 0;
    state.stats.fishCaught = data.stats?.fishCaught || 0;

    state.purchasedUpgrades = new Set(data.purchasedUpgrades || []);
    state.purchasedSecrets = new Set(data.purchasedSecrets || []);
    state.selectedSeed = data.selectedSeed || null;

    state.quests = data.quests || [];
    state.questProgress = data.questProgress || {};
    state.questsCompleted = new Set(data.questsCompleted || []);

    state.animals = data.animals || [];
    state.storage = data.storage || {};
    state.saveSlot = data.saveSlot || 'slot1';

    applyTheme(); resizeCanvas(); updateHUD(); renderInventory(); startAutosave();
  }

  function doSave(){
    localStorage.setItem(`pixelValley_pro_${state.saveSlot}`, JSON.stringify(saveBlob()));
    toast('Saved.');
  }
  function doLoad(slot=state.saveSlot){
    const s = localStorage.getItem(`pixelValley_pro_${slot}`);
    if (!s) return toast('No save.');
    try { hydrate(JSON.parse(s)); toast('Loaded.'); } catch(e){ toast('Load failed.'); }
  }

  saveBtn.addEventListener('click', doSave);
  loadBtn.addEventListener('click', ()=> doLoad());
  titleLoadBtn.addEventListener('click', ()=> doLoad());

  optionsBtn.addEventListener('click', ()=>{
    titleOverlay.style.display='grid';
    titleOptions.style.display='block';
  });
  optBtn.addEventListener('click', ()=>{
    titleOptions.style.display = titleOptions.style.display === 'none' ? 'block' : 'none';
  });
  optLight.addEventListener('click', ()=>{ state.theme='light'; applyTheme(); });
  optDark.addEventListener('click', ()=>{ state.theme='dark'; applyTheme(); });
  orientPortrait.addEventListener('click', ()=>{ orientationMode='portrait'; resizeCanvas(); });
  orientLandscape.addEventListener('click', ()=>{ orientationMode='landscape'; resizeCanvas(); });
  orientAuto.addEventListener('click', ()=>{ orientationMode='auto'; resizeCanvas(); });
  pix1.addEventListener('click', ()=>{ setPixelScale(1); resizeCanvas(); });
  pix2.addEventListener('click', ()=>{ setPixelScale(2); resizeCanvas(); });
  pix3.addEventListener('click', ()=>{ setPixelScale(3); resizeCanvas(); });
  autoOn.addEventListener('click', ()=>{ state.autosave=true; startAutosave(); toast('Autosave on'); });
  autoOff.addEventListener('click', ()=>{ state.autosave=false; startAutosave(); toast('Autosave off'); });

  document.getElementById('resetBtn').addEventListener('click', ()=>{
    if (!confirm('Reset game?')) return;
    Object.assign(state, {
      money:100, boosts:0, day:1, timeMin:360, stamina:100, seasonIndex:0, weatherIndex:0,
      inv:{}, stats:{ failures:0, harvests:0, upgrades:0, distinctSeeds:new Set(), soldMoney:0, fishCaught:0 },
      plots: Array.from({length: GRID_W*GRID_H}, ()=>({seedId:null, plantedAt:null, grow:0, ready:false, dead:false, color:null, sell:0, fail:false})),
      purchasedUpgrades:new Set(), purchasedSecrets:new Set(),
      selectedSeed:null, inInterior:false, interiorType:null,
      quests:[], questProgress:{}, questsCompleted:new Set(),
      animals:[], storage:{}
    });
    rollQuests(); updateHUD(); renderInventory(); closeAllOverlays(); toast('Reset.');
  });

  startBtn.addEventListener('click', ()=>{
    titleOverlay.style.display='none';
    state.ready = true;
    veil.style.opacity = '1';
    setTimeout(()=>{ veil.style.opacity = '0'; }, 60);
    setTimeout(()=>{ veil.style.display = 'none'; }, 1200);
    startAutosave();
  });

  /* ===== Init ===== */
  function init(){
    rollQuests(); applyTheme(); resizeCanvas(); updateHUD(); loop(); positionWorld();
    rollWeather();
    player.x = world.garden.x + tileSize*2;
    player.y = world.garden.y + tileSize*2;
  }
  init();

})();
</script>
</body>
</html>
