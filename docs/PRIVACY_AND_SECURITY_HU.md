# Adatvédelem és biztonság

## Mit távolítottunk el?

A publikus exportból helyettesítő értékre cserélve vagy eltávolítva:

- személyes és szolgáltatásfiók e-mail-címek;
- MELCloud-jelszó és azonosítók;
- iHost hozzáférési token;
- belső hálózati IP-címek;
- pontos település és eredeti koordináták;
- utcanevek és személynévvel jelölt helyiségek;
- Google Sheet-azonosítók;
- iHost-, Sonoff-, IKEA DIRIGERA- és Zigbee-eszközazonosítók;
- MySQL host- és adatbázisazonosítók.

Az exportban szereplő `192.0.2.0/24` címek dokumentációs célra fenntartott példacímek. Az `example.com` címek és a `REPLACE_WITH_…` értékek szintén szándékos helykitöltők.

## Közzététel előtti ellenőrzőlista

- Ne commitolj `flows_cred.json`, `.env`, SQL dump, service-account JSON vagy Node-RED credential secret fájlt.
- GitHubra feltöltés előtt keress e-mail-címekre, privát IP-tartományokra, tokenekre, jelszavakra és pontos címadatokra.
- Ha egy korábbi commitban titok szerepelt, a fájl későbbi törlése nem elég: a titkot vissza kell vonni, és a Git-történetet is tisztítani kell.
- A service accountnak, MySQL-felhasználónak és SMTP-fióknak csak a szükséges minimális jogosultságot add.
- A Node-RED admin felületet és UIBUILDER végpontokat ne tedd hitelesítés nélkül nyilvánosan elérhetővé.

## Jelentés

Ha a repositoryban valós titkot vagy személyes adatot találsz, ne nyiss olyan publikus issue-t, amelyben megismétled az adatot. Először vond vissza az érintett credentialt, majd a repository tulajdonosával privát csatornán egyeztess.
