# Purchased TLS certificate files

Place the certificate files here using these names:

- `fullchain.pem`: site certificate followed by intermediate CA certificate(s)
- `privkey.pem`: matching unencrypted private key

These files are mounted read-only into nginx and ignored by Git.
