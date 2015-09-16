Autentykacja
==========

W NGINX w bardzo prosty sposób można wprowadzić prostą [autentykację HTTP](https://en.wikipedia.org/wiki/Basic_access_authentication). Można dzięki niej chronić pewne zasoby za pomocą loginu i hasła.

`auth_basic`
----------

Aby zastosować autentykację, wystarczy użyć dwóch dyrektyw:

* auth\_basic *realm*|off - wprowadza w danym kontekście (`http`, `server`, `location`, `limit_except`) autentykację HTTP. *realm* to swego rodzaju dziedzina, czyli zbiór zasobów które są chronione tym samym hasłem. Jeżeli będziemy poruszać się po stronach z tej samej dziedziny, przeglądarka nie będzie musiała nas za każdym razem pytać o login i hasło. Jeżeli jednak zmienimy dziedzinę, potrzebne będą nowe dane. `off` jest specjalną wartością która ma sens tylko wtedy, gdy odziecziczono jakąś konfigurację `auth_basic`, a dla tego zasobu chcemy ją wyłączyć.
* auth\_basic\_user\_file *file* - plik z hasłami w formacie:

        name1:pass1
        name2:pass2:comment
        ...

  Hasła muszą być zaszyfrowane, na przykład z pomocą `openssl passwd`. Jeżeli zostanie podana ścieżka relatywna jako plik *file*, będzie ona używana względem lokacji nginxa.

Więcej informacji o [module ngx\_http\_auth\_basic\_module](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html).
