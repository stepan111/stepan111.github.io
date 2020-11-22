---
layout: post
title:  "ansible shell execute python script"
categories: Ansible
---


Here are example how to execute python code from ansble by using `shell` module:

```
  - name: Retrieve certificate for \{\{ site_addr \}\}
    shell: |
      import socket
      from OpenSSL import SSL
      from OpenSSL import crypto
      context = SSL.Context(SSL.SSLv23_METHOD)
      s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
      connection = SSL.Connection(context,s)
      connection.connect(('{{ site_addr | urlsplit('hostname') }}',443))
      connection.set_tlsext_host_name(str.encode("{{ site_addr | urlsplit('hostname') }}"))
      connection.setblocking(1)
      try:
        connection.do_handshake()
      except OpenSSL.SSL.WantReadError:
        print("Timeout")
        quit()
      for cert in connection.get_peer_cert_chain():
        subject_str = "".join("/{0:s}={1:s}".format(name.decode(),
          value.decode())
          for name, value in cert.get_subject().get_components() )
        if 'Site CA' in subject_str:
          print (crypto.dump_certificate(crypto.FILETYPE_PEM,
             cert).decode('ascii') )
    args:
      executable: "{{ ansible_python_interpreter }}"


```
