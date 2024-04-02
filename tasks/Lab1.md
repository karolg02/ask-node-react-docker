# Administracja sieciami komputerowymi 2024

---

# Lab1

**Cel ćwiczenia: Wykorzystanie konteneryzacji do stworzenia zdalnego środowiska deweloperskiego dla rozproszonej aplikacji webowej na przykładzie platformy Docker**

## Uruchomienie platformy AWS Academy

1. Zaloguj się do platformy AWS Academy. Każdy powinien otrzymać zaproszenie na uczelniany adres mailowy.

   Dane dostępowe:

   - login: <nr_indeksu>@student.agh.edu.pl
   - password: ustalony przy rejestracji

2. Z zakładki dashboard wejdź do kursu PH summer 2022/23 (Learner lab) i wybierz sekcję modules. Znajdują się w niej materiały przedstawiające korzystanie z platformy (Student Guide.pdf), terminal z dostępem do AWS (Learner lab) oraz ankieta podsumowująca (ja nie widzę wyników).
3. Uruchom konsolę wchodząc w Learner lab. Może być wymagane wyrażenie zgody na warunki korzystania z usługi. Rezultat, będący początkiem ćwiczenia, przedstawiono na poniższym rysunku.

   ![Learner lab console](/res/learner-lab-console.png)

4. By rozpocząć pracę z platformą, należy wystartować lab przyciskiem _Start lab_. Gdy przy linku do AWS zapali się zielona kontrolka, platforma jest gotowa do pracy. **UWAGA! Po zakończeniu pracy proszę wyłączać platformę przyciskiem _End lab_, by uniknąć niepotrzebnego wykorzystania środków**.

5. Wszelkie uwagi co do korzystania z platformy zostały zapisane w sekcji Readme, wyświetlanej domyślnie. Gdy nie jest ona widoczna, należy nacisnąć przycisk _Readme_.

## Zadania

**UWAGA! W ćwiczeniu należy korzystać wyłącznie z systemów Linuxowych**

Przydatne polecenia można znaleźć w [dockumentacji](https://docs.docker.com/engine/reference/commandline/cli/) oraz w [Docker cheat sheet](https://docs.docker.com/get-started/docker_cheatsheet.pdf).

### Przygotowanie platformy [1 pkt]

1. Stwórz maszynę wirtualną do pracy. Skorzystaj ze dostarczonego [opisu](/infra/README.md).
2. Jeżeli wszystkie czynności zostały wykonane prawidłowo, poniższe polecenie powinno wykonać się bez błędów i zwrócić pustą tabelę z uruchomionymi kontenerami.

   ```bash
   docker ps
   ```

   Przykładowy zrzut ekranu z konsoli:
   ![ssh to vm](/res/ssh-verify.png)

3. Wybierz preferowany sposób pracy: przez terminal lub IDE. W [opisie](/infra/README.md) konfiguracji infrastruktury znajduje się opis dla VS Code.
4. Skolnuj to repozytorium i przejdź do jego katalogu głównego (ask-node-react-docker, nazwa repozytorium).

---

### Baza danych [2 pkt]

1. Pobierz obraz bazy danych z repozytorium DockerHub na maszynę. Wspierane przez aplikację bazy to mysql lub mariadb.
2. Uruchom bazę danych w kontenerze, inicjalizując ją danymi ze skryptu [db.sql](/db/db.sql). Są trzy możliwe rozwiązania:
   - Stworzenie kontenera z bazą a następnie wczytanie skryptu z wykorzystaniem polecenia docker exec. Przykład dla mysql:
     ```bash
     docker exec -i <nazwa_kontenera> sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < /path/to/db.sql
     ```
   - Stworzenie własnego obrazu, w którym skrypt [db.sql](/db/db.sql) zostanie skopiowany do katalogu /docker-entrypoint-initdb.d w kontenerze.
   - Zmapowanie skryptu do kontenera jako wolumin
3. Zweryfikuj prawidłowe działanie kontenera z bazą za pomocą polecenia `docker ps`. Jeżeli kontener nie widnieje jako uruchomony, wówczas nastąpił błąd. Tip: `docker logs <nazwa_kontenera>` wyświetli na ekran logi kontenera, w których znaleźć można opisy błędów.
4. Zweryfikuj prawidłowy import bazy danych poleceniem:
   ```bash
   docker exec -it <nazwa_kontenera> mysql -udockerdb -p
   ```
   Należy podać hasło do bazy. Znaleźć je można w [skrypcie](/db/db.sql#L19)
5. Jeżeli udało się zalogować do bazy danych, sprawdź czy znajduje się tam baza o nazwie dockerdb. Jeżeli tak, wówczas czynności uruchomienia bazy w kontenerze zostały wykonane prawidłowo.

---

### Aplikacja backendowa [2 pkt]

1. Pobierz obraz nodejs z repozytorium DockerHub na maszynę. Wersja powinna być nie wyższa niż 16.
2. W katalogu server stwórz plik Dockerfile i uzupełnij go odpowiednimi instrukcjami, pozwalającymi na uruchomienie aplikacji.
   - Jako polecenie wykonawcze (CMD/ENTRYPOINT) wskaż `npm run dev`.
   - _Nie kopiuj_ kodu źródłowego!
3. Zbuduj obraz poleceniem `docker build -t <obraz_backendu>:0.1 .` w katalogu server.
4. Uruchom kontener z backendem poleceniem `docker run`. Uwzględnij:
   - Uruchom kontener tak, by nie blokował konsoli (deatched, -d).
   - Na potrzeby testów możesz opublikować port (-p), by zweryfikować poprawność działania aplikacji.
   - Wykorzystaj zmienne środowiskowe w pliku .env (-e lub --env-file).
   - Zmapuj kod źródłowy aplikacji jako wolumin do kontenera (-v i/lub `docker volume`).
5. Sprawdź, czy kontener jest uruchomiony i zweryfikuj poprawność jego działania, odwołując się do endpointu /api, który powinien zwrócić obiekt JSON z tekstem _Hello world!_.
6. Zmodyfikuj treść komunikatu zwracanego przez endpoint /api w pliku [api/index.ts](/server/src/api/index.ts#L8). Zapisz plik.
7. Jeżeli endpoint zwraca zmieniony tekst, wszystkie czynności zostały wykonane prawidłowo.

---

### Aplikacja frontendowa [2 pkt]

1. Pobierz lub wykorzystaj obraz nodejs z poprzedniej części. Wersja powinna być nie wyższa niż 16.
2. W katalogu client stwórz plik Dockerfile i uzupełnij go odpowiednimi instrukcjami pozwalającymi na uruchomienie.
   - Jako polecenie wykonawcze (CMD/ENTRYPOINT) wskaż `npm start`
   - _Nie kopiuj_ kodu źródłowego!
3. Zbuduj obraz poleceniem `docker build -t <obraz_frontendu>:0.1 .` w katalogu client.
4. Uruchom kontener z frontendem poleceniem `docker run`. Uwzględnij:
   - Uruchom kontener tak, by nie blokował konsoli (deatched, -d).
   - Opublikuj port (-p). Domyślny port aplikacji to 3000. Zmapuj go na 3333.
   - By klient uruchomił się poprawnie, niezbędne jest ustawienie zmiennej CI=true (-e).
   - Zmapuj kod źródłowy aplikacji z katalogu src jako wolumin do kontenera (-v).
   - Zmapuj katalog public jako wolumin do kontenera (-v).
5. Sprawdź, czy kontener jest uruchomiony i zweryfikuj poprawność jego działania, wchodząc pod adres maszyny (lub localhost przy port-forwardingu z VS Code) na port 3333.
6. Zmodyfikuj tekst dowolnego z dwóch przycisków: [_Load data_](/client/src/components/Layout/Layout.jsx#L30) lub [_Flush data_](/client/src/components/Layout/Layout.jsx#L35). Zapisz plik.
7. Jeżeli wyświetli się logo Reacta i dwa przyciski (mogą być ukryte po naciśnięciu logo) ze zmienionym tekstem, czynności zostały wykonane prawidłowo.

---

### Część problemowa: połączenie wszystkich komponentów aplikacji [3 pkt]

1. Bez wprowadzenia dodatkowych modyfikacji, działanie przycisku _Load data_ zwraca błąd (timeout lub internatl server error).
2. Połącz ze sobą kontenery backendu oraz bazy danych.
   - Aplikacja wykorzystuje zmienne środowiskowe do konfiguracji bazy. Ustaw zmienną [DB_HOST](/server/.env#L7) na właściwą.
   - Niektóre zmiany mogą wymagać ponownego stworzenia kontenera. Jeżeli to konieczne, usuń niewłaściwy i stwórz nowy.
3. Zweryfikuj poprawność połączenia, odpytując endpoint /api/products. Powinien on zwrócić listę produktów. Jeżeli występuje błąd, wykorzystaj polecenie `docker logs`, by znaleźć przyczynę.
4. Połącz ze sobą kontenery frontendu i backendu.
   - Aplikacja wykorzystuje wewnętrzny serwer proxy. Adres backendu, do którego proxowane są zapytania znajduje się w pliku [package.json](/client/package.json#L9) klienta.
   - Niektóre zmiany mogą wymagać ponownego stworzenia kontenera. Jeżeli to konieczne, usuń niewłaściwy i stwórz nowy.
5. Zweryfikuj poprawność połączenia poprzez załadowanie danych przyciskiem _Load data_. Finalny rezultat:
   ![ui-preview](res/ui-preview.png)
6. Jeżeli obydwa przyciski działają prawidłowo i wszystkie aplikacje uruchomione zostały jako kontenery dockera, wszystkie czynności w ramach tego ćwiczenia zostały wykonane prawidłowo.

Tip: Do wykonania tej sekcji przydatne będzie utworzenie własnej sieci w dockerze poleceniem `docker network` (lepsze rozwiązanie) lub `docker inspect` jeżeli kontenery umieszczone zostały w domyślnej sieci (gorsze rozwiązanie).

---

# Uwagi

1. Jako potwierdzenie wykonania ćwiczenia należy wykonać sprawozdanie w formie docx (pdf, preferowane) lub markdown.
2. Należy umieścić wszystkie niezbędne zdjęcia i opisy pokazujące przebieg ćwiczenia.
3. Sprawozdanie nie musi być bardzo szczegółowe, jednak powinno odzwierciedlać sekwencję wykonywanych kroków oraz uzasadnienie, co i dlaczego się stało.
4. Sprawozdanie należy umieścić na platformie UPEL do następnych zajęć.
5. Pozostałe dwa laboratoria wymagają wiedzy z tego ćwiczenia, gdyż będą wykorzystywać tą samą aplikację.

---

# Linki

1. https://docs.docker.com/network/network-tutorial-standalone/
2. https://www.digitalocean.com/community/questions/how-to-ping-docker-container-from-another-container-by-name