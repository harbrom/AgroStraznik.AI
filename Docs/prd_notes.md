<conversation_summary>

<decisions>
1. Lokalizacja stacji definiowana jako punkt GPS ±100 m, wprowadzany ręcznie przez użytkownika.  
2. Dane pogodowe: temperatura, wilgotność, wiatr, opady; prognoza na 24 godziny.  
3. Domyślna częstotliwość odświeżania danych: co 60 minut (z możliwością konfiguracji w przyszłości).  
4. Generowanie krótkich alertów z uzasadnionymi rekomendacjami akcji rolniczych; w przyszłości – opcjonalny agent AI ograniczony tematycznie.  
5. Kluczowe metryki sukcesu: liczba użytkowników, liczba stacji, liczba pobrań danych pogodowych, liczba wygenerowanych rekomendacji, uptime.  
6. Retencja danych: 12 miesięcy, z opcją natychmiastowego usunięcia przez użytkownika; dłuższe okresy (1–10 lat) jako rozszerzenie.  
7. Uwierzytelnianie oparte na JWT z krótkim TTL i refresh tokenem przechowywanym w bazie.  
8. Minimalny schemat danych stacji: id, owner_id, nazwa, lat, lon, timezone, utworzono, zmieniono.  
9. Dane wyłącznie w czasie polskim (bez obsługi innych stref).  
10. Brak wersjonowania promptów AI na etapie MVP.  
11. Maks. 4 alerty push dziennie na użytkownika, z min. odstępem 60 minut.  
12. Brak wdrożonych backupów na etapie MVP – zaplanowane później.  
13. Monitoring błędów: alert e-mail przy error rate >1% lub timeoutach >5%; retry z backoff, circuit breaker i cache ostatnich danych.  
14. Feedback użytkownika: prosty przycisk „użyteczne/nieużyteczne” przy rekomendacji.  
15. Hosting: Hetzner (EU, niski koszt, brak wymagań RODO); budżet – wyłącznie plany Free.  
16. Testy: jednostkowe (logika rekomendacji), integracyjne (mock API pogodowego), e2e (rejestracja → stacja → dane → rekomendacja) z uruchamianiem w CI/CD.  
17. CI/CD: GitHub Actions z automatycznymi testami i deploymentem.  
18. SLA: uptime 99%, generowanie rekomendacji <5s średnio.  
19. Web push jako jedyny kanał powiadomień w MVP, architektura modułowa pod przyszłe kanały.  
20. Projekt jednoosobowy – pełna odpowiedzialność użytkownika za development i utrzymanie.  
</decisions>

<matched_recommendations>
1. Użycie OpenWeatherMap (Free plan, później upgrade) z kalkulatorem wywołań dla kontroli kosztów.  
2. Zdefiniowanie fallbacków: cache, komunikaty degradacji, retry/backoff.  
3. Modularna architektura do łatwego rozszerzenia AI, języków i kanałów komunikacji.  
4. Monitorowanie i alerty SRE z e-mail powiadomieniami.  
5. Prostota MVP – brak map, brak płatności, brak IoT.  
6. Struktura CI/CD z minimalnym zestawem testów dla stabilności pipeline’u.  
7. Zbieranie danych o jakości rekomendacji przez feedback użytkowników.  
8. Ograniczenie kosztów – korzystanie wyłącznie z planów Free w API i AI.  
9. Przyjęcie realistycznych SLA (99% uptime, <5s generowanie rekomendacji).  
10. Wybór taniego hostingu z EU (Hetzner) dla prostoty i zgodności.  
</matched_recommendations>

<prd_planning_summary>
AgroStrażnik.AI to aplikacja webowa umożliwiająca rolnikom dodawanie wirtualnych stacji pogodowych (na bazie punktów GPS ±100 m) i generowanie przez AI krótkich rekomendacji rolniczych w oparciu o dane pogodowe (temperatura, wilgotność, wiatr, opady, prognoza 24h).

Użytkownik po rejestracji (auth JWT) może dodać stację, której dane pobierane są co 60 minut z OpenWeatherMap. Następnie system generuje rekomendację AI (model gpt-3.5-turbo). Dashboard prezentuje listę stacji, aktualne dane i rekomendacje. W przypadku błędów – system korzysta z cache i mechanizmu retry. Aplikacja wysyła web push do 4 razy dziennie (min. 60 min między alertami).

Testy obejmują logikę rekomendacji, integrację z API i e2e workflow. Deployment przez GitHub Actions na Hetzner. Monitoring zapewnia e-mail alerty o błędach i statystyki użyteczności rekomendacji.  
Metryki sukcesu: liczba użytkowników, stacji, pobrań danych, rekomendacji i uptime.

Aplikacja działa w czasie polskim, dane użytkowników i współrzędne przechowywane są przez 12 miesięcy (z możliwością usunięcia). Budżet ograniczony do planów Free. Projekt prowadzony jednoosobowo.

</prd_planning_summary>

<unresolved_issues>
1. Brak decyzji o dokładnym formacie i strukturze rekomendacji AI (np. szablon tekstu, długość, ton komunikatu).  
2. Brak ustalenia UI/UX dashboardu (układ, priorytety informacji, tryb mobilny).  
3. Nie zdefiniowano jeszcze struktury bazy danych dla użytkowników i relacji z tabelą stacji.  
4. Brak decyzji o frameworku front-end/backend (np. React, Vue, FastAPI, Django, NestJS).  
5. Nie ustalono jeszcze systemu logowania błędów i analityki (np. Sentry, Prometheus).  
6. Nie zdefiniowano dokładnych endpointów API wewnętrznego i sposobu przechowywania kluczy zewnętrznych usług.  
</unresolved_issues>

</conversation_summary>
