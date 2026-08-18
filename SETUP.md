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

## Znane pułapki (na bazie doświadczeń z Kalkulatora Premii — to samo konto/ekosystem)

**Mieszanie kont git na jednym komputerze:** jeśli na tej maszynie pracowano też nad projektem Roltes (konto `rolxxa0-dot`), zwykły `git push` w tym repo może failować z `Permission denied to rolxxa0-dot` (403), mimo że `gh auth status` poprawnie pokazuje zalogowane `andrzejdlu-ops` — Windows Credential Manager może mieć zapisane inne poświadczenia dla `git`. Naprawa: `gh auth setup-git` (podpina `gh` jako credential helper), a jeśli sesja przełącza się między kontami: `gh auth switch` na właściwe konto PRZED `git push`.

**Nowy komputer / świeże środowisko:** upewnić się, że `gh auth status` pokazuje `andrzejdlu-ops` przed jakimkolwiek pushem — nie zakładać, że istniejąca sesja `gh`/`git` jest już poprawnie skonfigurowana.

## Czego tu NIE ma (świadome decyzje projektowe)

- **Brak Firebase / bazy danych** — cennik trzyma się w `localStorage` przeglądarki, nie ma historii obliczeń w chmurze (inaczej niż Kalkulator Premii). Jeśli potrzebna wspólna historia między urządzeniami, trzeba by dodać Firestore analogicznie do `KalkulatorPremii/firebase/`.
- **Brak buildu/bundlera** — `index.html` to cały kod, edytować bezpośrednio, bez kroku kompilacji.
- **Brak ikon PWA** w `manifest.json` (tak samo jak w Kalkulatorze Premii) — manifest działa, ale bez pola `icons`.
