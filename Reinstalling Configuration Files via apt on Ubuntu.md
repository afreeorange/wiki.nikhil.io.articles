I accidentally hosed `/etc/nginx` thinking it would be reinstalled with `apt install nginx`. When no config files showed up, I had to do this to get things going.

```bash
# Purge, then reinstall
apt-get purge nginx nginx-common nginx-full
apt-get install nginx

# Old option
apt-get -o DPkg::options::=--force-confmiss --reinstall install nginx nginx-common nginx-full
```
