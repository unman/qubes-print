# Qubes printing
There are many options for printing in Qubes.
You can install print drivers in a template,and enable `cups` in every qube where you want to print.
You can install print drivers in a template, set up a disposableVM, then open files in a disposableVM, and print from there.
I dont like these approaches: I find them cumbersome when changing printers, or setting up a new printer, when on the move.
The best solution mimics normal practice in a non-Qubes system by using a print server qube running CUPS.  
Any qube that wishes to print accesses the print server over qubes-rpc  
This has the advantage that you only set up the printer in one qube, as if you were using an "ordinary" linux system.
Also, I prefer to disable cups by default in every qube, (along with almost every other service).

## Setting up the print server
This is straightforward, and follows the usual `cups` setup, whether the printer is accessed across the network or locally.
**But**, the `/etc/cups/printers.conf` file is bind-mounted so that it persists across reboots.

I usually disable cups in every qube, so it may be necessary to start cups from `/rw/config/rc.local`:
```
sudo touch /var/run/qubes-service/cups
sudo unmask cups
sudo start cups
```

What about print drivers?
If print drivers are needed, they are downloaded to the print qube, and loaded as needed.
E.g. in a Debian qube:  
```
#!/bin/bash
sudo apt install Downloads/printer-driver-postscript-hp
```

Together, these will start CUPS on the printer qube, configured to use the target printer.  

The printer has a qubes-rpc service defined, again bind-mounted in place in `/etc/qubes-rpc/qubes.Print`:  
```
#!/bin/sh
exec socat STDIO TCP:localhost:631
```

## Using the server qube

The client runs a socat listener,redirecting calls to the qrexec service on the server.  
Add this line to  `/rw/config/rc.local`:  
`socat TCP4-LISTEN:9100,reuseaddr,fork EXEC:"qrexec-client-vm print qubes.Print"`

All that is then needed is to create ~/.cups/client.conf, with this content:  
`ServerName 127.0.0.1:9100`

This is printing without cups, network discovery, or any unnecessary service or complicated setup on the clients.
All the work is done on the print server.

## Access to the print server qube
Access to the print server is controlled by the usual policy file in dom0.
