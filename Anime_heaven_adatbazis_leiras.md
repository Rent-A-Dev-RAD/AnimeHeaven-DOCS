# Anime Heaven Adatbázis Leírás

## Rövid összefoglaló

Ez az adatbázis egy anime-gyűjtemény / lejátszó oldal alapját képezi: tárolja az anime-adatlapokat, epizódokat, felhasználói profilokat, felhasználói előzményeket, forráslinkeket és címkéket.

---

## Fő táblák és szerepük

### `anime_adatlap`
Az anime fő adatai: cím(ek) (japán, angol), leírás, megjelenés dátuma, típus (TV/Film/OVA), státusz, készítő, besorolás, értékelés, borító/háttér képek, láthatóság. 

### `reszek`
Az anime-epizódok listája (`anime_id` → kapcsolódik `anime_adatlap`-hoz). Sorrend, epizódazonosító és láthatóság mezők.

### `cimke_lista` és `anime_cimke`
**Normalizált címkerendszer:** A `cimke_lista` tárolja az előre definiált címkéket (Action, Fantasy, Romance stb.), az `anime_cimke` kapcsolótábla pedig összeköti az animéket a címkékkel (Many-to-Many kapcsolat). Így egy anime több címkét kaphat, és ugyanaz a címke több animéhez is hozzárendelhető duplikáció nélkül.

### `studio_lista` és `anime_studio`
**Normalizált stúdiórendszer:** A `studio_lista` tárolja az animációs stúdiókat (MAPPA, UFOTABLE stb.), az `anime_studio` kapcsolótábla pedig összeköti az animéket a stúdiókkal (Many-to-Many kapcsolat). Egy anime több stúdióval is készülhet.

### `forras_tipus` és `forras_elem`
**Rugalmas forráskezelés:** A `forras_tipus` egyszerű lista formátumban tárolja a forrástípusokat (`id`, `nev`), például "Indavideo", "Videa", "YouTube". A `forras_elem` tábla kapcsolja össze az epizódokat a forráslinkekkel (`forras_id` → `forras_tipus`, `resz_id` → `reszek`). **Egy epizódhoz több forrás is tartozhat** (Szerver #1, #2, #3), így a felhasználók választhatnak a lejátszási lehetőségek között.

### `profil_adatlap`
Felhasználói fiókok (email, jelszó, profilkép, admin jog).

### `elozmeny`
Felhasználói megtekintési előzmények (`profil_id`, `anime_id`, `resz_id`, `megnezve` timestamp). Hasznos a folytatás / statisztika funkciókhoz.

### `lista_tipus` és `lista_elem`
Felhasználói listák (pl. kedvencek, megnézendők) típusa és az azokhoz tartozó elemek (`profil_id` + `anime_id` + `tipus_id`).

---

## Kapcsolatok / kulcsok

**Egy-a-többhöz kapcsolatok:**
- `reszek.anime_id` → `anime_adatlap.id` (egy anime → több epizód)
- `elozmeny.profil_id` → `profil_adatlap.id`, `elozmeny.anime_id` → `anime_adatlap.id`, `elozmeny.resz_id` → `reszek.id`
- `lista_elem.profil_id` → `profil_adatlap.id`, `lista_elem.anime_id` → `anime_adatlap.id`, `lista_elem.tipus_id` → `lista_tipus.id`

**Több-a-többhöz kapcsolatok (Many-to-Many junction tables):**
- `anime_cimke` kapcsolótábla: `anime_id` → `anime_adatlap.id` + `cimke_id` → `cimke_lista.id`
  - Egy anime több címkét kaphat, egy címke több animéhez is tartozhat
- `anime_studio` kapcsolótábla: `anime_id` → `anime_adatlap.id` + `studio_id` → `studio_lista.id`
  - Egy anime több stúdióval készülhet, egy stúdió több animét is készíthet

**Forráskezelés:**
- `forras_elem.resz_id` → `reszek.id` és `forras_elem.forras_id` → `forras_tipus.id`

- **Karakterkészlet:** `utf8mb4` (magyar collate: `utf8mb4_hungarian_ci`)
- **Normalizált szerkezet:** A címkék és stúdiók külön referencia táblákban vannak (`cimke_lista`, `studio_lista`) UNIQUE indexekkel, így elkerülhető a redundancia
- **Enum típusok:** A táblákban enumok vannak (pl. `statusz`, `tipus`, `besorolas`) — egyszerűsíti az alkalmazás logikát, de rugalmatlanabb bővítésnél
- **Egyedi email és nevek:** A `profil_adatlap.email`, `cimke_lista.cimke_nev` és `studio_lista.studio_nev` egyedi indexeltek (UNIQUE)
- **Cascade Delete/Update:** Az idegen kulcsok CASCADE-del vannak beállítva, így anime törlésénél automatikusan törlődnek a kapcsolódó epizódok, címkék, stúdiókapcsolatok is
- **AUTO_INCREMENT:** Minden tábla használ AUTO_INCREMENT-et az id mezőkhöz, így az INSERT utasítások nem tartalmaznak explicit id értékek
---

## Technikai megjegyzések
- **Karakterkészlet:** `utf8mb4` (magyar collate: `utf8mb4_hungarian_ci`)
- **Enum típusok:** A táblákban enumok vannak (pl. `statusz`, `tipus`, `besorolas`) — egyszerűsíti az alkalmazás logikát, de rugalmatlanabb bővítésnél
- **Egyedi email:** A `profil_adatlap.email` egyedi indexelt (UNIQUE), így egy email csak egyszer szerepelhet

