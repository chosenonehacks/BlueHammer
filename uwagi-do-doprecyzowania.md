# Uwagi i pytania do doprecyzowania — pentesty Portal Pacjenta i PRINTLUX.PL

Dokument zbiera otwarte kwestie, nieścisłości i braki w formularzach intake,
które należy doprecyzować z opiekunami aplikacji **przed potwierdzeniem zakresu
i estymacji**. Odpowiedzi mogą zmienić liczbę osobodni (MD) — przy każdym
punkcie zaznaczono potencjalny wpływ.

> Legenda wpływu: 🔴 duży (zmiana zakresu/MD) · 🟡 średni · 🟢 informacyjny.

---

## A. Portal Pacjenta

### A1. Aplikacje mobilne — czy są w zakresie? 🔴
Formularz nazywa rozwiązanie „aplikacją webową", ale stack technologiczny
i metodyki obejmują mobile (Android Kotlin 2.2.20 / Java 17, iOS Swift 5,
OWASP Mobile Top 10 / MASVS).
- **Czy aplikacje mobilne (Android, iOS) faktycznie istnieją i mają wejść
  w zakres testów?** (Web+API / Web+API+jeden mobile / Web+API+oba mobile)
- Jeśli tak — czy to natywni klienci, czy WebView/hybryda (np. ten sam Angular
  opakowany)? Ma to wpływ na metodykę i MD.

### A2. Certificate pinning w aplikacjach mobilnych 🔴
Kluczowe dla możliwości i czasu testów mobilnych.
- **Czy w buildzie testowym certificate pinning będzie wyłączony / czy
  dostarczycie build bez piningu**, tak aby tester mógł przechwytywać i
  analizować ruch (MITM)?
- Jeśli pinning **pozostaje aktywny** — czy zakres ma obejmować **próbę jego
  obejścia** (bypass, np. Frida/patch), czy testujemy aplikację **z założeniem
  zaufanego kanału** i pomijamy walkę z piningiem?
  - Wariant „walczymy z piningiem" zwiększa MD faz mobilnych.
- Czy dopuszczacie instalację na **zrootowanym/jailbreak** urządzeniu lub
  emulatorze? Czy aplikacja ma detekcję root/jailbreak, którą trzeba obejść?

### A3. Konta testowe i role 🟡
- Potwierdzenie dostępu do **min. 2 kont na każdą rolę** (demo, pełne) —
  niezbędne do testów IDOR/BOLA między pacjentami.
- Czy konto **demo** (bez potwierdzonej tożsamości) ma realnie ograniczony
  zestaw funkcji — prosimy o listę funkcji dostępnych dla demo vs pełne.
- Czy możliwy jest **samodzielny reset/odtwarzanie** kont i danych testowych?

### A4. MFA / autoryzacja operacji 🟡
Autoryzacja operacji: kod SMS / e-mail / autoryzacja mobilna.
- Czy na kontach testowych **działają realnie** kanały SMS/e-mail/mobilny
  (czy dostaniemy numery/skrzynki testowe)?
- Czy istnieje **tryb testowy/sandbox** kodów (np. stały kod), czy trzeba
  korzystać z realnej bramki?

### A5. Płatności online (PaymentCenter) 🔴
- Czy płatności działają w **trybie sandbox** (testowe karty/przelewy)?
- Czy testujemy **pełny proces płatności** (logika kwot, manipulacja
  parametrami, ponowienia, statusy), czy tylko integrację po stronie Portalu?
- Brak sandboxa istotnie ogranicza zakres i może zmienić MD.

### A6. Integracje (11 komponentów) 🟡
SSO, 4Net, CCS, P1, PaymentCenter, Alfavox, ePOZ, Codigital, Medalia, Zowie, ApteGO.
- Potwierdzenie, że integracje testujemy **wyłącznie z perspektywy Portalu
  Pacjenta** (granice zaufania, przekazywane parametry), a nie jako odrębne
  systemy.
- Czy w środowisku nieprodukcyjnym **wszystkie** integracje są podłączone i
  działające? Które mogą być zaślepione (mock)?
- **SSO** — jaki standard/provider (OIDC/SAML)? Czy logowanie do Portalu
  przechodzi przez zewnętrzny IdP, który jest poza zakresem?

### A7. Środowisko i stabilność 🟡
- Dostęp do środowiska nieprodukcyjnego: VPN/sieć, godziny dostępności.
- Czy środowisko nieprodukcyjne **odpowiada produkcyjnemu** pod kątem
  konfiguracji bezpieczeństwa (nagłówki, TLS, hardening)?
- Okno serwisowe / kontakt do eskalacji przy blokadach.

### A8. Zakres warstw 🟢
- Potwierdzenie, że **infrastruktura jest poza zakresem** (wg intake — Nie).
- Czy back-end to jeden API, czy wiele usług (.NET 8 oraz .NET Framework 4.8)
  — czy obie warstwy są w zakresie?

---

## B. PRINTLUX.PL

### B1. WAF (Wordfence) — sprzeczność w formularzu 🔴
W intake występuje rozbieżność:
- Sekcja **„Zespół Testów Bezpieczeństwa"**: aplikacja **NIE** jest za WAF-em.
- Sekcja **opiekuna**: jako kontrola wymieniony **Wordfence WAF** (filtrowanie
  żądań, blokady).

Do ustalenia:
- **Czy Wordfence WAF będzie aktywny podczas testów?**
- Jeśli tak — czy możliwe jest jego **wyłączenie na czas testów** lub
  **dodanie IP testerów do whitelisty**?
- Jeśli WAF ma pozostać aktywny — czy zakres obejmuje **testy obejścia WAF**?
  (zwiększa MD i wpływa na interpretację wyników).

### B2. Autoryzacja testów na obcym hostingu 🔴
Hosting zewnętrzny: Agencja HENNESSEY (Małgorzata Rachwał, mr@hennessey.pl),
obca domena.
- Czy uzyskano **pisemną zgodę właściciela hostingu** na przeprowadzenie
  testów bezpieczeństwa? (warunek konieczny rozpoczęcia).
- Czy obowiązują **ograniczenia czasowe/wolumenowe** ze strony dostawcy
  (rate limity, zakaz skanów automatycznych)?

### B3. Upload plików — przetwarzanie po stronie serwera 🔴
Przyjmowane typy: `.xlsx`, `.jpeg`, `.jpg`, `.png`, `.pdf`, `.eps`, `.tiff`.
- **Czy pliki są przetwarzane/konwertowane po stronie serwera** (np. generowanie
  podglądu, parsowanie xlsx, render eps/tiff/pdf)? Jeśli tak — rozszerza to
  powierzchnię ataku i potencjalnie MD.
- Gdzie pliki są zapisywane i czy są **dostępne publicznie** po wgraniu?
- Jakie są limity rozmiaru/typu i czy jest skan antywirusowy?

### B4. Środowisko testowe 🟡
- Potwierdzenie istnienia **środowiska testowego odpowiadającego produkcyjnemu**
  (URL, dostęp).
- Czy testujemy na środowisku testowym, czy istnieje ryzyko, że to instancja
  produkcyjna? (ważne przy testach destrukcyjnych/upload).

### B5. Wersja PHP i inwentaryzacja 🟡
- **Faktyczna wersja PHP serwera** (intake: „nieudokumentowana w repo").
- Lista zainstalowanych **wtyczek i motywów** (lub dostęp admin do
  inwentaryzacji) — kluczowe dla oceny znanych CVE.

### B6. Konta i role 🟢
- Dostęp do kont: min. **2 konta użytkownika** (kontrola dostępu/IDOR) oraz
  **1 konto administratora**.
- Czy działa **2FA e-mail** na kontach testowych (skrzynki testowe)?
- Zakres panelu administracyjnego — czy admin jest w pełnym zakresie testów?

---

## C. Kwestie wspólne (obie aplikacje)

### C1. Retesty 🟡
Żadna ze specyfikacji nie wymienia retestów.
- **Czy retest poprawek ma być częścią zlecenia?** Jeśli tak — w bazie czy jako
  opcja? (obecnie wyceniony jako opcja poza sumą bazową:
  Portal Pacjenta +3 MD, PRINTLUX.PL +1 MD).

### C2. Stawka dzienna i forma wyceny 🟡
- **Stawka dzienna (PLN/MD)** do uzupełnienia w dokumentach estymacji
  (obecnie placeholder `[STAWKA_DZIENNA]`).

### C3. Okno czasowe i dostępność 🟢
- Preferowane **terminy realizacji** i deadline raportu.
- Czy są okna, w których testów **nie wolno** prowadzić (np. godziny szczytu)?

### C4. Forma i odbiorcy raportu 🟢
- Format raportu (PL/EN), oczekiwana skala ocen (CVSS v3.1 / v4.0).
- Lista dystrybucyjna i kanał przekazania (szyfrowanie raportu).

### C5. Dane kontaktowe i eskalacja 🟢
- Osoby kontaktowe po stronie LUX MED dla każdej aplikacji.
- Procedura zgłaszania **krytycznych podatności w trakcie** testów
  (fast-track przed raportem końcowym).

---

## Podsumowanie pozycji o największym wpływie na estymację (🔴)

| # | Kwestia | Aplikacja | Wpływ |
|---|---------|-----------|-------|
| A1 | Czy mobile (Android/iOS) w zakresie | Portal Pacjenta | ±10–16 MD |
| A2 | Cert pinning: zdjęty vs walka z bypassem | Portal Pacjenta | ± kilka MD (fazy mobilne) |
| A5 | Sandbox płatności | Portal Pacjenta | zakres testów płatności |
| B1 | Wordfence WAF aktywny / wyłączony | PRINTLUX.PL | +0,5–1 MD przy aktywnym WAF |
| B2 | Pisemna zgoda hostingu (HENNESSEY) | PRINTLUX.PL | warunek startu |
| B3 | Serwerowe przetwarzanie uploadów | PRINTLUX.PL | możliwe rozszerzenie fazy 3 |
