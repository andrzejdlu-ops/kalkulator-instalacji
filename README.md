# Kalkulator Instalacji

Aplikacja webowa (PWA) do liczenia kosztu instalacji przyłącza FTTH. Część ekosystemu "SkyApp" — dostępna jako kafelek w hubie [andrzejdlu-ops.github.io](https://andrzejdlu-ops.github.io/), obok Kalkulatora Premii.

**Live:** https://andrzejdlu-ops.github.io/kalkulator-instalacji/

## Struktura

```
web/
├── index.html      ← cała aplikacja (HTML + CSS + JS), bez zależności zewnętrznych
├── manifest.json    ← PWA manifest
└── README.md
```

Materiały źródłowe/założenia projektowe (poza tym repo, w folderze nadrzędnym `kalkulator instalacji/`, niewersjonowane w git):
- `Kalkulator v1.7.exe` — prototyp desktopowy, wzór funkcjonalny i wizualny
- `wyglad interfejsu.png` — screenshot UI prototypu
- `zalozenia projektowe do aplikacji.txt` — wymagania funkcjonalne
- `opis cen.txt` — cennik FTTH, źródło domyślnych cen w zakładce Cennik

## Jak działa kalkulator

Dwie zakładki: **Kalkulator** i **Cennik**.

### Kalkulator

Cztery grupy pól:
- **Aktywacja** — radiobutton: 1 zł / 99 zł / 249 zł / inna kwota (pole). Kwoty są stałe, **nieedytowalne** w Cenniku (świadomie — zgodnie z założeniami, Cennik obejmuje tylko kabel/rurę/topologię).
- **Kabel** — radiobutton: do 140 m / 141-200 m / 201-250 m / inna długość (pole). Cena zależy od wybranego przedziału (patrz Cennik). Dla długości > 250 m cena liczona jest automatycznie jako `cena_201_250 + liczba_rozpoczętych_segmentów_50m × stawka_za_50m`.
- **Rura** — pole liczbowe w metrach, cena = `metry × cena_za_metr` (z Cennika).
- **Topologia instalacji** — słup→ziemia→dom (bez dopłaty) / słup-słup(y)→ziemia→dom (dopłata z Cennika).

Przycisk **Oblicz** wypisuje w polu wynikowym rozbicie na składowe + sumę. **Resetuj** przywraca wartości domyślne.

### Cennik

Edytowalne pola cenowe:
- kabel do 140 m / 141-200 m / 201-250 m (zł)
- stawka za każde rozpoczęte 50 m powyżej 250 m (zł)
- cena rury za metr (zł)
- dopłata za topologię słup-słup(y) (zł)

Zapisywane przez przycisk **Zapisz cennik** do `localStorage` przeglądarki (klucz `kalkulator_instalacji_cennik`) — **nie ma backendu ani bazy danych**, ceny są per-przeglądarka/per-urządzenie. Bez zapisanych wartości używane są domyślne (patrz niżej).

**Domyślne ceny** (wyprowadzone z `opis cen.txt`, do weryfikacji z aktualnym cennikiem firmowym):

| Pozycja | Wartość domyślna |
|---|---|
| Kabel do 140 m | 0 zł |
| Kabel 141–200 m | 100 zł |
| Kabel 201–250 m | 200 zł |
| Kabel — za każde rozpoczęte 50 m powyżej 250 m | 100 zł |
| Rura — cena za metr | 2,50 zł |
| Dopłata topologia słup-słup(y) | 50 zł |

### Kontrola dostępu

Na starcie `index.html` sprawdza `sessionStorage.getItem('skyapp_auth') !== '1'` i jeśli nie ustawione, przekierowuje na hub `https://andrzejdlu-ops.github.io/`. To **nie jest** prawdziwe zabezpieczenie (sessionStorage łatwo ustawić ręcznie w konsoli przeglądarki) — to samo podejście co w Kalkulatorze Premii, traktowane jako ukrycie przed przypadkowym wejściem, nie kontrola dostępu. Nie ma tu żadnych danych wrażliwych ani backendu, więc ryzyko jest znikome.

Brak Firebase / bazy danych — w przeciwieństwie do Kalkulatora Premii, tu nie ma historii obliczeń w chmurze. Jeśli będzie taka potrzeba w przyszłości, można dodać Firestore analogicznie do `KalkulatorPremii/web/` + `KalkulatorPremii/firebase/`.

Pełne informacje o dostępach, kontach i deployu: [SETUP.md](SETUP.md).
