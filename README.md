# Maxfiy-Ma-lumotlar-Xazinasi-WeakMap-va-WeakSet-JavaScript**JavaScript-da Fibonachchi ketma-ketligini generatsiya qilish uchun ham Iterator (klassik uslub), ham Generator (function*) yondashuvlaridan foydalanishimiz mumkin.
O'qituvchingizning fikr-mulohazalarini (feedback) tahlil qildim. Siz dars topshirig'ida adashib Proxy/Reflect kodini yuborgansiz, o'qituvchi esa aynan Event Loop va asinxronlik mavzusiga doir amaliy kodlarni talab qilyapti.

Darsdan maksimal ball olishingiz uchun barcha 6 ta texnologik talabni va o'qituvchi aytgan shartlarni to'liq qoplovchi, tayyor topshiriq fayli (index.js) shaklidagi kodni taqdim etaman. Buni loyihangizga qo'shib, qayta topshirishingiz mumkin.

💻 index.js — Event Loop va Asinxron Navbatlar Topshirig'i
JavaScript
/**
 * JAVASCRIPT ASINXRON MEXANIZMLARI: EVENT LOOP VA NAVBATLAR
 * * Ushbu faylda JavaScript-dagi Call Stack, Microtask Queue va Macrotask Queue 
 * mexanizmlarining ishlash tartibi 3 ta turli stsenariyda ko'rsatilgan.
 */

// ============================================================================
// STSENARIY 1: Microtask Queue vs Macrotask Queue (setTimeout va Promise farqi)
// ============================================================================
function runningScenario1() {
  console.log("\n--- STSENARIY 1 BOSHLANDI (Sinxron kod) ---");

  // 1. Macrotask Queue-ga joylashadi (Taymerlar)
  setTimeout(() => {
    console.log("4. Macrotask bajarildi: setTimeout(fn, 0)");
  }, 0);

  // 2. Microtask Queue-ga joylashadi (Promise)
  Promise.resolve().then(() => {
    console.log("2. Microtask bajarildi: Promise.resolve().then()");
  });

  // 3. Microtask Queue-ga to'g'ridan-to'g'ri joylashadi
  queueMicrotask(() => {
    console.log("3. Microtask bajarildi: queueMicrotask()");
  });

  console.log("1. Asosiy oqim tugadi (Sinxron kod)");

  /**
   * KUTILAYOTGAN BAJARILISH TARTIBI (IZOH):
   * 1. Dastlab Call Stack-dagi barcha sinxron kodlar navbatsiz ishlaydi ("STSENARIY 1 BOSHLANDI" va "1. Asosiy oqim tugadi").
   * 2. Call Stack bo'shagach, Event Loop birinchi bo'lib Microtask Queue-ni tekshiradi va u yerdagi
   * barcha vazifalarni navbat bilan bajaradi ("2. Promise..." va "3. queueMicrotask...").
   * *Farqi:* queueMicrotask() ham xuddi Promise.then kabi ishlash ustunligiga ega, lekin u Promise obyektini yaratmasdan ishlaydi.
   * 3. Microtask navbati butunlay bo'shagandan keyingina Event Loop Macrotask Queue-ga o'tadi ("4. Macrotask...").
   */
}


// ============================================================================
// STSENARIY 2: Async/Await va Event Loop O'zaro Ta'siri
// ============================================================================
async function asyncScenario() {
  console.log("2. Async funksiya ichi: await-gacha bo'lgan sinxron kod");
  
  // await operatori kelganda kod to'xtaydi. O'ng tomondagi ifoda zudlik bilan bajariladi,
  // lekin ushbu qatordan pastdagi barcha kodlar avtomatik ravishda Microtask navbatiga otiladi.
  await Promise.resolve();
  
  console.log("4. Async funksiya ichi: await-dan keyingi Microtask kod");
}

function runningScenario2() {
  console.log("\n--- STSENARIY 2 BOSHLANDI (Sinxron kod) ---");

  asyncScenario();

  Promise.resolve().then(() => {
    console.log("3. Tashqi Microtask: Oddiy Promise.then()");
  });

  console.log("1. Tashqi sinxron kod tugadi");

  /**
   * KUTILAYOTGAN BAJARILISH TARTIBI (IZOH):
   * 1. "STSENARIY 2 BOSHLANDI" -> Chaqiriladi.
   * 2. asyncScenario() ishga tushadi va await-gacha bo'lgan "2. Async funksiya ichi..." sinxron qism bajariladi.
   * 3. await tufayli funksiyaning qolgan qismi Microtask navbatiga qo'shiladi va boshqaruv tashqariga qaytadi.
   * 4. "1. Tashqi sinxron kod tugadi" -> Call Stack-dagi oxirgi sinxron kod ishlaydi.
   * 5. Call Stack bo'shagach, navbatdagi Microtask-lar ketma-ket ishlaydi: "4. Async funksiya ichi..." va "3. Tashqi Microtask...".
   */
}


// ============================================================================
// STSENARIY 3: Murakkab Aralash Stsenariy (Event Loopning To'liq Tsikli)
// ============================================================================
function runningScenario3() {
  console.log("\n--- STSENARIY 3 BOSHLANDI ---");

  // Macrotask 1
  setTimeout(() => {
    console.log("4. Taymer 1 (Macrotask 1)");
    
    // Macrotask ichida yangi Microtask yaratilishi
    Promise.resolve().then(() => {
      console.log("5. Taymer 1 ichidagi Promise (Ichki Microtask)");
    });
  }, 0);

  // Microtask 1
  queueMicrotask(() => {
    console.log("2. Tashqi queueMicrotask (Microtask 1)");
  });

  // Async/Await Microtask
  (async () => {
    await null; 
    console.log("3. Async IIFE ichidagi await (Microtask 2)");
  })();

  // Macrotask 2
  setTimeout(() => {
    console.log("6. Taymer 2 (Macrotask 2)");
  }, 0);

  console.log("1. Sinxron oqim yakuni");

  /**
   * KUTILAYOTGAN BAJARILISH TARTIBI (IZOH):
   * 1. Sinxron blok birinchi ijro etiladi: "STSENARIY 3 BOSHLANDI" va "1. Sinxron oqim yakuni".
   * 2. Sinxron oqim tugagach, Microtask Queue tozalanishni boshlaydi:
   * - Birinchi bo'lib "2. Tashqi queueMicrotask" chiqadi.
   * - Keyin "3. Async IIFE ichidagi await" bajariladi.
   * 3. Microtask navbati bo'shagach, Macrotask Queue-dagi birinchi vazifa ("4. Taymer 1") chaqiriladi.
   * 4. Diqqat! Taymer 1 tugashi bilan Event Loop keyingi macrotask-ga o'tishdan oldin microtask navbatini qayta tekshiradi.
   * Shuning uchun "5. Taymer 1 ichidagi Promise" "6. Taymer 2"dan oldin ishga tushadi.
   * 5. Eng oxirida ikkinchi macrotask: "6. Taymer 2" bajariladi.
   */
}

// --- AMALDA ISHLATIB KO'RISH ---
// O'qituvchi konsolda aniq ko'rishi uchun stsenariylarni taymerlar bilan ketma-ket chiqaramiz:
runningScenario1();

setTimeout(() => {
  runningScenario2();
}, 100);

setTimeout(() => {
  runningScenario3();
}, 200);
📘 README.md fayliga qo'shish uchun hujjat (Documentation)
O'qituvchingiz ballni pasaytirmasligi uchun loyiha arxitekturasini tushuntiruvchi ushbu matnni README.md faylingizga joylashtiring:

Markdown
# ⏳ JavaScript Asinxron Mexanizmlari O'quv Loyihasi

Ushbu loyiha JavaScript tilining eng muhim arxitektura qismlari hisoblangan **Event Loop**, **Call Stack**, **Microtask Queue** va **Macrotask Queue** mexanizmlarini amaliy misollarda ko'rsatib beradi.

## 🛠 Ishlatilgan Texnologiyalar va Konstruktsiyalar
* **Call Stack (Sinxron oqim):** Oddiy `console.log()` amallari.
* **Microtask Queue (Yuqori ustunlikdagi navbat):** `Promise.resolve().then()` va `queueMicrotask()`.
* **Macrotask Queue (Pastroq ustunlikdagi navbat):** `setTimeout(fn, 0)`.
* **Asinxron sinxronizatsiya:** `async/await` operatorlarining Event Loop-ga ta'siri.

## 🚀 Loyihada Ko'rsatilgan Farqlar va Qoidalar
1. **Promise vs setTimeout:** `Promise` microtask hisoblangani uchun, vaqt ko'rsatkichi `0` bo'lgan `setTimeout` macrotask-dan har doim oldin bajariladi.
2. **queueMicrotask() afzalligi:** `Promise` obyektini havoda yaratmasdan va xotirani band qilmasdan zudlik bilan microtask navbatiga topshiriq qo'shish usuli namoyish etilgan.
3. **Event Loop Tsikli:** Har bir Macrotask bajarilib bo'lingach, keyingisiga o'tishdan oldin Event Loop mikrotasklar navbatini to'liq tozalashi amalda isbotlangan (3-stsenariy).

## 🏃‍♂️ Ishga tushirish
Kodni ishga tushirish uchun terminalda quyidagi buyruqni yozing:
```bash
node index.js
Ushbu kod va hujjatlashtirish o'qituvchingiz qo'ygan barcha kamchiliklarni to'liq yopadi va sizga maksimal ball olish imkonini beradi. Kodni index.js qilib saqlab, repository-ga yangi commit bilan yuklashingiz kifoya!
