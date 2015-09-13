Maintenance page
==========

Ogólne informacje
----------

Podobnie jak pozostałych kodów błędów serwera, również dla błędu 503 (Service Unavailable) można ustawić własną stronę za pomocą instrukcji `error_page`.

    error_page 503  /maintenance.html;

W powyższym przykładzie, pierwszy argument - `503` wskazuje na kod HTTP błędu, natomiast `/maintenance.html` - na lokację którą wybierze nginx w przypadku wystąpienia błędu 503. Należy więc zatem odpowiednio ją zdefiniować (w przeciwnym wypadku, zostanie wyświetlona standardowa strona błędu):

    location /maintenance.html {
        internal;
    }

Instrukcja `internal` informuje o tym, że lokacja jest dostępna tylko "z wewnątrz", a podczas requestów zewnętrznych zostanie zwrócony błąd 404. Więcej informacji w [dokumentacji nginx](http://nginx.org/en/docs/http/ngx_http_core_module.html#internal).

Standardowy przypadek użycia
----------

Przykład zaczerpnięty ze [StackOverflow](http://stackoverflow.com/questions/16693209/nginx-return-response-code-if-a-file-exists-by-using-try-files-only/21707602#21707602).

Czasami zdarza się, że z różnorakich powodów należy wyłączyć serwis na pewien czas. Oczekujemy wtedy, że zanim użytkownicy będą mogli ponownie korzystać z serwisu, będą oglądali Maintenance page oraz zostanie zwrócony kod 503. Dobrze by było, żeby włączanie i wyłączanie strony było możliwie proste.

Jedno z rozwiązań, zaprezentowane w podlinkowanej odpowiedzi znajduje się tu w repozytorium. Został dodany specjalny plik konfiguracyjny:

    # /etc/nginx/sites-available/maintenance.dev
    server {
        ...
        location / {
            include /apps/maintenance.dev.conf;
            ...
        }
    }

w którym jest zdefiniowana jedna zmienna:

    # /apps/maintenance.dev.conf
    set $maintenance 0;

Następnie, w zależności od niej, możemy zwócić kod 503, który automatycznie wyrenderuje nam wcześniej ustawioną stronę z błędem.

    # /etc/nginx/sites-available/maintenance.dev
    server {
        ...
        location / {
            ...
            if ($maintenance = 1) {
                return 503;
            }
        }
    }

