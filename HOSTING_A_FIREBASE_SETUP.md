# 🚀 Kompletní průvodce: Firebase a Hosting pro Solární Kotelnu

Systém funguje **100% přes Google Firebase**:
- Živý stav a řízení přes Firebase Realtime Database.
- Dlouhodobá historie teplot ukládána každých **20 sekund přímo do Firebase DB** (ThingSpeak byl kompletně odstraněn).
- Administrátorské heslo je uloženo **přímo v databázi Firebase** pod `/kotelna/auth` (v kódu stránky není žádné heslo, nikdo ho v prohlížeči nevyčte).
- Web je hostován na **Firebase Hosting** (`https://kotelna-fa0d8.web.app`).

---

## 1. Správa hesla v Firebase Console

Administrátorské heslo pro ovládání kotelny **není v kódu HTML ani JS**, ale přímo ve vaší Firebase Realtime Databázi:

1. Otevřete [Firebase Console](https://console.firebase.google.com/) -> Váš projekt **kotelna-fa0d8**.
2. V menu vlevo klikněte na **Build** -> **Realtime Database**.
3. V datovém stromu uvidíte uzel:
   ```json
   "kotelna": {
     "auth": {
       "password": "kotelna123"
     }
   }
   ```
4. **Změna hesla:** Kdykoliv klikněte na hodnotu `"kotelna123"` a přepište ji na své vlastní heslo. Změna se okamžitě projeví na webu bez nutnosti cokoliv znovu nasazovat!

---

## 2. Struktura dat v Firebase Realtime Database

- **/kotelna/status** (ESP32 odesílá každé 2 s):
  - `pec`, `horni_n`, `stredni_n`, `dolni_n`, `horni_s`, `dolni_s`, `solarni`: Aktuální teploty (°C).
  - `cerpadlo`: boolean (stav sepnutí relé čerpadla).
  - `spirala`: "horni" / "spodni" (aktivní topná zóna).
  - `nocni`: boolean (nízký tarif HDO).
  - `uptime`: sekundy od spuštění ESP32.
  - `online`: boolean (heartbeat).

- **/kotelna/history/<timestamp>** (ESP32 ukládá každých 20 s):
  - Záznamy pro interaktivní graf na webu.
  - Ukládá se čas (Unix timestamp) a všech 7 teplot.

- **/kotelna/control** (ovládání z webu, ESP32 přijímá v reálném čase):
  - `autoswitch`: true/false (automatické přepínání spirál na 45 °C).
  - `pumpAlways`: true/false (vynucený trvalý chod čerpadla).
  - `spir`: "horni" / "spodni" (manuální volba ventilu, když je automatika vypnutá).
  - `teplotni_rozdil_min`: 3–20 °C (vypínací diference).
  - `teplotni_rozdil_max`: 20–50 °C (spínací diference).

- **/kotelna/auth**:
  - `password`: Administrátorské heslo.

---

## 3. Nahrání firmwaru do ESP32 (`ESP32_solary.ino`)

1. V Arduino IDE otevřete `ESP32_solary.ino`.
2. Zkontrolujte, zda máte nainstalovanou knihovnu **`Firebase ESP32 Client`** (od Mobizt).
3. V souboru doplňte své přihlašovací údaje k WiFi a Firebase:
   ```cpp
   const char* ssid = "Vase_SSID";
   const char* password = "Vase_Heslo";

   #define FIREBASE_HOST "https://kotelna-fa0d8-default-rtdb.europe-west1.firebasedatabase.app"
   #define FIREBASE_AUTH "VAS_DATABASE_SECRET_NEBO_API_KEY"
   ```
   *(Tajný klíč databáze: Firebase Console -> Ozubené kolo -> Nastavení projektu -> Servisní účty -> Tajné klíče databáze).*
4. Nahrajte firmware do ESP32.

---

## 4. Nasazení nové verze webu na Firebase Hosting

Kdykoliv upravíte web, stačí v kořenové složce projektu spustit:

```powershell
npx firebase-tools deploy
```

Web je dostupný na adrese:
👉 **https://kotelna-fa0d8.web.app**
