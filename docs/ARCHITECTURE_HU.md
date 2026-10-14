# Architektúra és adatfolyam

## 1. HVAC Core V1.0

### Bemenetek

| Forrás | Gyakoriság | Felhasznált adatok |
|---|---:|---|
| Sonoff iHost | 300 s, egyes dashboard-adatoknál 30 s | szobahőmérséklet, páratartalom, fűtési relék, redőnyök, kapcsolók |
| Open-Meteo | 300 s / 600 s áganként | aktuális kültéri hőmérséklet, páratartalom, szél, előrejelzés, napkelte és napnyugta |
| MELCloud | 600 s | klíma tápállapot, mód, szobahőmérséklet, célérték és kommunikációs állapot |
| IKEA DIRIGERA | periodikus lekérdezés | lámpák, vezérlők, mozgásérzékelő és hubállapot |
| Node-RED idő | minden HVAC-ciklus | óra, dátum, hónap és szezonális engedélyezés |

### Döntési lánc

```text
iHost + időjárás + idő + MELCloud
                │
                ▼
          Join snapshot
                │
                ▼
     Normalize snapshot v2.1
                │
                ▼
   Zone Context Builder v3.2
                │
        ┌───────┴────────┐
        ▼                ▼
Floor Arbitration   AC Arbitration
        │                │
        ▼                ▼
 Zone planner       Climate planner
        │                │
        ▼                ▼
Floor APPLY/iHost   A/C APPLY/MELCloud
```

Az összevárás legfeljebb 15 másodpercet vár a hat bemenetre. Hiányos snapshot esetén figyelmeztetés készül, majd a normalizáló továbbadja az elérhető adatokat.

### Zónák és alapértékek

| Zóna | Nappali alapérték |
|---|---:|
| Nappali | 24,2 °C |
| Hálószoba | 23,0 °C |
| Gyerekszoba | 23,5 °C |
| Fürdőszoba | 23,5 °C |
| WC | 22,5 °C |

Az időjárás-kompenzáció körülbelül −0,3 és +0,8 °C között módosítja az értékeket. 22:00 és 05:00 között 1,5 °C éjszakai csökkentés érvényesül. Egy kikapcsolt zóna a célérték alatt 1,0 °C-kal indul, egy már aktív zóna pedig a teljes célérték eléréséig működik.

### Hőtermelő-választás

- Enyhe átmeneti időben a klímafűtés kaphat elsőbbséget, ha az előrejelzett maximum legalább 12 °C, de 24 °C alatti, és a pillanatnyi kültéri hőmérséklet 5 °C feletti.
- Hidegebb időben a padlófűtés az elsődleges.
- Nyári hűtés csak júniusban, júliusban és augusztusban engedélyezett, ha a pillanatnyi és az előrejelzett hőmérséklet is eléri a 24 °C-ot.
- A hűtés 24,4 °C-nál indul és 24,0 °C-nál áll le.

### Hibavédelem

- A szobaszenzorok utolsó értéke legfeljebb 15 percig használható.
- Ismeretlen zóna-, kazán- vagy szivattyú-visszajelzés nem tekinthető igazolt állapotnak.
- Az azonos végrehajtási parancsok rövid időn belüli ismétlése szűrhető.
- A Node-RED logikai védelem nem helyettesíti a gépészet önálló hardveres védelmét.

### Dashboard-adatág

A szenzorok, kapcsolók, redőnyök, fények, klíma és időjárás normalizált állapota a `global.dash_state` kulcson tárolódik. A `NsCompact V1.0` és `NS desktop` UIBUILDER node-hoz tartozó replay Function csak a kérő socketnek küldi vissza az utolsó állapotot. A UI külön repositoryban található.

## 2. Datacollecting & Summary mailer

### Pillanatképek

A HVAC flow `Datacollector for SQL` link kimenete kapcsolódik az adatgyűjtő `ML&Dashboard SQL` link bemenetéhez. A `SNAPSHOT WRITER v1.2` öt kimenetre bontja az állapotot, és legfeljebb ötpercenként ír:

- `snap_weather`: kültéri időjárás;
- `snap_sensors_th`: szobahőmérséklet és páratartalom;
- `snap_ac`: klímaállapot;
- `snap_heating`: kazán, szivattyú és fűtési körök;
- `snap_shutters`: redőnyhelyzet és motorállapot.

A `live_snapshot` tábla a teljes, normalizált dashboard-állapot JSON-változatát tartja meg. Tíz írásonként törlődnek a 25 óránál régebbi sorok.

### Napi összefoglaló

Az injektáló node 08:05-kor, `Europe/Budapest` időzónában indítja a feldolgozást:

1. lekéri az előző 24 óra MySQL-adatait;
2. lekéri és elemzi az Open-Meteo napi előrejelzését;
3. lekéri az EUR/HUF árfolyamot;
4. Google Sheetsből beolvassa a napi menüt és a hulladékszállítási emlékeztetőt;
5. lekéri az aktuális névnapot;
6. összefűzi az ágakat;
7. HTML e-mailt készít és továbbít az SMTP node-nak.

A fűtési és kazán-üzemidő becslése az egymást követő minták közötti időt számolja. A nagy adatkimaradásokat egy mintavételi periódusra korlátozza, hogy egy hosszú kiesés ne jelenjen meg tévesen folyamatos működésként.

## Kapcsolódási szerződés

A két flow közötti fő szerződés a normalizált dashboard-payload:

```json
{
  "_ts": 0,
  "weather": {},
  "ac": {},
  "sensors": {
    "th": {},
    "switches": {},
    "shutters": {}
  },
  "lights": {}
}
```

A `room`, `label` és relénév értékeknek egyezniük kell a HVAC normalizálóiban, az SQL-lekérdezésben és a HTML-riport szobatérképében.
