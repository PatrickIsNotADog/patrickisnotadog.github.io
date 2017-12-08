I decided to write this post after a project I participated in, which required a web service to connect to a remote machine over SSL. Although I had already encountered the term “SSL” a lot of times, I soon realized that I had never fully understood how it really works and how it could be implemented in Java, so I started collecting information from the internet to make things clear. I will share these notes here, starting from the mathematical background of SSL to end up with the Java Secure Socket Extension.

## Theoretical background: Public-key (assymetric) cryptography

Before I describe SSL, I would like to take some time and talk about a widely used cryptographic protocol (or more accurately class of protocols), the public key cryptography. Public-key cryptography can be used to encrypt messages being exchanged between two peers and make sure that someone who “listens” to these messages can not decrypt them. It also gives the option to digitally sign the data exchanged in a way that nobody can counterfeit the sign, and when the sign is verified, the receiver of the message can be sure that the sender is the one who claims to be and that the message was received exactly as sent (nobody changed it in the meanwhile).

Public-key cryptography algorithms make use of two separate keys, a public and a private one. The two keys are mathematically linked in the following way: if a message is encrypted using the private key, only the public key can decrypt it and if a message is encrypted with the public key only the private key can decrypt it. The crucial point here is that, although it is easy to decrypt a message using one of the keys, it is computationally impractical to compute a private key from a public key. So, if two peers, who want to safely exchange some messages, make sure that their private keys remain secret, they can careless exchange public keys and use them to send encrypted messages over an insecure network, as the internet.

To make this a little more clear, let’s take the case that Alice wants to send an encrypted message to Bob. She can ask Bob to send his public key over the internet (not caring if someone else “hears” it) and then use this key to encrypt her message and send it back to Bob. Even though someone can “hear” the encrypted message, if he doesn’t have Bob’s private key, it will be impossible for him to decrypt it, making Bob the only person able to see Alice’s actual message.

If Bob is not sure that Alice is the one who claims to be, he can ask her to send him a digital signature. Alice can encrypt a random plaintext with her private key and send both the plaintext and the encrypted text to Bob. Then, Bob will use Alice’s public key (which he can easily get) and check that the decrypted message is actually the plaintext sent. This way he knows that the message was encrypted with Alice’s private key which is supposed to be only in Alice’s possesison.

We can take the example above one step further and use a combination of encryption techniques: Alice can create a random plaintext and encrypt it with her private key, thus producing a digital signature. Then, she can use Bob’s public key to encrypt both the plaintext and the signature and send them to him. Finally, Bob will receive Alice’s message and use first his private key to decrypt it, and then Alice’s public key to check if the signature is actually the plaintext encrypted.

But what if Trudy wants to fool Bob and pretend to be Alice? She will create a public and a private key, encrypt a plaintext with this private key and send both the raw and encrypted text to Bob, along with her public key, saying “I am Alice, use these to verify me”. Bob will decrypt the text, verify that it is the same with the plaintext and get fooled by Trudy. To prevent cases like this, entities called Certificate Authorities (CAs) emerged. CAs are commonly trusted entities, whose public keys are well known and can be used to verify that someone is actually the one he claims to be. In the example above, Alice would ask a CA to verify that she actually is Alice and not someone else, and the CA (after running some checks) would provide her a certificate, encrypted with CA’s own private key, saying that Alice’s public key is the one contained in the certificate. Bob (who trusts the CA, knows CA’s public key and can use it to verify that the certificate is valid) would only then accept Alice’s actual public key and would not get fooled by Trudy.

## An introduction to SSL/TLS

The SSL protocol is a standard security technology, developed at Netscape, to establish an encrypted link between a server and a client, leveraging public-key cryptography. It ensures that all data passed between the server and the client remain private and integral and for that reason, it was used to provide e-commerce transaction security on the Web, protecting customers’ personal data. It is implemented at the application level, directly on top of TCP, so that protocols above it (for example HTTP) can use it to offer communication security across a network while they keep operating unchanged.

Netscape published SSL v2 (1995) and SSL v3 (1996) as proprietary products and stopped updating the protocols ever since, leading SSL v3 to be considered insecure since 2014 and vulnerable to POODLE attack. The protocol’s upgrade was continued by IETF, which standardized SSL under the name Transport Security Layer (TLS) at 1999. So, TLS is actually the next version of SSL 3.0, having not dramatic differences from its ancestor (enough to preclude interoperability between TLS 1.0 and SSL though) and this is why the term “SSL” can often be used for TLS connections as well. TLS 1.0 was published in January 1999, TLS 1.1 in April 2006 and TLS 1.2 in August 2008.

The TLS protocol is designed to provide three essential services to all applications running above it:

* encryption: it ensures that only the parties to communicate can understand the messages or data sent between the parties.
* authentication: it ensures that the data could only have come from a known source.
* integrity: it ensures that the data received by one party were the data sent by the other party and were not manipulated or compromised during transmission.

In order to assure integrity, TLS protocol provides its own message framing mechanism and signs each message with a Message Authentication Code (MAC). The MAC algorithm is a one-way cryptographic hash function, the keys to which are negotiated by both connection peers. Whenever a TLS record is sent, a MAC value is generated and appended to that message, so that the receiver is able to compute and verify the sent MAC value to ensure message integrity.

In order to assure encryption both peers use a cipher-suite (a suite and not just a cipher, since the protocol is quite complex), selected from a list of available ones, to encrypt the data exchanged. The suite will use public-key cryptography to create and exchange a secret private key (common for both client and server) and once this key is securely exchanged, will make use of it to transmit the rest of the conversation encrypted with it. This is done because, although safer, public-key cryptography is much more computational and time consuming.

In order for the parties to select an encryption suite, TLS starts a negotiation protocol named “TLS Handshake” described below. During the handshake, the client also verifies server’s authentication, and the server can do the same thing with the client, if it is configured to do so.

## The TLS Handshake

Every SSL/TLS connection begins with a “handshake”, which is the negotiation between the two parties, so that they mutually decide the details of the encryption they will use. In detail, they decide about the version of the SSL/TLS protocol and the cipher suite that will be used. Also, the server’s identity is verified, and if asked, the client’s too. Below are the (summarized) steps of a typical handshake:

1. The client sends a “client hello” message with the list of protocols supported (in an order of preference) and its supported cipher suites.
1. The server selects a protocol version (usually the latest common one) and a cipher suite from the client’s list and sends back a “server hello” message with these two choices and also its digital certificate. If the server requires it, it will also send a request for the client’s digital certificate too, along with the acceptable Certificate Authorities (CAs).
1. The client verifies the server’s digital certificate and sends back to the server a random byte string (encrypted with server’s public key), that will be used by both the client and the server to compute a (shared) secret key, which will be used to encrypt session’s data. If asked, it also sends back its own digital certificate.
1. If client’s certificate was asked the server verifies it.
1. Client and server send a “finished” message encrypted with the secret key.
1. For the duration of the SSL/TLS session the server and the client exchange messages symmetrically encrypted with the (shared) secret key.

So, the client and the server use asymmetric encryption to generate a (shared) secret key, common for both the parts. This key is used later, when the session is established, to symmetrically encrypt the data exchanged between the client and the server. This is done because asymmetric encryption, although safer, requires more computational resources and time, so it is used to create a (shared) private key in both parts and then the “lighter” symmetric encryption uses this key to encrypt session’s messages.

## Authentication

As seen above, after both parts agree on one cipher suite, the server sends it’s authentication data. This authentication information will most probably be in the form of an X.509 certificate, commonly referred to as “SSL certificate”. An X.509 certificate acts as a container for the public key and is signed by a trusted Authority (Certificate Authority or simply CA). CAs are commonly trusted authorities, whose public keys are widely known and can be included in the OS. Servers that want to use public-key cryptography to ensure secure transactions with their clients, will make a request for a certificate to one of these authorities (CSR request) and the authority, after it runs some checks on the server and the company behind it, will provide a certificate which actually ensures that server’s public key is the one contained in it. The certificate itself comes encrypted with CA’s private key, so everybody is sure that the certificate is not fake, and as a result the server’s public key is the one contained in it.

## SSL related file types

Below is a list with file extensions commonly used with SSL/TLS:

* .csr: Certificate Signing Request. These files can be generated and submited to certificate authorities (CA) for approval. The actual format is PKCS10 which is defined in RFC 2986 and includes some or all of the key details of the requested certificate such as subject, organization, state, whatnot, as well as the public key of the certificate to be signed. They get signed by the CA and a certificate is returned. The returned certificate is the public certificate which itself can be  in a couple of formats.
* .pem: is a container format that may include just the public certificate, or may include an entire certificate chain including public/private key and root certificates.
* .cert .cer .crt: are all .pem formatted files with different extension, in order to be  recognized by Windows Explorer as certificates, since .pem is not.
* .key: is a PEM formatted file containing just the private key of a specific certificate and is merely a conventional name and not a standardized one. The rights of these files are very important and some programs will refuse to load these certificates if they are set wrong.
* .pkcs12 .pfx .p12 are passworded container formats that contain both public and private certificate pairs. Unlike .pem files, these containers are fuly encrypted. [Openssl](https://www.openssl.org/) can turn them into a .pem file with both public and private keys with the command `openssl pkcs12 -in file-to-convert.p12 -out converted-file.pem -nodes`.

## The use case and openssl

For my project’s needs I had to connect to a machine over an SSL secured port. For this reason I was given two files:

* trust.cer: which included two certificates. One self-signed under the identity huaweiossCA containing a public key and a signature, and one signed by huaweiossCA under the identity networkossCA.
* client.p12: protected with the passphrase “Changeme_123”. This file contained a certificate signed by networkossCA and also a client private key.

In order to connect to the desired machine, all actually needed is the client certificate and private key. So, I used the command:

    openssl pkcs12 -in client.p12 -out client.pem -nodes

to create a .pem file from the .p12 and then connected to the machine typing:

    openssl s_client -connect <ip-address>:<port> -cert client.pem

## Java implementation

In order to connect to the machine with Java, I used the [Java Secure Socket Extension (JSSE)](http://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.html), a framework dedicated to the SSL/TLS protocol. A new concept here are keystores. A keystore is a database of keys, used for a variety of purposes including authentication and data integrity. Keystores can be grouped into two different categories:

* key entries, which consist of entity’s identity (which includes public key) and its private key.
* trusted certificates, which contain entity’s identity (only the public key here).

The JDK implementation of keystore I am going to use is “JKS”, and allows keystores to contain both keys and trusted certificate entries. Using jks schema I will create two keystores: the one will contain client’s private key and certification (client.p12 file) and the other one will contain the trusted certificate entries (trust.cer). This is a logical distinction between our own certificates and corresponding private keys and others’ certificates, useful if you want to store these files under different directories, with different access restrictions.

Other keystore types can be found here: https://docs.oracle.com/javase/7/docs/technotes/guides/security/StandardNames.html#KeyStore

## Create keystore files with keytool

In order to create the jks files I will use Oracle’s [keytool](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/keytool.html). First, to create client’s keystore from client.p12 file:

    keytool -importkeystore -srckeystore client.p12 -srcstoretype pkcs12 -destkeystore client.jks -deststoretype jks -srcstorepass Changeme_123 -deststorepass 123456

The `-srcstorepass` parameter is the password, which locks the client.p12 file and is required to unlock it and create the client.jks file. Parameter `-deststorepass` on the other hand is the password of the newly created jks file. Every keystore contains a number of keys, each identified by an alias. In order to see keystore’s keys and their aliases you can type:

    keytool -list -keystore client.jks

and if you want to see information for a specific key only you can type:

    keytool -list -keystore client.jks -alias <alias>

In both occasions you will be prompted to give the keystore’s password, which is the one configured in the first command, that created the keystore (in my case 123456). To change this password you can use:

    keytool -storepasswd -keystore client.jks

Besides the keystore’s password every key has its own private password as well. In this case, this password is the same with the initial client.p12’s password (Changeme_123). To change this you can use:

    keytool -keypasswd -alias <key_name> -keystore client.jks

In order to create the keystore with the trusted certificates i used:

    keytool -importcert -file trust.cer -keystore trustedCerts.jks

## Java SSL client

In order to create an ssl session using JSSE, one has to configure and initiate an SSLContext object. SSLContext can be initiated with one or more KeyManager and one or more TrustManager objects. KeyManager is the interface responsible for the selection of the authentication details that will eventually be sent to the remote host, while TrustManager determines whether the remote authentication credentials (and thus the connection) should be trusted or not.

In order to get the desired KeyManager I created a KeyStore and loaded the client.jks file on it, using keystore’s password. Then, I created a KeyManagerFactory and initiated it with the keystore above and the key password. As already said, client keystore is locked with a password and it’s private key is also locked with its own password, so I had to provide them both.

```java
KeyStore keyStore = KeyStore.getInstance("JKS");
InputStream keyStoreIS = new FileInputStream("./src/test/resources/keystores/client.jks");
keyStore.load(keyStoreIS, "123456".toCharArray());
KeyManagerFactory kmf = KeyManagerFactory.getInstance(KeyManagerFactory.getDefaultAlgorithm());
kmf.init(keyStore, "456789".toCharArray());
```

In order to get the desired TrustManagers I created a KeyStore and loaded the trustedCerts.jks file on it. Then I created and initiated a TrustManagerFactory:

```java
KeyStore trustStore = KeyStore.getInstance("JKS");
InputStream trustStoreIS = new FileInputStream("./src/test/resources/keystores/trustedCerts.jks");
trustStore.load(trustStoreIS, "123123".toCharArray());
TrustManagerFactory trustFactory = TrustManagerFactory.getInstance(TrustManagerFactory.getDefaultAlgorithm());
trustFactory.init(trustStore);
```

and finally created the SSLContext:

```java
SSLContext context = SSLContext.getInstance("TLSv1.2");
context.init(kmf.getKeyManagers(), trustFactory.getTrustManagers(), new SecureRandom());
```

From this point, one can (after a number of steps) get the SSLSession and the input/output streams to this session:

```java
SSLSocketFactory factory = context.getSocketFactory();
SSLSocket socket = (SSLSocket) factory.createSocket("<ip-address", <port>);
SSLSession session = socket.getSession();
InputStream is = socket.getInputStream();
OutputStream os = socket.getOutputStream();
```

## Useful Links:

* http://chimera.labs.oreilly.com/books/1230000000545/ch04.html
* https://tools.ietf.org/html/rfc5246
* http://www-01.ibm.com/support/knowledgecenter/SSFKSJ_7.1.0/com.ibm.mq.doc/sy10660_.htm?lang=en
* http://security.stackexchange.com/questions/20803/how-does-ssl-tls-work
* https://en.wikipedia.org/wiki/Public-key_cryptography
* http://serverfault.com/questions/9708/what-is-a-pem-file-and-how-does-it-differ-from-other-openssl-generated-key-file
* http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html
