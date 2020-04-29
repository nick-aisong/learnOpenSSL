# OpenSSL攻略

#### root-ca.cnf

```
[default]
name                   = root-ca
domain_suffix          = example.com
aia_url                = http://$name.$domain_suffix/$name.crt
crl_url                = http://$name.$domain_suffix/$name.crl
ocsp_url               = http://ocsp.$name.$domain_suffix:9080
default_ca             = ca_default
name_opt               = utf8,esc_ctrl,multiline,lname,align

[ca_dn]
countryName            = "GB"
organizationName       = "Example"
commonName             = "Root CA"

[ca_default]
home                     = .
database                 = $home/db/index
serial                   = $home/db/serial
crlnumber                = $home/db/crlnumber
certificate              = $home/$name.crt
private_key              = $home/private/$name.key
RANDFILE                 = $home/private/random
new_certs_dir            = $home/certs
unique_subject           = no
copy_extensions          = none
default_days             = 3650
default_crl_days         = 365
default_md               = sha256
policy                   = policy_c_o_match

[policy_c_o_match]
countryName              = match
stateOrProvinceName      = optional
organizationName         = match
organizationalUnitName   = optional
commonName               = supplied
emailAddress             = optional

[req]
default_bits             = 4096
encrypt_key              = yes
default_md               = sha256
utf8                     = yes
string_mask              = utf8only
prompt                   = no
distinguished_name       = ca_dn
req_extensions           = ca_ext

[ca_ext]
basicConstraints         = critical,CA:true
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[sub_ca_ext]
authorityInfoAccess      = @issuer_info
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:true,pathlen:0
crlDistributionPoints    = @crl_info
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,keyCertSign,cRLSign
nameConstraints          = @name_constraints
subjectKeyIdentifier     = hash

[crl_info]
URI.0                    = $crl_url

[issuer_info]
caIssuers;URI.0          = $aia_url
OCSP;URI.0               = $ocsp_url

[name_constraints]
permitted;DNS.0=example.com
permitted;DNS.1=example.org
excluded;IP.0=0.0.0.0/0.0.0.0
excluded;IP.1=0:0:0:0:0:0:0:0/0:0:0:0:0:0:0:0

[ocsp_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
extendedKeyUsage         = OCSPSigning
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash

[server_ext]
authorityInfoAccess      = @issuer_info
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
crlDistributionPoints    = @crl_info
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,digitalSignature,keyEncipherment
subjectKeyIdentifier     = hash

[client_ext]
authorityInfoAccess      = @issuer_info
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
crlDistributionPoints    = @crl_info
extendedKeyUsage         = clientAuth
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash
```

#### sub-ca.cnf

```
[default]
name                     = sub-ca
domain_suffix            = example.com
aia_url                  = http://$name.$domain_suffix/$name.crt
crl_url                  = http://$name.$domain_suffix/$name.crl
ocsp_url                 = http://ocsp.$name.$domain_suffix:9081
default_ca               = ca_default
name_opt                 = utf8,esc_ctrl,multiline,lname,align

[ca_dn]
countryName              = "GB"
organizationName         = "Example"
commonName               = "Sub  CA"

[ca_default]
home                     = .
database                 = $home/db/index
serial                   = $home/db/serial
crlnumber                = $home/db/crlnumber
certificate              = $home/$name.crt
private_key              = $home/private/$name.key
RANDFILE                 = $home/private/random
new_certs_dir            = $home/certs
unique_subject           = no
copy_extensions          = copy
default_days             = 365
default_crl_days         = 30
default_md               = sha256
policy                   = policy_c_o_match

[policy_c_o_match]
countryName              = match
stateOrProvinceName      = optional
organizationName         = match
organizationalUnitName   = optional
commonName               = supplied
emailAddress             = optional

[req]
default_bits             = 4096
encrypt_key              = yes
default_md               = sha256
utf8                     = yes
string_mask              = utf8only
prompt                   = no
distinguished_name       = ca_dn
req_extensions           = ca_ext

[ca_ext]
basicConstraints         = critical,CA:true
keyUsage                 = critical,keyCertSign,cRLSign
subjectKeyIdentifier     = hash

[sub_ca_ext]
authorityInfoAccess      = @issuer_info
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:true,pathlen:0
crlDistributionPoints    = @crl_info
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,keyCertSign,cRLSign
nameConstraints          = @name_constraints
subjectKeyIdentifier     = hash

[crl_info]
URI.0                    = $crl_url

[issuer_info]
caIssuers;URI.0          = $aia_url
OCSP;URI.0               = $ocsp_url

[name_constraints]
permitted;DNS.0=example.com
permitted;DNS.1=example.org
excluded;IP.0=0.0.0.0/0.0.0.0
excluded;IP.1=0:0:0:0:0:0:0:0/0:0:0:0:0:0:0:0

[ocsp_ext]
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
extendedKeyUsage         = OCSPSigning
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash

[server_ext]
authorityInfoAccess      = @issuer_info
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
crlDistributionPoints    = @crl_info
extendedKeyUsage         = clientAuth,serverAuth
keyUsage                 = critical,digitalSignature,keyEncipherment
subjectKeyIdentifier     = hash

[client_ext]
authorityInfoAccess      = @issuer_info
authorityKeyIdentifier   = keyid:always
basicConstraints         = critical,CA:false
crlDistributionPoints    = @crl_info
extendedKeyUsage         = clientAuth
keyUsage                 = critical,digitalSignature
subjectKeyIdentifier     = hash
```



#### 创建demoCA命令

```shell
mkdir root-ca
cd root-ca
mkdir certs db private
chmod 700 private
touch db/index
openssl rand -hex 16 > db/serial
echo 1001 > db/crlnumber


mkdir sub-ca
cd sub-ca
mkdir certs db private
chmod 700 private
touch db/index
openssl rand -hex 16 > db/serial
echo 1001 > db/crlnumber


创建根证书请求
openssl req -new -config root-ca.conf -out root-ca.csr -keyout private/root-ca.key 

创建根证书
openssl ca -selfsign -config root-ca.conf -in root-ca.csr -out root-ca.crt -extensions ca_ext 

创建CRL
openssl ca -gencrl -config root-ca.conf -out root-ca.crl 

创建二级ca证书请求
openssl req -new -config root-ca.conf -out sub-ca.csr -keyout private/sub-ca.key 

创建二级ca证书
openssl ca -config root-ca.conf -in sub-ca.csr -out sub-ca.crt -extensions sub_ca_ext

吊销证书（unspecified、keyCompromise、CACompromise、affiliationChanged、superseded、cessationOfOperation、certificateHold、removeFromCRL）
openssl ca -config root-ca.conf -revoke certs/1002.pem -crl_reason keyCompromise 

创建用于OCSP签名的证书
openssl req -new -newkey rsa:2048 -subj "/C=GB/O=Example/CN=OCSP Root Responder" -keyout private/root-ocsp.key -out root-ocsp.csr 

使用根CA签发
openssl ca -config root-ca.conf -in root-ocsp.csr -out root-ocsp.crt -extensions ocsp_ext -days 30 

测试OCSP：
启动serv
openssl ocsp -port 9080 -index db/index -rsigner root-ocsp.crt -rkey private/root-ocsp.key -CA root-ca.crt -text

client访问
openssl ocsp -issuer root-ca.crt -CAfile root-ca.crt -cert root-ocsp.crt -url http://127.0.0.1:9080



二级CA生成
openssl req -new -config sub-ca.conf -out sub-ca.csr -keyout private/sub-ca.key

使用根CA来签发证书
openssl ca -config root-ca.conf -in sub-ca.csr -out sub-ca.crt -extensions sub_ca_ext

签发服务器证书
openssl ca -config sub-ca.conf -in server.csr -out server.crt -extensions server_ext

签发客户端证书
openssl ca -config sub-ca.conf -in client.csr -out client.crt -extensions client_ext 


创建服务器证书请求
openssl req -new -newkey rsa:1024 -subj "/C=GB/O=Example/CN=$(hostname -f)" -keyout private/server.key -out server.csr 

创建客户端证书请求
openssl req -new -newkey rsa:1024 -subj "/C=GB/O=Example/CN=$(hostname -f)" -keyout private/client.key -out client.csr 

删除服务器证书密码（不删除，启动web容器，读取证书，需要输入密码）
openssl rsa -in ./private/server.key -out ./private/server.key 

删除文档行结尾空格
sed -e 's/[ ]*$//g' temp > sub-ca.conf
```

