---
title: SSL/TLS for Laymen + Java folk and why it is important
updated: 2018-1-07
title_stub: ssl
contributor: Matt Kornfield
---

Hi from guest writer Matt here! If you are reading this, you are most likely using some sort of browser. This page itself is not served using HTTPS, but many many other sites, like Google, Facebook, etc. will transport data using SSL/TLS.

You might be aware of what it is, but could you explain Secure Socket Layer/ Transport Layer Security to a layman? Can you picture in your mind what is happening when you type https://google.com into the browser? Allow us to take a journey, and we'll start with a simple trip into the 90s, so prepare your VHS box sets.

## A brief history

Back in the Netscape days of yore, the first secured traffic was established by SSL v1. It was never released to the public, but soon after v2 was released. There were major issues with v2 that lead to a rushed v3. Soon after, Microsoft and Netscape fought because, well, it is what [Microsoft does best][Microsoft_netscape_fight], and when v3.1 of SSL should have appeared, we instead had a new thing called TLS, which is pretty much just v3.1 of SSL with a [nice RFC][TLS_rfc] attached to it (meaning it is an open standard as well). And since then, more versions of TLS have appeared to make the web even more secure (TLS 1.2 is the latest). [More info here][TLS_vs_SSL].

And how does this mystical technology work? How do you transfer something securely without having ever traded a private key before? How would I ship something to your house and ensure only you could open it? That my friends, is where asymmetric encryption comes in

## Asymmetric encryption

An overly simplified explanation of asymmetric encryption is a way of encrypting something that makes it very difficult to decrypt unless you have certain keys. Brute forcing of this type can take anywhere from [thousands of years to millions of years to break][decrypting_rsa]. The main mechanism for asymmetric encryption that is widely used is RSA (Rivest–Shamir–Adleman) encryption. It relies on two things:

* A public key, which is available to all parties
* A private key, which is only on the server

Before delving into the mathematics of RSA, I think it's time for a more real world example. The mechanism of our story is different but the principle is the same. Let us turn to Bill and Ted for a short anecdote.

### Bill and Ted's excellent mail security

_This is based on a description in the book [How Not to be Wrong by Jordan Ellenberg][how_not_to_be_wrong]_

Bill has just moved to a new address and wants to send Ted a letter. He is worried that evil-doers will intercept the letter; these evil-doers are clever and will be able to break any plain cipher or simple encoding mechanism. So Bill comes up with a great idea. He decides that he is going to put the letter in an impenetrable lockbox, and then send it to Ted. **The impenetrable lockbox will be locked with a key that only Bill has in his possession.** Bill also places another lock and a key, which he has his own copy of, inside the box.

How does this make it possible for Ted to retrieve the letter? Well, this is where it gets interesting.

We will say the box below is Bill's, secured by lock B

```
 [Bill's house] -> [ B ]  ->  [Ted's House]
```

Ted receives the impenetrable box. His first response is "Wohhh", but then, remembering what Bill had said to him earlier, he goes into his house and find his own impenetrable lock. He places the lock around the box, and then mails it back to Bill.

We now have a box with two locks on it.

```
 [Bill's house] <- [ B T ]  <-  [Ted's House]
```

Bill receives the box with two locks. What is he to do? He simply undoes his first lock and pops it back in the mail. The box is now

```
 [Bill's house] -> [ T ] -> [Ted's House]
```

Once it arrives on Ted's doorstep, he will be able to unlock his own lock and have access to the letter, which tells Ted that he is _'totally awesome'_ and _'far out'_. He also now has a lock that he can use to send a response to Bill, without having to do the same son and dance as before.

This is a good proxy for SSL/LTS, as we use it in modern webservers.

### Certifcation? You want to see my certification?

Public keys come in these nice formats called certificates. Other than just being a large number, they have additional metadata that is used to protect clients, like fingerprints and lists of hostnames. The general standard for the internet is X.509, which is the public key matched to a hostname.

The aforementioned assymetric encryption is done through means of RSA, where a certificate, or "public key," is sent by the server to client when the client first reaches out. This public key is almost always signed by a CA, or certificate authority, which is a third party deemed to be worthy of signing/ issuing certificates, so that some malicious entity doesn't pretend to do this key exchange with their own, made up certificate. Sending the public key is essentially the first pass, where Bill sends Ted a locked box.

This public key is then used by the client to send a new key, using the public key as the encryption mechanism. The public key is essentially a very large number. I navigated to google, downloaded the certificate and this is what the public key looks like:

```
04 e2 68 44 79 57 f7 3d 67 6a e3 90 75 db f6 b7 e5 ff a4 92 59 de b6 63 9c 06 ac a0 3f 19 c0 33 14 5e 09 4f 57 b4 da 6f 44 9d 95 fc 94 ec 81 04 71 a8 bf 10 0c b6 f5 36 cb 12 d3 75 ce ba 61 3b 59
```

*Gasp* didn't I just give something horrible away, you might think? Well no, because like the lockbox Bill sent, you can't do anything special with this. It is just something that you use to produce a very large number, which is the mechanism for delivering a shared private key. The server has a private key that allows it to easily retrieve the key you sent. The "easy mechanism" in Bill and Ted's version of the story is the lock that Ted placed around the box.

A slightly more in depth version of [the math is here][ssl_initial_connection_explained].

Once this new MAC (message authentication code) is established via a so called "handshake", things get [a bit more complicated][tls_complications] as there are additional keys involved, but essentially that is all invisible to the users of SSL/LTS. At this point, symmetric encryption is used to communicate.

### Invisible you say?

Yes invisible. One of the most important parts of SSL/LTS is that the application layer, or HTTP layer, is agnostic to this type of interaction. There is some overhead but it is fixed for symmetric encryption. "HTTPS" is really "HTTP" wrapped in SSL/LTS transport security.

### And now for something completely Java

Java has many different webservice connection mechanisms, but in your travels you will most likely encounter Apache's Http Commons. Http Commons, whose latest version is 3.x, is actually a library for which support has been dropped. It has significant issues (besides being old and unmaintained), but its worst crime is that it does not do proper certificate verification, which leaves it vulnerable to various attacks. The only way to fix it is to upgrade to a newer version, which for certain libraries means dropping support for them altogether or doing major upgrades.

The mechanism in Java to use SSL is generally via java.net. An example bit of code to create an SSL context and apply it to the connection manager for Apache Httpclient4 goes something like:


```java
  //Establish context used for socket connection
  SSLContext localSslContext = SSLContext.getInstance("SSL");
  localSslContext.init(getKeyManagers(), null, null);

  // Create a scheme registry
  SchemeRegistry registry = new SchemeRegistry();
  registry.register(new Scheme("http", PlainSocketFactory.getSocketFactory(), 80));
  registry.register( new Scheme("https", new SSLSocketFactory(localSslContext), 443));

  // Set on the connection manager
  ClientConnectionManager connectionManager = new SingleClientConnManager(schemeRegistry);

 ```

Newer code, using an HttpClientBuilder, would simply set the ssl context, using `httpClientBuilder.setSSLContext(localSslContext)`. Internally, when creating a socket, Java will initiate this assymetric key exchange before sending any data via HTTP.

Among the myriad things to know about Java and SSL is that the Java Virtual Machine (JVM) has its own keystore. The code above calls a `getKeyManagers()` method, which essentially returns an array of KeyManagers that manage the various private keys of the server, including the file specified by the property `"javax.net.ssl.keyStore"`. The keyStore is where Java stores its private key for encryption, and an additional place that is of note is the trustStore, which uses a similar property value for [access in Java][keystore_trustore]. When a certiicate is in the truststore, it will be trusted even if it is expired or not signed by a third party (these types of certificates are sometimes called _Client Certificates_).

To update these certificates one can also use a [keytool][keytool] to export or import certificates, or for testing purposes simply change the System properties to point to different files.

## Conclusion

SSL/LTS is very important because for many internet based systems it is the only mechanism of transport encryption. Usage of older versions of the protocol are dangerous because they expose values that are meant to be secure. You can think of the initial handshake mechanism as the locking of an already locked box, in order to exchange a special key. Java has a few different mechanisms for utilizing SSL, as well as ways to change what types of private keys or trusted certificates are used by Java.

[how_not_to_be_wrong]: https://www.amazon.com/How-Not-Be-Wrong-Mathematical/dp/0143127535
[nice_little_timeline]: https://www.feistyduck.com/ssl-tls-and-pki-history/
[ssl_initial_connection_explained]: https://security.stackexchange.com/questions/6290/how-is-it-possible-that-people-observing-an-https-connection-being-established-w
[TLS_vs_SSL]: https://security.stackexchange.com/questions/5126/whats-the-difference-between-ssl-tls-and-https
[TLS_rfc]: https://tools.ietf.org/html/rfc2246
[Microsoft_netscape_fight]: http://tim.dierks.org/2014/05/security-standards-and-name-changes-in.html
[decrypting_rsa]: https://www.digicert.com/TimeTravel/math.htm
[tls_complications]: https://crypto.stackexchange.com/questions/1139/what-is-the-purpose-of-four-different-secrets-shared-by-client-and-server-in-ssl
[keystore_trustore]: https://stackoverflow.com/a/6341566
[keytool]: https://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html
