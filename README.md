# Maxfiy-Ma-lumotlar-Xazinasi-WeakMap-va-WeakSet-JavaScript**JavaScript-da Fibonachchi ketma-ketligini generatsiya qilish uchun ham Iterator (klassik uslub), ham Generator (function*) yondashuvlaridan foydalanishimiz mumkin.
JavaScript-da Symbol — bu o'zgaruvchan bo'lmagan (immutable) va mutlaqo noyob (unikal) ma'lumot turi bo'lib, obyektrlarga xavfsiz va yashirin xususiyatlar qo'shish uchun ishlatiladi.

Siz so'ragan barcha shartlar (noyob IDlar, global reyestr, toPrimitive va iterator tizimli Symbollar) to'liq bajarilgan amaliy kod namunasi:

JavaScript
// ========================================================
// 1. NOYOQ IDENTIFIKATORLAR YARATISH (Symbol())
// ========================================================
const id1 = Symbol("user_id");
const id2 = Symbol("user_id"); // Bir xil tavsif berilgan taqdirda ham ular farq qiladi
const internalKey = Symbol("secret_key");

console.log("=== 1. Symbol Unikalligi Tekshiruvi ===");
// Ikki xil Symbol hech qachon o'zaro teng emasligini tekshiramiz
console.log(`id1 va id2 tengmi?: ${id1 === id2}`); // false
console.log(`id1 turi: ${typeof id1}`); // symbol


// ========================================================
// 2. GLOBAL REGISTRY (Symbol.for() va Symbol.keyFor())
// ========================================================
console.log("\n=== 2. Global Symbol Reyestri ===");

// Global registrdan Symbol yaratamiz (agar mavjud bo'lsa, o'shani oladi)
const globalApiKey = Symbol.for("api.config.key");
const globalApiKeyDuplicate = Symbol.for("api.config.key");

// Global reyestrdagi Symbollar bir-biriga teng bo'ladi
console.log(`Global kalitlar tengmi?: ${globalApiKey === globalApiKeyDuplicate}`); // true

// Symbol.keyFor() yordamida global Symbol kalit nomini olamiz
const keyName = Symbol.keyFor(globalApiKey);
console.log(`Global Symbolning kalit nomi (string): "${keyName}"`); // "api.config.key"


// ========================================================
// 3. OBYEKTDA [Symbol.toPrimitive] METODINI QO'LLASH
// ========================================================
console.log("\n=== 3. [Symbol.toPrimitive] (3 ta hint sinovi) ===");

const userAccount = {
  username: "Dilshod",
  balance: 350,
  
  // Obyektni har xil turlarga o'tkazish qoidasi
  [Symbol.toPrimitive](hint) {
    console.log(`-> Tizim so'ragan tur (hint): "${hint}"`);
    if (hint === "number") {
      return this.balance; // Raqam so'ralsa, balansni qaytaramiz
    }
    if (hint === "string") {
      return `Mijoz: ${this.username}`; // String so'ralsa, ismni qaytaramiz
    }
    // "default" holati (masalan, + operatori bilan ishlatilganda)
    return `${this.username} (${this.balance} USD)`;
  }
};

// Sinovlar:
console.log(+userAccount);                    // number hint -> 350
console.log(String(userAccount));             // string hint -> "Mijoz: Dilshod"
console.log("Hisob ma'lumoti: " + userAccount); // default hint -> "Dilshod (350 USD)"


// ========================================================
// 4. OBYEKTDA [Symbol.iterator] METODINI QO'LLASH
// ========================================================
console.log("\n=== 4. [Symbol.iterator] (for...of sinovi) ===");

const myCollection = {
  title: "Mening Kurslarim",
  items: ["Python", "JavaScript", "PostgreSQL", "React"],

  // Obyektni iteratsiya (aylantirish) qilish imkonini beruvchi metod
  [Symbol.iterator]() {
    let index = 0;
    const list = this.items;
    
    return {
      next() {
        if (index < list.length) {
          return { value: list[index++], done: false };
        } else {
          return { value: undefined, done: true };
        }
      }
    };
  }
};

// for...of tsikli yordamida obyektni to'g'ridan-to'g'ri aylanamiz
for (const item of myCollection) {
  console.log(`Kurs nomi: ${item}`);
}
📝 Muhim xulosalar va qoidalar:
Xavfsiz identifikatorlar: Symbol("user_id") orqali yaratilgan xususiyatlar obyektdagi tasodifiy ustma-ust tushishlardan (collision) himoya qiladi. Chunki har bir chaqiriq mutlaqo yangi manzil yaratadi.

Yashirinlik: Symbol kalitlari for...in yoki Object.keys() yordamida aylanilganda ko'rinmaydi. Ularni o'qish uchun maxsus Object.getOwnPropertySymbols() metodidan foydalanish lozim.

Tizimli Symbollar (Well-known Symbols): Symbol.toPrimitive va Symbol.iterator kabi Symbollar JavaScript dvigatelining obyektlar bilan qanday ishlashini (obyektni songa o'tkazish, for...ofda aylantirish) ichki tomondan o'zgartirish (override) imkonini beradi.
