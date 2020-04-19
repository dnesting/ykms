# ykms

yKMS is a proposed Key Management Service providing basic decryption services using hardware keys on commodity hardware.

The MVP will implement the Kubernetes [KMS service API](https://github.com/kubernetes/kubernetes/blob/master/staging/src/k8s.io/apiserver/pkg/storage/value/encrypt/envelope/v1beta1/service.proto) using an off-the-shelf Yubikey with PIV support.

The YubiKey supports the following [PIV algoriths](https://support.yubico.com/support/solutions/articles/15000014219-yubikey-5-series-technical-manual) for keys stored on the YubiKey:

- RSA 1024
- RSA 2048
- ECC P-256
- ECC P-384

## API

```proto
    // both request messages just deal in bytes
    rpc Decrypt(DecryptRequest) returns (DecryptResponse) {}
    rpc Encrypt(EncryptRequest) returns (EncryptResponse) {}
```

## Future directions

The MVP has a number of deficiencies:
- `Decrypt` operations will be slow because they all require the YubiKey.  Too slow for even a hobby service.
- `Encrypt` and `Decrypt` operations will be inherently slow just because they use asymmetric cryptograph.
- The entire system fails if the YubiKey fails, or is lost or stolen.
- The service must run on a single replica with access to the YubiKey, which adds additional availability risk.

A logical next step involves introducing an additional "master key".  Master keys would be used for all `Encrypt` and `Decrypt` operations.  The master key would only be stored unencrypted in memory.  In order to persist the key to storage, it would be wrapped using the YubiKey.  This increases the attack surface, but it's necessary to create a usable service.

Another potential future direction would be to export a RNG service for slow but cryptographically-secure randomness for applications to use as seeds?
