---
layout: post
title: "Fix cloud-init chef install on ubuntu 12.04 LTS"
date: 2013-06-20 15:55
comments: true
published: false
author: bkw
categories:
- chef
- cloud
- cloud-init
- ubuntu
---
Setting up a chef-client via cloud-init is a very converient way of
bootstrapping a vm, i.e. on Amazon AWS or OpenStack. The process is documented
in [several](https://cloudinit.readthedocs.org/en/latest/topics/examples.html#install-and-run-chef-recipes)
[places](http://docs.openstack.org/trunk/openstack-compute/admin/content/user-data.html#d6e7751) and usually uses Ubuntu 12.04 LTS (Precise Pangolin) as an
example.

Funny thing is - it doesn't bloody work. <!-- more -->
And [almost nobody seems to
notice](https://bugs.launchpad.net/ubuntu/+source/cloud-init/+bug/961142).
Seems that while running under cloud-init, several environment variables are
set that trigger some weird behaviour in the scripts around package `man-db`.
Since I don't need man-db on my production boxes, the workaround for me is to
simply remove that package.

While cloud-init supports installing packages, there doesn't seem to be support
for removing them. Fortunately, we can supply shell commands via the yaml key
`runcmd`.
``` yaml
runcmd:
- apt-get -y purge man-db
```

So here is an example of bootstrapping a cloud instance with cloud-init and
chef that actually works with Ubuntu 12.04 LTS:

``` yaml cloud-config.yml
#cloud-config
---
manage_etc_hosts: true
hostname: db1
fqdn: db1.mydomain
package_update: true
package_upgrade: true
runcmd:
- apt-get -y purge man-db
- echo 10.23.45.6 chef.mydomain >> /etc/hosts
apt_sources:
- source: deb http://apt.opscode.com/ $RELEASE-0.10 main
  key: |
     -----BEGIN PGP PUBLIC KEY BLOCK-----
     Version: GnuPG v1.4.9 (GNU/Linux)

     mQGiBEppC7QRBADfsOkZU6KZK+YmKw4wev5mjKJEkVGlus+NxW8wItX5sGa6kdUu
     twAyj7Yr92rF+ICFEP3gGU6+lGo0Nve7KxkN/1W7/m3G4zuk+ccIKmjp8KS3qn99
     dxy64vcji9jIllVa+XXOGIp0G8GEaj7mbkixL/bMeGfdMlv8Gf2XPpp9vwCgn/GC
     JKacfnw7MpLKUHOYSlb//JsEAJqao3ViNfav83jJKEkD8cf59Y8xKia5OpZqTK5W
     ShVnNWS3U5IVQk10ZDH97Qn/YrK387H4CyhLE9mxPXs/ul18ioiaars/q2MEKU2I
     XKfV21eMLO9LYd6Ny/Kqj8o5WQK2J6+NAhSwvthZcIEphcFignIuobP+B5wNFQpe
     DbKfA/0WvN2OwFeWRcmmd3Hz7nHTpcnSF+4QX6yHRF/5BgxkG6IqBIACQbzPn6Hm
     sMtm/SVf11izmDqSsQptCrOZILfLX/mE+YOl+CwWSHhl+YsFts1WOuh1EhQD26aO
     Z84HuHV5HFRWjDLw9LriltBVQcXbpfSrRP5bdr7Wh8vhqJTPjrQnT3BzY29kZSBQ
     YWNrYWdlcyA8cGFja2FnZXNAb3BzY29kZS5jb20+iGAEExECACAFAkppC7QCGwMG
     CwkIBwMCBBUCCAMEFgIDAQIeAQIXgAAKCRApQKupg++Caj8sAKCOXmdG36gWji/K
     +o+XtBfvdMnFYQCfTCEWxRy2BnzLoBBFCjDSK6sJqCu5Ag0ESmkLtBAIAIO2SwlR
     lU5i6gTOp42RHWW7/pmW78CwUqJnYqnXROrt3h9F9xrsGkH0Fh1FRtsnncgzIhvh
     DLQnRHnkXm0ws0jV0PF74ttoUT6BLAUsFi2SPP1zYNJ9H9fhhK/pjijtAcQwdgxu
     wwNJ5xCEscBZCjhSRXm0d30bK1o49Cow8ZIbHtnXVP41c9QWOzX/LaGZsKQZnaMx
     EzDk8dyyctR2f03vRSVyTFGgdpUcpbr9eTFVgikCa6ODEBv+0BnCH6yGTXwBid9g
     w0o1e/2DviKUWCC+AlAUOubLmOIGFBuI4UR+rux9affbHcLIOTiKQXv79lW3P7W8
     AAfniSQKfPWXrrcAAwUH/2XBqD4Uxhbs25HDUUiM/m6Gnlj6EsStg8n0nMggLhuN
     QmPfoNByMPUqvA7sULyfr6xCYzbzRNxABHSpf85FzGQ29RF4xsA4vOOU8RDIYQ9X
     Q8NqqR6pydprRFqWe47hsAN7BoYuhWqTtOLSBmnAnzTR5pURoqcquWYiiEavZixJ
     3ZRAq/HMGioJEtMFrvsZjGXuzef7f0ytfR1zYeLVWnL9Bd32CueBlI7dhYwkFe+V
     Ep5jWOCj02C1wHcwt+uIRDJV6TdtbIiBYAdOMPk15+VBdweBXwMuYXr76+A7VeDL
     zIhi7tKFo6WiwjKZq0dzctsJJjtIfr4K4vbiD9Ojg1iISQQYEQIACQUCSmkLtAIb
     DAAKCRApQKupg++CauISAJ9CxYPOKhOxalBnVTLeNUkAHGg2gACeIsbobtaD4ZHG
     0GLl8EkfA8uhluM=
     =zKAm
     -----END PGP PUBLIC KEY BLOCK-----
chef:
  install_type: packages
  server_url: https://chef.mydomain
  node_name: db1.mydomain
  environment: development
  validation_name: chef-validator
  validation_key: |
     -----BEGIN RSA PRIVATE KEY-----
     YOUR-ORGS-VALIDATION-KEY-HERE
     -----END RSA PRIVATE KEY-----
  run-list:
  - role[db]
```
