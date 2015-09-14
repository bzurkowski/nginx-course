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

### root

```
Syntax:	 root path;
Default: root html;
Context: http, server, location
```

* Sets the root directory for the requests.
* The path value can contain variables, except `$document_root` and `$realpath_root`.
* A path to the file is constructed by adding a URI to the value of the root directive.
* Don't put `root` directives in `location` blocks - gets messy!

For example, with the following configuration:

```
root /data/w3;

location /static {
  ...
}
```

The `/data/w3/static/top.gif` file will be sent in response to the `/static/top.gif` request.

### try_files

```
Syntax:	 try_files file ... uri;
         try_files file ... =code;
Default: -
Context: server, location
```

* Checks the existence of files in the specified order and uses the first found file for request processing.
* The path to a file is constructed from the file parameter according to the `root` and `alias` directives.
* Common use of global `$uri` variable: `$uri.html`, `$uri/.`.
* If none of the files were found, an internal redirect to the uri specified in the last parameter is made.
* The last parameter can point to a named location or response code:

```
location /images/ {
    try_files $uri /images/default.gif;
}

location = /images/default.gif {
    expires 30s;
}
```

```
try_files $uri/index.html $uri.html $uri =404;
```

```
try_files $uri/index.html $uri.html $uri @app;
```

### limit_except

```
Syntax:  limit_except method ... { ... }
Default: -
Context: location
```

* Limits allowed HTTP methods inside a location.
* The `method` parameter can be one of the following: `GET`, `HEAD`, `POST`, `PUT`, `PATCH`, `DELETE`, `MKCOL`, `COPY`, `MOVE`, `OPTIONS`, `PROPFIND`, `PROPPATCH`, `LOCK`, or `UNLOCK`
* Allowing the `GET` method makes the `HEAD` method also allowed.
* Access to other methods can be further limited using directives from `ngx_http_access_module` and `ngx_http_auth_basic_module`.

```
limit_except GET {
    allow 192.168.1.0/32;
    deny  all;
}
```

Example above will limit access to all methods except `GET` and `HEAD`. Other methods will be allowed only in network `192.168.1.0/32`.

## ngx_http_access_module

### allow/deny

```
Syntax:	 allow/deny address | CIDR | unix: | all;
Default: —
Context: http, server, location, limit_except
```

* Allows/denies access for the specified network or address. In example below, access is allowed only for IPv4 networks `10.1.1.0/16` and `192.168.1.0/24` excluding the address `192.168.1.1`, and for IPv6 network `2001:0db8::/32`.

```
location / {
    deny  192.168.1.1;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny  all;
}
```

* The rules are checked in sequence until the first match is found.
* Useful when we want to block single IPs (eg. bot).

## ngx_http_empty_gif_module
