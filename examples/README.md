# Kurs nginx - notatka
## include

Dyrektywa włączająca do pliku, w którym jest zawarta plik(i) podane jako argument.

```
# nginx.conf
user nginx nginx;
worker_processes 4;
include other_settings.conf;

# other_settings.conf
error_log logs/error.log;
pid logs/nginx.pid;

#result
user nginx nginx;
worker_processes 4;
error_log logs/error.log;
pid logs/nginx.pid;
```

Dyrektywy `include` można używać także w plikach włączanycgh.


## Moduł bazowy

### Dyrektywy

* `error_log` określa poziom i plik logowania błędów
* `user` pozwala zdefiniować użytkownika i grupę startujące procesy nginxa
* `worker_processes` określa liczbę procesów workerów jakie nginx odpali, zaleca się ustawić tę liczbę na równą liczbie rdzeni procesora
* `worker_rlimit_nofile` określa liczbę plików na jakich może pracować jednocześnie proces workera

## Moduł Events

Moduł Events zawiera dyrektywy pozwalające na na konfigurację mechanizmów sieciowych. Jego poprawne użycie może mieć decydujące znaczenie w sprawie wydajności aplikacji.

Wszystkie podane niżej dyrektywy muszą zostać umieszczone w głównym pliku konfiguracyjnym w bloku  `events`.

```
user nginx nginx;
master_process on;
worker_processes 4;
events {
    worker_connections 4;
    use epoll;
}
```

W przypadku umieszczenia powyższych dyrektyw poza głównym plikiem konfiguracyjnym test konfiguracji zakończy się niepowodzeniem.

### Dyrektywy

* `accept_mutex` ustawia mechanizm wzajemnego wykluczenia na socketach, na których nasłuchujemy
* `accept_mutex_delay` czas, jaki musi odczekać worker zanim spróbuje ponownie zająć zasób
* `debug_connections` ustawia zapis szczegółowego loga dla podanego adresu IP lub CIDR (wymaga opcji `--debug` przy kompilacji)
* `multi_accept` określa, czy nginx powinien akceptować wszystkie połączenia z kolejki nasłuchiwania naraz
* `use` ustawia model zdarzeń (opcje przy kompilacji określają zestaw dozwolonych wartości)
* `worker_connections` liczba połączeń, jakie może równolegle obsługiwać worker

## Bazowy moduł HTTP

Moduł HTTP jest największym modułem nginxa i udostępnia wszystkie dyrektywy potrzebne do postawienia serwera HTTP.

```
http {
    gzip on;
    
    server {
        server_name localhost;
        listen 80;
        
        location /downloads/ {
            gzip off;
        }
    }
}
```

### Bloki

* `http` blok ten powinien być umieszczony w głównym pliku konfiguracyjnym. Wydziela logicznie część konfiguracji odpowiedzialną za serwer HTTP
* `server` blok, którego można używać wewnątrz bloku `http`. Pozwala zadeklarować witrynę identyfikowaną przez jedną lub wiele nazw hosta i skonfigurować ją
* `location` pozwala skonfigurować cześć w obrębie witryny

### Dyrektywy

#### Server

* `listen` specyfikuje adres IP i/lub port nasłuchu socketu serwującego witrynę
* `server_name` przypisuje nazwę hosta do danego bloku `server`

#### Wszystkie bloki

* `tcp_nodelay` określa ustawienie flagi `TCP_NODELAY` na socketach (wyłączenie algorytmu Nagle'a)
* `tcp_nopush` określa ustawienie flagi `TCP_NOPUSH` na socketach, wymaga `sendfile` (transmitowanie wszystkich nagłówków odpowiedzi w jednym pakiecie TCP) 
* `sendfile` określa, czy przesyłanie plików będzie delegowane do wywołania systemowego jądra, czy obsługiwane przez nginx.
* `access_log` umożliwia określenie ścieżki, formatu i rozmiaru access loga lub jego wyłączenie


### Zmienne nagłówków żądania
* `$http_host`
* `$http_user_agent`
* `$http_referer`
* `$http_via`
* `$http_x_forwarded_for`
* `$http_cookie`

Dodatkowo można przechwytywać niestandardowe nagłówki tworząc zmienną zaczynajacą się od $http_ i odpowiadającą nazwie nagłówka zapisanej małymi literami i  z użyciem podkreślników zamiast myślników.

### Zmienne nagłówków odpowiedzi
Uwaga - nie wszystkie te nagłówki są dostępne cały czas, niektóre np. tylko w logach po wysłaniu odpowiedzi.
* `$sent_http_content_type`
* `$sent_http_content_length`
* `$sent_http_location`
* `$sent_http_last_modified`
* `$sent_http_connection`
* `$sent_http_keep_alive`
* `$sent_http_transfer_encoding`
* `$sent_http_cache_control`

Przy użyciu podobnej konwencji jak w nagłówkach żądania, możemy zdefiniować niestandardowe nagłówki odpowiedzi zaczynające się od $sent_http_

