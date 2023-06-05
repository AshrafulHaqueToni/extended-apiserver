# extended-apiserver
Kubernetes Extended APIServer using net/http library

```console
# Terminal 1
$ go run apiserver/main.go
2019/01/02 18:09:41 listening on 127.0.0.1:8443

# Terminal 2
export APISERVER_ADDR=127.0.0.1:8443

$ curl -k https://${APISERVER_ADDR}/core/pods
Resource: pods
```

```console
# Terminal 3
$ go run database-apiserver/main.go
2019/01/02 18:09:45 listening on 127.0.0.2:8443

# Terminal 2
export EAS_ADDR=127.0.0.2:8443

$ curl -k https://${EAS_ADDR}/database/postgres
Resource: postgres
```

```console
# Terminal 1
$ go run apiserver/main.go --send-proxy-request
2019/01/02 18:09:41 listening on 127.0.0.1:8443
forwarding request to https://127.0.0.2:8443/database/postgres

# Terminal 3
$ go run database-apiserver/main.go --receive-proxy-request
2019/01/02 18:09:45 listening on 127.0.0.2:8443

# Terminal 2
$ cd /tmp/extended-apiserver/

$ curl -k https://${APISERVER_ADDR}/core/pods
Resource: pods

$ curl -k https://${APISERVER_ADDR}/database/postgres
Resource: postgres requested by user[X-Remote-User]=

$ curl https://${APISERVER_ADDR}/database/postgres \
  --cacert ./apiserver-ca.crt \
  --cert ./apiserver-ashraful.crt \
  --key ./apiserver-ashraful.key
Resource: postgres requested by user[X-Remote-User]=john

$ curl https://${EAS_ADDR}/database/postgres \
  --cacert ./database-ca.crt \
  --cert ./apiserver-ashraful.crt \
  --key ./apiserver-ashraful.key
Resource: postgres requested by user[Client-Cert-CN]=john

$ curl https://${EAS_ADDR}/database/postgres \
  --cacert ./database-ca.crt \
  --cert ./database-pritam.crt \
  --key ./database-pritam.key
curl: (35) error:14094412:SSL routines:ssl3_read_bytes:sslv3 alert bad certificate

$ curl -k https://${EAS_ADDR}/database/postgres
Resource: postgres requested by user[-]=system:anonymous
```