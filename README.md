# ansible_ad

Sets up an AD DC to attempt integration testing.  Very much WIP.

Requires: vagrant, ansible, libvirt.

## Usage

```
vagrant up # Set up VM(s).

# Get keytab out of Windows VM.
vagrant winrm adserver -c 'cat svc.keytab.b64' > svc.keytab.b64
cat svc.keytab.b64 | tr -d '\r' | base64 -d > svc.keytab

# Address of Windows VM:
vagrant winrm-config
```

Configure a MIT krb5 installation to use realm AD.TEST with KDC at
adserver.ad.test, and add adserver.ad.test to /etc/hosts.
