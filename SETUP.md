# Setup i dostępy — Kalkulator Instalacji

Ten plik zawiera wszystko potrzebne, żeby kontynuować pracę nad projektem z innego komputera: konta, repozytoria, komendy deployu i znane pułapki.

## Konta / GitHub

- **Konto:** `andrzejdlu-ops` (e-mail: `andrzejdlu@gmail.com`)
- **Repo aplikacji:** https://github.com/andrzejdlu-ops/kalkulator-instalacji (publiczne)
- **Branch:** `master`
- **Repo huba (launcher z kafelkami):** https://github.com/andrzejdlu-ops/andrzejdlu-ops.github.io (publiczne, branch `main`)
  - Lokalny klon huba na tym komputerze: `Projekty Claude/GitHubHome/` (poza tym folderem projektu — osobne repo)
- Do operacji `git`/`gh` potrzebne zalogowanie: `gh auth login` (konto `andrzejdlu-ops`), potem `gh auth setup-git`, żeby `git push`/`pull` używały właściwych poświadczeń (patrz pułapki niżej).

## Struktura na dysku

```
kalkulator instalacji/                         ← folder projektu (niewersjonowany, materiały źródłowe)
├── Kalkulator v1.7.exe
├── wyglad interfejsu.png
├── zalozenia projektowe do aplikacji.txt
├── opis cen.txt
├── firebase/                                   ← config Firestore, NIE w git (patrz sekcja Firebase niżej)
│   ├── firebase.json
│   └── firestore.rules
└── web/                                        ← TO jest repo git (kalkulator-instalacji)
    ├── index.html
    ├── manifest.json
    ├── README.md
    └── SETUP.md
```

Working dir dla `git`/`gh` tej aplikacji: **`kalkulator instalacji/web/`** — tylko ten podfolder jest repozytorium (ten sam wzorzec co w projekcie Kalkulator Premii, gdzie `KalkulatorPremii/web/` jest repo, a folder nadrzędny nie).

## GitHub Pages

- **Live URL:** https://andrzejdlu-ops.github.io/kalkulator-instalacji/
- **Tryb:** legacy branch-deploy (source: branch `master`, path `/`) — **nie** GitHub Actions. Push do `master` = automatyczny deploy, bez custom workflow.
- Włączone jednorazowo komendą (nie trzeba powtarzać, ustawienie trwa w repo):
  ```bash
  gh api -X POST repos/andrzejdlu-ops/kalkulator-instalacji/pages -f "source[branch]=master" -f "source[path]=/"
  ```

## Kafelek w hubie

Hub (`andrzejdlu-ops.github.io`) to osobne repo z jednym plikiem `index.html` zawierającym siatkę kafelków. Kafelek tej aplikacji został dodany w sekcji `<div class="grid">` jako:

```html
<a class="tile" onclick="openApp('https://andrzejdlu-ops.github.io/kalkulator-instalacji/')" href="#">
  <div class="tile-icon blue">🔌</div>
  <div>
    <div class="tile-name">Kalkulator Instalacji</div>
    <div class="tile-desc">Obliczanie kosztu instalacji przyłącza (aktywacja, kabel, rura, topologia).</div>
  </div>
  <div class="tile-arrow">Otwórz →</div>
</a>
```

Żeby zmienić opis/ikonę kafelka: edytować `GitHubHome/index.html`, commitować i pushować w tamtym repo (branch `main`), nie w tym.

## Komendy — redeploy po zmianach

```bash
cd "kalkulator instalacji/web"
git add index.html manifest.json
git commit -m "opis zmiany"
git push
# GitHub Pages przebuduje się automatycznie po pushu do master (zwykle < 1 min)
```

Weryfikacja po deployu:
```bash
curl -s -o /dev/null -w "%{http_code}\n" https://andrzejdlu-ops.github.io/kalkulator-instalacji/
```

## Firebase — wspólny cennik między urządzeniami

Cennik (ceny kabla/rury/dopłaty topologii) jest współdzielony między wszystkimi komputerami korzystającymi z aplikacji przez Firestore — **własny, osobny projekt Firebase**, celowo NIE dzielony z Kalkulatorem Premii.

- **Projekt Firebase:** `kalkulator-instalacji-672ca` (Google Cloud project ID, jak zwykle w Firebase — nazwa różni się od repo z powodu globalnej unikalności ID)
- **Konsola:** https://console.firebase.google.com/project/kalkulator-instalacji-672ca/overview
- **Firestore:** baza `(default)`, region `eur3`, kolekcja `cennik`, jeden dokument o ID `config`
- **Config SDK** (wklejony bezpośrednio w `web/index.html`, standardowe dla Firebase Web SDK — apiKey tu nie jest sekretem):
  ```js
  apiKey: "AIzaSyApsuj77TS-KwyTmBGOYwBzvt7kzPpV-X4"
  authDomain: "kalkulator-instalacji-672ca.firebaseapp.com"
  projectId: "kalkulator-instalacji-672ca"
  storageBucket: "kalkulator-instalacji-672ca.firebasestorage.app"
  messagingSenderId: "449649988553"
  appId: "1:449649988553:web:e92ff389128327d7ae4e18"
  ```
- **Firestore rules i firebase.json:** `kalkulator instalacji/firebase/` (osobny folder, NIE w `web/`, NIE w git — analogicznie do `KalkulatorPremii/firebase/`). Brak `.firebaserc`, więc przy deployu trzeba jawnie podać projekt:
  ```bash
  cd "kalkulator instalacji/firebase"
  firebase deploy --only firestore:rules --project kalkulator-instalacji-672ca
  ```
- **Reguły:** wymóg `request.auth != null` na read/write + walidacja struktury dokumentu `cennik/config` (dozwolone tylko pola `kab140, kab200, kab250, kabOver, rura, topo, updatedAt`, wszystkie liczby w rozsądnym zakresie). Wszystko poza `cennik/config` zablokowane. Pełna treść `firestore.rules` (na wypadek gdyby folder `firebase/` zaginął — nie jest w git):
  ```
  rules_version = '2';
  service cloud.firestore {
    match /databases/{database}/documents {

      match /cennik/{docId} {
        function isValidCennik() {
          let d = request.resource.data;
          return docId == 'config'
            && d.keys().hasOnly(['kab140', 'kab200', 'kab250', 'kabOver', 'rura', 'topo', 'updatedAt'])
            && d.keys().hasAll(['kab140', 'kab200', 'kab250', 'kabOver', 'rura', 'topo'])
            && d.kab140 is number && d.kab140 >= 0 && d.kab140 <= 100000
            && d.kab200 is number && d.kab200 >= 0 && d.kab200 <= 100000
            && d.kab250 is number && d.kab250 >= 0 && d.kab250 <= 100000
            && d.kabOver is number && d.kabOver >= 0 && d.kabOver <= 100000
            && d.rura is number && d.rura >= 0 && d.rura <= 10000
            && d.topo is number && d.topo >= 0 && d.topo <= 100000;
        }

        allow read:   if request.auth != null;
        allow create: if request.auth != null && isValidCennik();
        allow update: if request.auth != null && isValidCennik();
        allow delete: if false;
      }

      match /{document=**} {
        allow read, write: if false;
      }
    }
  }
  ```
- **Anonymous Auth — WŁĄCZONE (2026-08-18).** Był to jednorazowy ręczny krok w konsoli Firebase: **Authentication → Sign-in method → Anonymous → Enable** (https://console.firebase.google.com/project/kalkulator-instalacji-672ca/authentication/providers) — **nie da się tego zrobić przez CLI/API** (błąd `auth/configuration-not-found` dopóki nie zostanie kliknięte w konsoli). Gdyby trzeba było odtworzyć projekt od zera, to jedyny krok wymagający ręcznej interwencji człowieka w przeglądarce.
- **Synchronizacja między urządzeniami — przetestowana i potwierdzona działająca (2026-08-18, ponownie na żywej stronie 2026-08-19).** Test: dwie niezależne sesje przeglądarki (osobny `localStorage`/kontekst, symulacja dwóch komputerów) — zapis ceny w sesji A trafia do Firestore i pojawia się natychmiast przy świeżym wejściu w sesji B, a już otwarta sesja B dostaje aktualizację na żywo (`onSnapshot`, bez odświeżania strony) po zmianie w A. Potwierdzono też, że obliczenia używają ceny z bazy, a nie z lokalnego cache (test z celowo zatrutym cache: baza wygrywa po zakończeniu synchronizacji).

> ### ⚠️ UWAGA: `cennik/config` to PRODUKCYJNE dane firmowe — nie nadpisywać "domyślnymi"
>
> Ceny w bazie są wpisywane przez realnych użytkowników i **różnią się od `DEFAULT_PRICES` w kodzie**. Przykład: 2026-08-19 w bazie była cena rury **4,50 zł/m**, podczas gdy w kodzie domyślna to 2,50 zł/m.
>
> Firestore nie ma tu włączonego Point-in-time recovery, więc **nadpisanie tego dokumentu jest nieodwracalne** — nie ma historii wersji ani cofnięcia. Zdarzyło się to raz (2026-08-19) przy sprzątaniu po teście automatycznym, który "przywracał wartości domyślne" — realna cena została skasowana i trzeba było ją odtwarzać z logów testu.
>
> **Zasady przy testowaniu:** testy automatyczne robić na odczycie, a jeśli test musi pisać — najpierw odczytać i zapamiętać oryginalny dokument, a na końcu przywrócić dokładnie te wartości (nigdy wartości założone/domyślne). Odczyt stanu bazy bez zmieniania czegokolwiek: otworzyć apkę i w konsoli przeglądarki wykonać `await firebase.firestore().collection('cennik').doc('config').get().then(s => s.data())`.
- **Zachowanie appki:** przy starcie od razu pokazuje ceny z lokalnego cache (`localStorage`, klucz `kalkulator_instalacji_cennik_cache`) lub domyślne — bez migania zerami — a następnie podłącza się pod `onSnapshot` na `cennik/config`, więc zmiana ceny na jednym urządzeniu pojawia się na żywo (bez odświeżania) na innych otwartych sesjach. `savePrices()` zapisuje jednocześnie do Firestore i do lokalnego cache; jeśli zapis do bazy się nie uda (np. brak internetu), dane i tak zostają zapisane lokalnie (offline-first) i appka pokazuje czerwony komunikat błędu zamiast się wywalić.

## Znane pułapki (na bazie doświadczeń z Kalkulatora Premii — to samo konto/ekosystem)

**Enable Firestore API na nowym projekcie GCP potrafi 403-ować przy pierwszym `firebase deploy --only firestore` / `firestore:databases:create`** mimo że CLI samo próbuje włączyć wymagane API ("ensuring required API firestore.googleapis.com is enabled..."). To propagacja, nie błąd konfiguracji — trzeba po prostu ponowić `firebase deploy --only firestore:rules --project <id>` po ok. 30-60 sekundach, zwykle wystarczy 1-2 ponowienia.

**Mieszanie kont git na jednym komputerze:** jeśli na tej maszynie pracowano też nad projektem Roltes (konto `rolxxa0-dot`), zwykły `git push` w tym repo może failować z `Permission denied to rolxxa0-dot` (403), mimo że `gh auth status` poprawnie pokazuje zalogowane `andrzejdlu-ops` — Windows Credential Manager może mieć zapisane inne poświadczenia dla `git`. Naprawa: `gh auth setup-git` (podpina `gh` jako credential helper), a jeśli sesja przełącza się między kontami: `gh auth switch` na właściwe konto PRZED `git push`.

**Nowy komputer / świeże środowisko:** upewnić się, że `gh auth status` pokazuje `andrzejdlu-ops` przed jakimkolwiek pushem — nie zakładać, że istniejąca sesja `gh`/`git` jest już poprawnie skonfigurowana.

## Czego tu NIE ma (świadome decyzje projektowe)

- **Brak historii obliczeń w chmurze** — Firestore przechowuje tylko cennik (jeden dokument), nie ma kolekcji z historią wyliczeń jak w Kalkulatorze Premii. Jeśli taka potrzeba się pojawi, dodać kolejną kolekcję + reguły w tym samym projekcie `kalkulator-instalacji-672ca`.
- **Brak buildu/bundlera** — `index.html` to cały kod, edytować bezpośrednio, bez kroku kompilacji.
- **Brak ikon PWA** w `manifest.json` (tak samo jak w Kalkulatorze Premii) — manifest działa, ale bez pola `icons`.
