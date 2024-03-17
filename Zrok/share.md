---
layout: home
parent: Zrok
title: How to share
---

## Share resource

Zrok lets us share resource in public and private

<hr/>

### To share a port publically run 
`.\zrok.exe share public localhost:port `

By default backend mode is proxy

<hr/>

### To share a port in private run 
`.\zrok.exe share private localhost:port `

<hr/>

### Access public port 
Public port can be accessed directly from the web browser 

<hr/>

### Access private port 
To access private port run this on cmd/powershell on client machine

`.\zrok.exe access private <access code>`

or

`.\zrok.exe access private <access code> --bind localhost:port` 

<hr/>

### Open tcp port
Tcp port forword is only available on private share.
To do so run these on server in order

1. `.\zrok.exe reserve private localhost:port --backend-mode tcpTunnel --unique-name <name>`
2. `.\zrok.exe share reserved <name>`

This will forword port specified in the command

To access this from client run
`.\zrok.exe access private <name>`

or

`.\zrok.exe access private <name> --bind localhost:port`

<hr/>

### Reserve public share
To get a consistent url consider reserved share when possible both for private and public share but remember to release it when not in use with `.\zrok.exe release <name>`

### Create a public reserve share
Run this in order
1. `.\zrok.exe reserve public localhost:port --unique-name <name>`
2. `.\zrok.exe share reserved <name>` 