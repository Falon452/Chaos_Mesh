# Chaos_Mesh

## Strona tytułowa

**Akronim:** Chaos Mesh  
**Tytuł:** Orchestrate complex fault scenarios   
**Autorzy:** Kuba Janczarski, Jan Rodzoń, Rafał Rodzoń, Maria Tkocz, Damian Tworek  
**Rok, Grupa:** 2024 Grupa 1

## 1. Wstęp

Chaos Mesh to narzędzie służące do testowania odporności systemów na awarie i nieprzewidywalne warunki w środowiskach produkcyjnych. Jest to rodzaj narzędzia do Chaos Engineering, które umożliwia inżynierom testowanie zachowania aplikacji w warunkach, które mogą być chaotyczne lub niestabilne. Eksperymenty przeprowadzane są w klastrze Kubernetes.

Do symulowanych rodzajów awarii należą:
- zakłocenia sieciowe, np. opóźnienia, utraty pakietów, przepuszczalność,
- zakłócenia systemowe, np. awarie dysków, zatrzymania procesów, wysokie zużycie zasobów (CPU, pamięć, dysk),
- zakłócenia procesów aplikacji, np. zabijanie procesów, zatrzymywanie serwisów, zmiany konfiguracji,
- utrata węzła.

Chaos Mesh dostarcza również narzędzia do monitorowania i analizy awarii, co pozwala na szybsze wykrywanie i rozwiązywanie problemów w klastrze Kubernetes.

Poprzez przeprowadzanie eksperymentów z użyciem Chaos Mesh zespoły deweloperskie mogą zrozumieć, jak aplikacja reaguje na zdarzenia powodujące awarie i zwiększyć jej odporność na tego typu sytuacje.

## 2. Tło teoretyczne

Chaos Mesh oferuje różne typy eksperymentów dla Kubernetesa:

- awaria pod'a,
- awaria sieci,
- stress testy,
- błędy przy zapisie/odczycie plików,
- awarie DNS,
- awarie czasowe,
- awarie JVM,
- błędy kernela Linuxa,
- błędy AWS, Azure, Google Cloud Platform,
- błędy HTTP,
- zdarzenia blokujące związane z urządzeniami.

Eksperymenty można uruchamiać cyklicznie lub jednorazowo.

Jako wiodąca platforma testów Chaos w branży Chaos Mesh ma następujące główne zalety:

- **stabilne podstawowe funkcje**: Chaos Mesh wywodzi się z platformy testowej TiDB i odziedziczył wiele istniejących doświadczeń z testów TiDB od chwili swojego pierwszego wydania,
- **w pełni uwierzytelniony**: Chaos Mesh jest używany w licznych firmach i organizacjach, a także w systemach testowych wielu znanych systemów rozproszonych, np. Apache APISIX i RabbitMQ,
- **łatwy w użyciu**: Chaos Mesh w pełni wykorzystuje automatyzację za pomocą operacji graficznych i opartych na Kubernetesie,
- **Cloud Native**: Chaos Mesh wspiera środowisko Kubernetes dzięki swojej potężnej zdolności automatyzacji,
- **różnorodne scenariusze symulacji awarii**: Chaos Mesh obejmuje większość scenariuszy podstawowej symulacji awarii w systemie testowym rozproszonym,
- **elastyczne możliwości orkiestracji eksperymentów**: możliwość projektowania własnych scenariuszy eksperymentów Chaos (w tym eksperymentów mieszanych) oraz sprawdzania stanu aplikacji,
- **bezpieczeństwo**: Chaos Mesh został zaprojektowany z wieloma warstwami kontroli bezpieczeństwa i zapewnia wysoki poziom bezpieczeństwa,
- **łatwa skalowalność**: dodawanie nowych typów testów awarii i funkcji do Chaos Mesh jest łatwe,
- **aktywna społeczność**: Chaos Mesh to inkubowany projekt hostowany przez CNCF. Ma coraz większą liczbę współtwórców i użytkowników na całym świecie.

## Analiza wybranych rodzajów eksperymentów

### 2.1. Awarie JVM (JVM Application Faults)

#### Akcja Return 

Jako helloworld Chaos Mesh podaje następujący przykład akcji Return

```java
public class Main {
    public static void main(String []args) {
        for(int x = 0; x < 1000; x = x+1) {
            try {
                Thread.sleep(1000);
                Main.sayhello(x);
            } catch (Exception e) {
                System.out.println("Got an exception!" + e);
            }
        }
    }

    public static void sayhello(int num) throws Exception {
        try {
            num = getnum(num);
            String s=String.valueOf(num);
            System.out.println(s + ". Hello World");
        } catch (Exception e) { 
            throw e;
        }
    }

    public static int getnum(int num) {
        return num;
    }
}
```

Ten program będzie wypisywał kolejno:

```java
0. Hello World
1. Hello World
2. Hello World
```

Możemy zmienić zachowanie tego programu używając JVMChaos, w taki sposób, żeby zawsze metoda getnum zwracała 9999.

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: JVMChaos
metadata:
 name: return
 namespace: helloworld
spec:
 action: return
 class: Main
 method: getnum
 value: '9999'
 mode: all
 selector:
   namespaces:
     - helloworld
```

Wstrzykując poniższą metodą:

```
kubectl apply -f ./jvm-return-example.yaml
```

program wypisuje:

``` java
9999. Hello World
9999. Hello World
9999. Hello World
```

#### Akcja Latency

Akcja Latency polega na zwiększeniu czasu wykonywania konkretnej metody w klasie mierzonej w milisekundach.

#### Akcja Exception

Akcja Exception pozwala na nadpisanie domyślnego zachowania metody i rzucenie wyjątku podanego w akcji.

#### Akcja Stress

Akcja Stress pozwala obciążyć maszynę JVM. Można obciążyć pamięć lub procesor. W przypadku pamięci możemy obciążyć zarówno stack jak i heap.

#### Akcja gc

Akcja gc wymusza garbage collection

#### Akcja ruleData

Akcja ruleData pozwala na odpalenie innych akcji poprzez użycie plików konfiguracyjnych Bytemana, którego chaos mesh pod spodem używa.


### 2.2. Awarie sieci (Network Faults)

#### Akcja Delay

Akcja Delay pozwala na ustawienie opóźnienia na sieci

#### Akcja Partition

Akcja Partition pozwala na ustawienie całkowitego blokowania połączeń między elementami

#### Akcja Bandwidth

Akcja Bandwidth pozwala na ustawienie limitu przepustowości między elementami

#### Akcja Reorder

Akcja Reorder pozwala na ustawienie szansy na przestawienie kolejności otrzymania pakietów

#### Akcja Loss

Akcja Loss pozwala na ustawienie symulacji gubienia części przesyłanych pakietów

#### Akcja Duplicate

Akcja Duplicate pozwala na ustawienie symulacji duplikacji części przesyłanych pakietów

#### Akcja Corrupt

Akcja Corrupt pozwala na ustawienie symulacji uszkodzenia części przesyłanych pakietów

### 2.3. Stress testy

Chaos Mesh pozwala na przeprowadzanie eksperymentów StressChaos pozwalających na symulację scenariuszy stresu wewnątrz kontenerów. Eksperymenty można tworzyć przy użyciu Chaos Dashboard lub bezpośrednio w plikach konfiguracyjnych *.yaml.

W eksperymentach StressChaos dostępne są następujące parametry:
- rodzaj stresu: CPU, pamięć,
- planowany czas trwania eksperymentu,
- tryb eksperymentu: losowy pod lub wybór spośród kwalifikujących się podów – wszystkie, wybrana liczba, wybrany/losowy procent,
- nazwa kontenera, do którego wstrzykiwany jest test,
- selektor określający docelowy pod.

#### 2.3.1. Obciążenie pamięci
W przypadku testów obciążeń pamięci - **MemoryStressor** - dostępne są następujące parametry:
- ```workers``` – liczba wątków obciążających pamięć,
- ```size``` – ostateczny rozmiar zajętej pamięci lub procent całkowitego rozmiaru pamięci,
- ```time``` – czas osiągnięcia rozmiaru pamięci określonego w size. Model wzrostu jest modelem liniowym,
- ```oomScoreAdj``` – modyfikacja wyniku procesu, którego używa linuxowy Out-Of-Memory Killer do określenia, który proces zabić w pierwszej kolejności. Wartość -1000 – proces jest całkowicie chroniony przed zabiciem, wartość +1000 – proces jest najbardziej prawdopodobnym kandydatem do zabicia w przypadku braku pamięci.

#### 2.3.2. Obciążenie procesora
W przypadku testów obciążeń procesora - **CPUStressor** - dostępne są następujące parametry:
- ```workers``` – liczba wątków obciążających procesor,
- ```load``` – procent zajętości procesora. Wartość 0 – nie jest dodawany żaden dodatkowy procesor, wartość 100 – pełne obciążenie. Ostateczna suma obciążenia procesora to workers * load.

#### 2.3.3 Przykładowa konfiguracja stress testu
Przykładowy eksperyment zdefiniowany w pliku *.yaml: utworzenie procesu w wybranym kontenerze, który będzie stale przydzielał oraz odczytywał i zapisywał w pamięci, zajmując do 256MB pamięci:

```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: memory-stress-example
  namespace: chaos-mesh
spec:
  mode: one
  selector:
    labelSelectors:
      'app': 'app1'
  stressors:
    memory:
      workers: 4
      size: '256MB'
```

Eksperyment zostanie utworzony po wykonaniu polecenia:

```
kubectl apply -f <filename>.yaml
```

### 2.4. Architektura Chaos Mesh

Chaos Mesh jest oparty na Kubernetes CRD (Custom Resource Definition). Do zarządzania różnymi eksperymentami definiuje wiele typów CRD opartych na różnych rodzajach awarii i implementuje osobne kontrolery dla różnych obiektów CRD. Do głównych komponentów Chaos Mesh należą:

- **Chaos Dashboard**: komponent wizualizacji Chaos Mesh, oferuje zestaw przyjaznych interfejsów internetowych, za pomocą których użytkownicy mogą manipulować i obserwować eksperymenty Chaos. Zapewnia również mechanizm zarządzania uprawnieniami RBAC.
- **Chaos Controller Manager**: główny komponent logiczny Chaos Mesh odpowiedzialny głównie za harmonogramowanie i zarządzanie eksperymentami Chaos. Zawiera kilka kontrolerów CRD takich jak Kontroler Przepływu Pracy, Kontroler Harmonogramu i Kontrolery różnych rodzajów awarii.
- **Chaos Daemon**: główny komponent wykonawczy. Działa w trybie DaemonSet i ma domyślnie uprawnienia Privileged (które można wyłączyć). Ten komponent głównie ingeruje w określone urządzenia sieciowe, systemy plików, jądra, łącząc się z przestrzeniami nazw docelowego Pod.

### 2.5. Stos technologiczny

Chaos Mesh, Docker, Kubernetes (Minikube), JVM (Java, Kotlin)

## 3. Koncepcja studium przypadku

Aplikacja, która została poddana testom to szyfrowany komunikator z czatem tekstowym i audiorozmową. System ten został zaimplementowany na urządzenia mobilne z systemem operacyjnym Android. Został zaprojektowany w oparciu o architekturę mikroserwisów. Do najważniejszych funkcjonalności aplikacji należą:
- przeglądanie czatów z innymi użytkownikami,
- wysyłanie wiadomości tekstowych i zdjęć w czasie rzeczywistym,
- nawiązywanie połączeń audio,
- rejestracja i logowanie użytkowników,
- import i eksport klucza prywatnego użytkownika używanego do odszyfrowywania wiadomości zaszyfrowanych jego kluczem publicznym,
- możliwość integracji z API systemu dowolnego innego klienta oferującego usługę videorozmów.

### 3.1. Architektura systemu

System opiera się na architekturze mikroserwisów. Wydzielono trzy mikroserwisy, każdy z nich posiada swoją własną bazę danych i jest niezależny od pozostałych. Jedną z głównych zalet takiego rozwiązania jest skalowalność - w sytuacji, gdy obciążenie systemu zwiększy się w konkretnym mikroserwisie to nie ma potrzeby duplikowania całej aplikacji. Takie podejście jest wymagane w przypadku skalowania horyzontalnego dla monolitu. W architekturze mikroserwisów duplikujemy tylko najbardziej obciążony mikroserwis. Kolejną zaletą tej architektury jest niezależność, dzięki której zalogowani użytkownicy mogą swobodnie korzystać z aplikacji nawet kiedy serwis odpowiedzialny za logowanie przestanie działać.

<br>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/65e25d5d-2493-48b8-9ce1-26318cdd1973" width="500">
</p>

#### 3.1.1. Backend

Backend składa się z trzech mikroserwisów: serwera autoryzacyjngo, chatu i połączeń. Endpointy pogrupowano ze względu na funkcjonalności. W każdej grupie funkcjonalności znajdują się takie pakiety jak:
- *controller* - zawiera klasy punktu wejścia do aplikacji, posiada metody HTTP takie jak POST, PATCH, PUT, GET oraz DELETE. Dzięki nim możliwa jest realizacja operacji CRUD. Kontroler po otrzymaniu konkretnego zapytania HTTP od klienta deleguje otrzymaną informację do serwisu, który wykonuje zadaną czynność.
- *service* - zawiera klasy serwisów, które są odpowiedzialne za wykonanie zadanego zapytania HTTP, np. utworzenie zasobu, aktualizację istniejącego zasobu lub usunięcie wskazanych zasobów. W celu aktualizacji konkretnego zasobu serwis za pośrednictwem repozytorium pobiera z bazy danych konkretny rekord tabeli w postaci encji bazodanowej (obiekt *DAO*, ang. *Data Access Object*) na podstawie otrzymanego od klienta ID obiektu. Następnie aktualizuje wybrane pola obiektu *DAO* na podstawie zapytania od klienta, zapisuje zaktualizowany obiekt *DAO* w bazie danych oraz zwraca odpowiednią informację do kontrolera, który oddelegowuje ją do klienta.
- *repository* - zawiera interfejsy repozytorium, z których każdy jest skorelowany z konkretną tabelą w bazie danych. Tabela reprezentowana jest w kodzie jako obiekt *DAO*.
- *model* - zawiera klasy zapytań od klienta (requesty), obiekty *DTO* (ang. *Data Transfer Object*, obiekty transferowane z klienta do serwera i odwrotnie) oraz klasy reprezentujące konkretne tabele w bazie danych - encje bazodanowe *DAO*.

<br>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/6eb12e25-16fe-4726-8878-9d85e8545caa" width="700">
</p>

#### 3.1.2. Frontend
    
Projekt klienta to aplikacja na system Android, która łączy się z konkretnymi mikroserwisami za pomocą protokołów HTTP oraz WS. Komunikacja pomiędzy klientem i serwisami odbywa się za pomocą formatu JSON zgodnie ze standardem REST.

Pliki projektu zostały pogrupowane w pakiety dotyczące wybranych ekranów aplikacji. Każdy ekran widoczny w aplikacji posiada odpowiadającą mu klasę dziedziczącą po klasie AppCompatActivity oraz plik XML definiujący jego wygląd. Ekran, który generuje zapytanie HTTP do serwera posiada swój interfejs, w którym zdefiniowane są wszystkie możliwe zapytania HTTP danego ekranu. Dane są przekazywane pomiędzy widokami za pomocą intentów co minimalizuje liczbę requestów HTTP do serwera.

### 3.2. Stos technologiczny i narzędzia

W projekcie aplikacji wykorzystano następujące technologie:

- aplikacja serwera: język Java, framework Spring Boot,
- aplikacja klienta: język Kotlin, środowisko Android Studio,
- baza danych PostgreSQL,
- Docker,
- Git.

### 3.3. Demo aplikacji

#### 3.3.1. Przypadki użycia

Przypadki użycia występujące w aplikacji przedstawiono w formie diagramu UML.

<br>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/43401ee1-cadd-465c-842a-5bb47465848e" width="600">
</p>

#### 3.3.2. Scenariusze przypadków użycia

**Wysłanie wiadomości tekstowej do wybranego użytkownika**

Zalogowany użytkownik z poziomu widoku prywatnych konwersacji lub widoku wyszukiwania użytkowników wybiera osobę, do której chce wysłać wiadomość. Po kliknięciu na danego użytkownika wybiera opcję wysłania wiadomości „Send message”. Po przejściu na ekran konwersacji z wybranym użytkownikiem naciska na pole tekstowe na dole ekranu z napisem „Write message here…”. Po pojawieniu się klawiatury użytkownik wprowadza treść wiadomości i zatwierdza jej przesłanie przyciskiem „Send”. Wiadomość zostaje przesłana do odbiorcy i pojawia się w widoku konwersacji.

**Przesłanie zdjęcia w wybranej konwersacji**

Zalogowany użytkownik wybiera osobę, do której chce przesłać zdjęcie z poziomu widoku prywatnych konwersacji lub widoku wyszukiwania użytkowników. Po kliknięciu na danego użytkownika wybiera opcję wysłania wiadomości „Send message”. Po przejściu na ekran konwersacji z wybranym użytkownikiem naciska na przycisk dodawania zdjęcia w lewym dolnym rogu ekranu. Wyświetlane jest okno dialogowe umożliwiające wybór zdjęcia z plików przechowywanych na urządzeniu. Wygląd okna jest zależny od systemu operacyjnego urządzenia. Użytkownik wybiera zdjęcie. Po wybraniu następuje wysłanie zdjęcia do odbiorcy.

**Nawiązanie połączenia audio z innym użytkownikiem**

Zalogowany użytkownik wybiera osobę, z którą chce nawiązać połączenie audio. Może tego dokonać w widoku prywatnych konwersacji lub w widoku wyszukiwania użytkowników. Po kliknięciu na wybraną osobę wyświetlane są dodatkowe opcje, z których użytkownik wybiera opcję wysłania wiadomości „Send message”. Po przejściu na ekran konwersacji z wybranym użytkownikiem należy nacisnąć przycisk „Call” w prawym górnym rogu ekranu. Wyświetlone zostanie okno dialogowe z wyborem dostawcy oprogramowania obsługującego połączenia. Użytkownik wybiera jedną z opcji, po czym następuje próba nawiązania połączenia z odbiorcą. Do czasu odebrania połączenia przez odbiorcę nadawany jest sygnał dźwiękowy dzwonienia. W przypadku odebrania połączenia przez odbiorcę użytkownicy mogą ze sobą rozmawiać. Opcjonalnie mogą wyciszyć mikrofon. W przypadku anulowania połączenia przychodzącego przez nadawcę lub po upływie minuty połączenie jest przerywane.

**Eksportowanie klucza prywatnego**

Zalogowany użytkownik na głównym ekranie wybiera opcję menu – przycisk z trzema kropkami położony w prawym górnym rogu ekranu. Z listy dostępnych operacji wybiera opcję „Export private key”. Po wyświetleniu okna dialogowego z prośbą o potwierdzenie operacji hasłem użytkownik podaje swoje hasło i zatwierdza czynność przyciskiem „Accept”. Wyświetlane jest okno dialogowe umożliwiające zapis pliku z kluczem prywatnym w wybranej lokalizacji w pamięci urządzenia. Wygląd okna jest zależny od systemu operacyjnego urządzenia. Użytkownik wybiera miejsce, w którym chce zapisać plik oraz opcjonalnie zmienia jego nazwę. Zatwierdza operację przyciskiem „Save”.

**Importowanie klucza prywatnego**

Zalogowany użytkownik na głównym ekranie wybiera opcję menu – przycisk z trzema kropkami położony w prawym górnym rogu ekranu. Z listy dostępnych operacji wybiera opcję „Import private key”. Po wyświetleniu okna dialogowego z informacją o możliwości utraty klucza prywatnego użytkownik akceptuje ostrzeżenie. Wyświetlane jest okno dialogowe umożliwiające wybór pliku z kluczem prywatnym z wybranej lokalizacji w pamięci urządzenia. Wygląd okna jest zależny od systemu operacyjnego urządzenia. Użytkownik wybiera plik z kluczem prywatnym i potwierdza operację importu.

#### 3.3.3. Widoki aplikacji

Zamieszczono zrzuty ekranu przedstawiające przykładowe widoki aplikacji.

<br>
<p align="center">Przykładowy czat z innym użytkownikiem</p>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/f033b36f-b62a-43a4-8124-b2f8d1732973" width="300">
</p>

<br>
<p align="center">Przesłanie zdjęcia</p>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/f565f5d3-e6e5-4235-b6af-3c408563e3c2" width="300">
</p>

<br>
<p align="center">Połączenie przychodzące</p>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/a91590de-b1c9-4101-a7f5-5cbce10e3e09" width="300">
</p>

<br>
<p align="center">Rozmowa głosowa</p>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/53a62bff-5d2e-4dee-836b-198f85bc2751" width="300">
</p>

<br>
<p align="center">Import klucza prywatnego</p>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/8b84eb24-fb0d-4904-a623-8f2afc975e37" width="300">
</p>

<br>
<p align="center">Informacja o błędnym kluczu prywatnym użytkownika</p>
<p align="center">
<img src="https://github.com/Vertemi/Chaos_Mesh/assets/72327045/da833424-6056-4e34-ba89-71764909553c" width="300">
</p>

#### 3.3.4. Film demo aplikacji

Załączono film prezentujący przykładowe użytkowanie aplikacji:

https://github.com/Sweepner/Chaos_Mesh/assets/72269056/127a4655-bb66-4a76-919e-9c0688dbd4e5

## 4. Architektura rozwiązania

## 5. Konfiguracja środowiska

### 5.1 Przygotowanie środowiska
Środowisko utworzono na własnym sprzęcie. Zakupiono serwer posiadający 32 GB ramu oraz 1 TB SSD. Za pomocą frps - https://github.com/fatedier/frp wystawiono serwer do internetu. W celu posiadania adresu IP za NATem skorzystano z zawsze darmowej maszyny wirtualnej Oracle (dalej nazwanej tutaj proxy) z 1 GB ramu oraz 1 corem. Maszyna ta służy jako proxy do prywatnej maszyny. Na proxy uruchomiono frps w następującej konfiguracji (frps.toml - plik konfiguracyjny):

```
[common]
bindPort = 7000
vhost_https_port = 443
tls_enable = true
tls_cert_file = /etc/letsencrypt/live/www.rodzon.site/fullchain.pem
tls_key_file = /etc/letsencrypt/live/www.rodzon.site/privkey.pem
```
Stworzono usługę systemd dla frps w celu automatycznego uruchomienia przekierowania na prywatny serwer po restarcie proxy. Zawartość pliku frps.service:

```
[Unit]
Description=FRPS Service
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/frps -c /usr/local/bin/frps.toml

[Install]
WantedBy=multi-user.target
```

Uruchomiono usługę komendą:

```
sudo systemctl start frps.service
```

Dodano usługę do autostartu maszyny komendą:
```
sudo systemctl enable frps.service
```

Aby frps oraz frpc komunikowały się otwarto porty 7000 oraz 6000 na proxy używając komend:
```
sudo firewall-cmd --permanent --zone=public --add-port=7000/udp
sudo firewall-cmd --permanent --zone=public --add-port=7000/tcp
sudo firewall-cmd --permanent --zone=public --add-port=6000/udp
sudo firewall-cmd --permanent --zone=public --add-port=6000/tcp
sudo firewall-cmd --reload
```
Port 7000 służy do komunikacji pomiędzy klientem a serwerem frp. Port 6000 służy do przekierowania na port 22 na prywatnej maszynie.


Na prywatnej maszynie zainstalowano system operacyjny Rocky Linux - https://rockylinux.org/pl/. Następnie skonfigurowano oraz uruchomiono frpc z następującą konfiguracją (zawartość pliku frpc.toml):
```
serverAddr = "158.101.209.102"
serverPort = 7000

[[proxies]]
name = "ssh"
type = "tcp"
localIP = "127.0.0.1"
localPort = 22
remotePort = 6000

[[proxies]]
name = "postgres"
type = "tcp"
localIP = "127.0.0.1"
localPort = 5432
remotePort = 5432

[[proxies]]
name = "http"
type = "tcp"
localIP = "127.0.0.1"
localPort = 80
remotePort = 80

[[proxies]]
name = "https"
type = "tcp"
localIP = "127.0.0.1"
localPort = 443
remotePort = 443
```

W celu umożliwienia przekierowania z portu 6000 na proxy na port 22 na lokalnej maszynie wystawiono port 22 na prywatnej maszynie za pomocą komend:

```
sudo firewall-cmd --permanent --zone=public --add-port=22/udp
sudo firewall-cmd --permanent --zone=public --add-port=22/tcp
sudo firewall-cmd --reload
```

Uruchomiono  usługę ssh na porcie 22.

W celu umożliwienia zdalnego rebootowania prywatnego serwera utworzono usługę systemd dla frpc.
Plik frpc.service:

```
[Unit]
Description=FRPC Service
After=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/frpc -c /usr/local/bin/frpc.toml

[Install]
WantedBy=multi-user.target
```
Uruchomiono usługę komendą:

```
sudo systemctl start frpc.service
```

Dodano usługę do autostartu maszyny komendą:
```
sudo systemctl enable frpc.service
```

Na prywatnym serwerze zainstalowano:
- minikube
- docker
- kubectl

### 5.2 Uruchomienie usług w Kubernetesie

#### 5.2.1 Postgres

Utworzono pliki do deploymentu bazy danych Postgres w Kubernetesie:

Utworzenie konfig mapy:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres-config
  labels:
    app: postgres
data:
  POSTGRES_DB: postgres
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: myPassword
```

```
kubectl apply -f postgres-configmap.yaml 
```

Utworzenie volumenu dla kontenera:

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
  name: postgres-pv-volume
  labels:
    type: local
    app: postgres
spec:
  storageClassName: manual
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: "/mnt/data"
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: postgres-pv-claim
  labels:
    app: postgres
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
```

```
kubectl apply -f postgres-volume.yaml
```

Utworzenie deploymentu oraz serwisu:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:latest
          ports:
            - containerPort: 5432
          envFrom:
            - configMapRef:
                name: postgres-config
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgres-volume
      volumes:
        - name: postgres-volume
          persistentVolumeClaim:
            claimName: postgres-pv-claim
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  type: NodePort
  ports:
    - port: 5432
  selector:
    app: postgres
```

```
kubectl apply -f postgres-deployment.yaml
```
#### 5.2.2 Spring Boot

Utworzenie plików do deploymentu aplikacji Spring Bootowych w Kubernetesie:

Serwis autoryzacyjny:

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chat-authorization-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: chat-authorization-deployment
  template:
    metadata:
      labels:
        app: chat-authorization-deployment
    spec:
      containers:
        - name: chat-authorization
          image: rodzonvm/chat-authorization-service:1.0.0
          ports:
            - containerPort: 8081
```

Serwis:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: chat-authorization-service
spec:
  type: LoadBalancer
  selector:
    app: chat-authorization-deployment
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 8081
```

Serwis do wysyłania wiadomości:

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chat-message-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: chat-message-deployment
  template:
    metadata:
      labels:
        app: chat-message-deployment
    spec:
      containers:
        - name: chat-message
          image: rodzonvm/chat-message-service:1.0.0
          ports:
            - containerPort: 8082

```

Serwis:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: chat-message-service
spec:
  type: LoadBalancer
  selector:
    app: chat-message-deployment
  ports:
    - protocol: TCP
      port: 8082
      targetPort: 8082

```

Serwis do audio rozmów:

Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: chat-call-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: chat-call-deployment
  template:
    metadata:
      labels:
        app: chat-call-deployment
    spec:
      containers:
        - name: chat-call
          image: rodzonvm/chat-call-service:1.0.0
          ports:
            - containerPort: 8083

```

Serwis:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: chat-call-service
spec:
  type: LoadBalancer
  selector:
    app: chat-call-deployment
  ports:
    - protocol: TCP
      port: 8083
      targetPort: 8083

```

Wszystki trzy aplikacje uruchomiono w następujący sposób:

```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

## 6. Metoda instalacji

### 6.1. Instalacja aplikacji bazowej

W celu użytkowania aplikacji należy zainstalować ją na wybranym urządzeniu z systemem operacyjnym Android.

Minimalne wymagania sprzętowe i programowe:
- 150MB wolnego miejsca na dysku,
- 2GB pamięci RAM,
- system operacyjny w wersji *Nougat* (Android 7 - SDK 24) lub wyższej.

W celu zainstalowania aplikacji należy pobrać udostępniony przez administratora systemu plik .apk programu oraz zainstalować go na urządzeniu mobilnym spełniającym minimalne wymagania sprzętowe i programowe.

Aby móc używać aplikacji należy zbudować obrazy dockerowe za pomocą polecenia ```docker build -tag <nazwa_obrazu>``` z plików *dockerfile* dla trzech mikroserwisów oraz bazy danych, utworzyć sieć dockerową za pomocą polecenia ```docker network create <nazwa_sieci>```, uzupełnić skrypt *docker-compose.yml* o nazwy utworzonych obrazów dockerowych mikroserwisów, bazy danych oraz o nazwę utworzonej sieci dockerowej, a następnie uruchomić skrypt *docker-compose.yml* za pomocą polecenia ```docker-compose up -d```, który uruchomi wszystkie serwisy wraz z bazą danych. W celu uruchomienia aplikacji bez Dockera można wystawić serwisy jako usługi systemowe.

Istotną sprawą jest wystawienie konkretnych portów za NAT aby usługi były dostępne poza siecią lokalną localhost. Sugerowane jest skonfigurowanie serwera proxy (serwer pośredniczący) i wystawienie usług do Internetu za jego pośrednictwem.

### 6.2. Instalacja aplikacji z Chaos Mesh

## 7. Sposób odtworzenia krok po kroku

## 8. Demonstracyjny sposób wdrożenia

### JVM

### JVM
Eksperyment polega na rzuceniu wyjątku, kiedy ktoś się spróbuje zalogować.
Dokument, który taki eksperyment przygotowuje używając Chaos Mesh wygląda następująco:
```
apiVersion: chaos-mesh.org/v1alpha1
kind: JVMChaos
metadata:
  name: throw-exception
  namespace: default
spec:
  action: exception
  class: pl.rodzon.endpoints.auth.controller.AuthController
  method: registration
  exception: java.lang.RuntimeException
  mode: all
  selector:
    namespaces:
      - default
```
Podajemy metodę w klasie, której zmienimy zachowanie. Zmieniamy zachowanie w taki sposób, żeby zawsze rzucała wyjątek. 
[Film z demonstracji działania](https://youtu.be/xgSpCa-w6wI)
Po kilku minutach pod się restartuje i wraca do poprawnego stanu.

## 9. Podsumowanie i wnioski

## 10. Bibliografia

- https://chaos-mesh.org/docs/
- https://minikube.sigs.k8s.io/docs/
- https://kubernetes.io/pl/docs/tutorials/hello-minikube/
