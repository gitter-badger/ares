# Ares
Phishing toolkit for red teams and pentesters.

Ares allows security testers to create a phishing environment easily. Ares acts as a proxy between the original site and the phished site, and allows modifications and injects. 

# Getting started

## Docker

Make sure the config toml is located and valid. 

```
docker run -d -p 8080:8080 --name ares -v $(pwd)/config.toml:/etc/ares.toml dutchsec/ares
```

## Features

* transparant 1 to 1 of original site
* modify specific paths to return static files, rendered as Go templates
* create redirects (short urls)
* inject scripts into target site
* support ssl (using lets encrypt)
* multiple targets / hosts
* enhanced filtering on path, method, ip addresses and useragent
* all requests and responses are being logged into Elasticsearch
* all data is being stored for caching / retrieval

## Todo

* create frontend for configuration, monitoring and dashboard
* send emails from toolkit

## Installation

Make sure you have Go 1.7 installed. 

```
git clone git@github.com:dutchcoders/ares.git
go run main.go -c config.toml
```

## Injects

The injects can be inserted in the target site, currently we have the following injects:

* **location** will ask the client for longitude and latitude and post to server
* **snap** will generate screenshots and post to server
* **clipboard** will copy text from clipboard and post to server

## Configuration

```
listener = "0.0.0.0:8080"
tlslistener = "0.0.0.0:8443"

# uncomment if you want to store all responses to disk
#data = "/data"

# uncomment if you want to log all request and responses to elasticsearch
#elasticsearch_url = "http://127.0.0.1:9200/"

# uncomment if you want to use socks proxy for outgoing traffic
#socks = "socks5://127.0.0.1:9050"

[[host]]
host = "wikipedia.lvh.me"
target = "https://en.wikipedia.org"

[[host.action]]
path = "^.*"
action = "inject"
method = ["GET"]
scripts = ["injects/location.js", "injects/snap.js", "injects/clipboard.js"]

# this action will serve as dump endpoint and will be logged
[[host.action]]
path = "^/dump"
action = "serve"
body = ""

# this action will replace all regex occurences 
[[host.action]]
path = "^/login.html"
action = "replace"
method = ["GET"]
regex = "wikipedia"
replace = "aidepikiw"

# this action will serve a login template for all get requests to /login.html
[[host.action]]
path = "^/login.html"
action = "file"
method = ["GET"]
file = "static/login.html"

# this action will serve a login failed template for post requests to /login.html
[[host.action]]
path = "^/login.html"
action = "file"
method = ["POST"]
file = "static/login-failed.html"

# this action will redirect with statuscode 302
[[host.action]]
path = "^/short-url"
statuscode = 302
action = "redirect"
location = "/login.html"

[[logging]]
output = "stdout"
level = "info"
```

## Gophish

Ares will work seamless with Gophish, where you'll use Ares for the landing page functionality. 
