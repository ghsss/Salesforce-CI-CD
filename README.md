COMMANDS TO CREATE THE DIGITAL CERTIFICATE:

mkdir certificates

cd certificates

openssl genrsa -des3 -passout pass:<password> -out server.pass.key 2048

openssl rsa -passin pass:<password> -in server.pass.key -out server.key

rm server.pass.key

openssl req -new -key server.key -out server.csr

openssl x509 -req -sha256 -days 365 -in server.csr -signkey server.key -out server.crt

openssl enc -aes-256-cbc -k <passphrase> -P -md sha1 -nosalt

openssl enc -nosalt -aes-256-cbc -in server.key -out server.key.enc -base64 -K <key-value> -iv <iv-value>