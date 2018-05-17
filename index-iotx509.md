# X.509 ECC Cert with Aure IoT Hub 

## Preliminary reading
* [Conceptual understanding of X.509 CA certificates in the IoT industry](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-x509ca-concept)
* [Device Authentication using X.509 CA Certificates](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-x509ca-overview)
* [PowerShell scripts to manage CA-signed X.509 certificates](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-security-x509-create-certificates#createcerts)
* [Set up X.509 security in your Azure IoT hub](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-security-x509-get-started#registercerts)
* [Azure IoT certification with Bash script](https://github.com/Azure/azure-iot-sdk-c/blob/master/tools/CACertificates/certGen.sh)

## ATECC508A Microchip
* [ATECC508A product page](http://www.microchip.com/wwwproducts/en/ATECC508A#additional-features)
* [Node Authentication Example Using Asymmetric PKI](http://ww1.microchip.com/downloads/en/AppNotes/Atmel-8983-CryptoAuth-ATECC508A-Node-Example-Asymmetric-PKI-ApplicationNote.pdf)

The ATECC508A is a secure element from the Microchip CryptoAuthentication portfolio with advanced Elliptic Curve Cryptography (ECC) capabilities. With ECDH and ECDSA being built right in, this device is ideal for the rapidly growing IoT market by easily supplying the full range of security such as confidentiality, data integrity, and authentication to systems with MCU or MPUs running encryption/decryption algorithms (i.e. AES). Similar to all Microchip CryptoAuthentication products, the new ATECC508A employs ultra-secure hardware-based cryptographic key storage and cryptographic countermeasures which are more robust than software-based key storage.

### Additional Features 
* Easy way to run ECDSA and ECDH Key Agreement
* ECDH key agreement makes encryption/decryption easy
* Cryptographic accelerator with Secure Hardware-based Key Storage
* Performs High-Speed Public Key (PKI) Algorithms
* NIST Standard P256 Elliptic Curve Support
* SHA-256 Hash Algorithm with HMAC Option
* 256-bit Key Length
* Storage for up to 16 Keys
* Guaranteed Unique 72-bit Serial Number
* Internal High-quality FIPS Random Number Generator (RNG)
* 10Kb EEPROM Memory for Keys, Certificates, and Data


## Self-Signed ECC certificate

* [Create a self-signed ECC certificate](https://msol.io/blog/tech/create-a-self-signed-ecc-certificate/)
* [ecdhCurve, prime256v1 (aka NIST P-256)](https://github.com/nodejs/node/issues/1495)

```bash
openssl ecparam -genkey -name prime256v1 -out key.pem
openssl req -new -sha256 -key key.pem -out csr.csr
openssl req -x509 -sha256 -days 365 -key key.pem -in csr.csr -out certificate.pem
openssl req -in csr.csr -text -noout | grep -i "Signature.*SHA256" && echo "All is well" || echo "This certificate will stop working in 2017! You must update OpenSSL to generate a widely-compatible certificate"
```
## Azure IoT Hub certificate creation

Use the script to create the root and intermediate certificates
```bash
./certGen.sh create_root_and_intermediate
```

IoT hub createtion and preparation
```bash
#Check you version of Azure CLI
az --verion (2.0.31)
#Create a resource group
az group create -l "West Europe" -n iotCert
#Create Azure IoT Hub
az iot hub create -g IoTCert -n iotCertHub --sku S1 -l "West Europe" --partition-count 2
#Upload Root CA certificate to iot HUB
az iot hub certificate create --hub-name iotCertHub -n
ECC_Test_Certificate -p ./certs/azure-iot-test-only.root.ca.cert.pem
#List all certificates for a specific iotHub and get etag
az iot hub certificate list -g iotCert --hub-name iotCerthub
#Generate Certificat Verification code
az iot hub certificate generate-verification-code --hub-name iotCertHub -n ECC_Test_Certificate --etag AAAAAAHtArU=
```

Sign the verification code from IoT hub
```bash
 ./certGen.sh create_verification_certificate D08BEA59EBD60D5CD91DBD9BA18BA5C64D7F47294ED78928
```
Azure IoT HUB
```sh
#Get the updated etag
az iot hub certificate list -g iotCert --hub-name iotCerthub
# Verfy the certificate
az iot hub certificate verify --hub-name iotCertHub -n ECC_Test_Certificate --path ./certs/verification-code.cert.pem --etag AAAAAAHtBEU=
```

Create Device Certificates
```bash
#create the new device certificate
certGen.sh create_device_certificate my-iot-device 
#To get the public key
cd ./cert
cat new-device.cert.pem azure-iot-test-only.intermediate.cert.pem azure-iot-test-only.root.ca.cert.pem > new-device-full-chain.cert.pem
#device's private key is in: ./private/new-device.cert.pem 
```