# sonarcloud-bitbucket-connect-test

## How to install the test app on bitbucket cloud ?

### Prerequisite

* Install ngrok: https://ngrok.com/download

### Start the web app (ubuntu)

1. In terminal:

```
python -m SimpleHTTPServer 8000
```

2. In other terminal:

```
ngrok http 8000
```

3. Copy the https link from ngrok
4. Replace all previous ngrog url with the new one:

```
sed -i 's/<old url>/<new url>/g' *
```

5. Browse your ngrok url `/connect-app/connect.html`

### Kill SimpleHTTPServer

1. In terminal:

```
kill -9 `ps -ef |grep SimpleHTTPServer |grep 8000 |awk '{print $2}'`
```
