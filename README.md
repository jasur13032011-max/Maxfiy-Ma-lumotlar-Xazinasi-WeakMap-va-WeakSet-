# Maxfiy-Ma-lumotlar-Xazinasi-WeakMap-va-WeakSet-JavaScript**JavaScript-da Fibonachchi ketma-ketligini generatsiya qilish uchun ham Iterator (klassik uslub), ham Generator (function*) yondashuvlaridan foydalanishimiz mumkin.
JavaScript-dagi Event Loop, Microtask Queue (V8 dvigatelidagi Promise va queueMicrotask navbati) hamda Macrotask Queue (Taymerlar va tarmoq so'rovlari navbati) asinxron kodlarning bajarilish tartibini belgilaydi.

Quyida so'ralgan barcha 3 ta stsenariy, ularning ishlash mantig'i va kutilgan natijalari izohlar bilan keltirilgan.

1-Stsenariy: Microtask vs Macrotask (setTimeout, Promise, queueMicrotask farqi)
Ushbu stsenariyda sinxron kod, setTimeout(fn, 0) (Macrotask), Promise.resolve().then() (Microtask) va queueMicrotask() larning o'zaro farqi va ustunlik darajasi ko'rsatilgan.

JavaScript
console.log("--- 1-STSENARIY BOSHLANDI (Sinxron 1) ---");

// Macrotask Queue-ga tushadi
setTimeout(() => {
  console.log("Macrotask: setTimeout(fn, 0) bajarildi");
}, 0);

// Microtask Queue-ga tushadi (Promise)
Promise.resolve().then(() => {
  console.log("Microtask: Promise.resolve().then() bajarildi");
});

// Microtask Queue-ga tushadi (To'g'ridan-to'g'ri navbat)
queueMicrotask(() => {
  console.log("Microtask: queueMicrotask(fn) bajarildi");
});

console.log("--- 1-STSENARIY TUGADI (Sinxron 2) ---");

/*
Kutilayotgan bajarilish tartibi:
1. Sinxron 1 -> Birinchi bo'lib asosiy oqimda (Call Stack) bajariladi.
2. Sinxron 2 -> Asosiy oqimdagi keyingi sinxron kod.
3. Microtask (Promise) -> Call Stack bo'shagach, birinchi navbatda Microtask navbati tekshiriladi.
4. Microtask (queueMicrotask) -> Microtask navbatidagi keyingi element (tartib bo'yicha).
5. Macrotask (setTimeout) -> Microtask navbati mutlaqo bo'shagandan keyingina Event Loop Macrotask navbatiga o'tadi.

Konsoldagi natija:
--- 1-STSENARIY BOSHLANDI (Sinxron 1) ---
--- 1-STSENARIY TUGADI (Sinxron 2) ---
Microtask: Promise.resolve().then() bajarildi
Microtask: queueMicrotask(fn) bajarildi
Macrotask: setTimeout(fn, 0) bajarildi
*/
2-Stsenariy: Async/Await va Event Loop o'zaro ta'siri
async/await ostida aslida Promise yotadi. await operatori kelgan joyda kod to'xtaydi va undan keyingi barcha qatorlar avtomatik ravishda Microtask navbatiga (.then() ichiga) joylashtiriladi.

JavaScript
async function asyncScenario() {
  console.log("Async: Funksiya ichidagi sinxron kod");
  
  // await iborasi o'ng tomonidagi kod sinxron bajariladi, 
  // lekin undan pastdagi qatorlar Microtask-ga otiladi.
  await Promise.resolve(); 
  
  console.log("Async: await-dan keyingi Microtask kod");
}

console.log("--- 2-STSENARIY BOSHLANDI ---");

asyncScenario();

Promise.resolve().then(() => {
  console.log("Oddiy Microtask: Tashqi Promise");
});

console.log("--- 2-STSENARIY TUGADI ---");

/*
Kutilayotgan bajarilish tartibi:
1. 2-STSENARIY BOSHLANDI -> Sinxron.
2. Async: Funksiya ichidagi sinxron kod -> Funksiya chaqirilganda await-gacha bo'lgan qism sinxron ishlaydi.
3. 2-STSENARIY TUGADI -> Call Stack-dagi oxirgi sinxron kod.
4. Async: await-dan keyingi Microtask kod -> await tufayli navbatga qo'yilgan birinchi microtask.
5. Oddiy Microtask: Tashqi Promise -> Navbatdagi ikkinchi microtask.

Konsoldagi natija:
--- 2-STSENARIY BOSHLANDI ---
Async: Funksiya ichidagi sinxron kod
--- 2-STSENARIY TUGADI ---
Async: await-dan keyingi Microtask kod
Oddiy Microtask: Tashqi Promise
*/
3-Stsenariy: Aralash (Murakkab) Stsenariy
Barcha konstruktsiyalar aralashib kelganda, Event Loop har safar bitta macrotask-dan keyin (yoki dastlabki sinxron kod tugagach) barcha yig'ilib qolgan microtask-larni to'liq bajarib tugatish qoidasiga bo'ysunadi.

JavaScript
console.log("1. Asosiy oqim (Sinxron boshlanishi)");

setTimeout(() => {
  console.log("2. Taymer 1 (Macrotask)");
  
  // Macrotask ichida yangi Microtask yaratish
  Promise.resolve().then(() => {
    console.log("3. Taymer 1 ichidagi Promise (Microtask)");
  });
}, 0);

queueMicrotask(() => {
  console.log("4. Tashqi queueMicrotask (Microtask)");
});

async function murakkabAsync() {
  await null; // Zudlik bilan hal bo'luvchi va'da (Microtask yaratadi)
  console.log("5. Async funksiya ichidagi await natijasi (Microtask)");
}
murakkabAsync();

setTimeout(() => {
  console.log("6. Taymer 2 (Macrotask)");
}, 0);

console.log("7. Asosiy oqim (Sinxron yakuni)");

/*
Kutilayotgan bajarilish tartibi va mantig'i:
1. Sinxron kodlar birinchi ishlaydi: "1" keyin "7". (Taymerlar va Microtasklar navbatlarga joylashdi).
2. Dastlabki sinxron blok tugagach, Microtask navbati tozalanishni boshlaydi:
   - "4. Tashqi queueMicrotask" bajariladi.
   - "5. Async funksiya ichidagi await natijasi" bajariladi.
3. Microtask navbati bo'shadi. Endi navbat Macrotask Queue-dagi birinchi elementga keladi:
   - "2. Taymer 1" ishga tushadi.
   - Uning ichidagi Promise zudlik bilan Microtask navbatiga yoziladi.
4. "2" tugashi bilan Event Loop yana Microtask navbatini tekshiradi (Taymer 2-ga o'tishdan oldin!):
   - "3. Taymer 1 ichidagi Promise" bajariladi.
5. Microtask navbati yana bo'sh. Macrotask navbatidagi keyingi elementga o'tiladi:
   - "6. Taymer 2" bajariladi.

Konsoldagi yakuniy natija:
1. Asosiy oqim (Sinxron boshlanishi)
7. Asosiy oqim (Sinxron yakuni)
4. Tashqi queueMicrotask (Microtask)
5. Async funksiya ichidagi await natijasi (Microtask)
2. Taymer 1 (Macrotask)
3. Taymer 1 ichidagi Promise (Microtask)
6. Taymer 2 (Macrotask)
*/
💡 Xulosa (Asosiy farqlar):
Sinxron kod: Har doim birinchi va uzilishlarsiz navbatsiz bajariladi.

Microtask (Promise.resolve().then, queueMicrotask, await): Ustunlik darajasi yuqori. Call stack bo'shashi bilan yoki har qanday macrotask tugashi bilan navbatdagi macrotask-ga o'tmasdan oldin barcha mavjud microtasklar bajarilib tugatiladi.

Macrotask (setTimeout): Ustunlik darajasi pastroq. Faqatgina barcha microtasklar bajarilib bo'linganidan keyin Event Loop tomonidan navbatma-navbat (bitta tsiklda bittadan) chaqiriladi.
