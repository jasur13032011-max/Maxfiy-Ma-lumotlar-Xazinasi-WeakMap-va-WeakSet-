# Maxfiy-Ma-lumotlar-Xazinasi-WeakMap-va-WeakSet-JavaScript-dagi WeakMap va WeakSet obyektlari xotira bilan samarali ishlash (Memory Management) va ma'lumotlar xavfsizligini ta'minlash uchun juda qulay. Ularning asosiy farqi — faqat obyektlarni kalit (key) yoki qiymat (value) sifatida qabul qiladi va agar obyektga boshqa hech qayerdan murojaat qolmasa, u "Garbage Collector" (axlat yig'uvchi) tomonidan xotiradan avtomatik o'chiriladi.

Siz so'ragan barcha shartlar bajarilgan to'liq va tushunarli kod namunasi quyidagicha:

JavaScript
// --- 1. MA'LUMOTLAR OMBORINI YARATISH ---
// Tokenlarni saqlash uchun WeakMap
const userTokens = new WeakMap();

// Faol sessiyalarni kuzatish uchun WeakSet
const activeSessions = new WeakSet();


// --- 2. FOYDALANUVCHILARNI YARATISH ---
// Eslatma: WeakMap/WeakSet faqat obyektlar bilan ishlaydi
let user1 = { id: 101, username: "Jasur" };
let user2 = { id: 102, username: "Laylo" };


// --- 3. TOKЕN VA SESSIYALARNI O'RNATISH (Simulyatsiya) ---
// Foydalanuvchilarga token biriktiramiz
userTokens.set(user1, "JWT_TOKEN_JASUR_998877");
userTokens.set(user2, "JWT_TOKEN_LAYLO_112233");

// Foydalanuvchilarni faol sessiyalar ro'yxatiga qo'shamiz
activeSessions.add(user1);
activeSessions.add(user2);


// --- 4. FUNKSIYALARNI YARATISH ---

/**
 * Foydalanuvchining tokenini olib beradi
 * @param {Object} user 
 * @returns {string|undefined}
 */
function getToken(user) {
    return userTokens.get(user);
}

/**
 * Foydalanuvchi faol sessiyadami yoki yo'qligini tekshiradi
 * @param {Object} user 
 * @returns {boolean}
 */
function isActive(user) {
    return activeSessions.has(user);
}


// --- 5. NATIJALARNI KONSOLGA CHIQARISH ---

console.log("=================== FOYDALANUVCHI 1 ===================");
console.log(`Foydalanuvchi: ${user1.username}`);
console.log(`Tokeni: ${getToken(user1)}`);
console.log(`Sessiya faolmi?: ${isActive(user1) ? "Ha ✅" : "Yo'q ❌"}`);

console.log("\n=================== FOYDALANUVCHI 2 ===================");
console.log(`Foydalanuvchi: ${user2.username}`);
console.log(`Tokeni: ${getToken(user2)}`);
console.log(`Sessiya faolmi?: ${isActive(user2) ? "Ha ✅" : "Yo'q ❌"}`);


// --- 6. SESSIYANI YAKUNLASH VA O'CHIRISH (SINOV) ---
console.log("\n=================== TIZIMDAN CHIQISH SINOVI ===================");
console.log(`${user1.username} tizimdan chiqmoqda...`);

// Jasurning faol sessiyasini o'chiramiz
activeSessions.delete(user1);

console.log(`${user1.username} faolmi?: ${isActive(user1) ? "Ha ✅" : "Yo'q ❌"}`);
console.log(`${user2.username} faolmi?: ${isActive(user2) ? "Ha ✅" : "Yo'q ❌"}`);
💡 Nima uchun aynan WeakMap va WeakSet?
Agar loyihamizda, masalan, user1 = null; qilib foydalanuvchini butunlay o'chirib yuborsak, oddiy Map yoki Set ishlatilganda bu foydalanuvchi xotirada baribir saqlanib qolaverardi (xotira to'lib ketishiga olib kelardi).

WeakMap va WeakSet ishlatilganda esa, JavaScript ushbu foydalanuvchiga tegishli token va sessiyalarni xotiradan (Garbage Collector yordamida) avtomatik va izsiz tozalab yuboradi.
