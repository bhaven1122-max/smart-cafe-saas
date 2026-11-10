<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Smart Cafe QR & Multi-Cafe SaaS Platform</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@400;600;700;800&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #FAF6F0; color: #1C130E; }
    .no-scrollbar::-webkit-scrollbar { display: none; }
    .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    @keyframes pulse-border { 0%, 100% { border-color: #ef4444; } 50% { border-color: transparent; } }
    .animate-pulse-red { animation: pulse-border 1.5s infinite; }
  </style>
</head>
<body class="min-h-screen pb-24">

  <div id="app"></div>

  <script>
    const SUPER_ADMIN_PIN = "9999";

    const defaultMenuTemplate = [
      { id: 1, name: "Signature Hazelnut Cold Coffee", category: "Coffee", price: 149, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1517701604599-bb29b565090c?w=500&q=80" },
      { id: 2, name: "Classic Caramel Frappe", category: "Coffee", price: 159, is_veg: true, is_bestseller: false, in_stock: true, img: "https://images.unsplash.com/photo-1572490122747-3968b75cc699?w=500&q=80" },
      { id: 3, name: "Hot Cappuccino with Cocoa Dust", category: "Coffee", price: 119, is_veg: true, is_bestseller: false, in_stock: true, img: "https://images.unsplash.com/photo-1534778101976-62847782c213?w=500&q=80" },
      { id: 4, name: "Belgian Dark Chocolate Shake", category: "Coffee", price: 169, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1577805947697-89e18249d767?w=500&q=80" },
      { id: 5, name: "Kulhad Masala Adrak Chai", category: "Chai & Coolers", price: 49, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1561336313-0bd5e0b27ec8?w=500&q=80" },
      { id: 6, name: "Blue Lagoon Refreshing Mojito", category: "Chai & Coolers", price: 119, is_veg: true, is_bestseller: false, in_stock: true, img: "https://images.unsplash.com/photo-1513558161293-cdaf765ed2fd?w=500&q=80" },
      { id: 7, name: "Maharaja Double Cheese Burger", category: "Burgers & Sandwiches", price: 169, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=500&q=80" },
      { id: 8, name: "Tandoori Paneer Tikka Sandwich", category: "Burgers & Sandwiches", price: 159, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1528735602780-2552fd46c7af?w=500&q=80" },
      { id: 9, name: "Farmhouse Loaded Cheese Pizza [8 Inch]", category: "Pizza & Pasta", price: 249, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1513104890138-7c749659a591?w=500&q=80" },
      { id: 10, name: "Creamy Alfredo White Sauce Pasta", category: "Pizza & Pasta", price: 189, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1645112411341-6c4fd023714a?w=500&q=80" },
      { id: 11, name: "Kurkure Cheese Burst Paneer Momos", category: "Snacks & Momos", price: 149, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1625220194771-7ebdea0b70b9?w=500&q=80" },
      { id: 12, name: "Pahadi Tadka Double Masala Maggi", category: "Snacks & Momos", price: 99, is_veg: true, is_bestseller: false, in_stock: true, img: "https://images.unsplash.com/photo-1612929633738-8fe44f7ec841?w=500&q=80" },
      { id: 13, name: "Peri-Peri Loaded Cheesy Fries", category: "Snacks & Momos", price: 129, is_veg: true, is_bestseller: false, in_stock: true, img: "https://images.unsplash.com/photo-1576107232684-1279f3908594?w=500&q=80" },
      { id: 14, name: "Sizzling Hot Chocolate Walnut Brownie", category: "Desserts", price: 179, is_veg: true, is_bestseller: true, in_stock: true, img: "https://images.unsplash.com/photo-1606313564200-e75d5e30476c?w=500&q=80" }
    ];

    const initialSaaSData = {
      gemini_api_key: "",
      cafes: {
        "sukoon": {
          slug: "sukoon",
          is_active: true,
          cafe_name: "Sukoon The Cafe",
          tagline: "Sip, Bite & Find Your Sukoon",
          upi_id: "sukooncafe@upi",
          whatsapp_number: "919876543210",
          chef_pin: "1122",
          owner_pin: "8899",
          total_tables: 12,
          google_review_link: "https://g.page/r/sukoon-cafe/review",
          rush_hour_discount: false,
          menu: JSON.parse(JSON.stringify(defaultMenuTemplate)),
          orders: []
        }
      }
    };

    let saasDB = JSON.parse(localStorage.getItem("multi_cafe_saas_db")) || initialSaaSData;

    function saveSaaS() {
      localStorage.setItem("multi_cafe_saas_db", JSON.stringify(saasDB));
    }

    const urlParams = new URLSearchParams(window.location.search);
    let activeCafeSlug = (urlParams.get("cafe") || "sukoon").toLowerCase();
    if (!saasDB.cafes[activeCafeSlug]) {
      activeCafeSlug = Object.keys(saasDB.cafes)[0] || "sukoon";
    }
    let currentTable = urlParams.get("table") || "1";

    function currentCafe() {
      return saasDB.cafes[activeCafeSlug];
    }

    let cart = {};
    let selectedCategory = "All";
    let searchQuery = "";
    let bestsellerOnly = false;
    let activeOrderId = localStorage.getItem("active_order_" + activeCafeSlug) || null;
    let cartStep = 0;
    let selectedNotes = [];
    let customNote = "";
    let customerName = "";
    let paymentMode = "COUNTER";
    let utrNumber = "";
    let soundEnabled = false;
    let pinUnlocked = false;
    let ownerTab = "sales";
    let showScannerModal = false;
    let isScanningMenu = false;
    let bulkMenuText = "";

    function getAutoFoodImage(nameOrCat) {
      const q = (nameOrCat || "").toLowerCase();
      if (q.includes("coffee") || q.includes("latte") || q.includes("frappe") || q.includes("shake")) return "https://images.unsplash.com/photo-1517701604599-bb29b565090c?w=500&q=80";
      if (q.includes("chai") || q.includes("tea") || q.includes("mojito") || q.includes("cooler")) return "https://images.unsplash.com/photo-1561336313-0bd5e0b27ec8?w=500&q=80";
      if (q.includes("burger") || q.includes("sandwich")) return "https://images.unsplash.com/photo-1568901346375-23c9450c58cd?w=500&q=80";
      if (q.includes("pizza") || q.includes("pasta")) return "https://images.unsplash.com/photo-1513104890138-7c749659a591?w=500&q=80";
      if (q.includes("momo") || q.includes("maggi") || q.includes("fries") || q.includes("chilli") || q.includes("roll")) return "https://images.unsplash.com/photo-1625220194771-7ebdea0b70b9?w=500&q=80";
      if (q.includes("brownie") || q.includes("waffle") || q.includes("cake") || q.includes("ice cream")) return "https://images.unsplash.com/photo-1606313564200-e75d5e30476c?w=500&q=80";
      return "https://images.unsplash.com/photo-1504674900247-0877df9cc836?w=500&q=80";
    }

    window.addEventListener("storage", () => {
      const updated = JSON.parse(localStorage.getItem("multi_cafe_saas_db"));
      if (updated) {
        const oldLen = currentCafe().orders.length;
        saasDB = updated;
        const newLen = currentCafe().orders.length;
        if (newLen > oldLen && soundEnabled) playKitchenBell();
        render();
      }
    });

    function playKitchenBell() {
      try {
        const ctx = new (window.AudioContext || window.webkitAudioContext)();
        const playTone = (freq, start, duration) => {
          const osc = ctx.createOscillator();
          const gain = ctx.createGain();
          osc.type = "sine";
          osc.frequency.setValueAtTime(freq, ctx.currentTime + start);
          gain.gain.setValueAtTime(0.4, ctx.currentTime + start);
          gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + start + duration);
          osc.connect(gain);
          gain.connect(ctx.destination);
          osc.start(ctx.currentTime + start);
          osc.stop(ctx.currentTime + start + duration);
        };
        playTone(880, 0, 0.35);
        playTone(660, 0.3, 0.6);
      } catch (e) {}
    }

    function getDiscountedPrice(price) {
      return currentCafe().rush_hour_discount ? Math.round(price * 0.9) : price;
    }

    function getRoute() {
      const mode = urlParams.get("mode");
      const hash = window.location.hash;
      if (mode === "superadmin" || hash === "#superadmin") return "SUPERADMIN";
      if (mode === "kitchen" || hash === "#kitchen-kds") return "KDS";
      if (mode === "owner" || hash === "#owner-boss") return "OWNER";
      return "CUSTOMER";
    }

    function renderScannerModal() {
      if (!showScannerModal) return "";
      return `
        <div class="fixed inset-0 z-50 bg-black/75 flex items-center justify-center p-4">
          <div class="bg-white text-stone-900 w-full max-w-lg rounded-3xl p-5 max-h-[90vh] overflow-y-auto shadow-2xl">
            <div class="flex justify-between items-center pb-3 border-b">
              <div>
                <h3 class="text-lg font-extrabold text-amber-700">📸 AI Menu Photo Scanner & Importer</h3>
                <p class="text-xs text-stone-500">Adding items to: <b>${currentCafe().cafe_name}</b></p>
              </div>
              <button onclick="showScannerModal=false; render()" class="text-stone-400 font-bold text-xl">✕</button>
            </div>

            <div class="my-4 p-4 bg-amber-50/70 border-2 border-amber-400 border-dashed rounded-2xl text-center">
              <p class="text-xs font-bold text-amber-900 mb-2"> तरीका 1: पुराने मेन्यू कार्ड की फोटो खींचें या अपलोड करें (AI Auto-Read)</p>
              <input type="password" placeholder="Optional: Paste Free Gemini API Key for Direct Photo Scan" value="${saasDB.gemini_api_key || ''}" onchange="saasDB.gemini_api_key=this.value.trim(); saveSaaS()" class="w-full border rounded-xl px-3 py-1.5 text-xs mb-2.5 bg-white" />
              <label class="cursor-pointer inline-block bg-[#1C130E] text-amber-400 font-extrabold text-xs px-5 py-3 rounded-xl shadow hover:opacity-95">
                ${isScanningMenu ? '⏳ AI is Reading Menu Photo...' : '📷 Click / Upload Menu Card Photo'}
                <input type="file" accept="image/*" capture="environment" class="hidden" onchange="handleMenuPhotoUpload(event)" ${isScanningMenu ? 'disabled' : ''} />
              </label>
              <p class="text-[11px] text-stone-500 mt-1.5">फोटो चुनते ही AI सारे नाम और रेट पढ़कर नीचे बॉक्स में भर देगा।</p>
            </div>

            <div class="my-3">
              <div class="flex justify-between items-center mb-1">
                <label class="text-xs font-extrabold text-stone-700">तरीका 2: Bulk Menu List (Category | Dish Name | Price)</label>
                <button onclick="fillSampleBulkText()" class="text-[11px] text-amber-700 font-bold underline">Load Example</button>
              </div>
              <textarea id="bulkMenuInput" rows="6" oninput="bulkMenuText=this.value" placeholder="Coffee | Cold Coffee | 120&#10;Snacks | Paneer Momos | 140" class="w-full border-2 border-stone-200 rounded-xl p-3 text-xs font-mono focus:outline-none focus:border-amber-600">${bulkMenuText}</textarea>
            </div>

            <div class="flex gap-2 mt-4">
              <button onclick="importBulkMenu(false)" class="flex-1 bg-amber-600 hover:bg-amber-700 text-white font-extrabold py-3 rounded-xl text-xs shadow">
                ➕ Add to Existing Menu
              </button>
              <button onclick="if(confirm('पुराना मेन्यू हटाकर सिर्फ यह नया मेन्यू सेट करें?')) importBulkMenu(true)" class="flex-1 bg-[#1C130E] text-amber-400 font-extrabold py-3 rounded-xl text-xs shadow">
                🔄 Replace Full Menu
              </button>
            </div>
          </div>
        </div>
      `;
    }

    function fillSampleBulkText() {
      bulkMenuText = "Coffee | Special Kulhad Cold Coffee | 130\nChai & Coolers | Tandoori Masala Chai | 40\nSnacks & Momos | Kurkure Veg Momos | 120\nBurgers & Sandwiches | Double Cheese Paneer Burger | 150\nPizza & Pasta | Tandoori Paneer Pizza | 220";
      render();
    }

    async function handleMenuPhotoUpload(event) {
      const file = event.target.files[0];
      if (!file) return;
      if (!saasDB.gemini_api_key) {
        alert("Direct AI Photo Scan के लिए अपनी फ्री Google Gemini API Key डालें!");
        return;
      }
      isScanningMenu = true;
      render();
      try {
        const base64Data = await new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.onload = () => resolve(reader.result.split(",")[1]);
          reader.onerror = reject;
          reader.readAsDataURL(file);
        });
        const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${saasDB.gemini_api_key}`, {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            contents: [{
              parts: [
                { text: "Extract all food and drink menu items and prices. Return ONLY plain text lines in this exact format: Category | Item Name | Price (number only)." },
                { inline_data: { mime_type: file.type || "image/jpeg", data: base64Data } }
              ]
            }]
          })
        });
        const data = await response.json();
        const extractedText = data?.candidates?.[0]?.content?.parts?.[0]?.text || "";
        if (extractedText) bulkMenuText = extractedText.trim();
      } catch (err) {
        alert("Scan failed.");
      }
      isScanningMenu = false;
      render();
    }

    function importBulkMenu(replaceAll) {
      const raw = (document.getElementById("bulkMenuInput")?.value || bulkMenuText).trim();
      if (!raw) { alert("कृपया पहले मेन्यू लिस्ट दें!"); return; }
      const lines = raw.split("\n").map(l => l.trim()).filter(Boolean);
      const newItems = [];
      lines.forEach((line, idx) => {
        let category = "Specials";
        let name = "";
        let price = 99;
        if (line.includes("|")) {
          const parts = line.split("|").map(p => p.trim());
          if (parts.length >= 3) {
            category = parts[0] || "Specials";
            name = parts[1];
            price = parseInt(parts[2].replace(/[^0-9]/g, "")) || 99;
          } else if (parts.length === 2) {
            name = parts[0];
            price = parseInt(parts[1].replace(/[^0-9]/g, "")) || 99;
          }
        } else {
          name = line;
        }
        if (name) {
          newItems.push({
            id: Date.now() + idx,
            name,
            category,
            price,
            is_veg: true,
            is_bestseller: idx < 3,
            in_stock: true,
            img: getAutoFoodImage(category + " " + name)
          });
        }
      });
      if (replaceAll) currentCafe().menu = newItems;
      else currentCafe().menu = [...newItems, ...currentCafe().menu];
      saveSaaS();
      showScannerModal = false;
      bulkMenuText = "";
      alert(`🎉 ${newItems.length} आइटम्स सफलतापूर्वक जुड़ गए हैं!`);
      render();
    }

    function renderSuperAdmin() {
      if (!pinUnlocked) {
        return `
          <div class="min-h-screen bg-stone-950 text-white flex items-center justify-center p-4">
            <div class="bg-stone-900 p-6 rounded-3xl max-w-sm w-full text-center border-2 border-amber-500">
              <span class="text-xs font-extrabold uppercase px-3 py-1 bg-amber-500 text-black rounded-full">Developer Master Control</span>
              <h2 class="text-xl font-extrabold text-amber-400 mt-3 mb-1">🚀 Super Admin SaaS Panel</h2>
              <p class="text-xs text-stone-400 mb-4">Enter Developer PIN (Default: 9999)</p>
              <input id="superPin" type="password" maxlength="4" placeholder="••••" class="w-full text-center text-2xl tracking-widest bg-stone-800 rounded-xl py-3 mb-4 text-white border border-stone-700" />
              <button onclick="if(document.getElementById('superPin').value === SUPER_ADMIN_PIN){pinUnlocked=true; render();}else{alert('Wrong PIN!');}" class="w-full bg-amber-500 text-black py-3 rounded-xl font-extrabold">Unlock Master Panel</button>
            </div>
          </div>
        `;
      }

      const cafeList = Object.values(saasDB.cafes);
      const baseUrl = window.location.origin + window.location.pathname;

      return `
        <div class="min-h-screen bg-stone-950 text-white p-4">
          <div class="max-w-6xl mx-auto">
            <div class="bg-stone-900 p-5 rounded-3xl border border-amber-500/40 flex flex-wrap justify-between items-center gap-4 mb-6">
              <div>
                <h1 class="text-2xl font-extrabold text-amber-400">🚀 Multi-Cafe SaaS Super Admin</h1>
                <p class="text-xs text-stone-400">Client Cafes Control Center</p>
              </div>
              <span class="text-xs bg-stone-800 px-3 py-2 rounded-xl border border-stone-700">Total Cafes: <b class="text-amber-400">${cafeList.length}</b></span>
            </div>

            <div class="bg-stone-900 p-5 rounded-3xl border border-stone-800 mb-6">
              <h2 class="text-base font-extrabold text-amber-400 mb-3">➕ Create New Cafe Website</h2>
              <div class="grid grid-cols-1 sm:grid-cols-5 gap-3">
                <input id="newSlug" type="text" placeholder="Cafe ID (e.g. hangout)" class="bg-stone-800 border border-stone-700 rounded-xl px-3 py-2.5 text-xs text-white" />
                <input id="newName" type="text" placeholder="Cafe Name" class="bg-stone-800 border border-stone-700 rounded-xl px-3 py-2.5 text-xs text-white" />
                <input id="newUpi" type="text" placeholder="UPI ID" class="bg-stone-800 border border-stone-700 rounded-xl px-3 py-2.5 text-xs text-white" />
                <input id="newPhone" type="text" placeholder="WhatsApp" class="bg-stone-800 border border-stone-700 rounded-xl px-3 py-2.5 text-xs text-white" />
                <button onclick="createNewCafeClient()" class="bg-amber-500 hover:bg-amber-400 text-black font-extrabold rounded-xl py-2.5 text-xs shadow">⚡ Launch</button>
              </div>
            </div>

            <div class="space-y-4">
              ${cafeList.map(c => {
                const custUrl = `${baseUrl}?cafe=${c.slug}&table=1`;
                const kdsUrl = `${baseUrl}?cafe=${c.slug}&mode=kitchen`;
                const ownUrl = `${baseUrl}?cafe=${c.slug}&mode=owner`;
                return `
                  <div class="bg-stone-900 rounded-3xl p-5 border ${c.is_active ? 'border-stone-800' : 'border-red-600 opacity-75'}">
                    <div class="flex flex-wrap justify-between items-start gap-3 pb-4 border-b border-stone-800">
                      <div>
                        <div class="flex items-center gap-2">
                          <h3 class="text-xl font-extrabold text-white">${c.cafe_name}</h3>
                          <span class="text-xs font-mono bg-stone-800 text-amber-400 px-2.5 py-0.5 rounded-full">ID: ${c.slug}</span>
                        </div>
                        <p class="text-xs text-stone-400 mt-1">UPI: <b>${c.upi_id}</b> • Menu Items: <b>${c.menu.length}</b> • Orders Recorded: <b>${c.orders.length}</b></p>
                      </div>
                      <div class="flex gap-2">
                        <button onclick="activeCafeSlug='${c.slug}'; showScannerModal=true; render()" class="bg-amber-500 text-black font-extrabold px-3 py-2 rounded-xl text-xs">📸 Scan Menu</button>
                        <button onclick="toggleCafeActive('${c.slug}')" class="px-3 py-2 rounded-xl text-xs font-bold ${c.is_active ? 'bg-red-950 text-red-300' : 'bg-green-700 text-white'}">${c.is_active ? '🔒 Suspend' : '🔓 Activate'}</button>
                      </div>
                    </div>
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-3 mt-4 text-xs">
                      <div class="bg-stone-950 p-3 rounded-2xl border border-stone-800">
                        <div class="font-bold text-amber-400 mb-1">1. Customer Menu</div>
                        <input type="text" readonly value="${custUrl}" class="w-full bg-stone-900 text-stone-300 p-1.5 rounded text-[11px] mb-2" />
                        <a href="${custUrl}" target="_blank" class="block text-center bg-amber-600/20 text-amber-400 py-1.5 rounded-lg font-bold">Open ↗</a>
                      </div>
                      <div class="bg-stone-950 p-3 rounded-2xl border border-stone-800">
                        <div class="font-bold text-green-400 mb-1">2. Chef KDS (PIN: ${c.chef_pin})</div>
                        <input type="text" readonly value="${kdsUrl}" class="w-full bg-stone-900 text-stone-300 p-1.5 rounded text-[11px] mb-2" />
                        <a href="${kdsUrl}" target="_blank" class="block text-center bg-green-600/20 text-green-400 py-1.5 rounded-lg font-bold">Open ↗</a>
                      </div>
                      <div class="bg-stone-950 p-3 rounded-2xl border border-stone-800">
                        <div class="font-bold text-purple-400 mb-1">3. Owner Panel (PIN: ${c.owner_pin})</div>
                        <input type="text" readonly value="${ownUrl}" class="w-full bg-stone-900 text-stone-300 p-1.5 rounded text-[11px] mb-2" />
                        <a href="${ownUrl}" target="_blank" class="block text-center bg-purple-600/20 text-purple-400 py-1.5 rounded-lg font-bold">Open ↗</a>
                      </div>
                    </div>
                  </div>
                `;
              }).join('')}
            </div>
          </div>
        </div>
        ${renderScannerModal()}
      `;
    }

    function createNewCafeClient() {
      const slug = (document.getElementById("newSlug").value || "").trim().toLowerCase().replace(/[^a-z0-9-_]/g, "");
      const name = (document.getElementById("newName").value || "").trim();
      const upi = (document.getElementById("newUpi").value || "").trim() || "merchant@upi";
      const phone = (document.getElementById("newPhone").value || "").trim() || "919876543210";
      if (!slug || !name) { alert("Cafe ID और Name भरें!"); return; }
      if (saasDB.cafes[slug]) { alert("यह ID पहले से है!"); return; }

      saasDB.cafes[slug] = {
        slug,
        is_active: true,
        cafe_name: name,
        tagline: "Fresh Food • Fast Table Ordering",
        upi_id: upi,
        whatsapp_number: phone,
        chef_pin: "1122",
        owner_pin: "8899",
        total_tables: 12,
        google_review_link: "https://google.com",
        rush_hour_discount: false,
        menu: JSON.parse(JSON.stringify(defaultMenuTemplate)),
        orders: []
      };
      saveSaaS();
      render();
    }

    function toggleCafeActive(slug) {
      if (saasDB.cafes[slug]) {
        saasDB.cafes[slug].is_active = !saasDB.cafes[slug].is_active;
        saveSaaS();
        render();
      }
    }

    function renderCustomer() {
      const cafe = currentCafe();
      if (!cafe.is_active) {
        return `<div class="min-h-screen flex items-center justify-center p-6 text-center"><div class="bg-white p-8 rounded-3xl shadow-xl max-w-sm"><h2 class="text-xl font-extrabold text-red-600 mb-2">⚠️ Service Paused</h2><p class="text-xs text-stone-600">This cafe system is currently paused.</p></div></div>`;
      }

      const menu = cafe.menu;
      const categories = ["All", ...new Set(menu.map(i => i.category))];
      const filtered = menu.filter(item => {
        const matchCat = selectedCategory === "All" || item.category === selectedCategory;
        const matchSearch = item.name.toLowerCase().includes(searchQuery.toLowerCase());
        const matchBest = !bestsellerOnly || item.is_bestseller;
        return matchCat && matchSearch && matchBest;
      });

      const cartItems = Object.values(cart);
      const totalQty = cartItems.reduce((sum, i) => sum + i.qty, 0);
      const subtotal = cartItems.reduce((sum, i) => sum + (getDiscountedPrice(i.price) * i.qty), 0);
      const gst = Math.round(subtotal * 0.05);
      const grandTotal = subtotal + gst;
      const activeOrder = cafe.orders.find(o => o.id === activeOrderId);

      return `
        <header class="sticky top-0 z-30 bg-[#1C130E] text-[#FAF6F0] px-4 py-3 shadow-lg">
          <div class="max-w-xl mx-auto flex items-center justify-between">
            <div>
              <h1 class="text-xl font-extrabold tracking-tight text-amber-400">${cafe.cafe_name}</h1>
              <p class="text-xs text-stone-300">${cafe.tagline}</p>
            </div>
            <div class="flex items-center gap-2 bg-stone-800 px-3 py-1.5 rounded-full border border-amber-500/40">
              <span class="text-xs font-bold text-amber-300">📍 Table #</span>
              <select onchange="currentTable=this.value; render()" class="bg-transparent text-white font-bold text-sm focus:outline-none">
                ${Array.from({length: cafe.total_tables}, (_, i) => i + 1).map(num => `<option value="${num}" class="text-black" ${currentTable == num ? 'selected' : ''}>${num}</option>`).join('')}
              </select>
            </div>
          </div>
          ${cafe.rush_hour_discount ? `<div class="max-w-xl mx-auto mt-2 bg-gradient-to-r from-amber-600 to-orange-600 text-white text-xs font-bold px-3 py-1 rounded-lg text-center">⚡ Happy Hours Active! Flat 10% OFF!</div>` : ''}
        </header>

        <main class="max-w-xl mx-auto px-4 pt-4">
          ${activeOrder ? `
            <div class="mb-6 bg-white rounded-2xl p-4 shadow-md border-2 border-amber-500">
              <div class="flex justify-between items-center mb-2">
                <span class="text-xs font-bold uppercase bg-amber-100 text-amber-800 px-2.5 py-1 rounded-full">Order #${activeOrder.order_number} • Table ${activeOrder.table_number}</span>
                <button onclick="activeOrderId=null; localStorage.removeItem('active_order_'+activeCafeSlug); render()" class="text-xs text-stone-400 underline">Dismiss</button>
              </div>
              <div class="grid grid-cols-3 gap-2 my-3 text-center">
                <div class="p-2 rounded-xl ${activeOrder.order_status === 'new' ? 'bg-amber-500 text-white font-bold' : 'bg-stone-100 text-stone-600'} text-xs">1. 🔔 Sent</div>
                <div class="p-2 rounded-xl ${activeOrder.order_status === 'preparing' ? 'bg-amber-500 text-white font-bold' : 'bg-stone-100 text-stone-600'} text-xs">2. 👨‍🍳 Preparing</div>
                <div class="p-2 rounded-xl ${(activeOrder.order_status === 'ready' || activeOrder.order_status === 'completed') ? 'bg-green-600 text-white font-bold' : 'bg-stone-100 text-stone-600'} text-xs">3. 🍽️ Ready!</div>
              </div>
              <div class="flex gap-2 mt-3">
                <a href="https://wa.me/${cafe.whatsapp_number}?text=${encodeURIComponent(`Hi ${cafe.cafe_name}! Order #${activeOrder.order_number} (Table #${activeOrder.table_number}): ${activeOrder.items.map(x=>x.qty+'x '+x.name).join(', ')} - Total ₹${activeOrder.total_amount}`)}" target="_blank" class="flex-1 text-center bg-green-600 text-white text-xs font-bold py-2.5 rounded-xl">💬 WhatsApp Backup</a>
                <a href="${cafe.google_review_link}" target="_blank" class="flex-1 text-center bg-amber-100 text-amber-900 border border-amber-400 text-xs font-bold py-2.5 rounded-xl">⭐ Rate & Get 10% Off</a>
              </div>
            </div>
          ` : ''}

          <div class="flex gap-2 mb-3">
            <input type="text" placeholder="Search dish..." value="${searchQuery}" oninput="searchQuery=this.value; render()" class="flex-1 bg-white px-4 py-2.5 rounded-xl border border-stone-200 text-sm shadow-sm focus:outline-none focus:border-amber-600" />
            <button onclick="bestsellerOnly=!bestsellerOnly; render()" class="px-3 py-2 rounded-xl text-xs font-bold border ${bestsellerOnly ? 'bg-amber-600 text-white border-amber-600' : 'bg-white text-stone-700 border-stone-200'}">🔥 Bestsellers</button>
          </div>

          <div class="flex gap-2 overflow-x-auto no-scrollbar pb-3 mb-2">
            ${categories.map(cat => `<button onclick="selectedCategory='${cat}'; render()" class="whitespace-nowrap px-4 py-2 rounded-full text-xs font-bold ${selectedCategory === cat ? 'bg-[#1C130E] text-amber-400 shadow' : 'bg-white text-stone-700 border border-stone-200'}">${cat}</button>`).join('')}
          </div>

          <div class="space-y-3">
            ${filtered.map(item => {
              const inCart = cart[item.id]?.qty || 0;
              const finalPrice = getDiscountedPrice(item.price);
              return `
                <div class="bg-white rounded-2xl p-3 shadow-sm border border-stone-200/80 flex gap-3 items-center ${!item.in_stock ? 'opacity-50 grayscale' : ''}">
                  <img src="${item.img}" onerror="this.src='https://images.unsplash.com/photo-1504674900247-0877df9cc836?w=400&q=80'" class="w-20 h-20 rounded-xl object-cover flex-shrink-0 bg-stone-100" />
                  <div class="flex-1 min-w-0">
                    <div class="flex items-center gap-1.5 mb-1">
                      <span class="w-3 h-3 border border-green-600 flex items-center justify-center rounded-sm"><span class="w-1.5 h-1.5 bg-green-600 rounded-full"></span></span>
                      ${item.is_bestseller ? `<span class="text-[10px] font-bold bg-amber-100 text-amber-800 px-2 py-0.5 rounded-full">🔥 Bestseller</span>` : ''}
                    </div>
                    <h3 class="font-bold text-sm text-stone-900 leading-snug">${item.name}</h3>
                    <div class="mt-1 flex items-center gap-2">
                      <span class="font-extrabold text-base text-stone-900">₹${finalPrice}</span>
                      ${cafe.rush_hour_discount ? `<span class="text-xs text-stone-400 line-through">₹${item.price}</span>` : ''}
                    </div>
                  </div>
                  <div>
                    ${!item.in_stock ? `<span class="text-xs font-bold text-red-600 bg-red-50 px-2.5 py-1.5 rounded-lg">Out of Stock</span>` : inCart === 0 ? `<button onclick="updateCart(${item.id}, 1)" class="bg-amber-50 hover:bg-amber-100 text-amber-800 border border-amber-500 font-extrabold text-xs px-4 py-2 rounded-xl shadow-sm">+ ADD</button>` : `<div class="flex items-center bg-[#1C130E] text-white rounded-xl overflow-hidden shadow"><button onclick="updateCart(${item.id}, -1)" class="px-3 py-1.5 font-bold text-amber-400">-</button><span class="px-2 text-xs font-bold">${inCart}</span><button onclick="updateCart(${item.id}, 1)" class="px-3 py-1.5 font-bold text-amber-400">+</button></div>`}
                  </div>
                </div>
              `;
            }).join('')}
          </div>
        </main>

        ${totalQty > 0 ? `<button onclick="cartStep=1; render()" class="fixed left-4 bottom-4 z-40 bg-[#1C130E] text-white px-5 py-3.5 rounded-2xl shadow-2xl border-2 border-amber-500 flex items-center gap-3"><div class="text-left"><div class="text-[11px] text-amber-400 font-bold uppercase">${totalQty} Item${totalQty > 1 ? 's' : ''} Added</div><div class="text-sm font-extrabold">₹${grandTotal} • View Cart ➔</div></div></button>` : ''}

        ${cartStep > 0 ? `
          <div class="fixed inset-0 z-50 bg-black/60 flex items-end justify-center p-0 sm:p-4">
            <div class="bg-white w-full max-w-lg rounded-t-3xl sm:rounded-3xl p-5 max-h-[90vh] overflow-y-auto">
              ${cartStep === 1 ? `
                <div class="flex justify-between items-center pb-3 border-b"><h2 class="text-lg font-extrabold">Step 1: Review Order (Table #${currentTable})</h2><button onclick="cartStep=0; render()" class="text-stone-400 font-bold text-xl">✕</button></div>
                <div class="divide-y my-3">
                  ${cartItems.map(i => `<div class="py-2.5 flex justify-between items-center"><div><p class="font-bold text-sm">${i.name}</p><p class="text-xs text-stone-500">₹${getDiscountedPrice(i.price)} x ${i.qty}</p></div><span class="font-bold text-sm">₹${getDiscountedPrice(i.price) * i.qty}</span></div>`).join('')}
                </div>
                <div class="my-3">
                  <div class="flex flex-wrap gap-1.5 mb-2">
                    ${["Less Spicy", "Extra Crispy", "Less Sugar", "No Onion/Garlic"].map(chip => `<button onclick="toggleNote('${chip}')" class="text-xs px-2.5 py-1 rounded-full border ${selectedNotes.includes(chip) ? 'bg-amber-600 text-white border-amber-600' : 'bg-stone-100 text-stone-700'}">${chip}</button>`).join('')}
                  </div>
                  <input type="text" placeholder="Custom note..." value="${customNote}" oninput="customNote=this.value" class="w-full border rounded-xl px-3 py-2 text-xs mb-2" />
                  <input type="text" placeholder="Your Name (Optional)" value="${customerName}" oninput="customerName=this.value" class="w-full border rounded-xl px-3 py-2 text-xs" />
                </div>
                <div class="bg-stone-50 p-3 rounded-xl text-xs space-y-1 my-3">
                  <div class="flex justify-between"><span>Subtotal</span><span>₹${subtotal}</span></div>
                  <div class="flex justify-between"><span>GST (5%)</span><span>₹${gst}</span></div>
                  <div class="flex justify-between font-extrabold text-sm pt-1 border-t"><span>Grand Total</span><span>₹${grandTotal}</span></div>
                </div>
                <button onclick="cartStep=2; render()" class="w-full bg-amber-600 text-white font-extrabold py-3.5 rounded-2xl shadow-lg">Proceed to Confirm ➔</button>
              ` : `
                <div class="flex justify-between items-center pb-3 border-b"><h2 class="text-lg font-extrabold">Step 2: Payment & Confirm</h2><button onclick="cartStep=1; render()" class="text-xs text-amber-700 font-bold">← Back</button></div>
                <div class="my-4 space-y-3">
                  <label class="flex items-center gap-3 p-3 border-2 rounded-2xl cursor-pointer ${paymentMode === 'COUNTER' ? 'border-amber-600 bg-amber-50/40' : 'border-stone-200'}">
                    <input type="radio" name="paymode" ${paymentMode === 'COUNTER' ? 'checked' : ''} onchange="paymentMode='COUNTER'; render()" />
                    <div><div class="font-bold text-sm">Pay at Counter / After Meal</div><div class="text-xs text-stone-500">Cash or UPI at Counter</div></div>
                  </label>
                  <label class="flex items-center gap-3 p-3 border-2 rounded-2xl cursor-pointer ${paymentMode === 'UPI' ? 'border-amber-600 bg-amber-50/40' : 'border-stone-200'}">
                    <input type="radio" name="paymode" ${paymentMode === 'UPI' ? 'checked' : ''} onchange="paymentMode='UPI'; render()" />
                    <div><div class="font-bold text-sm">Pay Now via Direct UPI (₹${grandTotal})</div><div class="text-xs text-stone-500">Zero Commission Transfer</div></div>
                  </label>
                  ${paymentMode === 'UPI' ? `<div class="p-4 bg-stone-50 rounded-2xl border text-center space-y-2"><a href="upi://pay?pa=${cafe.upi_id}&pn=${encodeURIComponent(cafe.cafe_name)}&am=${grandTotal}&cu=INR" class="block w-full bg-green-600 text-white font-bold py-2.5 rounded-xl text-sm shadow">📱 Open GPay / PhonePe / Paytm</a><input type="text" maxlength="4" placeholder="Last 4 digits of UTR (Optional)" value="${utrNumber}" oninput="utrNumber=this.value" class="w-full border rounded-xl px-3 py-2 text-xs text-center font-bold" /></div>` : ''}
                </div>
                <button onclick="placeFinalOrder(${subtotal}, ${gst}, ${grandTotal})" class="w-full bg-[#1C130E] text-amber-400 font-extrabold py-4 rounded-2xl shadow-xl text-base">🔔 Confirm & Send Order</button>
              `}
            </div>
          </div>
        ` : ''}
      `;
    }

    function updateCart(id, delta) {
      const item = currentCafe().menu.find(x => x.id === id);
      if (!item) return;
      if (!cart[id]) cart[id] = { ...item, qty: 0 };
      cart[id].qty += delta;
      if (cart[id].qty <= 0) delete cart[id];
      render();
    }

    function toggleNote(chip) {
      if (selectedNotes.includes(chip)) selectedNotes = selectedNotes.filter(c => c !== chip);
      else selectedNotes.push(chip);
      render();
    }

    function placeFinalOrder(subtotal, gst, total) {
      const cafe = currentCafe();
      const orderNum = Math.floor(100 + Math.random() * 900);
      const newOrder = {
        id: "ord_" + Date.now(),
        order_number: orderNum,
        table_number: currentTable,
        customer_name: customerName || "Guest",
        items: Object.values(cart).map(i => ({ id: i.id, name: i.name, price: getDiscountedPrice(i.price), qty: i.qty })),
        cooking_notes: [...selectedNotes, customNote].filter(Boolean).join(", "),
        subtotal,
        gst_amount: gst,
        total_amount: total,
        payment_method: paymentMode === "UPI" ? `UPI ${utrNumber ? '(UTR:' + utrNumber + ')' : ''}` : "CASH/COUNTER",
        payment_status: paymentMode === "UPI" ? "PAID" : "PENDING",
        order_status: "new",
        created_at: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })
      };
      cafe.orders.unshift(newOrder);
      activeOrderId = newOrder.id;
      localStorage.setItem("active_order_" + activeCafeSlug, activeOrderId);
      cart = {};
      cartStep = 0;
      selectedNotes = [];
      customNote = "";
      saveSaaS();
      render();
    }

    function renderKDS() {
      const cafe = currentCafe();
      if (!pinUnlocked) {
        return `
          <div class="min-h-screen bg-[#1C130E] text-white flex items-center justify-center p-4">
            <div class="bg-stone-900 p-6 rounded-3xl max-w-sm w-full text-center border border-amber-500/30">
              <h2 class="text-xl font-extrabold text-amber-400 mb-1">👨‍🍳 Kitchen KDS</h2>
              <p class="text-xs text-stone-400 mb-4">Enter Chef PIN (Default: ${cafe.chef_pin})</p>
              <input id="pinInput" type="password" maxlength="4" placeholder="••••" class="w-full text-center text-2xl tracking-widest bg-stone-800 rounded-xl py-3 mb-4 text-white border border-stone-700" />
              <button onclick="if(document.getElementById('pinInput').value === currentCafe().chef_pin){pinUnlocked=true; render();}else{alert('Wrong PIN!');}" class="w-full bg-amber-600 py-3 rounded-xl font-extrabold">Unlock Kitchen</button>
            </div>
          </div>
        `;
      }

      return `
        <div class="min-h-screen bg-stone-950 text-white p-4">
          <div class="max-w-6xl mx-auto flex flex-wrap justify-between items-center gap-3 pb-4 border-b border-stone-800">
            <div>
              <h1 class="text-2xl font-extrabold text-amber-400">👨‍🍳 ${cafe.cafe_name} — Live Kitchen KDS</h1>
              <p class="text-xs text-stone-400">Orders are securely logged</p>
            </div>
            <button onclick="soundEnabled=!soundEnabled; if(soundEnabled) playKitchenBell(); render()" class="px-4 py-2 rounded-xl text-xs font-extrabold ${soundEnabled ? 'bg-green-600 text-white' : 'bg-stone-800 text-stone-300 border border-stone-700'}">
              ${soundEnabled ? '🔊 Sound Alert ON' : '🔇 Enable Kitchen Bell'}
            </button>
          </div>
          <div class="max-w-6xl mx-auto grid grid-cols-1 md:grid-cols-3 gap-4 mt-6">
            ${cafe.orders.length === 0 ? `<p class="text-stone-500 col-span-3 text-center py-12">No orders yet...</p>` : ''}
            ${cafe.orders.map(o => `
              <div class="bg-stone-900 rounded-2xl p-4 border-2 ${o.order_status === 'new' ? 'border-red-500 animate-pulse-red' : o.order_status === 'preparing' ? 'border-amber-500' : 'border-green-600 opacity-75'}">
                <div class="flex justify-between items-start mb-3">
                  <div>
                    <span class="text-2xl font-extrabold text-amber-400">Table #${o.table_number}</span>
                    <p class="text-xs text-stone-400">Order #${o.order_number} • ${o.created_at}</p>
                  </div>
                  <span class="text-[11px] font-bold px-2.5 py-1 rounded-full bg-stone-800 text-amber-300">${o.payment_method}</span>
                </div>
                <div class="divide-y divide-stone-800 my-3">
                  ${o.items.map(i => `<div class="py-1.5 text-sm font-bold">${i.qty}x ${i.name}</div>`).join('')}
                </div>
                ${o.cooking_notes ? `<div class="bg-red-950/60 border border-red-800 text-red-200 text-xs p-2 rounded-lg mb-3">📝 Note: ${o.cooking_notes}</div>` : ''}
                <div class="flex gap-2 mt-3">
                  ${o.order_status === 'new' ? `
                    <button onclick="updateOrderStatus('${o.id}', 'preparing')" class="flex-1 bg-amber-600 text-white font-bold py-2.5 rounded-xl text-xs">👨‍🍳 Start Preparing</button>
                  ` : o.order_status === 'preparing' ? `
                    <button onclick="updateOrderStatus('${o.id}', 'ready')" class="flex-1 bg-green-600 text-white font-bold py-2.5 rounded-xl text-xs">🔔 Mark Ready / Completed</button>
                  ` : `
                    <span class="flex-1 text-center bg-stone-800 text-green-400 font-bold py-2 rounded-xl text-xs">✔ Completed & Saved</span>
                  `}
                </div>
              </div>
            `).join('')}
          </div>
        </div>
      `;
    }

    function updateOrderStatus(id, st) {
      const ord = currentCafe().orders.find(x => x.id === id);
      if (ord) { ord.order_status = st; saveSaaS(); render(); }
    }

    function renderOwner() {
      const cafe = currentCafe();
      if (!pinUnlocked) {
        return `
          <div class="min-h-screen bg-[#1C130E] text-white flex items-center justify-center p-4">
            <div class="bg-stone-900 p-6 rounded-3xl max-w-sm w-full text-center border border-amber-500/30">
              <h2 class="text-xl font-extrabold text-amber-400 mb-1">👑 Owner Panel</h2>
              <p class="text-xs text-stone-400 mb-4">Enter Owner PIN (Default: ${cafe.owner_pin})</p>
              <input id="ownerPin" type="password" maxlength="4" placeholder="••••" class="w-full text-center text-2xl tracking-widest bg-stone-800 rounded-xl py-3 mb-4 text-white border border-stone-700" />
              <button onclick="if(document.getElementById('ownerPin').value === currentCafe().owner_pin){pinUnlocked=true; render();}else{alert('Wrong PIN!');}" class="w-full bg-amber-600 py-3 rounded-xl font-extrabold">Unlock Boss Dashboard</button>
            </div>
          </div>
        `;
      }

      const totalRev = cafe.orders.reduce((s, o) => s + o.total_amount, 0);
      const upiRev = cafe.orders.filter(o => o.payment_status === 'PAID').reduce((s, o) => s + o.total_amount, 0);
      const cashRev = totalRev - upiRev;

      return `
        <div class="max-w-5xl mx-auto p-4">
          <div class="bg-[#1C130E] text-white p-5 rounded-3xl shadow-lg flex flex-wrap justify-between items-center gap-3 mb-6">
            <div>
              <h1 class="text-xl font-extrabold text-amber-400">👑 ${cafe.cafe_name} — Owner Admin</h1>
              <p class="text-xs text-stone-300">Sales Cashbook & Permanent Order History</p>
            </div>
            <div class="flex flex-wrap gap-2">
              <button onclick="ownerTab='sales'; render()" class="px-4 py-2 rounded-xl text-xs font-bold ${ownerTab === 'sales' ? 'bg-amber-500 text-black' : 'bg-stone-800'}">📊 Sales Cashbook</button>
              <button onclick="ownerTab='menu'; render()" class="px-4 py-2 rounded-xl text-xs font-bold ${ownerTab === 'menu' ? 'bg-amber-500 text-black' : 'bg-stone-800'}">📸 Menu & Scanner</button>
              <button onclick="ownerTab='branding'; render()" class="px-4 py-2 rounded-xl text-xs font-bold ${ownerTab === 'branding' ? 'bg-amber-500 text-black' : 'bg-stone-800'}">⚙️ Settings & QRs</button>
            </div>
          </div>

          ${ownerTab === 'sales' ? `
            <div class="grid grid-cols-2 md:grid-cols-4 gap-3 mb-6">
              <div class="bg-white p-4 rounded-2xl border shadow-sm"><p class="text-xs text-stone-500">Total Revenue</p><p class="text-2xl font-extrabold">₹${totalRev}</p></div>
              <div class="bg-white p-4 rounded-2xl border shadow-sm"><p class="text-xs text-stone-500">Direct UPI</p><p class="text-2xl font-extrabold text-green-600">₹${upiRev}</p></div>
              <div class="bg-white p-4 rounded-2xl border shadow-sm"><p class="text-xs text-stone-500">Cash / Counter</p><p class="text-2xl font-extrabold text-amber-600">₹${cashRev}</p></div>
              <div class="bg-white p-4 rounded-2xl border shadow-sm"><p class="text-xs text-stone-500">Total Orders</p><p class="text-2xl font-extrabold">${cafe.orders.length}</p></div>
            </div>
            <div class="bg-white rounded-2xl p-4 border shadow-sm">
              <div class="flex justify-between items-center mb-3">
                <h3 class="font-bold">Permanent Order History (Safe & Secured)</h3>
                <span class="text-xs text-stone-400">Records cannot be accidentally deleted</span>
              </div>
              <div class="divide-y max-h-96 overflow-y-auto">
                ${cafe.orders.length === 0 ? `<p class="text-stone-400 text-xs py-4 text-center">No orders recorded yet.</p>` : ''}
                ${cafe.orders.map(o => `
                  <div class="py-2.5 flex justify-between items-center text-sm">
                    <div>
                      <span class="font-bold">Table #${o.table_number} (Order #${o.order_number})</span>
                      <p class="text-xs text-stone-500">${o.items.map(x=>x.qty+'x '+x.name).join(', ')} • ${o.created_at}</p>
                    </div>
                    <div class="text-right">
                      <p class="font-extrabold">₹${o.total_amount}</p>
                      <span class="text-[10px] font-bold px-2 py-0.5 rounded bg-green-50 text-green-700">${o.order_status.toUpperCase()}</span>
                    </div>
                  </div>
                `).join('')}
              </div>
            </div>
          ` : ownerTab === 'menu' ? `
            <div class="bg-white rounded-2xl p-4 border shadow-sm space-y-4">
              <div class="flex flex-wrap justify-between items-center gap-2 pb-3 border-b">
                <div>
                  <h3 class="font-extrabold text-base">🍔 Menu & Stock Manager (${cafe.menu.length} Items)</h3>
                  <p class="text-xs text-stone-500">मेन्यू कार्ड फोटो स्कैन करें या सिंगल आइटम जोड़ें</p>
                </div>
                <button onclick="showScannerModal=true; render()" class="bg-[#1C130E] text-amber-400 font-extrabold px-4 py-2.5 rounded-xl text-xs shadow-lg">📸 Scan Menu Photo / Bulk Add</button>
              </div>

              <div class="grid grid-cols-1 sm:grid-cols-4 gap-2 bg-stone-50 p-3 rounded-xl border">
                <input id="singleCat" type="text" placeholder="Category" class="border rounded-lg px-3 py-2 text-xs" />
                <input id="singleName" type="text" placeholder="Dish Name" class="border rounded-lg px-3 py-2 text-xs" />
                <input id="singlePrice" type="number" placeholder="Price (₹)" class="border rounded-lg px-3 py-2 text-xs" />
                <button onclick="addSingleDish()" class="bg-amber-600 text-white font-bold rounded-lg py-2 text-xs">+ Add Dish</button>
              </div>

              <div class="divide-y max-h-96 overflow-y-auto">
                ${cafe.menu.map(item => `
                  <div class="py-3 flex justify-between items-center gap-3">
                    <div class="flex items-center gap-3">
                      <img src="${item.img}" class="w-12 h-12 rounded-lg object-cover bg-stone-100" />
                      <div>
                        <span class="text-[10px] font-bold text-amber-700 uppercase">${item.category}</span>
                        <p class="font-bold text-sm">${item.name}</p>
                        <input type="number" value="${item.price}" onchange="updateItemPrice(${item.id}, this.value)" class="border rounded px-2 py-0.5 text-xs w-20 font-bold mt-1" />
                      </div>
                    </div>
                    <div class="flex items-center gap-2">
                      <button onclick="toggleStock(${item.id})" class="px-3 py-1.5 rounded-xl text-xs font-bold ${item.in_stock ? 'bg-green-100 text-green-800' : 'bg-red-100 text-red-800'}">${item.in_stock ? '✔ In Stock' : '✕ Out'}</button>
                      <button onclick="deleteMenuItem(${item.id})" class="text-red-500 text-xs px-2">🗑️</button>
                    </div>
                  </div>
                `).join('')}
              </div>
            </div>
          ` : `
            <div class="grid md:grid-cols-2 gap-6">
              <div class="bg-white p-5 rounded-2xl border shadow-sm space-y-3">
                <h3 class="font-extrabold text-base">🏷️ Cafe Settings & Happy Hours</h3>
                <div><label class="text-xs font-bold text-stone-600">Cafe Name</label><input type="text" value="${cafe.cafe_name}" oninput="currentCafe().cafe_name=this.value; saveSaaS()" class="w-full border rounded-xl px-3 py-2 text-sm font-bold" /></div>
                <div><label class="text-xs font-bold text-stone-600">Owner Direct UPI ID</label><input type="text" value="${cafe.upi_id}" oninput="currentCafe().upi_id=this.value; saveSaaS()" class="w-full border rounded-xl px-3 py-2 text-sm font-bold" /></div>
                <div><label class="text-xs font-bold text-stone-600">Google Review Link</label><input type="text" value="${cafe.google_review_link}" oninput="currentCafe().google_review_link=this.value; saveSaaS()" class="w-full border rounded-xl px-3 py-2 text-sm" /></div>
                <div class="pt-2 flex items-center justify-between border-t"><span class="text-sm font-bold">⚡ Enable Happy Hours (10% OFF)</span><input type="checkbox" ${cafe.rush_hour_discount ? 'checked' : ''} onchange="currentCafe().rush_hour_discount=this.checked; saveSaaS(); render()" class="w-5 h-5" /></div>
              </div>
              <div class="bg-white p-5 rounded-2xl border shadow-sm">
                <div class="flex justify-between items-center mb-3"><h3 class="font-extrabold text-base">📱 Table QR Codes</h3><button onclick="window.print()" class="bg-[#1C130E] text-amber-400 px-3 py-1.5 rounded-xl text-xs font-bold">🖨️ Print QRs</button></div>
                <div class="grid grid-cols-2 sm:grid-cols-3 gap-3 max-h-96 overflow-y-auto p-1">
                  ${Array.from({length: cafe.total_tables}, (_, i) => i + 1).map(num => {
                    const tableUrl = `${window.location.origin}${window.location.pathname}?cafe=${cafe.slug}&table=${num}`;
                    const qrImg = 'https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=' + encodeURIComponent(tableUrl);
                    return `<div class="border-2 border-stone-800 rounded-2xl p-2.5 text-center bg-[#FAF6F0]"><p class="text-[10px] font-extrabold text-amber-800 uppercase truncate">${cafe.cafe_name}</p><img src="${qrImg}" class="w-24 h-24 mx-auto my-1.5 rounded" /><p class="text-xs font-extrabold">TABLE #${num}</p></div>`;
                  }).join('')}
                </div>
              </div>
            </div>
          `}
        </div>
        ${renderScannerModal()}
      `;
    }

    function addSingleDish() {
      const cat = (document.getElementById("singleCat").value || "Specials").trim();
      const name = (document.getElementById("singleName").value || "").trim();
      const price = parseInt(document.getElementById("singlePrice").value) || 99;
      if (!name) return alert("Dish Name डालें!");
      currentCafe().menu.unshift({
        id: Date.now(),
        name,
        category: cat,
        price,
        is_veg: true,
        is_bestseller: false,
        in_stock: true,
        img: getAutoFoodImage(cat + " " + name)
      });
      saveSaaS();
      render();
    }

    function toggleStock(id) {
      const item = currentCafe().menu.find(x => x.id === id);
      if (item) { item.in_stock = !item.in_stock; saveSaaS(); render(); }
    }

    function updateItemPrice(id, val) {
      const item = currentCafe().menu.find(x => x.id === id);
      if (item) { item.price = Number(val) || item.price; saveSaaS(); render(); }
    }

    function deleteMenuItem(id) {
      currentCafe().menu = currentCafe().menu.filter(x => x.id !== id);
      saveSaaS();
      render();
    }

    function render() {
      const route = getRoute();
      const root = document.getElementById("app");
      if (route === "SUPERADMIN") root.innerHTML = renderSuperAdmin();
      else if (route === "KDS") root.innerHTML = renderKDS();
      else if (route === "OWNER") root.innerHTML = renderOwner();
      else root.innerHTML = renderCustomer();
    }

    window.addEventListener("hashchange", () => { pinUnlocked = false; render(); });
    render();
  </script>
</body>
</html>
