SSLyze
======

Fast and full-featured SSL scanner for Python 2.7.


Description
-----------

SSLyze is a Python tool that can analyze the SSL configuration of a server by connecting to it. It is designed to be 
fast and comprehensive, and should help organizations and testers identify mis-configurations affecting their SSL
servers.

Key features include:
* Multi-processed and multi-threaded scanning: it's very fast.
* Support for all SSL protocols, from SSL 2.0 to TLS 1.2.
* **NEW:** SSLyze can also be used as a library, in order to run scans and process the results directly from Python.
* Performance testing: session resumption and TLS tickets support.
* Security testing: weak cipher suites, insecure renegotiation, CRIME, Heartbleed and more.
* Server certificate validation and revocation checking through OCSP stapling.
* Support for StartTLS handshakes on SMTP, XMPP, LDAP, POP, IMAP, RDP, PostGres and FTP.
* Support for client certificates when scanning servers that perform mutual authentication.
* Scan results can be written to an XML or JSON file for further processing.
* And much more !


Getting Started
---------------

SSLyze can be installed directly via pip:
    
    pip install sslyze

It is also easy to directly clone the repository and the fetch the requirements:

    git clone https://github.com/nabla-c0d3/sslyze.git
    cd sslyze
    pip install -r requirements.txt --target ./lib
    
Then, the command line tool can be used to scan servers:

    python sslyze_cli.py --regular www.yahoo.com:443 www.google.com

On Linux, the `python-devel` package needs to be installed first so that the nassl C extension can be compiled:

    sudo apt-get install python-devel

SSLyze has been tested on the following platforms: Windows 7 (32 and 64 bits), Debian 7 (32 and 64 bits), OS X El 
Capitan.

Usage as a library
------------------

Starting with version 0.13.0, SSLyze can be used as a Python module in order to run scans and process the results 
directly in Python:

```python
# Script to get the list of SSLv3 cipher suites supported by smtp.gmail.com
hostname = 'smtp.gmail.com'
try:
    # First we must ensure that the server is reachable
    server_info = ServerConnectivityInfo(hostname=hostname, port=587,
                                         tls_wrapped_protocol=TlsWrappedProtocolEnum.STARTTLS_SMTP)
    server_info.test_connectivity_to_server()
except ServerConnectivityError as e:
    raise RuntimeError('Error when connecting to {}: {}'.format(hostname, e.error_msg))

# Get the list of available plugins
sslyze_plugins = PluginsFinder()

# Create a process pool to run scanning commands concurrently
plugins_process_pool = PluginsProcessPool(sslyze_plugins)

# Queue a scan command to get the server's certificate
plugins_process_pool.queue_plugin_task(server_info, 'sslv3')

# Process the result and print the certificate CN
for plugin_result in plugins_process_pool.get_results():
    if plugin_result.plugin_command == 'sslv3':
        # Do something with the result
        print 'SSLV3 cipher suites'
        for cipher in plugin_result.accepted_cipher_list:
            print '    {}'.format(cipher.name)
```

The scan commands are same as the ones described in the `sslyze_cly.py --help` text. 

They will all be run concurrently using Python's multiprocessing module. Each command will return a `PluginResult` 
object with attributes that contain the result of the scan command run on the server (such as list of supported cipher 
suites for the `--tlsv1` command). These attributes are specific to each plugin and command but are all documented 
(within each plugin's module).

See _api\_sample.py_ for more examples of SSLyze's Python API.


Windows executable
------------------

A pre-compiled Windows executable is available in the [Releases](https://github.com/nabla-c0d3/sslyze/releases) tab. 
The package can also be generated by running the following command:

    python.exe setup_py2exe.py py2exe
    

How does it work ?
------------------

SSLyze is all Python code but it uses an 
[OpenSSL wrapper written in C called nassl](https://github.com/nabla-c0d3/nassl), which was specifically developed for
allowing SSLyze to access the low-level OpenSSL APIs needed to perform deep SSL testing.


Where do the trust stores come from?
------------------------------------

The Mozilla, Microsoft, Apple and Java trust stores are downloaded using the following tool: 
https://github.com/nabla-c0d3/catt/blob/master/sslyze.md .


License
-------

GPLv2 - See LICENSE.txt.
