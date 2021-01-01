# webparametrics

This repository documents a Virtual Private Server holding several test projects and builds. It will consist of a static web site, served via nginx, and hold a number of LXD containers. Each container will be a separate project. If these projects serve html, the VPS static web site will link to directories serving the project html.

The nginx configuration will act as a reverse proxy, forwarding calls to those directories which link to the appropriate lxd containers.

Therefore creating a new project is simple: create an lxd container and populate with the project. Alter the VPS host static html to link to the container. Add a proxy location to the nginx configuration.

Removing a project is just reversing the above steps.

The VPS therefore needs to be configured with LXD, nginx, and certificates to serve https.

It needs ssh capability to allow remote access - via keys rather than passwords.

An MQTT broker service is required on the VPS, listenning on the briged LXD network, as projects are likely to communicate via an mqtt broker.

Include git capability, so projects on github can be easily cloned on the VPS.

The resultant service can be viewed at

https://webparametrics.co.uk/

Documentation set in this repository can be viewed at

https://bernie-skipole.github.io/webparametrics/

