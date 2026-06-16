# Zakres testów bezpieczeństwa i estymacja czasowa — Portal Pacjenta

## 1. Metadane

| Pole | Wartość |
|------|---------|
| Aplikacja | Portal Pacjenta (LX) |
| Typ | Aplikacja self-service dla pacjentów (Web + API + Mobile) |
| Właściciel / opiekun | Opiekun aplikacji LUX MED (do uzupełnienia) |
| Środowisko testowe | Nieprodukcyjne |
| Hosting | On-prem |
| Podejście | Gray box (utworzone konta wszystkich ról) |
| Lokalizacja zespołu | Zdalnie / w siedzibie usługodawcy |
| Klasyfikacja danych | Ogólnodostępne, osobowe, **medyczne** |
| Data dokumentu | 2026-06-16 |
| Wersja | 1.0 |
| Autor | Zespół Testów Bezpieczeństwa IT |

## 2. Opis aplikacji i funkcjonalności krytyczne

Aplikacja self-service dla pacjentów LX. Funkcjonalności krytyczne objęte
testami:

1. Prezentacja historii leczenia
2. Umawianie usług
3. Płatności online
4. Zarządzanie kontem
5. Inbox
6. Ankiety Medalia
7. Konsultacje online
8. Twoje zgody
9. CoDigital
10. Deklaracje POZ
11. Twoje pliki
12. Weryfikacja abonamentu
13. Mental

**Skala aplikacji** (wg intake):
- ~20 stron z podstronami
- ~400 metod GET, ~200 metod POST (~600 endpointów)
- ~1000 argumentów przekazywanych od użytkownika

**Integracje LUX MED (11 komponentów):** SSO, 4Net, CCS, P1, PaymentCenter,
Alfavox, ePOZ, Codigital, Medalia, Zowie, ApteGO.

**Uwierzytelnianie:** login/hasło, odcisk palca, PIN.
**Autoryzacja operacji:** kod SMS / e-mail / autoryzacja mobilna.
**Sesja:** cookie szyfrowane oraz podpisywane.

**Role:**
- Konto demo — bez potwierdzonej tożsamości (ograniczony dostęp)
- Konto pełne — pełen dostęp do Portalu Pacjenta

**Technologie:** .NET 8, .NET Framework 4.8, Angular 19; Android (Kotlin
2.2.20 / Java 17); iOS (Swift 5); MS SQL 2019.

## 3. Klasyfikacja danych i ryzyko

Aplikacja przetwarza dane **medyczne i osobowe** oraz obsługuje **płatności
online** — najwyższa kategoria wrażliwości wg standardu LUX MED (Klasyfikacja
i postępowanie z informacją). Wymusza to podwyższoną staranność testów
kontroli dostępu (IDOR/BOLA między pacjentami), ochrony danych w tranzycie
i spoczynku oraz bezpieczeństwa procesu płatności. Naruszenie poufności danych
medycznych niesie ryzyko prawne (RODO/ustawa o prawach pacjenta) i reputacyjne.

## 4. Zakres prac

### W zakresie (in scope)
- Aplikacja webowa (Angular 19) — warstwa front-end i logika biznesowa
- Warstwa API/back-end (.NET 8 / .NET Framework 4.8)
- Aplikacja mobilna **Android** (Kotlin)
- Aplikacja mobilna **iOS** (Swift)
- Mechanizmy uwierzytelniania, autoryzacji operacji (MFA), zarządzania sesją
- Kontrola dostępu między rolami (demo ↔ pełne) i między użytkownikami
- Granice zaufania względem 11 integracji (z perspektywy aplikacji)

### Poza zakresem (out of scope)
- Testy infrastruktury, na której działa aplikacja (wg intake — **Nie**)
- Testy wewnętrzne integrowanych systemów (SSO, P1, PaymentCenter itd.) jako
  odrębnych aplikacji — testowane wyłącznie z perspektywy Portalu Pacjenta
- DoS/DDoS, testy obciążeniowe
- Socjotechnika
- Retesty poprawek (oferowane jako opcja — patrz sekcja 8)

## 5. Podejście i metodyki

- **Podejście:** Gray box — utworzone konta dla wszystkich ról (demo, pełne).
- **WAF:** brak (testy bez WAF).
- **Metodyki:**
  - OWASP Top 10
  - OWASP Web Security Testing Guide (WSTG) v4
  - OWASP Application Security Verification Standard (ASVS) v4
  - OWASP API Security Top 10 (2019)
  - OWASP Mobile Top 10 / Mobile Application Security Verification Standard (MASVS) i MSTG
  - OWASP Top 10 Mobile Controls
  - CWE/SANS TOP 25
  - OASIS Web Application Security
  - Własne metodyki Zleceniobiorcy

## 6. Założenia i wymagania wstępne

Warunki konieczne do rozpoczęcia i dotrzymania estymacji:
1. Dostęp do środowiska nieprodukcyjnego (sieć/VPN), stabilny przez cały okres testów.
2. Konta testowe dla **wszystkich ról** (demo, pełne) — min. 2 konta na rolę
   (do testów IDOR/BOLA) z możliwością resetu.
3. Działające kanały MFA na kontach testowych (SMS/e-mail/autoryzacja mobilna).
4. Buildy aplikacji mobilnych (Android `.apk`/`.aab`, iOS `.ipa`) możliwe do
   instalacji na urządzeniu/emulatorze testerów oraz dostępność środowiska
   bez wymuszonego, niemożliwego do obejścia w teście certificate pinning
   (lub build testowy umożliwiający inspekcję ruchu).
5. Środowisko testowe odpowiadające produkcyjnemu pod kątem konfiguracji.
6. Dane testowe (np. testowe płatności w PaymentCenter w trybie sandbox).
7. Lista kontaktowa po stronie klienta do eskalacji blokad.

## 7. Estymacja czasowa i kosztowa

> Wartość `[STAWKA_DZIENNA]` (PLN/MD) do uzupełnienia. Koszt = MD × `[STAWKA_DZIENNA]`.

| # | Faza | MD |
|---|------|----|
| 0 | Przygotowanie: dostępy, weryfikacja kont (wszystkie role), recon, modelowanie zagrożeń | 2 |
| 1 | Testy aplikacji web (Angular) — logika biznesowa 12+ funkcji krytycznych, WSTG/ASVS | 9 |
| 2 | Testy API (OWASP API Top 10, BOLA/IDOR, walidacja danych, integracje) | 5 |
| 3 | Uwierzytelnianie / autoryzacja operacji (MFA SMS/mail/mobilna), sesja, role demo↔pełne, SSO | 3 |
| 4 | Aplikacja Android (MASVS/MSTG: storage, IPC, traffic, pinning, hardening) | 5 |
| 5 | Aplikacja iOS (MASVS/MSTG) | 5 |
| 6 | Raportowanie: analiza, ocena ryzyka (CVSS), raport końcowy, QA | 4 |
| 7 | Zarządzanie: kickoff, statusy, spotkanie zamykające | 1 |
| | **Suma bazowa** | **34 MD** |

**Koszt bazowy = 34 × `[STAWKA_DZIENNA]`.**

Przykład (stawka poglądowa **2 000 PLN/MD**, do nadpisania):
34 MD × 2 000 PLN = **68 000 PLN netto**.

## 8. Opcje dodatkowe

| Opcja | MD | Uwaga |
|-------|----|-------|
| Retest poprawek po stronie klienta | +3 | Poza sumą bazową — specyfikacja nie wymienia retestów |

## 9. Harmonogram orientacyjny i zespół

- Przy zespole **2 pentesterów** część faz realizowana równolegle
  (np. web/API + mobile), kalendarzowo ~3–4 tygodnie robocze.
- Przy **1 pentesterze** ~7 tygodni roboczych (34 MD).
- Sugerowany podział kompetencji: 1 osoba web/API + 1 osoba mobile (Android/iOS).

## 10. Ryzyka i uwagi do potwierdzenia

- **Certificate pinning** w aplikacjach mobilnych może wymagać buildu testowego
  lub uzgodnienia metody obejścia; brak może wydłużyć fazy 4–5.
- **Płatności online** — testy w trybie sandbox; brak sandboxa ograniczy zakres
  testów procesu płatności.
- **Integracje** testowane z perspektywy Portalu; pełne testy systemów
  zewnętrznych (P1, PaymentCenter itd.) wymagałyby odrębnych zleceń.
- Intake określa aplikacje jako „webowe”, lecz stack i metodyki obejmują
  mobile — zakres mobilny (Android + iOS) potwierdzony z Zamawiającym.
- Estymacja zakłada stabilność środowiska nieprodukcyjnego; częste przestoje
  mogą wymagać korekty MD.
