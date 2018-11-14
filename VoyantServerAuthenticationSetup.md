# Voyant Server Authentication Setup


## Voyant setup
```
<VoyantServerHome>/server-setting.txt
```

```
# The port to use for the web server (default is 8888). If a port is already in use,
# this can be changed (8889, etc.). If the server terminated unexpectedly the port
# may continue being used and restarting the machine is one way of resetting things.
port = 8080

# The URI path (after the domain) to use (default is /). This is a good way to provide
# load a corpus by default, ex: uri_path = /?input=http://cbc.ca/news/ or /?corpus=333.333
# (where 333.333 is an existing local corpus).
# It's also possible to build a custom menu for the open menu on the front page. Separate
# the corpus ID and a label by a colon and separate multiple entries by commas, making sure
# to escape characters (like spaces), eg:  /?openMenu=444.444:Corpus+One;555.555:Corpus+Two
uri_path = /voyant
```

## Jetty setup (Voyant server embedded web server)


Config file to set context path

```
<VoyantServerHome>/_app/WEB-INF/jetty-web.xml
```



```xml
<Configure class="org.eclipse.jetty.webapp.WebAppContext">
	<Set name="contextPath">/voyant</Set>
    ...
</Configure>
```

X-Forwarded- issue solution

```
<VoyantServerHome>/_app/resources/jsp/pre_app.jsp
```

Change from

```xml
<script type="text/javascript" src="<%= base %>/resources/voyant/current/voyant.jsp?v=11<%= (request.getParameter("debug")!=null ? "&debug=true" : "") %>"></script>
<script type="text/javascript" src="<%= base %>/resources/voyant/current/voyant-locale.jsp?v=11&lang=<%= lang %>"></script>
```
to

```xml
<script type="text/javascript" src="<%= base %>/resources/voyant/current/voyant.min.js"></script>
<script type="text/javascript" src="<%= base %>/resources/voyant/current/voyant-locale-en.js"></script>
```

For more detail, refer to: [SSL Voyant #17](https://github.com/sgsinclair/VoyantServer/issues/17#issuecomment-433812114)

## Nginx setup

*Mac OS X*

```
 /usr/local/etc/nginx/nginx.conf
```

*Ubuntu*

```
 /etc/nginx/nginx.conf
```

### nginx.conf
```
# HTTPS server
    
server {

    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;

    server_name  localhost;

    ssl_certificate      cert.crt;
    ssl_certificate_key  cert.key;


    location = /oauth2/callback {
        auth_request off;
        proxy_pass http://127.0.0.1:4180;

        # copy from https://github.com/sgsinclair/VoyantServer/issues/17
        client_max_body_size 0;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        # copy end
    }

    location / {
        client_max_body_size 0;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header  X-Real-IP  $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        #proxy_pass http://127.0.0.1:8080;
        proxy_pass http://127.0.0.1:4180;
    }
}
```

### Self-signed SSL certificate

[Setup Nginx reverse proxy (with ssl termination) on OSX](http://artemave.github.io/2015/04/01/ssl-reverse-proxy-osx/)

[How To Create a Self-Signed SSL Certificate for Nginx in Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-18-04) 

## OAuth2 proxy for Nginx

HcsVlab includes OAuth2 feature thru [Doorkeeper](https://github.com/doorkeeper-gem/doorkeeper).

For OAuth2 Provider setup, visit [OAuth2 Provider - staging](https://staging.alveo.edu.au/oauth/applications) or [OAuth2 Provider - production](https://app.alveo.edu.au/oauth/applications), admin right is required.


There is an [OAuth2 proxy](https://github.com/Alveo/oauth2_proxy) implementation for Nginx, including Doorkeeper support to make Voyant Server service resource protected by Doorkeeper.

*OAuth2 proxy* is a `GO` program, so assume `<OAUTH2_PROXY_HOME>=$GOPATH/src/github.com/Alveo/oauth2_proxy`, and after `go install`, it locates in `$GOPATH/bin/oauth2_proxy`

For `GO` environment preparation, refer to [Getting Started](https://golang.org/doc/install)

### doorkeeper.cfg

```
# <OAUTH2_PROXY_HOME>/config/doorkeeper.cfg

## OAuth2 Proxy Config File
## https://github.com/bitly/oauth2_proxy

## <addr>:<port> to listen on for HTTP/HTTPS clients
# http_address = "127.0.0.1:4180"
# https_address = ":443"

## TLS Settings
# tls_cert_file = ""
# tls_key_file = ""

## the OAuth Redirect URL.
# defaults to the "https://" + requested host header + "/oauth2/callback"
# redirect_url = "https://internalapp.yourcompany.com/oauth2/callback"

# dev (external IP)
#redirect_url = "https://10.125.71.103/oauth2/callback"

# staging
#redirect_url = "https://130.56.244.159/oauth2/callback"

# production
#redirect_url = "https://130.56.244.159/oauth2/callback"

## the http url(s) of the upstream endpoint. If multiple, routing is based on path
upstreams = [
	"http://127.0.0.1:8080/"
]

## Log requests to stdout
request_logging = true

## pass HTTP Basic Auth, X-Forwarded-User and X-Forwarded-Email information to upstream
# pass_basic_auth = true
# pass_user_headers = true
## pass the request Host Header to upstream
## when disabled the upstream Host is used as the Host Header
# pass_host_header = true

## Email Domains to allow authentication for (this authorizes any email on this domain)
## for more granular authorization use `authenticated_emails_file`
## To authorize any email addresses use "*"
email_domains = [
    "*"
]

## OAuth Provider
provider = "doorkeeper"
#email-domain = "*"
#login-url = "https://github.com/login/oauth/authorize"
login-url = "https://staging.alveo.edu.au/oauth/authorize"
redeem-url = "https://staging.alveo.edu.au/oauth/token"

## The OAuth Client ID, Secret
#client_id = "05d6bbb94484b143c577"
#client_secret = "510c2eaf42a397e981d1a75beb2b40de77eff068"
client_id = "d32863b804eb8214134cba1050b32bbd87134578aa36a2a673dd4bb867529baa"
client_secret = "c525a4d2b124b17e08d1fba6bff9a2f4cc510b072b9db7ecb2ceeaeebfaac2af"

## Pass OAuth Access token to upstream via "X-Forwarded-Access-Token"
# pass_access_token = false

## Authenticated Email Addresses File (one email per line)
# authenticated_emails_file = ""

## Htpasswd File (optional)
## Additionally authenticate against a htpasswd file. Entries must be created with "htpasswd -s" for SHA encryption
## enabling exposes a username/login signin form
# htpasswd_file = ""

## Templates
## optional directory with custom sign_in.html and error.html
# custom_templates_dir = ""

## skip SSL checking for HTTPS requests
# ssl_insecure_skip_verify = false


## Cookie Settings
## Name     - the cookie name
## Secret   - the seed string for secure cookies; should be 16, 24, or 32 bytes
##            for use with an AES cipher when cookie_refresh or pass_access_token
##            is set
## Domain   - (optional) cookie domain to force cookies to (ie: .yourcompany.com)
## Expire   - (duration) expire timeframe for cookie
## Refresh  - (duration) refresh the cookie when duration has elapsed after cookie was initially set.
##            Should be less than cookie_expire; set to 0 to disable.
##            On refresh, OAuth token is re-validated.
##            (ie: 1h means tokens are refreshed on request 1hr+ after it was set)
## Secure   - secure cookies are only sent by the browser of a HTTPS connection (recommended)
## HttpOnly - httponly cookies are not readable by javascript (recommended)
cookie_name = "_oauth2_proxy"
cookie_secret = "UjdndVR0RGhJdUQ2QmlCMytQQlJtUT09"
cookie_domain = "10.125.71.103"
# cookie_expire = "168h"
# cookie_refresh = ""
cookie_secure = false
# cookie_httponly = true

```

### Run

```
$ oauth2_proxy -config=$GOPATH/config/doorkeeper.cfg &
```





