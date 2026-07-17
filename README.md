# Maxfiy-Ma-lumotlar-Xazinasi-WeakMap-va-WeakSet-JavaScript**JavaScript-da Fibonachchi ketma-ketligini generatsiya qilish uchun ham Iterator (klassik uslub), ham Generator (function*) yondashuvlaridan foydalanishimiz mumkin.
JavaScript-da Proxy (vositachi) va Reflect (aks ettirish) API-laridan foydalanib, obyekt ustida qat'iy nazorat (validatsiya) o'rnatishimiz mumkin. Bu usul ma'lumotlarni noto'g'ri kiritishdan himoya qilish uchun juda qulay.

Siz so'ragan barcha talablar (validatsiya qoidalari, Reflect metodlari, try/catch sinovlari) bajarilgan to'liq kod namunasi:

JavaScript
// 1. ASOSIY TARGET OBYEKT (Asl ma'lumotlar saqlanadigan joy)
const targetUser = {
  name: "Jasur",
  age: 20
};

// 2. HANDLER (Tuzoqlarni boshqaruvchi obyekt)
const handler = {
  // GET Tuzog'i: Ma'lumot o'qilayotganda ishga tushadi
  get(target, prop) {
    if (prop in target) {
      return Reflect.get(target, prop);
    }
    // Mavjud bo'lmagan xususiyat uchun maxsus xabar qaytaramiz
    return `Xato: "${prop}" xususiyati obyektda mavjud emas!`;
  },

  // SET Tuzog'i: Yangi ma'lumot yozilayotganda ishga tushadi
  set(target, prop, value) {
    // Ism validatsiyasi (string bo'lishi va kamida 2 ta harfdan iborat bo'lishi shart)
    if (prop === "name") {
      if (typeof value !== "string" || value.trim().length < 2) {
        throw new TypeError("Ism kamida 2 ta harfdan iborat matn (string) bo'lishi kerak!");
      }
    }

    // Yosh validatsiyasi (musbat butun son va 0-120 oralig'ida bo'lishi shart)
    if (prop === "age") {
      if (typeof value !== "number" || !Number.isInteger(value) || value < 0 || value > 120) {
        throw new TypeError("Yosh 0 dan 120 gacha bo'lgan butun son bo'lishi kerak!");
      }
    }

    // Agar barcha tekshiruvlardan o'tsa, qiymatni yozamiz
    return Reflect.set(target, prop, value);
  }
};

// 3. PROXY OBYEKTINI YARATISH
const userProxy = new Proxy(targetUser, handler);


// ==========================================
// 🧪 SINOV (TESTING) QISMI
// ==========================================

console.log("=== 1. TO'G'RI QIYMATLAR SINOVI (Kamida 3 ta) ===");

try {
  // Sinov 1: To'g'ri ism kiritish
  userProxy.name = "Alisher";
  console.log(`✅ Ism o'zgartirildi: ${userProxy.name}`);

  // Sinov 2: To'g'ri yosh kiritish
  userProxy.age = 25;
  console.log(`✅ Yosh o'zgartirildi: ${userProxy.age}`);

  // Sinov 3: Mavjud bo'lgan xususiyatni o'qish
  console.log(`✅ Obyekt holati: Ismi - ${userProxy.name}, Yoshi - ${userProxy.age}`);
} catch (error) {
  console.error(`❌ Kutilmagan xato: ${error.message}`);
}


console.log("\n=== 2. NOTO'G'RI QIYMATLAR SINOVI (Kamida 3 ta try/catch bilan) ===");

// Noto'g'ri Sinov 1: Ism o'rniga raqam berish
try {
  console.log("Harakat: Ismga son berish (123)...");
  userProxy.name = 123; // Xatolik berishi kerak
} catch (error) {
  console.log(`💡 Tutib qolingan xato: ${error.message}`);
}

// Noto'g'ri Sinov 2: Yoshga manfiy son berish
try {
  console.log("\nHarakat: Yoshga manfiy son berish (-5)...");
  userProxy.age = -5; // Xatolik berishi kerak
} catch (error) {
  console.log(`💡 Tutib qolingan xato: ${error.message}`);
}

// Noto'g'ri Sinov 3: Yoshga string tipli qiymat berish
try {
  console.log("\nHarakat: Yoshga yozuv berish ('yigirma')...");
  userProxy.age = "yigirma"; // Xatolik berishi kerak
} catch (error) {
  console.log(`💡 Tutib qolingan xato: ${error.message}`);
}


console.log("\n=== 3. MAVJUD BO'LMAGAN XUSUSIYAT SINOVI ===");
// Obyektda yo'q xususiyatni chaqirib ko'ramiz
console.log(userProxy.email); 
console.log(userProxy.location);
🔑 Asosiy tushunchalar:
Reflect.get() va Reflect.set(): Proxy ichida maqsadli obyektning asl xatti-harakatini (qiymat olish/yozish) xavfsiz va standart usulda bajarish uchun ishlatiladi.

TypeError: Kiritilgan ma'lumot turi yoki formati biz o'rnatgan qoidalarga mos kelmaganida dastur ishini buzmasdan, xatoni dasturchiga chiroyli yetkazish uchun ishlatildi.
