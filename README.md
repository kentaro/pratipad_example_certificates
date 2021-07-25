# How to create TLS certificates for the example system using Pratipad

This is the document of how I created TSL certificates for [the example system using Pratipad](https://github.com/kentaro/pratipad_example).

## Setup a Vault server

### Start a Valut server

Start Vault server using `config/vault.hcl`:

```sh
⟩ vault server -config=config/vault.hcl
==> Vault server configuration:

                     Cgo: disabled
              Go Version: go1.16.5
              Listener 1: tcp (addr: "0.0.0.0:8200", cluster address: "0.0.0.0:8201", max_request_duration: "1m30s", max_request_size: "33554432", tls: "disabled")
               Log Level: info
                   Mlock: supported: false, enabled: false
           Recovery Mode: false
                 Storage: file
                 Version: Vault v1.7.3
             Version Sha: 5d517c864c8f10385bf65627891bc7ef55f5e827+CHANGES

==> Vault server started! Log data will stream in below:

2021-07-25T23:19:01.706+0900 [INFO]  proxy environment: http_proxy="" https_proxy="" no_proxy=""
2021-07-25T23:19:01.706+0900 [WARN]  no `api_addr` value specified in config or in VAULT_API_ADDR; falling back to detection if possible, but this value should be manually set
```

### Initialize the Vault server

We're going to communicate to this server from another terminal.

Initialize the Vault server:

```sh
⟩ vault operator init
Unseal Key 1: +bYVazQ2ZtEoliKAPeBlx9aWOdF+2iHW8w7ImbcTM9Ex
Unseal Key 2: 9ytVgtxXGnFx7f5c1KrD/oj8IZGxenCTZpJ9fMcdcQRA
Unseal Key 3: E7F+Hh7Z5clI6DHcVdvg+5jBjkvj/VrpufNuE81a0Re9
Unseal Key 4: LUEo+rxb1emvlnCaTKbqjUZdcrXN/04dvJU49Hw2yrOF
Unseal Key 5: IzMuVcsNDRhqjZ2GTdCfITj4/fp7kfl2A7lLM1wuI37A

Initial Root Token: s.B7mRKzF3zAGbPJt1jHAd6KC0

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

Unseal the Vault server:

```sh
⟩ vault operator unseal +bYVazQ2ZtEoliKAPeBlx9aWOdF+2iHW8w7ImbcTM9Ex
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    1/3
Unseal Nonce       0652bf5b-6e6f-a281-9a7b-8185447a1249
Version            1.7.3
Storage Type       file
HA Enabled         false

⟩ vault operator unseal 9ytVgtxXGnFx7f5c1KrD/oj8IZGxenCTZpJ9fMcdcQRA
Key                Value
---                -----
Seal Type          shamir
Initialized        true
Sealed             true
Total Shares       5
Threshold          3
Unseal Progress    2/3
Unseal Nonce       0652bf5b-6e6f-a281-9a7b-8185447a1249
Version            1.7.3
Storage Type       file
HA Enabled         false

⟩ vault operator unseal E7F+Hh7Z5clI6DHcVdvg+5jBjkvj/VrpufNuE81a0Re9
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.7.3
Storage Type    file
Cluster Name    vault-cluster-bef7f964
Cluster ID      c7f3c5b2-13fb-28cf-549a-d23c6205f12f
HA Enabled      false
```

Log in to the Vault server:

```sh
⟩ vault login
Token (will be hidden):
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.B7mRKzF3zAGbPJt1jHAd6KC0
token_accessor       L8UaJMfTEmXeVpKe7qWjF8we
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
```

## Generate a CA certificate

Enable PKI Secrets Engine:

```sh
⟩ vault secrets enable -path=pratipad -description="Root CA for Pratipad Example" pki
Success! Enabled the pki secrets engine at: pratipad/

⟩ vault secrets tune -max-lease-ttl=87600h pratipad
Success! Tuned the secrets engine at: pratipad/
```

You can see the secrets just enabled above:

```sh
⟩ vault secrets list
Path          Type         Accessor              Description
----          ----         --------              -----------
cubbyhole/    cubbyhole    cubbyhole_db5adfaa    per-token private secret storage
identity/     identity     identity_514fb6a3     identity store
pratipad/     pki          pki_5b4da43d          Root CA for Pratipad Example
sys/          system       system_549faebd       system endpoints used for control, policy and debugging
```

Generate a self-signed CA certificate:

```sh
⟩ vault write pratipad/root/generate/internal common_name="Root CA for Pratipad Example" ttl=87600h key_bites=4096 exclude_cn_from_sans=true
Key              Value
---              -----
certificate      -----BEGIN CERTIFICATE-----
MIIDPzCCAiegAwIBAgIUXy1xy84rqXS808YnH0VAhEr0JpwwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNDQ0NDdaFw0zMTA3MjMxNDQ1MTZaMCcxJTAjBgNVBAMTHFJvb3QgQ0Eg
Zm9yIFByYXRpcGFkIEV4YW1wbGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCjMwjVwiDhl4E2NrlB0zMlYLydGJoPZlUFrlKa5vGyZeI2jPnvAgC0VRGp
x3Saa2dLVItLmRZ5/3z6JRhSBhAy4iBYNA1fdEULvvpS2S9tuKjUEAJLSh9yiqeg
PU72wIgjCPGHC/gvnE7Ch4i1V9GhELMvlQXAREeuspf3CdzeTzpVl+7/tsi6G02O
wRKG1diMRI+1rHFBUKUlcH265o5+K1YUrBveUoXBOmkPcCm7W3NFELTQWGyNitg8
pdEFXSIC2A5eG0Ahuhq3v+FEpp7BY1M8p09mXfIjDei0OOwcsyTJvRBsvcbPM7qv
RHucZ0uZOgkxek0duCwKtKPGRsezAgMBAAGjYzBhMA4GA1UdDwEB/wQEAwIBBjAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBS6ayhLuZumy/EPY7FeZTGYB02TVzAf
BgNVHSMEGDAWgBS6ayhLuZumy/EPY7FeZTGYB02TVzANBgkqhkiG9w0BAQsFAAOC
AQEAbEAPCvphPgd/KydvS+XU+eEOL2L3S4HDOnPFdXAZm6YjL8siCfiTwwZmi+qT
424yaiFa/5K5sffBLW/fRai0ydH1eEHC3JUhmv1OZjtMFOdmFSSqMKM+nzu5wtIQ
GVpRF3T7M98c9GYTPZyHfYfxIIzzQdlg2Iw054UFea224nnFobOHx75XEAZT+J7f
Irtc9Vz5CBJcKqNUOnbBboXpCvS9e1YoXte0D0flhitgvYa+UpUHlhK87zrTDqSM
XOMPNWKVBELfiXRQ3aOBX2g5HQJU0BYixlwH+FwAaIJ8x1XVSpBAn7/Fnl5BUp3q
kDo2P0RFKVxIAih2P5ZYUnpDZw==
-----END CERTIFICATE-----
expiration       1942584316
issuing_ca       -----BEGIN CERTIFICATE-----
MIIDPzCCAiegAwIBAgIUXy1xy84rqXS808YnH0VAhEr0JpwwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNDQ0NDdaFw0zMTA3MjMxNDQ1MTZaMCcxJTAjBgNVBAMTHFJvb3QgQ0Eg
Zm9yIFByYXRpcGFkIEV4YW1wbGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCjMwjVwiDhl4E2NrlB0zMlYLydGJoPZlUFrlKa5vGyZeI2jPnvAgC0VRGp
x3Saa2dLVItLmRZ5/3z6JRhSBhAy4iBYNA1fdEULvvpS2S9tuKjUEAJLSh9yiqeg
PU72wIgjCPGHC/gvnE7Ch4i1V9GhELMvlQXAREeuspf3CdzeTzpVl+7/tsi6G02O
wRKG1diMRI+1rHFBUKUlcH265o5+K1YUrBveUoXBOmkPcCm7W3NFELTQWGyNitg8
pdEFXSIC2A5eG0Ahuhq3v+FEpp7BY1M8p09mXfIjDei0OOwcsyTJvRBsvcbPM7qv
RHucZ0uZOgkxek0duCwKtKPGRsezAgMBAAGjYzBhMA4GA1UdDwEB/wQEAwIBBjAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBS6ayhLuZumy/EPY7FeZTGYB02TVzAf
BgNVHSMEGDAWgBS6ayhLuZumy/EPY7FeZTGYB02TVzANBgkqhkiG9w0BAQsFAAOC
AQEAbEAPCvphPgd/KydvS+XU+eEOL2L3S4HDOnPFdXAZm6YjL8siCfiTwwZmi+qT
424yaiFa/5K5sffBLW/fRai0ydH1eEHC3JUhmv1OZjtMFOdmFSSqMKM+nzu5wtIQ
GVpRF3T7M98c9GYTPZyHfYfxIIzzQdlg2Iw054UFea224nnFobOHx75XEAZT+J7f
Irtc9Vz5CBJcKqNUOnbBboXpCvS9e1YoXte0D0flhitgvYa+UpUHlhK87zrTDqSM
XOMPNWKVBELfiXRQ3aOBX2g5HQJU0BYixlwH+FwAaIJ8x1XVSpBAn7/Fnl5BUp3q
kDo2P0RFKVxIAih2P5ZYUnpDZw==
-----END CERTIFICATE-----
serial_number    5f:2d:71:cb:ce:2b:a9:74:bc:d3:c6:27:1f:45:40:84:4a:f4:26:9c
```

Save the certificate part from the output above to `certs/ca.crt`.

## Generate certificate and private key pairs

We have three hosts that run as TLS server/client:

* device001.pratipad.local (172.16.0.118)
* dataflow.pratipad.local (172.16.0.136)
* server.pratipad.local (172.16.0.129)

Thus we have to generate certificate and private key pairs for each hosts. Each pair is used for authenticating both server and client connections.

### device001.pratipad.local

Generate a TSL server certificate and a private key and save them to `certs/device001.pratipad.local.crt` and `certs/device001.pratipad.local.key` respectivelly.

```sh
⟩ vault write pratipad/roles/device001.pratipad.local key_bites=2048 max_ttl=8760h allow_any_name=true
Success! Data written to: pratipad/roles/device001.pratipad.local

⟩ vault write pratipad/issue/device001.pratipad.local common_name="device001.pratipad.local" ip_sans="172.16.0.118" ttl=8760h foramt=pem
Key                 Value
---                 -----
certificate         -----BEGIN CERTIFICATE-----
MIIDdjCCAl6gAwIBAgIUAlI3BjwiaOp9RNMr/vMCkMqUutAwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNDU5MjdaFw0yMjA3MjUxNDU5NTdaMCMxITAfBgNVBAMTGGRldmljZTAw
MS5wcmF0aXBhZC5sb2NhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
AL4WbDaDfi7Z49YOBTVyfHpFZiG2vEiji0WD091s2VZTC3/uFwTTfm4JS5NxuGKH
ZTnTTYNP0dQVgyKd0yUAA83UHp3JQUsPvZ0MKTSW31MTPjVCvLf1nI0Ib9hbW4k/
oAERTKmnDaRt6mR+Q1dr3mo/SxH19Jvtxif/UNP5v1VRUejAAqHBMMyDaAb3TxYS
ekjjb8WpfDKcI64uxtWQ98bL58j/1zdvZNcv/2MwcfBwqiYfsN4Fet8BDCms6nxg
ee5DOrDp2oSMc93BNBuZf8HCS6hBN/PBduftFZT9QD0rXbEusmsCQwFDBzYrlL1c
sJEeGrQL5gzqoBPAbzA5LYsCAwEAAaOBnTCBmjAOBgNVHQ8BAf8EBAMCA6gwHQYD
VR0lBBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMB0GA1UdDgQWBBQctUAx5mZ5x7cY
88XYMrQLNbPkOTAfBgNVHSMEGDAWgBS6ayhLuZumy/EPY7FeZTGYB02TVzApBgNV
HREEIjAgghhkZXZpY2UwMDEucHJhdGlwYWQubG9jYWyHBKwQAHYwDQYJKoZIhvcN
AQELBQADggEBAJCyrzGyep1Re84YzhLhygOzQRKX6Ll/2sDZdC6IPviCj30bf5P2
szDjZQ+zW4T3ducqYu1+UjmHIVDvHvvmzsKptWMb2yvHX2v2B3GW+TKgygW3uCll
RPv3bPrhjcy+ZflI92r/AXbo8xxnEOSFhvSiG2iXERGsmNg/ot/fRlYIkXsT3snF
7p40ytROd+t86QoCfVVZfpz4RhodV5S34zGlc5gYbwP2taS3TCQnrWMLwaIgTzHU
l2TjRU7VOCookOzWlH5f/UgCxAmrfraOoQjAjvKs46FagFD9epHInQ1LQSJ/TZrr
gUKfoOwUzY4oRhIonA2UzzOPvi2tf30lDNo=
-----END CERTIFICATE-----
expiration          1658761197
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDPzCCAiegAwIBAgIUXy1xy84rqXS808YnH0VAhEr0JpwwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNDQ0NDdaFw0zMTA3MjMxNDQ1MTZaMCcxJTAjBgNVBAMTHFJvb3QgQ0Eg
Zm9yIFByYXRpcGFkIEV4YW1wbGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCjMwjVwiDhl4E2NrlB0zMlYLydGJoPZlUFrlKa5vGyZeI2jPnvAgC0VRGp
x3Saa2dLVItLmRZ5/3z6JRhSBhAy4iBYNA1fdEULvvpS2S9tuKjUEAJLSh9yiqeg
PU72wIgjCPGHC/gvnE7Ch4i1V9GhELMvlQXAREeuspf3CdzeTzpVl+7/tsi6G02O
wRKG1diMRI+1rHFBUKUlcH265o5+K1YUrBveUoXBOmkPcCm7W3NFELTQWGyNitg8
pdEFXSIC2A5eG0Ahuhq3v+FEpp7BY1M8p09mXfIjDei0OOwcsyTJvRBsvcbPM7qv
RHucZ0uZOgkxek0duCwKtKPGRsezAgMBAAGjYzBhMA4GA1UdDwEB/wQEAwIBBjAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBS6ayhLuZumy/EPY7FeZTGYB02TVzAf
BgNVHSMEGDAWgBS6ayhLuZumy/EPY7FeZTGYB02TVzANBgkqhkiG9w0BAQsFAAOC
AQEAbEAPCvphPgd/KydvS+XU+eEOL2L3S4HDOnPFdXAZm6YjL8siCfiTwwZmi+qT
424yaiFa/5K5sffBLW/fRai0ydH1eEHC3JUhmv1OZjtMFOdmFSSqMKM+nzu5wtIQ
GVpRF3T7M98c9GYTPZyHfYfxIIzzQdlg2Iw054UFea224nnFobOHx75XEAZT+J7f
Irtc9Vz5CBJcKqNUOnbBboXpCvS9e1YoXte0D0flhitgvYa+UpUHlhK87zrTDqSM
XOMPNWKVBELfiXRQ3aOBX2g5HQJU0BYixlwH+FwAaIJ8x1XVSpBAn7/Fnl5BUp3q
kDo2P0RFKVxIAih2P5ZYUnpDZw==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEAvhZsNoN+Ltnj1g4FNXJ8ekVmIba8SKOLRYPT3WzZVlMLf+4X
BNN+bglLk3G4YodlOdNNg0/R1BWDIp3TJQADzdQenclBSw+9nQwpNJbfUxM+NUK8
t/WcjQhv2FtbiT+gARFMqacNpG3qZH5DV2veaj9LEfX0m+3GJ/9Q0/m/VVFR6MAC
ocEwzINoBvdPFhJ6SONvxal8Mpwjri7G1ZD3xsvnyP/XN29k1y//YzBx8HCqJh+w
3gV63wEMKazqfGB57kM6sOnahIxz3cE0G5l/wcJLqEE388F25+0VlP1APStdsS6y
awJDAUMHNiuUvVywkR4atAvmDOqgE8BvMDktiwIDAQABAoIBAQCEqU14diiYNgTm
HP7RoEbzZn+yw79/ynOmXix8ejzsHCUIcXerLJct4FrYWiNe0DN7Odb877X3F9Rf
UlpLlkkRWxrK7+wboK6qbhRL2YeeiO1/akYe9ND/NIYqLwghL0BRvmsMc8P3n6ZX
4C7LUkL1T5rqvAas1DLZMzyL098OgO4mVJaRZsVWYVtlVflXvDUYXbr3slVygiRs
kqzYgys4kupFTINbKlYDkLR3WZnKqdVTuRdw9S5L2/6fC6o5Ca3Efb3b5B3+hM6q
5nN8yA2rivtuRVGr/jfXlshV6lpKrqgDPJ1dfDPiaMjIXOQOM11Lc6bImljsTJ9V
paJzQgABAoGBAPnZYo8vMN9QIVktdp9LjrYJgL5XYE2EQJgX4WkqQziAhHEZwBAD
3Sp0YY/N5aXOoioAhoX6trPEJsE7XsGqInnI0YxEb58zba77X9aYAABhuuBsePTq
BgfYvB4lxL+EcJxqIpm7Bbxap4rZI0jq67ENUrBZ5Q5ya8mmX4Ar6u2LAoGBAMLE
Z5OhgTO2zTilFXQmaQ13MfURv1Slppu5otkW9UrKCLcmlgaVs4NT770JFaQWxXHM
Y7KSjKE9hP2BinpVXD4mYxrDYMyUkmjeop0zvifwhRzvL/n0eZoI8QDbJrmr/xSY
tJOtb8M1PXKprk3ApVhQVGAEu9QQAOHxz8znMsABAoGAZzoGe4wO0CTmMlcTTItG
IjXY6EtncX9zxKMRMYcRkNWgYq416Sf/h5vf9y8lc5Tk8R+YdOB5/dnL/UgPRUqK
xfBPi3l2+Lqh1YrsNNhGH+JA+Jo4e0/5P+KvDnGiUVJhyG4db5CStRhrYnWGG4lb
6aHMiSoK9iYWHJFNocIDZTMCgYAYpUvKBaTYy2f6pAEr+nROrOeYcE96wZ9skzgF
Kn+NoDUsH+jaGnVlx+hNTmn7opoHhWqUPTEociVzAsJoKocKokbmKxUDrkU8mfeP
1u1YFnpxp961TXdZw4njptempRoZHB21ljvPQtxstwYEdr01iKy0ncS61++Up8m4
zwTAAQKBgQC9mlc1lJ7PevzUFhxu6lFvqpP+3+UTiVDskERpGpNRbJwb4MXJiLcz
Xii3JpBQdn+5br8JhGrC/5irtCYbnuwGF1/O8fBfjgX73N8ufY9Y6tvnvxosGJXT
+/jb7prJL5G0ARtstRyziH3vGzuKf22ojITmMUeMDWng7VoeUcV8rQ==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       02:52:37:06:3c:22:68:ea:7d:44:d3:2b:fe:f3:02:90:ca:94:ba:d0
```

### dataflow.pratipad.local

Generate a TSL server certificate and a private key and save them to `certs/dataflow.pratipad.local.crt` and `certs/dataflow.pratipad.local.key` respectivelly.

```sh
⟩ vault write pratipad/roles/dataflow.pratipad.local key_bites=2048 max_ttl=8760h allow_any_name=true
Success! Data written to: pratipad/roles/dataflow.pratipad.local

⟩ vault write pratipad/issue/dataflow.pratipad.local common_name="dataflow.pratipad.local" ip_sans="172.16.0.136" ttl=8760h foramt=pem
Key                 Value
---                 -----
certificate         -----BEGIN CERTIFICATE-----
MIIDdDCCAlygAwIBAgIUU078BHPondmfTDaEcMaCG+CuyKwwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNTA0MDhaFw0yMjA3MjUxNTA0MzhaMCIxIDAeBgNVBAMTF2RhdGFmbG93
LnByYXRpcGFkLmxvY2FsMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA
46AtRiQiwdBthoSwGP5miIO96dklrOVPEWfWTYIYlWZ4ww7S4xRj/P8QQwMU2Olo
IOiRCnlHtJZVnG4hP5UIVo9KFcPWFpoUri71Ucxc1xk/J19UEPnioqiZoh9nI7uP
mKCVzNAgiUwvMRT9H2KP6lYFRKljPEQWKbi+09ukZ7WODqmnoBkRfZW1Xn/Tz9J/
7GpkTKeNXBSDNTQiZAlFnqXpfODixmP+nQo5HJSj6+MYLYqC/Ny7FqTmwTvemFRE
lHjWqoMdHJGMIHDhcW9PK95a/Qu7Qzv/iANRIOf4P/K+1dROGPGn/n2J36q2fzp4
S+U/CI2NNB6sMLQNzPzfwwIDAQABo4GcMIGZMA4GA1UdDwEB/wQEAwIDqDAdBgNV
HSUEFjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwHQYDVR0OBBYEFDH4Z81tXOWegv1+
RNgwu0INFLX6MB8GA1UdIwQYMBaAFLprKEu5m6bL8Q9jsV5lMZgHTZNXMCgGA1Ud
EQQhMB+CF2RhdGFmbG93LnByYXRpcGFkLmxvY2FshwSsEACIMA0GCSqGSIb3DQEB
CwUAA4IBAQCfQWjvotWJkpVBSx3oreVKCyIt9igj+7q0ultIsOQywlAsUothJI7o
oAe17ZX5pRBfCRrMOkiwxgY+UGShV1LNkk0DBk252clqyZb2TzBrpsR47mqr4DS5
E8+Kiyx97Ht3GeAHmV+myz91xa6bFtDcHYl1ACqVgyRuGgseU/twiQ5Ls0wZ3aG5
h5c3V96yzfhF69g/UhTgqCi8ybU/Pht+7vvXcjRy3iz5PYLrNi2KnZX7mwpovvKN
hQeE+Uhg2I+wLtkRIuA0V/5vddSQ9IhaUkt6FIGjpSucdhbC/yDXUQuxRkuUszPF
6nNNkC8E+6OHhU/O9oeGKkq+AUd5E9hL
-----END CERTIFICATE-----
expiration          1658761478
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDPzCCAiegAwIBAgIUXy1xy84rqXS808YnH0VAhEr0JpwwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNDQ0NDdaFw0zMTA3MjMxNDQ1MTZaMCcxJTAjBgNVBAMTHFJvb3QgQ0Eg
Zm9yIFByYXRpcGFkIEV4YW1wbGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCjMwjVwiDhl4E2NrlB0zMlYLydGJoPZlUFrlKa5vGyZeI2jPnvAgC0VRGp
x3Saa2dLVItLmRZ5/3z6JRhSBhAy4iBYNA1fdEULvvpS2S9tuKjUEAJLSh9yiqeg
PU72wIgjCPGHC/gvnE7Ch4i1V9GhELMvlQXAREeuspf3CdzeTzpVl+7/tsi6G02O
wRKG1diMRI+1rHFBUKUlcH265o5+K1YUrBveUoXBOmkPcCm7W3NFELTQWGyNitg8
pdEFXSIC2A5eG0Ahuhq3v+FEpp7BY1M8p09mXfIjDei0OOwcsyTJvRBsvcbPM7qv
RHucZ0uZOgkxek0duCwKtKPGRsezAgMBAAGjYzBhMA4GA1UdDwEB/wQEAwIBBjAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBS6ayhLuZumy/EPY7FeZTGYB02TVzAf
BgNVHSMEGDAWgBS6ayhLuZumy/EPY7FeZTGYB02TVzANBgkqhkiG9w0BAQsFAAOC
AQEAbEAPCvphPgd/KydvS+XU+eEOL2L3S4HDOnPFdXAZm6YjL8siCfiTwwZmi+qT
424yaiFa/5K5sffBLW/fRai0ydH1eEHC3JUhmv1OZjtMFOdmFSSqMKM+nzu5wtIQ
GVpRF3T7M98c9GYTPZyHfYfxIIzzQdlg2Iw054UFea224nnFobOHx75XEAZT+J7f
Irtc9Vz5CBJcKqNUOnbBboXpCvS9e1YoXte0D0flhitgvYa+UpUHlhK87zrTDqSM
XOMPNWKVBELfiXRQ3aOBX2g5HQJU0BYixlwH+FwAaIJ8x1XVSpBAn7/Fnl5BUp3q
kDo2P0RFKVxIAih2P5ZYUnpDZw==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA46AtRiQiwdBthoSwGP5miIO96dklrOVPEWfWTYIYlWZ4ww7S
4xRj/P8QQwMU2OloIOiRCnlHtJZVnG4hP5UIVo9KFcPWFpoUri71Ucxc1xk/J19U
EPnioqiZoh9nI7uPmKCVzNAgiUwvMRT9H2KP6lYFRKljPEQWKbi+09ukZ7WODqmn
oBkRfZW1Xn/Tz9J/7GpkTKeNXBSDNTQiZAlFnqXpfODixmP+nQo5HJSj6+MYLYqC
/Ny7FqTmwTvemFRElHjWqoMdHJGMIHDhcW9PK95a/Qu7Qzv/iANRIOf4P/K+1dRO
GPGn/n2J36q2fzp4S+U/CI2NNB6sMLQNzPzfwwIDAQABAoIBAQCLQ1RvtWwOaBaa
VlPm9r6EhxWKHPCP9zuIyp6yjQW9YhRAQrGDfNYv011/okL+8s5iU+dpIQpd0hxO
uJJe9C9FxiTtbANvkJVWfCRbB01FzIx26jvkpv2hxsU4Cz5u/jG8j4MW6778QUAe
J1k1+ndSR46xk3DDTmTA4ebed2L+j04WLbsLmZ+/AI1yGXAYN8r+Msg4AT6nEeB9
8ysyb3oi0NDa8J/Dj5l/BXIsZQBhGwO2QjaG3pmtfp2wfLe/nTssQtzayzG1WafQ
pH1lMAgFunVmDUQzPYuZKKX9NuHby4H0ou2sdqvC9DBZGywH/nCf7/OMTc1hWSUg
FBxX7/YBAoGBAPN/xP2mwSmZ8TYwgYsTyUG8WQuBAXVjXUSxrkrCHJOQ4Rk7doE1
kjJ37dTOZepj9fs1I/m7vRluMmLx5TxHfH0IfnUX8mM2gHVczTCJkiwfMk0zvu3X
11gRZxVz0Gwi2cpZvhRjghjwNnUbqumcn3v8tAXucnmO4lLMJ8+ryFN7AoGBAO9P
ycoX25z0ZOCEKwOQrA6lw6sQPIZzMjC5VcVpxgAwBdzZAP271K0za1mRQJ9XICjq
D593VcUhcqXVTJH+k5LGOAZC1UtdG1KPqoxFwrJWAN+b7R67RGFxyCpFuzSMxP+E
Ol+cr8L9upjefGV7+aDJ+oxHqH/VNPPk5cDPm25ZAoGAZ3qV8aGLFy3Xp0rH0p3O
+oObZ9skDonymf3Ubuq9EC0SrBFsFA77GT2EMdqgzxI5986mgju5afQ9r3TTEWHj
0pLogsRxep4vyzBr9sOP/fYn/00NR7BhUIjcwO4d1cadvXOT5sA/CnATBIOEh5DK
6fsDWj3yIhyJq9wc0xFSqb8CgYARZz4HgmCoM2W6piHyqmy4y/lE0XN1W59Ex9Wi
+6Q4k0V54BYgXa6Dwf+GjfejHtTp5MuqDyWfpmUOBksBOwBEZkHgwq98QZMhF+2R
MemMypBZsp814uyAIaQq3tNUaQBSjK0qEtz9UzJkt5lYUAHBXa7o0LVCRqEJM5Y5
xV9KUQKBgQCDNI+WQMcd+mmdvUhI3/iEUvffk0VPIrYYlZIXFM66HPe0K0mJdLvZ
m29n+Q9fPiKCmR64u4r4RpQAekyyB4ucjPd5hwTTNzB/6Rg8C/exFEhdnqZqtm2y
C1+LoKj64G/7DCyC341c9ufi1aOda/bxH5nNipqJvp7YQcF+x915IQ==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       53:4e:fc:04:73:e8:9d:d9:9f:4c:36:84:70:c6:82:1b:e0:ae:c8:ac
```

### server.pratipad.local

Generate a TSL server certificate and a private key and save them to `certs/server.pratipad.local.crt` and `certs/server.pratipad.local.key` respectivelly.

```sh
⟩ vault write pratipad/roles/server.pratipad.local key_bites=2048 max_ttl=8760h allow_any_name=true
Success! Data written to: pratipad/roles/server.pratipad.local

⟩ vault write pratipad/issue/server.pratipad.local common_name="server.pratipad.local" ip_sans="172.16.0.129" ttl=8760h foramt=pem
Key                 Value
---                 -----
certificate         -----BEGIN CERTIFICATE-----
MIIDcDCCAligAwIBAgIUWxPzSc5uSdqzqZQz42aJ0heH1CAwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNTA2MzlaFw0yMjA3MjUxNTA3MDlaMCAxHjAcBgNVBAMTFXNlcnZlci5w
cmF0aXBhZC5sb2NhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPMl
6ej5kkBgVRL9GV50xJkPDYcD+9sada/1gKM4fg3pjPObWy5otNjpq33G0D7GJHPl
ZOQFUGloduz0oQDsAmJ7OInuWWD6Vpbg9GY97NnqKsnnQ0t0Nhm0gIXrrWWI16r8
9vaKOTEZvBOOcs6glz6qS0QxqTFU+/oCG4Ra7dkVG6LpOU719LNxRS5zz/dzMsmX
cznAlTvjnwm+zS4RW0t+CqX6nehKhW/siaX8/WK5k/erQCAwBKbWQk8RLhjjHuHK
0tiWCwg0F+z3xNvXxQDfZhv/GT0MdJwQTrW3z7piBUFHjiJ73bMPZxPlcGFm7lRb
GacazrMeEHoCaeDQvskCAwEAAaOBmjCBlzAOBgNVHQ8BAf8EBAMCA6gwHQYDVR0l
BBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMB0GA1UdDgQWBBQ/o5U1zfrP+mP5sQMr
eljUai2UXzAfBgNVHSMEGDAWgBS6ayhLuZumy/EPY7FeZTGYB02TVzAmBgNVHREE
HzAdghVzZXJ2ZXIucHJhdGlwYWQubG9jYWyHBKwQAIEwDQYJKoZIhvcNAQELBQAD
ggEBABUd3q2u8HZCfSXuGB/saJttRQResjgGzzjyr5xBEK6PIHGha3Pa75dqnET1
NpTsAShIoQ1tRctupykWYn9+iAtgAYWxqeq+WPuhu7NYKqd4O4E/8L151y5MMaWy
9ZXeMoUKbZv3ZAayX6vB8WgHo5GPo/uFHBrGoRNqzf+Qe/6QB1wsU3ns/nPcnNno
ffpNQgDQxwnfnCPuPv4tMrfS/8wajFhJLqe4Vrz4u4IuiVDnIewtfYA02ZsByLyI
28UESEA7x7Ydhygnw03G/5ISOPd2q2J6rJtOHPaVHH1t31LnP2plFCTkuRRT+Mk3
VPPw3C6WUpFgBGifk7LyjSUxyGg=
-----END CERTIFICATE-----
expiration          1658761629
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDPzCCAiegAwIBAgIUXy1xy84rqXS808YnH0VAhEr0JpwwDQYJKoZIhvcNAQEL
BQAwJzElMCMGA1UEAxMcUm9vdCBDQSBmb3IgUHJhdGlwYWQgRXhhbXBsZTAeFw0y
MTA3MjUxNDQ0NDdaFw0zMTA3MjMxNDQ1MTZaMCcxJTAjBgNVBAMTHFJvb3QgQ0Eg
Zm9yIFByYXRpcGFkIEV4YW1wbGUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQCjMwjVwiDhl4E2NrlB0zMlYLydGJoPZlUFrlKa5vGyZeI2jPnvAgC0VRGp
x3Saa2dLVItLmRZ5/3z6JRhSBhAy4iBYNA1fdEULvvpS2S9tuKjUEAJLSh9yiqeg
PU72wIgjCPGHC/gvnE7Ch4i1V9GhELMvlQXAREeuspf3CdzeTzpVl+7/tsi6G02O
wRKG1diMRI+1rHFBUKUlcH265o5+K1YUrBveUoXBOmkPcCm7W3NFELTQWGyNitg8
pdEFXSIC2A5eG0Ahuhq3v+FEpp7BY1M8p09mXfIjDei0OOwcsyTJvRBsvcbPM7qv
RHucZ0uZOgkxek0duCwKtKPGRsezAgMBAAGjYzBhMA4GA1UdDwEB/wQEAwIBBjAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBS6ayhLuZumy/EPY7FeZTGYB02TVzAf
BgNVHSMEGDAWgBS6ayhLuZumy/EPY7FeZTGYB02TVzANBgkqhkiG9w0BAQsFAAOC
AQEAbEAPCvphPgd/KydvS+XU+eEOL2L3S4HDOnPFdXAZm6YjL8siCfiTwwZmi+qT
424yaiFa/5K5sffBLW/fRai0ydH1eEHC3JUhmv1OZjtMFOdmFSSqMKM+nzu5wtIQ
GVpRF3T7M98c9GYTPZyHfYfxIIzzQdlg2Iw054UFea224nnFobOHx75XEAZT+J7f
Irtc9Vz5CBJcKqNUOnbBboXpCvS9e1YoXte0D0flhitgvYa+UpUHlhK87zrTDqSM
XOMPNWKVBELfiXRQ3aOBX2g5HQJU0BYixlwH+FwAaIJ8x1XVSpBAn7/Fnl5BUp3q
kDo2P0RFKVxIAih2P5ZYUnpDZw==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA8yXp6PmSQGBVEv0ZXnTEmQ8NhwP72xp1r/WAozh+DemM85tb
Lmi02OmrfcbQPsYkc+Vk5AVQaWh27PShAOwCYns4ie5ZYPpWluD0Zj3s2eoqyedD
S3Q2GbSAheutZYjXqvz29oo5MRm8E45yzqCXPqpLRDGpMVT7+gIbhFrt2RUbouk5
TvX0s3FFLnPP93MyyZdzOcCVO+OfCb7NLhFbS34Kpfqd6EqFb+yJpfz9YrmT96tA
IDAEptZCTxEuGOMe4crS2JYLCDQX7PfE29fFAN9mG/8ZPQx0nBBOtbfPumIFQUeO
Invdsw9nE+VwYWbuVFsZpxrOsx4QegJp4NC+yQIDAQABAoIBABU3zsS02q1heqsQ
iSE0AV/171FD6LuDAJgdTV9w85cVNWagvQE8w+NV6NAuBEgmFFJEx1walpzML+yX
oGErNz4O0K1Arm4HCn1aHhm596xAggFie/3eo0X0+W42VchRe7iBIK/8+eat/zqZ
qK0dWHVweOb3inMynlZ/zpTrNmxF/rrVcH3OvRos4qnV9gJpQREPYZ35c/VUAXAc
hot9hVzSILOHDH7t5Z4GTtw564HOKltlMHyHU/k9XqH4npc6iHfPHDtmcoQJPzRM
0qKnl9PlCMyjxLBzlZxJaXYOdr4uuldL04NpWdGfn37Ed4AvZXF64onw/W1cT8nd
WwzVkAECgYEA+4gN3YOtuvkOiiTcvKE0oyJQaXYrQnTEvzV6o3MFqES0HOTaP1+i
Ekjjxg91IpUpdNj6D5kTwCZXqvI0D201xJ5lUF0rPkbEKLT8CqEftcs5gBdwSiqc
B9+fdK5ys/sl+Z9IcCCs79FWfVoF7nSdFHkQG7L8odQReKHVooxEoMkCgYEA93e7
j+6XAzXjNTVXKSokFbRq89vi8g37brUoVfSylSd9F4ilidIYvGNsRwiB3k6KQBXN
+j4nqOFNOSp+dyyLZBiISexp3RKO/T/zLjvfYnJxRVU9gSEy32zC9VuP6pzsypdK
as9SV+G/Tw1bn55SzEERyLF8m94126cQOT5oLgECgYA6cvIt4FR1lzxes5QrrRYr
NmUTLKd+yN2TRR0bcDYHVPe5oyBoC5QAxblQI/VnNNwuT+FD0KF7TC2hBqk8UHdn
GhuW4h+TWCRrBStwWOKifvf8oPWx9lbNqZRHK+ZxllHLwMy3aZBmJfIALPQl5ik+
QaeRmDUGcd4hdxHKtOeZqQKBgQDb6BxW1RCBG9viJppjzDzwxLjeJyJPMzmhsX48
lAw2GzdAOH/SL08n6boIjXjKkkSsmjPGEoGvwzaafDaRtJXRxzMlbd7NQ3apebCh
/zaNB2G82PikzVmlzcKZwlnrhLOvfC33KHDmA4e7ugUXnNu7An/JNl+jKx31KUpz
dA+kAQKBgQCZsyJK2wQGBLnVtmphcJGGZpNR3my2yzt08N5I/oz4XPo2Digu2/l+
nnbmfq2kINhxCyEhBv/ImyjBpw+m+bOHMVK5PDDQxR16W1pUPgJ71XvgdfcZFKS4
k24JTpYAncUZgQ3iiP9sK6c1xg6hTMZv/PuyosnESelqj2MC2a1rHg==
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       5b:13:f3:49:ce:6e:49:da:b3:a9:94:33:e3:66:89:d2:17:87:d4:20
```
