# N(app)ra Gépek — telepítési útmutató

Önálló webapp (PWA) QR-kódos, jelszavas belépéssel. Androidon és iPhone-on is a kezdőképernyőre tehető, és appként indul.
Az adatok a Firebase (Google) felhőben vannak, minden eszközön élőben frissülnek.

## Szerepek

| Szerep | Mit tud |
|---|---|
| **Aktiválásra vár** | Regisztrált, de még semmit nem lát. |
| **Beíró** | Mindent lát. Tankolást, hibát, menetlevelet, mozgást, óraállást **csak kérelemként** küldhet be. |
| **Parancsnok** | Mindent lát, minden kérelmet követ, ő is küldhet be kérelmet. Jóváhagyni nem tud. |
| **Technikus** | Jóváhagyja vagy elutasítja a kérelmeket („Jóváhagyva technikus által” pipa), és közvetlenül is írhat. |
| **Admin** | Mint a technikus, és ő aktiválja a regisztrációkat, ő osztja ki a szerepeket. |

A jogosultságokat a **szerver** ellenőrzi (`firestore.rules`). Egy beíró akkor sem tud közvetlenül adatot módosítani, ha a böngészőben trükközik.

---

## 1. Firebase-projekt (kb. 10 perc)

1. Nyisd meg: https://console.firebase.google.com → **Projekt hozzáadása** → név: `napra-gepek` → a Google Analyticset kikapcsolhatod.
2. Bal menü **Build → Authentication** → **Get started** → **Sign-in method** fül → **Email/Password** → **Enable** → Save.
   - (A felhasználónévből az app automatikusan képez belső címet, valódi e-mail nem kell.)
3. Bal menü **Build → Firestore Database** → **Create database** →
   - hely: **europe-west3 (Frankfurt)**,
   - **Start in production mode** → Create.
4. Firestore → **Rules** fül → töröld a tartalmát, másold be a `firestore.rules` fájl teljes tartalmát → **Publish**.
5. Fogaskerék → **Project settings** → lent **Your apps** → **</>** (Web) ikon → becenév: `napra-gepek` → **Register app**.
   Megjelenik egy `firebaseConfig = { ... }` blokk. Ezeket az értékeket másold be a **`firebase-config.js`** fájlba az `IDE_JON...` helyére.

## 2. Feltöltés GitHubra és Vercelre (kb. 10 perc)

Ugyanúgy, mint a common rail appnál:

1. GitHubon új repó, pl. `napra-gepek` (lehet **Private**).
2. Töltsd fel a mappa **összes** fájlját (Add file → Upload files): `index.html`, `firebase-config.js`, `manifest.webmanifest`, `sw.js`, a három `.png` ikon. A `firestore.rules` és a `README.md` maradhat a repóban, nem árt.
3. https://vercel.com → **Add New → Project** → válaszd a repót → Framework: **Other** → **Deploy**.
4. Kapsz egy címet, pl. `napra-gepek.vercel.app`.
5. Vissza a Firebase-be: **Authentication → Settings → Authorized domains → Add domain** → írd be a Vercel-címet (pl. `napra-gepek.vercel.app`).

## 3. Első belépés és admin jog

1. Nyisd meg a Vercel-címet → **Regisztráció** → töltsd ki → „Aktiválásra vár” képernyő jön.
2. Firebase konzol → **Firestore Database → Data** → `users` gyűjtemény → a te dokumentumod → a `role` mező értékét írd át `pending`-ről **`admin`**-ra.
3. Az app magától továbblép. Innentől minden mást az appból csinálsz.

## 4. QR-kód és telepítés a telefonokra

- Appban: **Kérelmek** menü alján **Felhasználók és szerepek** → **QR-kód** gomb → **Nyomtatás** (vagy bárki a saját profiljából is előhívhatja).
- **Android (Chrome):** QR beolvasása → megnyílik → ⋮ menü → **Alkalmazás telepítése** / **Hozzáadás a kezdőképernyőhöz**.
- **iPhone:** QR beolvasása a kamerával → **Safariban** nyisd meg → Megosztás gomb → **Főképernyőhöz adás**. (iPhone-on csak Safariból telepíthető.)

## 5. Napi üzemeltetés

- **Új ember:** QR → Regisztráció → te (admin) a Kérelmek menüben látod „új regisztráció” jelöléssel → szerep gomb: **Beíró / Parancsnok / Technikus**.
- **Kilépett ember:** szerepét állítsd **Tiltva** állapotra — azonnal semmit nem lát.
- **Elfelejtett jelszó:** Firebase konzol → Authentication → Users → az illető sorában ⋮ → **Delete account**, és Firestore → `users` → az ő dokumentumát is töröld. Utána újra regisztrál.
- **Jelszócsere:** bárki a saját profiljában (jobb felső név) megadhat új jelszót.

## 6. Költség

Az ingyenes **Spark** csomagban marad: napi 50 000 olvasás, 20 000 írás, 1 GB tárhely.
Az app ezért csak az utolsó **45 nap** adatait tölti be induláskor; a régebbi adatokat akkor kéri le, amikor egy gép lapját vagy egy régebbi hónapot megnyitsz.
25 fő, napi 3–4 megnyitás bőven belefér. Ha valaha elérnétek a keretet, aznap a mentés hibát jelez, és másnap újra működik.

A fotók (menetlevél, hiba) tömörítve, az adatbázisban tárolódnak (kb. 100–200 kB/kép), 1 GB-ba több ezer kép fér.

## 7. Fájlok

| Fájl | Mi ez |
|---|---|
| `index.html` | Maga az app |
| `firebase-config.js` | A Firebase-projekted azonosítói (ezt kell kitölteni) |
| `firestore.rules` | Szerveroldali jogosultságok — a Firebase konzolba kell bemásolni |
| `manifest.webmanifest`, `sw.js`, `*.png` | Telepíthető app (ikon, név, offline indulás) |

Ha a `firebase-config.js` nincs kitöltve, az app **demó módban** indul: minden működik, de az adatok csak az adott telefonon maradnak.
