Serwowanie plików statycznych (root, alias, location)
===========

Ogólne informacje
-----------

Jedną ze specjalności NGINXa jest serwowanie plików statycznych. To, jakie pliki serwować, skąd i kiedy definiujemy za pomocą poniższych instrukcji:

### `location`

Bloki `location` znajdują się w definicjach wirtualnych hostów, czyli w środku bloków `server` (mogą być zagnieżdżone). Na podstawie argumentów przekazanych tej opcji, tworzone są zasady według których NGINX będzie kierował żądania użytkowników. Instrukcja wygląda następująco:

    location [matcher] uri { ... }

gdzie `matcher` informuje o sposobie interpretacji argumentu `uri` - czy jest to wyrażenie regularne, czy dopasowanie powinno być pełne i tym podobne. Dokładny opis [w dokumentacji](http://nginx.org/en/docs/http/ngx_http_core_module.html#location).

Ogólnie, jeżeli URI żądania pasuje do któregoś z bloków `location`, zostanie ono przetworzone zgodnie z zasadami zdefiniowanymi w bloku.

### `root`

`root` jest instrukcją, którą można umieścić w dowolnym z bloków `http`, `server`, `location` czy `if`. Link do [dokumentacji](http://nginx.org/en/docs/http/ngx_http_core_module.html#root). Pobiera ona jeden argument - `path` - który staje się katalogiem głównym dla przychodzących żądań. Można myśleć o tej instrukcji jak o aktualnym folderze w terminalu:

    cd /apps/static.dev  ~~  root /apps/static.dev

Lokacja pobrana z bloku `location` jest przyłączana do ścieżki z `root`.

### `alias`

Instrukcja `alias` jest podobna do `root`, w odróżnieniu od `root` można ją umieścić jedynie w bloku `location`, a ścieżka podana w `location` jest zamieniana, a nie dołączana do argumentu `aliasu`. Jeżeli chcemy pozostać przy porównaniach z Unixowymi komendami, najodpowiedniejszą analogią wydaje się być link:

    ln -s /apps/static.dev/assets/javascripts /apps/static.dev/js
    ~~
    location /js {
        alias /apps/static.dev/assets/javascripts;
    }

Teraz, wykonanie zapytania do `/js/application.js` powoduje zaserwowanie pliku `/apps/static.dev/assets/javascripts/application.js`. Link do [dokumentacji](http://nginx.org/en/docs/http/ngx_http_core_module.html#alias).

Żródła i ciekawe linki
-----------

1. https://www.nginx.com/resources/admin-guide/serving-static-content/
2. http://nginx.org/en/docs/http/request\_processing.html
