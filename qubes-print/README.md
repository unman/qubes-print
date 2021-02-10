# Qubes printing
There are many options for printing in Qubes.
You can install print drivers in a template,and enable `cups` in every qube where you want to print.
You can install print drivers in a template, set up a disposableVM, then open files in a disposableVM, and print from there.
I dont like these approaches: I find them cumbersome when changing printers, or setting up a new print.  
The best solution mimics normal practice in an ordinary system - it uses a print server qube running CUPS.  
Any qube that wishes to print uses the print server.

## Setting up a print server
This is straightforward, and follows the usual `cups` setup, whether the printer is accessed across the network or locally.
**But**, the `/etc/cups/printers.conf` file is bind-mounted so that it persists across reboots.

What about print drivers?
If print drivers are needed, they are downloaded to the print qube, and loaded as needed.
E.g. in a Debian qube:  
```
#!/bin/bash
sudo apt install Downloads/printer-driver-postscript-hp
sudo touch /var/run/qubes-service/cups
sudo unmask cups
sudo start cups
```

This will start CUPS on the printer qube, configured to use the target printer.  
The printer has a qubes-rpc service defined, again bind-mounted in place in `/etc/qubes-rpc`  
See `qubes.Print`  
```
#!/bin/sh
exec socat STDIO TCP:localhost:631
```

## Using the server qube

The client runs a socat listener,redirecting calls to the qrexec service on the server.
`socat TCP4-LISTEN:9100,reuseaddr,fork EXEC:"qrexec-client-vm print qubes.Print"`

All that is then needed is to create ~/.cups/client.conf:
`ServerName 127.0.0.1:9100`	

This is printing without cups, network discovery, or any unnecessary service or complicated setup on the clients.
All the work is done on the print server.
