# graylog

Creating SSL certificate for graylog 2

#<b>Step 1</b> change the default password for Java CAcert store. Default password is <b>changeit</b>

If you have oracle java installed use the second line. If you have openjdk then use the first.

<b>open java cacert store</b> 

sudo keytool -storepasswd -keystore /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/cacerts 

<b>oracle java cacert store</b>

sudo keytool -storepasswd -keystore /usr/lib/jvm/java-8-oracle/jre/lib/security/cacerts


#<b>Step 2</b> Generate Certs for Graylog

keytool -genkey -alias <b>dns.name.of.server</b> -keyalg RSA -validity 365 -keystore keystore.jks

openssl req -x509 -days 365 -nodes -newkey rsa:2048 -keyout pkcs5-plain.pem -out cert.pem

openssl pkcs8 -in pkcs5-plain.pem -topk8 -nocrypt -out pkcs8-plain.pem

openssl pkcs8 -in pkcs5-plain.pem -topk8 -v2 des3 -out pkcs8-encrypted.pem -passout pass:'password'

keytool -list -v -keystore keystore.jks -alias <b>dns.name.of.server</b>

keytool -importkeystore -srckeystore keystore.jks -destkeystore keystore.p12 -deststoretype PKCS12

openssl pkcs12 -in keystore.p12 -nokeys -out graylog-certificate.pem

openssl pkcs12 -in keystore.p12 -nocerts -out graylog-pkcs5.pem

openssl pkcs8 -in graylog-pkcs5.pem -topk8 -out graylog-key.pem

<b>If you have oracle java use this line. The password it requests is the password you changed in step 1.</b>

keytool -import -trustcacerts -file graylog-certificate.pem -alias dns.name.of.server  -keystore /usr/lib/jvm/java-8-oracle/jre/lib/security/cacerts

<b>If you have openjdk use this line. The password it requests is the password you changed in step 1.</b>

keytool -import -trustcacerts -file graylog-certificate.pem -alias dns.name.of.server  -keystore /usr/lib/jvm/java-8-openjdk-amd64/jre/lib/security/cacerts

#<b>Step 3</b> Setup graylog server.conf and move certificates to graylog folder

move all *.pem, *.p12, and *.jks files to /etc/graylog and chown graylog:graylog -R /etc/graylog so that the graylog user has access to them.

<b>edit /etc/graylog/server.conf</b>

change 
rest_listen_uri = http://dns.name.of.server:9000/api/ 

rest_enable_tls = true

rest_tls_cert_file = /etc/graylog/graylog-certificate.pem

rest_tls_key_file = /etc/graylog/graylog-key.pem

rest_tls_key_password = password (this is the password you assigned your cert from step 2. -passout pass:)

web_enable_tls = true

web_listen_uri = http://dns.name.of.server:9000/

web_tls_cert_file = /etc/graylog/graylog-certificate.pem

web_tls_key_file = /etc/graylog/graylog-key.pem

web_tls_key_password = password (Same as rest_tls_key_password.)

#<b>Step 4</b> restart graylog and tail -f /var/log/graylog/server.log

If all goes well open chrome and go to https://dns.name.of.server:9000 and you should be able to log in.



