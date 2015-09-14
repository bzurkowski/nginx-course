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

### empty_gif

```
Syntax:	 empty_gif;
Default: -
Context: location
```

* Emits single-pixel transparent GIF:

```
location = /_.gif {
    empty_gif;
}
```

## ngx_http_gzip_module

* Filter that compresses responses using the `gzip` method.
* Helps to reduce the size of transmitted data (by half or even more).

* Increases TTFB (Time To First Byte).
* Increases CPU usage.

### gzip

```
Syntax:	 gzip on | off;
Default: gzip off;
Context: http, server, location
```

* Enables or disables gzipping of responses.

### gzip_vary

```
Syntax:	 gzip_vary on | off;
Default: gzip_vary off;
Context: http, server, location
```

* When the directives `gzip`, `gzip_static`, or `gunzip` are active.
* Enables or disables inserting the `Vary: Accept-Encoding` response header field.
* Browsers send requests with headers, among others: `Accept-Encoding` informing whether it supports decompressing server responses.
* When eg. index.html requested, CDN will cache compressed or uncompressed version of file depending on which client requested first - client with or without compression support.
* We need to inform CDN to cache separate entries for compressed and uncompressed version of response depending. How?
* Including `Vary` header in server's response - it's for CDNs (other intermediary brokers?)!

* [Accept-Encoding is very important!](https://www.maxcdn.com/blog/accept-encoding-its-vary-important/)


## ngx_http_gzip_static_module

* Allows sending precompressed files with the `.gz` filename extension instead of regular files.

### gzip_static

```
Syntax:	 gzip_static on | off | always;
Default: gzip_static off;
Context: http, server, location
```

* Enables or disables checking the existence of precompressed files.
* With the `always` value, gzipped file is used in all cases, without checking if the client supports it.
* The files can be compressed using the `gzip` command, or any other compatible one.

## Variables

* `$args` - arguments in the request line.
* `$content_length` - `Content-Length` request header field.
* `$content_type` - `Content-Type` request header field.
* `$document_root` - `root` or `alias` directive’s value for the current request.
* `$host` - in this order of precedence: host name from the request line, or host name from the `Host` request header field, or the server name matching a request.
* `$hostname` - host`s name.
* `$https` - `on` if connection operates in SSL mode, or an empty string otherwise.
