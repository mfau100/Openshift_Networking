
### Using tcpdump to see traffic from an app using http, no https.
### -i = interface, -A aschii = output, -n = no dns info.

sudo tcpdump -i eth0 -A -n port 80 | grep "angular"


### Use the curl command with the -I and -v options to retrieve the connection headers.

curl -I -v https://todo-https.apps.ocp4.example.com

*  subjectAltName: host "todo-https.apps.ocp4.example.com" matched cert's "*.apps.ocp4.example.com"
*  issuer: O=EXAMPLE.COM; CN=Red Hat Training Certificate Authority
*  SSL certificate verify ok.