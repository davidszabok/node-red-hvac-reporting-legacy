# Hardverjegyzék

Ez a jegyzék azt a fizikai környezetet dokumentálja, amelyhez az archivált Node-RED flow-k készültek. A publikus flow-exportokban a tényleges eszközazonosítók anonimizált helykitöltőkre vannak cserélve.

## Összefoglaló

| Terület | Eszköz | Mennyiség | Szerep |
|---|---|---:|---|
| Központ | Sonoff iHost | 1 db | Zigbee/LAN eszközök központja és a Node-RED futtatási környezete |
| Fűtésvezérlés | Sonoff 4CH PRO R3 | 2 db | zónák, kazán és keringetőszivattyú relés vezérlése |
| Padlófűtés | Danfoss termoelektromos aktuátor | 10 db | 10 fűtési kör nyitása és zárása, 5 logikai zónába csoportosítva |
| WC szenzor | Sonoff SNZB-02P | 1 db | hőmérséklet és relatív páratartalom |
| Fürdőszobai szenzor | Sonoff SNZB-02P | 1 db | hőmérséklet és relatív páratartalom |
| Szobaszenzorok | Sonoff SNZB-02D | helyiségenként 1 db | kijelzős hőmérséklet- és páratartalom-mérés |
| Fali kapcsolók | Sonoff TX Ultimate | telepítés szerint | világítási kapcsolás és további okosotthon-bemenetek |
| IKEA központ | IKEA DIRIGERA hub | 1 db | IKEA lámpák és vezeték nélküli vezérlők integrációja |
| IKEA vezérlők | IKEA BILRESA és STYRBAR | telepítés szerint | vezeték nélküli világításvezérlés |
| IKEA fényforrások | IKEA TRÅDFRI izzók és LED-vezérlők | telepítés szerint | szabályozható világítás |
| Klímaberendezés | Mitsubishi Electric MSZ-AY35 | 1 db | átmeneti fűtés és nyári hűtés |
| Klímakapcsolat | gyári Mitsubishi Wi-Fi modul + MELCloud | 1 készlet | állapotlekérdezés és távoli parancsok |

## Fűtési topológia

```text
Node-RED az iHoston
        │
        ├── Sonoff 4CH PRO R3 #1 ── zónák / kazán
        │
        └── Sonoff 4CH PRO R3 #2 ── zónák / szivattyú
                                      │
                                      ▼
                    10 Danfoss aktuátor / 10 padlófűtési kör
                                      │
                                      ▼
                              5 logikai fűtési zóna
```

A tíz hidraulikai kör nem tíz külön szabályozási zónát jelent. Az azonos helyiséghez tartozó aktuátorok közös zónaparancsot kapnak; a flow öt logikai zónával dolgozik:

- nappali;
- hálószoba;
- gyerekszoba;
- fürdőszoba;
- WC.

A két 4 csatornás relémodul nyolc relékimenetet biztosít. A flow öt zónakimenet mellett külön kazán- és szivattyúparancsot kezel; az egyes relécsatornákhoz a helyi vezetékezés szerinti egy vagy több Danfoss aktuátor tartozik.

## Szenzorok

- A WC-ben és a fürdőszobában egy-egy Sonoff SNZB-02P méri a hőmérsékletet és a páratartalmat.
- A lakószobákban helyiségenként egy Sonoff SNZB-02D kijelzős érzékelő működik.
- A flow az öt zóna mérési értékeit normalizált helyiségkulcsokra alakítja.
- Hiányzó mérésnél a legutóbbi érvényes érték csak korlátozott ideig használható; a szenzor hardveres elhelyezése és kalibrálása továbbra is helyszíni feladat.

## Világítás és kezelőszervek

A rendszer két párhuzamos ökoszisztémát egyesít:

- a Sonoff TX Ultimate kapcsolók az iHost/Sonoff oldalon jelennek meg;
- az IKEA DIRIGERA hub kezeli a BILRESA és STYRBAR vezeték nélküli vezérlőket, valamint a TRÅDFRI izzókat és LED-vezérlőket;
- a Node-RED Unifier közös dashboard-állapotba rendezi a Sonoff és IKEA eszközök elérhetőségét, kapcsolási állapotát és – ahol van – akkumulátorszintjét.

## Klímaberendezés

A rendszer egy **Mitsubishi Electric MSZ-AY35** klímát vezérel. A beltéri egység gyári Wi-Fi modulja a MELCloud szolgáltatáshoz kapcsolódik, a Node-RED pedig a `node-red-contrib-melcloud` node-okon keresztül:

- lekéri az aktuális tápállapotot és üzemmódot;
- figyeli a mért és beállított hőmérsékletet;
- fűtési vagy hűtési parancsot küld;
- a flow döntése alapján leállítja az általa indított üzemet.

A repository nem tartalmaz MELCloud-fiókadatot, building ID-t vagy device ID-t.

## Integrációs kapcsolatok

| Kapcsolat | Útvonal |
|---|---|
| Sonoff szenzorok, relék és TX Ultimate | eszköz → iHost → `node-red-contrib-ihost` → Node-RED |
| IKEA lámpák és vezérlők | eszköz → DIRIGERA → `node-red-contrib-dirigera` → Node-RED |
| Mitsubishi klíma | MSZ-AY35 Wi-Fi modul → MELCloud → `node-red-contrib-melcloud` → Node-RED |
| NSPanel felület | böngésző/panel → UIBUILDER → normalizált `dash_state` |

## Telepítési figyelmeztetés

A relékiosztás, az aktuátorok feszültsége, a kazán engedélyező kontaktusa és a szivattyú kapcsolása telepítésfüggő. A dokumentáció nem villamos bekötési terv. A hálózati feszültségű és kazánvezérlő áramköröket megfelelő képesítéssel rendelkező szakembernek kell ellenőriznie, és a fizikai gépészeti védelmeknek a Node-RED-től függetlenül is működniük kell.
