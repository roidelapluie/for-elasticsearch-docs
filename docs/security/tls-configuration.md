---
layout: default
title: TLS Certificates
parent: Security
nav_order: 2
---

# Configure TLS certificates

TLS is configured in `elasticsearch.yml`. There are two main configuration sections: transport layer and REST layer. TLS is optional for the REST layer and mandatory for the transport layer.

You can find an example configuration template with all options on [GitHub](https://www.github.com/opendistroforelasticsearch/security-ssl/blob/v0.0.7.0/opendistrosecurity-ssl-config-template.yml)
{: .note }


## X.509 PEM certificates and PKCS \#8 keys

The following tables contain the settings you can use to configure the location of your PEM certificates and private keys.


### Transport layer TLS

Name | Description
:--- | :---
opendistro_security.ssl.transport.pemkey_filepath | Path to the certificate's key file (PKCS \#8), which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.transport.pemkey_password | Key password. Omit this setting if the key has no password. Optional.
opendistro_security.ssl.transport.pemcert_filepath | Path to the X.509 node certificate chain (PEM format), which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.transport.pemtrustedcas_filepath | Path to the root CA(s) (PEM format), which must be under the `config/` directory, specified using a relative path . Required.


### REST layer TLS

Name | Description
:--- | :---
opendistro_security.ssl.http.pemkey_filepath | Path to the certificate's key file (PKCS \#8), which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.http.pemkey_password | Key password. Omit this setting if the key has no password. Optional.
opendistro_security.ssl.http.pemcert_filepath | Path to the X.509 node certificate chain (PEM format), which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.http.pemtrustedcas_filepath | Path to the root CA(s) (PEM format), which must be under the `config/` directory, specified using a relative path . Required.


## Keystore and truststore files

As an alternative to certificates and private keys in PEM format, you can instead use keystore and truststore files in JKS or PKCS12/PFX format. The following settings configure the location and password of your keystore and truststore files. If desired, you can use different keystore and truststore files for the REST and the transport layer.


### Transport layer TLS

Name | Description
:--- | :---
opendistro_security.ssl.transport.keystore\_type | The type of the keystore file, JKS or PKCS12/PFX. Optional. Default is JKS.
opendistro_security.ssl.transport.keystore\_filepath | Path to the keystore file, which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.transport.keystore\_alias: my\_alias | Alias name. Optional. Default is the first alias.
opendistro_security.ssl.transport.keystore_password | Keystore password. Default is `changeit`.
opendistro_security.ssl.transport.truststore_type | The type of the truststore file, JKS or PKCS12/PFX. Default is JKS.
opendistro_security.ssl.transport.truststore_filepath | Path to the truststore file, which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.transport.truststore\_alias | Alias name. Optional. Default is all certificates.
opendistro_security.ssl.transport.truststore_password | Truststore password. Default is `changeit`.


### REST layer TLS

Name | Description
:--- | :---
opendistro_security.ssl.http.enabled | Whether to enable TLS on the REST layer. If enabled, only HTTPS is allowed. Optional. Default is false.
opendistro_security.ssl.http.keystore\_type | The type of the keystore file, JKS or PKCS12/PFX. Optional. Default is JKS.
opendistro_security.ssl.http.keystore\_filepath | Path to the keystore file, which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.http.keystore\_alias | Alias name. Optional. Default is the first alias.
opendistro_security.ssl.http.keystore_password | Keystore password. Default is `changeit`.
opendistro_security.ssl.http.truststore_type | The type of the truststore file, JKS or PKCS12/PFX. Default is JKS.
opendistro_security.ssl.http.truststore_filepath | Path to the truststore file, which must be under the `config/` directory, specified using a relative path. Required.
opendistro_security.ssl.http.truststore\_alias | Alias name. Optional. Default is all certificates.
opendistro_security.ssl.http.truststore_password | Truststore password. Default is `changeit`.


## Configure node certificates

The Security plugin needs to identify inter-cluster requests (i.e. requests between the nodes). The simplest way of configuring node certificates is to list the Distinguished Names (DNs) of these certificates in `elasticsearch.yml`. The Security plugin supports wildcards and regular expressions:

```yaml
opendistro_security.nodes_dn:
  - 'CN=node.other.com,OU=SSL,O=Test,L=Test,C=DE'
  - 'CN=*.example.com,OU=SSL,O=Test,L=Test,C=DE'
  - 'CN=elk-devcluster*'
  - '/CN=.*regex/'
```

If your node certificates have an OID identifier in the SAN section, you can omit this configuration.


## Configure admin certificates

Admin certificates are regular client certificates that have elevated rights to perform administrative tasks. You need an admin certificate to change the the Security plugin configuration using the REST API. Admin certificates are configured in `elasticsearch.yml` by stating their DN(s):

```yaml
opendistro_security.authcz.admin_dn:
  - CN=admin,OU=SSL,O=Test,L=Test,C=DE
```

For security reasons, you cannot use wildcards nor regular expressions here.


## OpenSSL

The Security plugin supports OpenSSL. We recommend OpenSSL in production for enhanced performance and a wider range of modern cipher suites. In order to use OpenSSL, you need to install OpenSSL, the Apache Portable Runtime, and a Netty version with OpenSSL support matching your platform on all nodes.

If OpenSSL is enabled, but for one reason or another the installation does not work, the Security plugin falls back to the Java JCE as the security engine.

Name | Description
:--- | :---
opendistro_security.ssl.transport.enable\_openssl\_if\_available | Enable OpenSSL on the transport layer if available. Optional. Default is true.
opendistro_security.ssl.http.enable\_openssl\_if\_available | Enable OpenSSL on the REST layer if available. Optional. Default is true.

1. Install [OpenSSL 1.1.0](https://www.openssl.org/community/binaries.html) on every node.
1. Install [Apache Portable Runtime ](https://apr.apache.org) on every node:

  ```
  sudo yum install apr
  ```

1. Download [netty-tcnative](https://netty.io/wiki/forked-tomcat-native.html) for RPM-based distributions (`_linux-x86_64-fedora.jar_`) and place it into `plugins/opendistro_security/` on every node.


## (Advanced) Hostname verification and DNS lookup

In addition to verifying the TLS certificates against the root CA and/or intermediate CA(s), the Security plugin can apply additional checks on the transport layer.

With `hostname_verification` enabled, the Security plugin verifies that the hostname of the communication partner matches the hostname in the certificate. For example, if the hostname of your node is `node-0.example.com`, then the hostname in the TLS certificate has to be set to `node-0.example.com`, as well. Otherwise, an error is thrown. The hostname is taken from the `subject` or `SAN` entries of your certificate.

In addition, when `resolve_hostnames` is enabled, the Security plugin resolves the (verified) hostname against your DNS. If the hostname does not resolve, an error is thrown.

Name | Description
:--- | :---
opendistro_security.ssl.transport.enforce\_hostname\_verification | Whether to verify hostnames on the transport layer. Optional. Default is true.
opendistro_security.ssl.transport.resolve\_hostname | Whether to resolve hostnames against DNS on the transport layer. Optional. Default is true. Only works if hostname verification is also enabled.


## (Advanced) Client authentication

With TLS client authentication enabled, REST clients can send a TLS certificate with the HTTP request to provide identity information to the Security plugin. There are three main usage scenarios for TLS client authentication:

- Providing an admin certificate when using the REST management API.
- Configuring roles and permissions based on a client certificate.
- Providing identity information for tools like Kibana, Logstash or Beats.

TLS client authentication has three modes:

* `NONE`: The Security plugin does not accept TLS client certificates. If one is sent, it is discarded.
* `OPTIONAL`: The Security plugin accepts TLS client certificates if they are sent, but does not require them.
* `REQUIRE`: The Security plugin only accepts REST requests when a valid client TLS certificate is sent.

For the REST management API, the client authentication modes has to be OPTIONAL at a minimum.

You can configure the client authentication mode by using the following setting:

Name | Description
:--- | :---
opendistro_security.ssl.http.clientauth_mode | The TLS client authentication mode to use. Can be one of `NONE`, `OPTIONAL` (default) or `REQUIRE`. Optional.


## (Advanced) Enabled ciphers and protocols

You can limit the allowed ciphers and TLS protocols for the REST layer. For example, you can only allow strong ciphers and limit the TLS versions to the most recent ones.

If this setting is not enabled, the ciphers and TLS versions are negotiated between the browser and the Security plugin automatically, which in some cases can lead to a weaker cipher suite being used. You can configure the ciphers and protocols using the following settings.

Name | Description
:--- | :---
opendistro_security.ssl.http.enabled_ciphers | Array, enabled TLS cipher suites for the REST layer. Only Java format is supported.
opendistro_security.ssl.http.enabled_protocols | Array, enabled TLS protocols for the REST layer. Only Java format is supported.
opendistro_security.ssl.transport.enabled_ciphers | Array, enabled TLS cipher suites for the transport layer. Only Java format is supported.
opendistro_security.ssl.transport.enabled_protocols | Array, enabled TLS protocols for the transport layer. Only Java format is supported.

### Example settings

```yaml
opendistro_security.ssl.http.enabled_ciphers:
  - "TLS_DHE_RSA_WITH_AES_256_CBC_SHA"
  - "TLS_DHE_DSS_WITH_AES_128_CBC_SHA256"
opendistro_security.ssl.http.enabled_protocols:
  - "TLSv1.1"
  - "TLSv1.2"
```

Because it is insecure, the Security plugin disables `TLSv1` by default. If you need to use `TLSv1` and accept the risks, you can still enable it:

```yaml
opendistro_security.ssl.http.enabled_protocols:
  - "TLSv1"
  - "TLSv1.1"
  - "TLSv1.2"
```


## (Advanced) Disable client initiated renegotiation for Java 8

Set `-Djdk.tls.rejectClientInitiatedRenegotiation=true` to disable secure client initiated renegotiation, which is enabled by default. This can be set via `ES_JAVA_OPTS` in `config/jvm.options`.
