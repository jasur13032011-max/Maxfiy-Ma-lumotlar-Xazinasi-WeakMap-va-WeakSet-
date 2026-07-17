# Maxfiy-Ma-lumotlar-Xazinasi-WeakMap-va-WeakSet-JavaScript**JavaScript-da Fibonachchi ketma-ketligini generatsiya qilish uchun ham Iterator (klassik uslub), ham Generator (function*) yondashuvlaridan foydalanishimiz mumkin.

Siz so'ragan barcha talablarga (jumladan yield* va cheksiz generatorni to'xtatish shartiga) muvofiq tayyorlangan to'liq kod namunasi:

JavaScript
// ==========================================
// 1. CLASSIC ITERATOR: fibonacciIterator
// ==========================================
const fibonacciIterator = {
  [Symbol.iterator]() {
    let prev = 0;
    let curr = 1;
    let step = 0;
    const maxSteps = 10; // To'xtash sharti: dastlabki 10 ta sonni chiqaradi

    return {
      next() {
        if (step < maxSteps) {
          const value = prev;
          // Keyingi Fibonachchi sonini hisoblash
          const nextValue = prev + curr;
          prev = curr;
          curr = nextValue;
          step++;
          return { value: value, done: false };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};


// ==========================================
// 2. GENERATORS: function* VA yield/yield*
// ==========================================

// Birinchi generator: Cheksiz Fibonachchi generatori (to'g'ri to'xtash sharti bilan)
function* infiniteFibonacciGenerator(limit) {
  let prev = 0;
  let curr = 1;
  while (true) {
    // To'xtash sharti: Agar limit berilgan bo'lsa va undan oshsa, generatsiyani tugatadi
    if (limit !== undefined && prev > limit) {
      return; // done: true qaytaradi
    }
    yield prev; // done: false bilan qiymat qaytaradi
    const nextVal = prev + curr;
    prev = curr;
    curr = nextVal;
  }
}

// Ikkinchi generator: Boshqa generatordan ma'lumotlarni delegatsiya qiluvchi (yield*) generator
function* delegatedFibonacciGenerator() {
  // Limit 50 gacha bo'lgan Fibonachchi sonlarini infiniteFibonacciGenerator-dan oladi
  yield* infiniteFibonacciGenerator(50); 
}


// ==========================================
// 3. AMALDA SINASH (for...of)
// ==========================================

console.log("=== 1. Classic Iterator (Dastlabki 10 ta son) ===");
for (const num of fibonacciIterator) {
  console.log(num);
}

console.log("\n=== 2. Cheksiz Generator (To'xtash sharti limit: 20 gacha) ===");
// Bu yerda generator cheksiz aylanish o'rniga limit (20) ga yetganda to'xtaydi
for (const num of infiniteFibonacciGenerator(20)) {
  console.log(num);
}

console.log("\n=== 3. Delegated Generator (yield* yordamida limit: 50 gacha) ===");
for (const num of delegatedFibonacciGenerator()) {
  console.log(num);
}
🔍 Kod tushuntirishi:
Symbol.iterator va next(): fibonacciIterator obyektimiz ushbu metodlar orqali standart JavaScript iteratsiya protokoli asosida qurildi. Har bir qadamda done: false va eng oxirida done: true qaytariladi.

yield* delegatsiyasi: delegatedFibonacciGenerator ichida yield* ishlatilib, navbatdagi qiymatlarni ishlab chiqarish vazifasi to'g'ridan-to'g'ri infiniteFibonacciGenerator zimmasiga yuklatildi.

To'g'ri to'xtash sharti: Cheksiz generator while (true) tsiklida ishlasa-da, parametr sifatida uzatilgan limit qiymatiga qarab, if (prev > limit) return; bloki yordamida tsiklni xavfsiz va to'g'ri yakunlaydi.**
