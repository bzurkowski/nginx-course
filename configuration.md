# Configuration

## Standard directives

### server

```
Syntax:  server { ... };
Default: -
Context: http
```

* Sets configuration for [virtual server](https://en.wikipedia.org/wiki/Virtual_hosting).

```
server {
    listen      80;
    server_name example.net www.example.net;
    ...
}

server {
    listen      80;
    server_name example.com www.example.com;
    ...
}
```

### server_name

```
Syntax:  server_name name ...;
Default: server_name "";
Context: server
```

* Determines which server block should be used for given request.
* May be defined using:

	* exact names:
		`example.org`, `example.net`
	* wildcard names:
		`*.example.com`, `mail.*`, `.example.org`

      * A wildcard name may contain an asterisk only on the name’s start or end, and only on a dot border.
      * An asterisk can match several name parts. The name `*.example.org` matches not only `www.example.org` but `www.sub.example.org` as well.
      * A special wildcard name in the form `.example.org` can be used to match both the exact name `example.org` and the wildcard name `*.example.org`.

	* regular expressions:
		`~^(?<user>.+)\.example\.net$`

      * A named regular expression capture can be used later as a variable:

      ```
      server {
          server_name ~^(?<user>.+)\.example\.net$;

          location / {
              root   /sites/$user;
          }
      }
      ```

* When searching for a virtual server by name, if name matches more than one of the specified variants, the first matching variant will be chosen, in the following order of precedence:

  1. exact name
  2. longest wildcard name starting with an asterisk
  3. longest wildcard name ending with an asterisk
  4. first matching regular expression

  In example below, request with host set to `example.com`, will be processed by the first block:
  ```
  server {
      listen      80;
      server_name example.com;
      ...
  }

  server {
      listen      80;
      server_name ~^(www\.)?(?<domain>.+)$;
      ...
  }
  ```

#### Preventing processing requests with undefined server names

If requests without the `Host` header field should not be allowed, a server that just drops the requests can be defined:

```
server {
    listen      80;
    server_name "";
    return      444;
}
```
