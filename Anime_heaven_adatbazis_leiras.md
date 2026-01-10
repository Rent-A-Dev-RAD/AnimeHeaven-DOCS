# Anime Heaven Adatbázis Leírás

## Rövid összefoglaló

Ez az adatbázis egy anime-gyűjtemény / lejátszó oldal alapját képezi: tárolja az anime-adatlapokat, epizódokat, felhasználói profilokat, felhasználói előzményeket, forráslinkeket és címkéket.

---

## Fő táblák és szerepük

### `anime_adatlap`
Az anime fő adatai: cím(ek) (japán, angol), leírás, megjelenés dátuma, típus (TV/Film/OVA), státusz, készítő, besorolás, értékelés, borító/háttér képek, láthatóság. 

### `reszek`
Az anime-epizódok listája (`anime_id` → kapcsolódik `anime_adatlap`-hoz). Sorrend, epizódazonosító és láthatóság mezők.

### `cimke`
Anime-hez tartozó címkék / tagek (`anime_id`, `cimke`). Segít a keresésben és a kategorizálásban.

### `studiok`
Egy anime-hez kapcsolt stúdiók (több stúdió is lehet egy anime-hez).

### `forras_tipus` és `forras_elem`
Források típusai (pl. Vimeo, YouTube, Videa stb.) és az egyes epizódokhoz tartozó forráslinkek (`forras_id` → `forras_tipus`, `resz_id` → `reszek`). Ezzel szervezhető, honnan játszható le egy rész.

### `profil_adatlap`
Felhasználói fiókok (email, jelszó, profilkép, admin jog).

### `elozmeny`
Felhasználói megtekintési előzmények (`profil_id`, `anime_id`, `resz_id`, `megnezve` timestamp). Hasznos a folytatás / statisztika funkciókhoz.

### `lista_tipus` és `lista_elem`
Felhasználói listák (pl. kedvencek, megnézendők) típusa és az azokhoz tartozó elemek (`profil_id` + `anime_id` + `tipus_id`).

---

## Kapcsolatok / kulcsok


- `reszek.anime_id` → `anime_adatlap.id` (egy-a-többhöz: egy anime-hez több epizód tartozhat)
- `cimke.anime_id` → `anime_adatlap.id` (egy-a-többhöz: egy anime-hez több címke tartozhat)
- `studiok.anime_id` → `anime_adatlap.id` (egy-a-többhöz: egy anime-hez több stúdió tartozhat)
- `forras_elem.resz_id` → `reszek.id` és `forras_elem.forras_id` → `forras_tipus.id`
- `elozmeny` kapcsolja a `profil_adatlap`, `anime_adatlap` és `reszek` táblákat (több idegen kulcs)
- `lista_elem` kapcsolja a profilokat az animékhez és a lista típusához

> A dump tartalmazza az idegen kulcsos megszorításokat és az AUTO_INCREMENT beállításokat is.

---

## Technikai megjegyzések
0
- **Karakterkészlet:** `utf8mb4` (magyar collate: `utf8mb4_hungarian_ci`)
- **Enum típusok:** A táblákban enumok vannak (pl. `statusz`, `tipus`, `besorolas`) — egyszerűsíti az alkalmazás logikát, de rugalmatlanabb bővítésnél
- **Egyedi email:** A `profil_adatlap.email` egyedi indexelt (UNIQUE), így egy email csak egyszer szerepelhet