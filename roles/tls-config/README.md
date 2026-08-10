# tls-config

Installs a third-party TLS key pair for Splunk Web (default tcp/8000).

Used by `tls-config.yml`.

## Files you provide

Place the pair in the project's `certs/` directory before running:

- `certs/cert.pem` - the certificate, including any intermediate chain
- `certs/privkey.pem` - the matching private key

Both are gitignored. The role uploads them to
`$SPLUNK_HOME/etc/auth/mycerts/`, with the key mode `0600`, and sets
`serverCert`, `privKeyPath` and `enableSplunkWebSSL` in
`$SPLUNK_HOME/etc/system/local/web.conf`.

## Notes

- Only those three keys are managed; the rest of `web.conf` is left alone.
  This matters because Splunk writes its own content there - an encrypted
  `sslPassword` once TLS is enabled, and a large block of `[expose:...]`
  stanzas in Splunk 10.
- `web.conf` is managed at mode `0600`, matching what Splunk sets once it
  stores `sslPassword`.
- The role can run standalone: `web.conf` is created if it does not exist.
- Splunk is restarted through the `restart splunk` handler, so nothing is
  restarted when the certificate and settings are already in place.
