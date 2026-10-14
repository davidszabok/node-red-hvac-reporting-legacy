# Telepítés és konfiguráció

## 1. Előkészítés

1. Exportáld és mentsd el a működő Node-RED flow-kat és credential fájlokat.
2. Használj külön tesztpéldányt vagy tiltsd le a fizikai kimeneti node-okat.
3. Telepítsd a README-ben felsorolt node-csomagokat.
4. Hozd létre a MySQL/MariaDB adatbázist a `sql/schema.sql` futtatásával.

## 2. Importálás

1. Node-RED: **Menu → Import → Clipboard/File**.
2. Importáld a `flows/hvac-core-v1.0.public.json` fájlt új flow-ként.
3. Importáld a `flows/datacollecting-summarymailer.public.json` fájlt.
4. Még ne deployolj éles kimenetekkel.

## 3. Kötelezően cserélendő mintaértékek

| Minta | Hol található | Mit kell megadni |
|---|---|---|
| `192.0.2.10` | iHost és UIBUILDER szerkesztési URL | az iHost/Node-RED helyi címe |
| `192.0.2.20` | MySQL konfiguráció | az adatbázis-szerver helyi címe |
| `REPLACE_WITH_IHOST_TOKEN` | `api-server` config | iHost hozzáférési token |
| `00000000-0000-4000-8000-…` | device node-ok és Function térképek | valós iHost/DIRIGERA eszközazonosítók |
| `REPLACE_WITH_MELCLOUD_*` | MELCloud credential/device node-ok | fiók, building ID és device ID |
| `recipient…@example.com` | e-mail node | címzettek |
| `sender@example.com` | e-mail node | SMTP-felhasználó; a jelszót a credential mezőben add meg |
| `REPLACE_WITH_GOOGLE_SHEET_ID` | két GSheet node | a megfelelő munkafüzet azonosítója |
| `service-account@…` | Google auth config | a saját service account konfigurációja |
| `47.4979`, `19.0402` | Open-Meteo URL-ek | a kívánt hely koordinátái |

Az exportban több Function node saját eszköztérképet tartalmaz. Keresd meg az összes `00000000-0000-4000-8000-` és `0x000000000000` mintát, és ugyanazt a fizikai eszközt minden előfordulásban ugyanarra az azonosítóra cseréld.

## 4. iHost és relék

1. Állítsd be az `iHost (configure me)` konfigurációs node-ot.
2. A `get-devices`, `event-state` és `put-device-state` node-okban válaszd ki a valós eszközöket.
3. Ellenőrizd a `Home registry`, `Normalize Heating Sonoff Switches`, szobaszenzor-normalizáló és redőny-normalizáló Function node térképeit.
4. Ellenőrizd külön a kazán, szivattyú és minden zóna csatornaszámát.

## 5. DIRIGERA és MELCloud

1. Párosítsd a DIRIGERA konfigurációs node-ot a helyi hubbal.
2. Minden DIRIGERA node-ban válaszd ki újra a megfelelő lámpát, kontrollert vagy szenzort.
3. A MELCloud credential node-ban add meg a fiókot a Node-RED credential store használatával.
4. A két `melcloud-device` node-ban állítsd be ugyanazt a valós building/device párost, majd ellenőrizd a fűtési és hűtési mód kódjait.

## 6. MySQL, Google Sheets és e-mail

1. Futtasd a `sql/schema.sql` fájlt egy üres adatbázison.
2. Állítsd be a MySQL hostot, adatbázisnevet, felhasználót, jelszót és megfelelő jogosultságokat.
3. A Google service accountnak csak olvasási hozzáférést adj a használt munkafüzethez.
4. Az eredeti kialakítás a `Today!A1` cellából menüt, a `Today!A2` cellából hulladék-emlékeztetőt olvas; szükség esetén módosítsd a két GSheet node-ot.
5. Állítsd be az SMTP szervert, portot, TLS-t, felhasználót, jelszót és a címzetteket.
6. Először a `Preview HTML` debug node-on ellenőrizd az üzenetet, és csak utána engedélyezd az e-mail node-ot.

## 7. Biztonságos próba

1. Hagyd kikapcsolva a `HVAC CONTROL ENABLE` ágat.
2. Futtasd a bemeneti lekérdezéseket, és ellenőrizd a normalizált `hvac/state` és `hvac/zone-context` payloadokat.
3. Figyeld meg legalább egy teljes 300 másodperces és egy 600 másodperces ciklust.
4. A `Floor executor (dry run)` és debug node-ok alapján hasonlítsd össze a tervezett állapotokat a valós reléállapotokkal.
5. Teszteld a hiányzó szenzort, elérhetetlen klímát, sikertelen reléparancsot és Node-RED-újraindítást.
6. Csak ezután engedélyezd egyenként a fizikai végrehajtókat.

## 8. Visszaállítás

Hiba esetén tiltsd le a vezérlést, válaszd le a `put-device-state` és MELCloud végrehajtó node-okat, majd importáld vissza a telepítés előtti mentést. Az adatbázis-táblák törlése nem szükséges a flow visszaállításához.
