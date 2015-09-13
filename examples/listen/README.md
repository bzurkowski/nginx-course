Instrukcja Listen
=============

Ogólne informacje
-------------

Instrukcja `listen` informuje o sockecie, na którym serwer będzie oczekiwał połączeń. Informacja o sockecie składa się z adresu i/lub portu bądź ścieżki do pliku reprezentującego Unixowy socket.

`listen` pozwala również skonfigurować zasady i parametry przyjmowania i utrzymywania połączeń. Między innymi:

* `ssl` pozwala sprecyzować, czy na wybranym sockecie powinny być obsługiwane połączenia SSL
* `default_server` informuje, czy wybrany serwer jest domyślnym serwerem dla tego socketu

Pełną listę parametrów oraz dokładne informacje można znaleźć w [dokumentacji](http://nginx.org/en/docs/http/ngx_http_core_module.html#listen).
