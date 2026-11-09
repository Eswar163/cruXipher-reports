# boxy proxy



[Original report, page 6](../../cruXipher_CTF_BreakingBad_Report.pdf#page=6)

**Category** Web / cryptography
**Points** 500 (dynamic)

The service at https://proxy.cruxipher.crux-bphc.com/ advertised a custom proxy protocol. Its supplied CA certificate
and HKDF hint suggested that registration involved an authenticated key exchange. The original investigation found that
POST /register-proxy attempted an outbound connection even when the HTTP response was 500 {"success":false}.
The callback, rather than the HTTP body, carried the useful result.

Protocol investigation
Registering a public TCP listener exposed a plaintext destination line, recorded as 144.24.133.118:8053, followed by
four-byte big-endian length-prefixed messages: a greeting, an RSA public key, a raw 32-byte X25519 public key, and an
RSA-PSS signature over that X25519 key. The expected reply was a CA-signed certificate, our X25519 public key, and its
signature made with the certificate's private key.

The supplied CN=Pre-established CA certificate used an RSA modulus whose two factors were recorded in
ca_factors.json. We can reconstruct the signing key because knowing both primes gives the private exponent:

```
d = pow(e, -1, (p - 1) * (q - 1))
```

The solver checks p * q == pub.n before constructing the RSA CRT parameters. The saved factors pass that check and
reconstruct a working signing key. The original factorization was obtained from FactorDB.

Completing the handshake
The documented verifier accepted the self-signed CA certificate itself as the client certificate. The solver therefore
verifies the server's RSA-PSS/SHA-256 signature, signs our own raw X25519 public key using the recovered CA key, and
sends the three reply frames. Both peers derive the session key as follows:

```
shared = client_key.exchange(X25519PublicKey.from_public_bytes(server_pub))
aes_key = derive_aes_key(shared)
plaintext = AESGCM(aes_key).decrypt(msg[:12], msg[12:], None)
```

derive_aes_key uses HKDF-SHA-256, length 32, no salt, and info=b"handshake encryption". The first 12 bytes of the
final message are the GCM nonce. The recorded plaintext begins “Hello Bob!” and contains:

```
cruXipher{som4h4W_b4tt4r_th4n_eMud4ra}
```

From the working directory, the supplied solver can be invoked with a public endpoint forwarded to local TCP port 4444:

```
python boxy-proxy/solve_boxy_proxy.py --port 4444 \
  --public-host <tunnel-host> --public-port <tunnel-port>
```

The security failure is the recoverable trust-anchor key combined with acceptance of a CA as a leaf identity. X25519 and
AES-GCM do not compensate for compromised authentication. The RSA key sent in the same callback establishes
signature consistency; that check alone does not establish an external identity.

Step-by-step procedure

1. Establish what registration actually does
The initial request accepts a host and port, rather than a proxy URL or credentials:

```
POST /register-proxy
Content-Type: application/json
```

```
{"host":"example.com","port":443}
```

The HTTP 500 did not distinguish a rejected registration from a callback that had already started. The recorded delay of
roughly ten seconds for an unreachable address showed that the application was attempting a connection. That
observation justified moving the investigation to a listener we controlled.

2. Capture the callback before implementing the response
The original setup used a Pinggy tunnel forwarded to local port 4444:

```
ssh -p 443 -R0:localhost:4444 tcp@a.pinggy.io
```

Once the returned host and port were registered, the server sent its destination line and greeting. The greeting specified
both the frame format and the required authentication exchange. The solver reads each four-byte header as >I, then
loops until the declared number of bytes has arrived. A single recv() would be insufficient because TCP does not
preserve application message boundaries.

3. Recover the signing identity
Load the PEM certificate, read its RSA public numbers, and load the two saved factors. Verify their product before
calculating the inverse exponent. The solver constructs dmp1, dmq1 and iqmp as well as d, supplying the CRT parameters
required by the cryptography library. This recovers a usable signing key without changing the certificate sent to the
verifier.

4. Match the server's cryptographic parameters exactly
The server's signature covers the raw X25519 public-key bytes, not their hex representation or a PEM wrapper. RSA-PSS
uses SHA-256, MGF1-SHA-256 and maximum salt length. Our reply signs the same representation with the recovered CA
private key. The reply order is certificate, X25519 public key, then signature; all three use the same length-prefix format.

5. Decrypt the application message
The X25519 exchange supplies the shared secret. HKDF derives 32 bytes using the exact context string handshake
encryption; changing that string produces a different AES key. Split the encrypted frame into a 12-byte nonce and the
remaining GCM ciphertext/tag, and decrypt with no additional authenticated data. A valid GCM tag confirms that the
key exchange and message parsing agree with the server. The flag arrives here, independently of the registration
endpoint's HTTP status.

**Source files** boxy-proxy-writeup.md, boxy-proxy/README.md, boxy-proxy/solve_boxy_proxy.py, boxy-proxy/cert/ca_cert.pem, boxy-proxy/cert/ca_factors.json.
