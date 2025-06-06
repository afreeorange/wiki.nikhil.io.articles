### Listing

```bash
python -c 'import yum, pprint; yb = yum.YumBase(); pprint.pprint(yb.conf.yumvar, width=1)'
```

### Creating

E.g. `$osname`

```bash
echo "Scientific Linux" > /etc/yum/vars/osname
```
