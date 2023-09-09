# https签名流程

首先，浏览器缓存必须 **全部** 清除

### 1.设置 `openssl.cnf`

```
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req

[req_distinguished_name]
countryName = Country Name (2 letter code)
countryName_default = AU
stateOrProvinceName = State or Province Name (full name)
stateOrProvinceName_default = Some-State
localityName = Locality Name (eg, city)
# localityName_default = ShenZhen
organizationalUnitName  = Organizational Unit Name (eg, section)
organizationalUnitName_default  = Internet Widgits Pty Ltd
commonName = Internet Widgits Ltd
commonName_max  = 64

[ v3_req ]
# Extensions to add to a certificate request
authorityKeyIdentifier=keyid,issuer
basicConstraints = CA:TRUE # Android 安装这里必须是True
keyUsage = nonRepudiation, digitalSignature, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]

# 改成自己的域名
DNS.1 = localhost
#DNS.1 = kb.example.com
#DNS.2 = helpdesk.example.org
#DNS.3 = systems.example.net

# 改成自己的ip
IP.1 = 192.168.113.1
IP.2 = 10.81.40.105
```

### 2.命令行生成 `key` 和 `crt`

```
openssl genrsa -des3 -out ca.key 2048
Enter PEM pass phrase: <your password>
Verifying - Enter PEM pass phrase: <your password>

openssl req -x509 -new -nodes -key ca.key -sha256 -days 114514 -out ca.crt

openssl x509 -in ca.crt -noout -text

openssl genrsa -out test_gpu_dev.key 2048

openssl req -new -key test_gpu_dev.key -out test_gpu_dev.csr

openssl req -text -noout -in test_gpu_dev.csr

openssl x509 -req -in test_gpu_dev.csr -out test_gpu_dev.crt -days 114514 -CAcreateserial -CA ca.crt -CAkey ca.key -extensions v3_req -CAserial serial -extfile openssl.cnf

openssl verify -CAfile ca.crt test_gpu_dev.crt
```

### 3.配置本地`webpack.config.js`

```javascript
devServer: {
      host: '0.0.0.0',
      port: 443,
      https: {
        key : fs.readFileSync('./https/test_gpu_com.key'),
        cert: fs.readFileSync('./https/test_gpu_com.crt')
      },
      allowedHosts : ['testgpu.com'], 
      devMiddleware: {
        publicPath: "/dist/",
      },
      static: {
        directory: "./",
        serveIndex: true,
      },
    },
```

### 4.安装证书

`firefox` 的话要设置 `security.enterprise_roots.enabled`

这样就可以内部IP直接访问了



然后也需要把证书传给别人进行安装
