# Dokument wymagań produktu (PRD) - AgroStrażnik.AI

## 1. Przegląd produktu

AgroStrażnik.AI to aplikacja webowa skierowana do rolników i właścicieli pól, umożliwiająca tworzenie „wirtualnych stacji pogodowych”.  
Każda stacja reprezentuje punkt GPS (±100 m) i pobiera dane z publicznego API pogodowego (np. OpenWeatherMap), które są następnie analizowane przez moduł AI w celu wygenerowania krótkich rekomendacji rolniczych.

System ma działać w czasie polskim, z prostym interfejsem webowym, wykorzystując JWT do uwierzytelniania użytkowników i web push do powiadomień.  
Projekt prowadzony jest jednoosobowo i hostowany na serwerze Hetzner (EU).  
Priorytetem MVP jest prostota, stabilność i niskie koszty utrzymania (preferowane plany Free).

---

## 2. Problem użytkownika

Rolnicy i właściciele pól często nie mają łatwego dostępu do lokalnych prognoz i analiz pogody dostosowanych do ich konkretnych lokalizacji.  
Dostępne aplikacje rolnicze są często zbyt rozbudowane lub wymagają fizycznych czujników IoT, co zwiększa koszt i złożoność.

AgroStrażnik.AI rozwiązuje ten problem, umożliwiając:
- szybkie utworzenie wirtualnej stacji pogodowej bez sprzętu,
- uzyskanie danych o temperaturze, wilgotności, wietrze i opadach w oparciu o dokładną lokalizację,
- otrzymywanie inteligentnych rekomendacji AI (np. czy dziś warto wykonać oprysk),
- otrzymywanie powiadomień push tylko wtedy, gdy warunki są istotne dla użytkownika.

---

## 3. Wymagania funkcjonalne

### 3.1 Autoryzacja i zarządzanie użytkownikiem
- Logowanie i rejestracja użytkowników z wykorzystaniem JWT (krótki TTL) i refresh tokenów przechowywanych w bazie.
- Możliwość unieważnienia refresh tokenów.
- Dane użytkownika: id, email, hasło (hashowane), data rejestracji, ostatnie logowanie.

### 3.2 Zarządzanie stacjami pogodowymi (CRUD)
- Dodanie nowej stacji (nazwa, współrzędne GPS ±100 m, timezone = Europe/Warsaw).
- Edycja i usuwanie stacji.
- Lista wszystkich stacji użytkownika z bieżącymi danymi pogodowymi.
- Minimalny schema: id, owner_id, nazwa, lat, lon, timezone, utworzono, zmieniono.

### 3.3 Pobieranie danych pogodowych
- Dane: temperatura, wilgotność, wiatr, opady, prognoza 24h.
- Częstotliwość aktualizacji: co 60 minut.
- API: OpenWeatherMap (plan Free, z możliwością przejścia na płatny).
- Mechanizmy niezawodności: retry z backoff, circuit breaker, cache ostatnich poprawnych danych.

### 3.4 Generowanie rekomendacji AI
- Model: gpt-3.5-turbo (możliwość łatwej zamiany modelu w przyszłości).
- Format: krótki opis sytuacji + rekomendacja (np. „Wysoka wilgotność, oprysk niewskazany”).
- System zbiera dane o jakości rekomendacji (przycisk „użyteczne/nieużyteczne”).
- Limit rekomendacji: maks. 4 alerty dziennie, minimum 60 minut przerwy między nimi.

### 3.5 Dashboard użytkownika
- Widok listy wszystkich stacji z aktualnymi danymi i rekomendacją AI.
- Status synchronizacji (ostatnia aktualizacja, błędy).
- Przycisk „Usuń dane” umożliwiający natychmiastowe skasowanie konta lub danych stacji.

### 3.6 Powiadomienia
- Web push jako jedyny kanał w MVP.
- Kolejkowanie komunikatów, by nie przekraczać limitu alertów dziennie.

### 3.7 Dane i prywatność
- Retencja danych: 12 miesięcy.
- Automatyczne usunięcie danych po upływie okresu lub na żądanie użytkownika.
- Dłuższe okresy (1–10 lat) planowane jako opcja rozszerzona.
- Brak wymagań RODO, ale zgodność z praktykami EU-hostingu.

### 3.8 Monitoring i niezawodność
- E-mail alerty przy:
  - error rate > 1% w ciągu godziny,
  - timeoutach > 5%.
- Retry i cache awaryjny w przypadku błędów API.
- SLA: uptime 99%, czas generowania rekomendacji <5 s średnio.

### 3.9 CI/CD i testy
- GitHub Actions do automatycznych testów i deploymentu.
- Testy:
  - jednostkowe (logika rekomendacji),
  - integracyjne (mock API pogodowego),
  - e2e (rejestracja → stacja → dane → rekomendacja).

---

## 4. Granice produktu

### Poza zakresem MVP
1. Brak fizycznych czujników IoT (tylko dane z API).
2. Brak integracji MQTT lub innych protokołów sprzętowych.
3. Brak wizualizacji na mapie i wykresów historycznych.
4. Brak systemu płatności i subskrypcji.
5. Brak obsługi wielu stref czasowych (tylko Polska).
6. Brak zaawansowanego planera działań rolniczych.
7. Brak systemu backupów (zaplanowane w kolejnych etapach).

---

## 5. Historyjki użytkowników

### US-001 – Rejestracja i logowanie
**Opis:**  
Jako użytkownik chcę się zarejestrować i zalogować, aby móc korzystać z aplikacji i zarządzać stacjami.  
**Kryteria akceptacji:**  
- Możliwość rejestracji konta z e-mailem i hasłem.  
- Logowanie zwraca token JWT i refresh token.  
- Sesja automatycznie wygasa po TTL.  
- Możliwość odświeżenia sesji bez ponownego logowania.  

---

### US-002 – Dodanie stacji pogodowej
**Opis:**  
Jako użytkownik chcę dodać wirtualną stację pogodową, aby śledzić lokalne dane pogodowe.  
**Kryteria akceptacji:**  
- Formularz z nazwą i współrzędnymi GPS.  
- Dane zapisują się w bazie (z owner_id).  
- System automatycznie zaczyna pobierać dane pogodowe co 60 minut.  

---

### US-003 – Podgląd stacji i danych
**Opis:**  
Jako użytkownik chcę zobaczyć listę moich stacji z aktualnymi danymi pogodowymi.  
**Kryteria akceptacji:**  
- Widok listy stacji z nazwą, temperaturą, wilgotnością, wiatrem i opadami.  
- Widok aktualizuje się po pobraniu nowych danych.  

---

### US-004 – Generowanie rekomendacji AI
**Opis:**  
Jako użytkownik chcę otrzymać krótką rekomendację AI na podstawie danych pogodowych, aby wiedzieć, jakie działania rolnicze są zalecane.  
**Kryteria akceptacji:**  
- Rekomendacja generowana po pobraniu danych.  
- Czas generacji ≤ 5 s.  
- Możliwość oceny rekomendacji jako „użyteczne” lub „nieużyteczne”.  

---

### US-005 – Powiadomienia push
**Opis:**  
Jako użytkownik chcę otrzymywać powiadomienia o ważnych zmianach pogodowych lub rekomendacjach, aby nie przegapić kluczowych momentów.  
**Kryteria akceptacji:**  
- Maks. 4 alerty dziennie.  
- Min. 60 minut między powiadomieniami.  
- Powiadomienia są wysyłane w czasie polskim.  

---

### US-006 – Usunięcie danych
**Opis:**  
Jako użytkownik chcę móc natychmiast usunąć dane moich stacji lub konta, aby mieć kontrolę nad prywatnością.  
**Kryteria akceptacji:**  
- Przycisk „Usuń dane” powoduje natychmiastowe skasowanie danych z bazy.  
- Dane są również automatycznie usuwane po 12 miesiącach.  

---

### US-007 – Monitoring i niezawodność
**Opis:**  
Jako administrator chcę otrzymywać powiadomienia e-mail, gdy aplikacja przekracza próg błędów lub timeoutów, aby reagować na problemy.  
**Kryteria akceptacji:**  
- Alert e-mail przy error rate > 1% w 1h lub timeoutach > 5%.  
- Retry/backoff i cache w razie błędów API.  

---

## 6. Metryki sukcesu

1. **Liczba użytkowników z aktywnymi stacjami** – wskaźnik adopcji.  
2. **Liczba stacji pogodowych** – wskaźnik zaangażowania.  
3. **Liczba wygenerowanych rekomendacji** – miara użycia AI.  
4. **Odsetek pozytywnych ocen rekomendacji („użyteczne”) ≥ 50%** – wskaźnik jakości AI.  
5. **Uptime ≥ 99%** – wskaźnik stabilności.  
6. **Średni czas generacji rekomendacji ≤ 5 s** – wskaźnik wydajności.  
7. **Error rate ≤ 1%** – wskaźnik niezawodności systemu.

---
