# Ansible-Splunk-Base ./certs/

To upload custom SSL certs to install with Splunk for the Web UI (default tcp/8000):

1. Place your public key (PEM format) in certs/cert.pem (include intermediate chain after the public key if available).

2. Place your private key (PEM format) in certs/privkey.pem

3. This project will upload the certs to their default locations and restart the Phantom nginx server.