# type something  
## type something  
### type something else  
#### try something else


```
when RULE_INIT {
  set static::ASDL_Split_debug 1
}
when HTTP_REQUEST {
    # disable serverside SSL
    SSL::disable serverside
    if { [TCP::client_port] % 10  == 0} {
        if { ([HTTP::uri] contains "v1/locker/entry/authorize")
            or ( [HTTP::uri] contains "v1/locker/entry/cancel") } {
            set http_uri [HTTP::uri]
            set http_method [HTTP::method]
            if { $static::ASDL_Split_debug } {
                log local0. http_method=$http_method,http_uri=$http_uri
            }
            #Send to new URI
            HTTP::uri [string map {"/auth/locker/provisioning/v1/locker/entry" "/digitallocker/provisioning"} [HTTP::uri]]
            set http_uri [HTTP::uri]
            #re-enable SSL
            SSL::enable serverside
            #send to the AWS Pool
            pool aws_asdl_pool
            if { $static::ASDL_Split_debug } {
                log local0. http_method=$http_method,http_uri=$http_uri
            }
        }
    }
}

```  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service-nodeport
  namespace: default
  labels:
    app: nginx
    cis.f5.com/as3-tenant: nginx
    cis.f5.com/as3-app: nginx_app
    cis.f5.com/as3-pool: nginx_pool
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - name: nginx
      protocol: TCP
      port: 80
      targetPort: 80
```