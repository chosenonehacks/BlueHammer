# Zakres testów bezpieczeństwa i estymacja czasowa — PRINTLUX.PL

## 1. Metadane

| Pole | Wartość |
|------|---------|
| Aplikacja | PRINTLUX.PL (właściciel zewnętrzny, domena obca) |
| Typ | Dedykowany portal WordPress do zamówień druku wizytówek LUX MED |
| Właściciel hostingu | Agencja HENNESSEY — Małgorzata Rachwał (mr@hennessey.pl) |
| Środowisko testowe | Testowe, odpowiadające produkcyjnemu |
| Hosting | Subskrypcja dostawcy (Agencja HENNESSEY) |
| Podejście | Gray box |
| Lokalizacja zespołu | Zdalnie |
| Klasyfikacja danych | Ogólnodostępne dane osobowe (adres placówki, służbowy e-mail i telefon) |
| Data dokumentu | 2026-06-16 |
| Wersja | 1.0 |
| Autor | Zespół Testów Bezpieczeństwa IT |

## 2. Opis aplikacji i funkcjonalności krytyczne

Dedykowany portal do składania zamówień druku wizytówek firmowych LUX MED,
zbudowany na WordPressie. Funkcjonalności krytyczne:

1. **Formularz** z polami tekstowymi i polami wyboru (35 pól do wypełnienia,
   możliwość jednoczesnego wysłania do 10 formularzy).
2. **Przyjmowanie plików** od użytkownika: `.xlsx`, `.jpeg`, `.jpg`, `.png`,
   `.pdf`, `.eps`, `.tiff`.

**Skala aplikacji:** podstawowe strony systemu WordPress + 1 strona własna.
Liczba metod HTTP i argumentów — zgodna ze standardem WordPress, plus 35 pól
formularza własnego.

**Uwierzytelnianie:** ID login, hasło indywidualne, kod mailem.
**Autoryzacja / kontrole:** logowanie WordPress (login + hasło), 2FA e-mail,
reCAPTCHA, Wordfence WAF.
**Sesja:** cookie **niezaszyfrowane**, ale **podpisywane**.

**Role:**
- Każdy zalogowany użytkownik — dostęp wyłącznie do funkcjonalności formularza.
- Administrator — dodatkowo panel administracyjny.

**Technologie:** WordPress 6.7.1; PHP min. 8.1 (faktyczna wersja serwera
nieudokumentowana w repo); MySQL 8.0.45.

**Integracje LUX MED:** brak — aplikacja nie komunikuje się z komponentami LUX MED.

## 3. Klasyfikacja danych i ryzyko

Przetwarzane są dane **ogólnodostępne osobowe służbowe** (adres placówki, e-mail
i telefon służbowy) — niska wrażliwość treści. Główny wektor ryzyka nie wynika
z wrażliwości danych, lecz z **przyjmowania plików od użytkownika** (6 typów,
w tym formaty podatne na nadużycia: `.xlsx`, `.eps`, `.pdf`, `.tiff`) oraz
z faktu, że aplikacja działa **na obcej domenie/hostingu zewnętrznym**, ale jest
brandowana jako LUX MED — ryzyko reputacyjne w razie przejęcia/defacementu.

## 4. Zakres prac

### W zakresie (in scope)
- Rdzeń WordPress 6.7.1 (znane podatności, konfiguracja, hardening)
- Wtyczki i motyw (wersje, znane CVE, konfiguracja)
- Strona/formularz własny (35 pól) — walidacja, injection, logika biznesowa
- Mechanizm uploadu plików (6 typów rozszerzeń)
- Uwierzytelnianie, sesja, 2FA e-mail, reCAPTCHA
- Kontrola dostępu: użytkownik zwykły vs administrator, panel administracyjny

### Poza zakresem (out of scope)
- Testy infrastruktury hostingowej (wg intake — **Nie**)
- Integracje z LUX MED (brak)
- DoS/DDoS, testy obciążeniowe
- Socjotechnika
- Retesty poprawek (oferowane jako opcja — patrz sekcja 8)

## 5. Podejście i metodyki

- **Podejście:** Gray box (konta: użytkownik, administrator).
- **WAF:** wg sekcji ZTB — brak; opiekun wskazuje Wordfence WAF (patrz sekcja 10).
- **Metodyki:**
  - OWASP Top 10
  - OWASP Web Security Testing Guide (WSTG) v4
  - OWASP Application Security Verification Standard (ASVS) v4
  - OWASP API Security Top 10
  - CWE/SANS TOP 25
  - OASIS Web Application Security
  - Własne metodyki Zleceniobiorcy

## 6. Założenia i wymagania wstępne

1. Dostęp do środowiska testowego odpowiadającego produkcyjnemu.
2. Konta testowe: min. 2 konta użytkownika (do testów kontroli dostępu) oraz
   1 konto administratora.
3. Działający kanał 2FA e-mail dla kont testowych.
4. Zgoda właściciela hostingu (Agencja HENNESSEY) na przeprowadzenie testów
   oraz, w razie potrzeby, **whitelista IP** testerów w Wordfence.
5. Potwierdzenie statusu WAF (Wordfence) na czas testów (patrz sekcja 10).
6. Lista wtyczek/motywów lub dostęp do panelu admin do inwentaryzacji.

## 7. Estymacja czasowa i kosztowa

> Wartość `[STAWKA_DZIENNA]` (PLN/MD) do uzupełnienia. Koszt = MD × `[STAWKA_DZIENNA]`.

| # | Faza | MD |
|---|------|----|
| 0 | Przygotowanie / recon / setup, weryfikacja kont (user, admin) | 0.5 |
| 1 | WordPress core / wtyczki / motyw (WPScan, znane CVE, konfiguracja, hardening) | 1 |
| 2 | Formularz własny: 35 pól (injection, XSS, logika, masowa wysyłka 10 form., rate limiting, reCAPTCHA bypass) | 1.5 |
| 3 | Bezpieczeństwo uploadu plików (6 typów, malicious upload, path traversal, content-type bypass, parsowanie xlsx/eps/tiff) | 1.5 |
| 4 | Uwierzytelnianie / sesja / 2FA e-mail, kontrola dostępu user vs admin, panel administracyjny | 1 |
| 5 | Raportowanie: analiza, ocena ryzyka (CVSS), raport końcowy, QA | 1.5 |
| 6 | Zarządzanie: kickoff, spotkanie zamykające | 0.5 |
| | **Suma bazowa** | **7.5 MD** |

**Koszt bazowy = 7.5 × `[STAWKA_DZIENNA]`.**

Przykład (stawka poglądowa **2 000 PLN/MD**, do nadpisania):
7.5 MD × 2 000 PLN = **15 000 PLN netto**.

## 8. Opcje dodatkowe

| Opcja | MD | Uwaga |
|-------|----|-------|
| Retest poprawek po stronie wykonawcy/hostingu | +1 | Poza sumą bazową — specyfikacja nie wymienia retestów |

## 9. Harmonogram orientacyjny i zespół

- Realizacja przez **1 pentestera**, kalendarzowo ~1,5–2 tygodnie robocze
  (7,5 MD) z uwzględnieniem czasu na raport i ustalenia z hostingiem.

## 10. Ryzyka i uwagi do potwierdzenia

- **Sprzeczność dot. WAF:** sekcja „Zespół Testów Bezpieczeństwa” deklaruje
  **brak WAF**, podczas gdy opiekun wymienia **Wordfence WAF** jako aktywną
  kontrolę. Estymacja zakłada testy **bez aktywnego WAF** lub z **whitelistą IP**
  testerów. Aktywny, niewyłączony WAF wydłuża testy i zaniża wyniki — **do
  uzgodnienia przed startem** (potencjalna korekta +0,5–1 MD na obejścia).
- **Hosting zewnętrzny (Agencja HENNESSEY):** wymagana pisemna autoryzacja
  właściciela na testy; bez niej testy nie mogą się rozpocząć.
- **Wersja PHP** serwera nieudokumentowana — do potwierdzenia (wpływ na ocenę
  podatności środowiska uruchomieniowego).
- **Upload plików** to główny wektor ryzyka; jeśli pliki są przetwarzane/
  konwertowane po stronie serwera (np. eps/tiff/pdf), warto rozważyć rozszerzenie
  fazy 3.
