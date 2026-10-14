# Node-RED HVAC és napi összefoglaló – archivált rendszer

Ez a repository egy 2026. október 14-én archivált, otthoni Node-RED-rendszer két összetartozó flow-ját őrzi meg:

- `HVAC Core V1.0`: ötzónás padlófűtés, kazán, szivattyú és Mitsubishi/MELCloud klíma vezérlése; iHost-, IKEA DIRIGERA-, Open-Meteo- és UIBUILDER-integrációval.
- `Datacollecting & Summary mailer`: ötpercenkénti MySQL-pillanatképek és minden reggel 08:05-kor összeállított HTML-összefoglaló időjárással, EUR/HUF árfolyammal, névnappal, menüvel és hulladékszállítási emlékeztetővel.

A közzétett exportok anonimizáltak. Nem tartalmazzák az eredeti jelszavakat, hozzáférési tokeneket, e-mail-címeket, belső IP-címeket, pontos földrajzi helyet, Google Sheet-azonosítókat, MELCloud-azonosítókat vagy valós eszközazonosítókat. Emiatt import után kötelező a helyi konfiguráció beállítása.

## Tartalom

```text
flows/
  hvac-core-v1.0.public.json
  datacollecting-summarymailer.public.json
docs/
  ARCHITECTURE_HU.md
  INSTALLATION_HU.md
  PRIVACY_AND_SECURITY_HU.md
sql/
  schema.sql
```

## Függőségek

Az exportokban rögzített verziók:

| Csomag | Verzió | Szerep |
|---|---:|---|
| `node-red-contrib-ihost` | 1.3.0 | Sonoff iHost eszközök és relék |
| `node-red-contrib-dirigera` | 1.3.0 | IKEA DIRIGERA fények, vezérlők és szenzorok |
| `node-red-contrib-uibuilder` | 7.7.4 | NSPanel/asztali dashboard adatkapcsolata |
| `node-red-contrib-melcloud` | 1.6.0 | Mitsubishi klíma lekérdezése és vezérlése |
| `node-red-node-email` | 5.2.5 | HTML e-mail küldése |
| `node-red-node-mysql` | 3.0.3 | pillanatképek tárolása és lekérdezése |
| `node-red-contrib-google-sheets` | 1.1.2 | menü és hulladéknaptár beolvasása |

## Gyors indulás

1. Készíts mentést a jelenlegi Node-RED-ről.
2. Telepítsd a fenti node-csomagokat a Palette Managerben.
3. Hozd létre az adatbázist a `sql/schema.sql` alapján.
4. Importáld először a HVAC, majd az adatgyűjtő flow-t.
5. Kövesd a [telepítési és konfigurációs útmutatót](docs/INSTALLATION_HU.md).
6. Először letiltott kimenetekkel vagy tesztkörnyezetben ellenőrizd a döntéseket.
7. Csak igazolt eszköz-visszajelzések és gépészeti védelmek mellett engedélyezd az éles vezérlést.

## Fontos biztonsági megjegyzés

Ez archív referencia, nem kulcsrakész fűtéstechnikai termék. Kazánt, szivattyút, szelepeket és klímát vezérel; hibás eszközazonosító vagy bekötés anyagi kárt okozhat. A fizikai túlmelegedés-, fagy-, áramlás- és nyomásvédelmet nem helyettesíti. Import után a `HVAC CONTROL ENABLE` ágat csak ellenőrzött szárazpróba után használd.

## Működési összefoglaló

- A HVAC fő ciklusa 300 másodpercenként fut, a MELCloud állapot lekérdezése 600 másodpercenként.
- A rendszer időjárásfüggően választ padlófűtés és klímafűtés között.
- Az ötzónás padlófűtés külön célértékeket, éjszakai csökkentést, időjárás-kompenzációt és hiszterézist használ.
- Hűtés csak június–augusztus között engedélyezett, külön be- és kikapcsolási küszöbbel.
- A dashboard-adatok a `global.dash_state` objektumba kerülnek; a két UIBUILDER kliens kérésre visszakaphatja az utolsó állapotot.
- Az adatgyűjtő legfeljebb ötpercenként ír időjárás-, hőmérséklet-, klíma-, fűtés- és redőny-pillanatképet.
- A reggeli e-mail az előző 24 óra átlagait, minimumait, maximumait és becsült fűtési időit foglalja össze.

Részletes adatfolyam: [ARCHITECTURE_HU.md](docs/ARCHITECTURE_HU.md).

## Állapot és korlátok

- A flow-k egy konkrét otthon eszközmodelljére készültek; a példaként szereplő szobák és csatornák átnevezhetők.
- Az anonimizált hardverazonosítók szándékosan nem működnek valós rendszerrel.
- Az SQL-séma a flow által használt mezők alapján készült, ezért meglévő adatbázisra alkalmazás előtt ellenőrizendő.
- Az Open-Meteo, az árfolyam- és névnap-szolgáltatás internetkapcsolatot igényel.
- A Google Sheets- és SMTP-hitelesítést a Node-RED credential store-ban kell megadni; a repository nem tartalmaz credential fájlt.

## Licenc

A repository jelenleg nem ad külön nyílt forráskódú licencet. A kód megtekinthető és archiválási céllal közzétett; újrafelhasználáshoz a tulajdonos engedélye szükséges.
