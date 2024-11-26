# Lab #2,22110075, Ha Quoc Tien, INSE330380E_02FIE
# Task 1: Public-key based authentication 
**Question 1**: 
Implement public-key based authentication step-by-step with openssl according the following scheme.
![alt text](image-1.png)

**Answer 1**:
- In this lab, I will use the VM of Seed Labs and the ping-pong lab.
- **Bob**: will be the client.
- **Alice**: will be the server.

![alt text](/imgs/1.png)

- **Step 1**: We needto create a **private** and **public** key for the client.
Get in to the client (Bob):
```sh
docker exec -it e26c3013f3b5 /bin/bash
```
**Create Private key**: 
```sh
openssl genpkey -algorithm RSA -out client_private_key.pem -pkeyopt rsa_keygen_bits:2048
```
- **openssl genpkey**: generate a private key.
- **-algorithm RSA**: using the RSA (Rivest–Shamir–Adleman) algorithm.
- **-out client_private_key.pem**: the output file that contain the private key for the client. **Note**: PEM (Privacy-Enhanced Mail).
- **-pkeyopt rsa_keygen_bits:2048**: an option for the key length. In this case, the private key length is 2048 bits.

![alt text](/imgs/2.png)

Watch the private key using:
```sh
cat client_private_key.pem
```
![alt text](/imgs/3.png)

See the infomation about the key:
```sh
openssl pkey -in client_private_key.pem -text -noout
```
- **-in client_private_key.pem**: choose the key file.
- **-text**: show the infomation about the key (module, component).
- **-noout**: Do not reprint PEM format at the beginning.

**Create Public key**: Now we will create a public key base on the private key.
```sh
openssl rsa -in client_private_key.pem -pubout -out client_public_key.pem
```
- **openssl rsa**: using RSA.
- **-in client_private_key.pem**: the input file will be the private key that we have.
- **-pubout**: This option is used to export the public key from the private key.
- **-out client_public_key.pem**: file output contain the public key.

![alt text](/imgs/4.png)

**Note**: use the commands like private key to watch the public key.

- **Step 2**: Transfer public key from bob (client) to alice (server).

On the host machine, run the following command to copy the client_public_key.pem file from the bob container to the host machine:
```sh
docker cp bob-10.9.0.6:/client_public_key.pem /tmp/client_public_key.pem
```
The public key file will now be located on the host machine at **/tmp/client_public_key.pem**.

Copy files from the host computer to the alice container.
```sh
docker cp /tmp/client_public_key.pem alice-10.9.0.5:/
```
- **/tmp/client_public_key.pem**: File path on the host machine.
- **alice-10.9.0.5:/**: where to save the file on Alice container.

![alt text](/imgs/5.png)

- **Step 3**: send the challenge message from server(Alice).
```sh
docker exec -it 52d58a5b55ff /bin/bash
```
**Generate the message**
```sh
echo "Alice_challenge_message" > challenge.txt
```

**Encrypt the challenge with the client's public key**
```sh
openssl rsautl -encrypt -inkey client_public_key.pem -pubin -in challenge.txt -out encrypted_challenge.bin
```
![alt text](/imgs/6.png)

**Copy files from the container server to the host computer**
```sh
docker cp alice-10.9.0.5:/encrypted_challenge.bin /tmp/encrypted_challenge.bin
```
**Copy files from the host computer to the client container**
```sh
docker cp /tmp/encrypted_challenge.bin bob-10.9.0.6:/
```
![alt text](/imgs/7.png)

- **Step 4**: The client decodes the challenge message.
```sh
openssl rsautl -decrypt -inkey /client_private_key.pem -in /encrypted_challenge.bin -out /decrypted_challenge.txt
```
![alt text](/imgs/8.png)

Before and after decrupted

![alt text](/imgs/9.png)

- **Step 5**: Bob signs the challenge message with private key
```sh
openssl dgst -sha256 -sign /client_private_key.pem -out /challenge_signature.bin /decrypted_challenge.txt
```
![alt text](/imgs/10.png)

Transfer signature to Alice

- Copy to host
```sh
docker cp bob-10.9.0.6:/challenge_signature.bin /tmp/challenge_signature.bin
```
- Copy to alice
```sh
docker cp /tmp/challenge_signature.bin alice-10.9.0.5:/challenge_signature.bin
```
- **Step 6**: Alice verifies the signature
```sh
openssl dgst -sha256 -verify /client_public_key.pem -signature /challenge_signature.bin /challenge.txt
```
![alt text](/imgs/11.png)

# Task 2: Encrypting large message 
Create a text file at least 56 bytes.
**Question 1**:
Encrypt the file with aes-256 cipher in CFB and OFB modes. How do you evaluate both cipher as far as error propagation and adjacent plaintext blocks are concerned. 
**Answer 1**:
- **Step 1**: Create large file on Bob (at least 56 bytes)
```sh
echo "This is a large message to test AES encryption in CFB and OFB modes." > /large_message_Bob.txt
```
Watch the file size.
```sh
wc -c /large_message_Bob.txt
```
![alt text](/imgs/12.png)
- **Step 2**: Create Key and IV
256 bits(32 bytes) key
```sh
openssl rand -hex 32 > /key.txt
```
IV (16 bytes)
```sh
openssl rand -hex 16 > /iv.txt
```
Check the contents.
```sh
cat /key.txt
cat /iv.txt
```
- **Step 3**: Encrypt files in CFB and OFB modes

Implement CFB encoding:
```sh
openssl enc -aes-256-cfb -in /large_message_Bob.txt -out /encrypted_cfb.bin -K $(cat /key.txt) -iv $(cat /iv.txt)
```

Encryption using AES-256 in OFB mode:
```sh
openssl enc -aes-256-ofb -in /large_message_Bob.txt -out /encrypted_ofb.bin -K $(cat /key.txt) -iv $(cat /iv.txt)
```
![alt text](/imgs/13.png)

- **Step 4**: Transfer encrypted files from Bob to Alice
On the host machine:
Copy from bob to host
```sh
docker cp bob-10.9.0.6:/encrypted_cfb.bin /tmp/encrypted_cfb.bin
docker cp bob-10.9.0.6:/encrypted_ofb.bin /tmp/encrypted_ofb.bin
docker cp bob-10.9.0.6:/key.txt /tmp/key.txt
docker cp bob-10.9.0.6:/iv.txt /tmp/iv.txt
```
Copy from host to alice
```sh
docker cp /tmp/encrypted_cfb.bin alice-10.9.0.5:/encrypted_cfb.bin
docker cp /tmp/encrypted_ofb.bin alice-10.9.0.5:/encrypted_ofb.bin
docker cp /tmp/key.txt alice-10.9.0.5:/
docker cp /tmp/iv.txt alice-10.9.0.5:/
```
![alt text](/imgs/14.png)

- **Step 5**: Analyze the error by modifying the 8th byte
Download xxd
```sh
apt-get update
apt-get install vim-common
```
On alice
- Use **xxd** to convert binary files to hex
```sh
xxd /encrypted_cfb.bin > /encrypted_cfb.hex
xxd /encrypted_ofb.bin > /encrypted_ofb.hex
```
Check the hex file
![alt text](/imgs/15.png)

**Open the hex file and edit the 8th byte**
```sh
nano /encrypted_cfb.hex
nano /encrypted_ofb.hex
```
![alt text](/imgs/16.png)
- **Identify the 8th bytes**: Here, the 8th byte is 0x11 (the 8th value in the second pair of digits: 11fa). I will change it to 0x99
- In nano, press Ctrl + O to save, then Enter, and Ctrl + X to exit.

**Convert the Hex back to Binary**
```sh
xxd -r /encrypted_cfb.hex /modified_encrypted_cfb.bin
xxd -r /encrypted_ofb.hex /modified_encrypted_ofb.bin
```
- **Step 6**: Decrypt corrupted files and analyze the results
**CFB**
- Perform decoding
```sh
openssl enc -d -aes-256-cfb -in /modified_encrypted_cfb.bin -out /decrypted_cfb.txt -K $(cat /key.txt) -iv $(cat /iv.txt)
openssl enc -d -aes-256-cfb -in /modified_encrypted_ofb.bin -out /decrypted_ofb.txt -K $(cat /key.txt) -iv $(cat /iv.txt)
```
![alt text](/imgs/17.png)

- CFB Mode:
An error in the 8th byte will affect many subsequent bytes (depending on the block length) due to the dependence of the CFB mode on the previous ciphertext.
- OFB Mode:
The error in the 8th byte only affects that exact byte in the plaintext, because OFB generates the key stream independently of the ciphertext.

**Question 2**:
Modify the 8th byte of encrypted file in both modes (this emulates corrupted ciphertext).
Decrypt corrupted file, watch the result and give your comment on Chaining dependencies and Error propagation criteria.

**Answer 2**:





