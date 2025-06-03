Head to [the download page](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html) (login required since Oracle is terrible) and download these files 

    instantclient-basic-macos.x64-12.2.0.1.0-2.zip
    instantclient-sdk-macos.x64-12.2.0.1.0-2.zip
    instantclient-sqlplus-macos.x64-12.2.0.1.0-2.zip

Make sure that you get the 64-bit versions (for 64-bit Pythons), else you'll see something like this at import

    ImportError: dlopen(/Users/nikhil/.pyenv/versions/manifold-api/lib/python2.7/site-packages/cx_Oracle.so, 2): Symbol not found: _OCIAttrGet  
      Referenced from: /Users/nikhil/.pyenv/versions/manifold-api/lib/python2.7/site-packages/cx_Oracle.so  
      Expected in: flat namespace

### Easy Way

Uses Homebrew. Copy the files downloaded above to `/Users/$USER/Library/Caches/Homebrew`. Then run [these commands](https://github.com/InstantClientTap/homebrew-instantclient)

```bash
brew install InstantClientTap/instantclient/instantclient-basic
brew install InstantClientTap/instantclient/instantclient-sdk
brew install InstantClientTap/instantclient/instantclient-sqlplus
```

Done! To set up `tnsnames.ora`,

```bash
mkdir -p /usr/local/opt/instantclient-basic/network/admin
touch /usr/local/opt/instantclient-basic/network/admin/tnsnames.ora
```

and add stuff to that file.

### Laborious Way

* Extract the downloaded files at the command line. You'll get a folder called `instantclient_12_2`

* Move it to `/usr/local`

```bash
sudo mkdir /usr/local/oracle  
sudo mv instantclient_12_2 /usr/local/oracle/instantclient  
sudo chown user:group /usr/local/oracle
```

* Set some environment variables

```bash
export ORACLE_HOME=/usr/local/oracle/instantclient  
export OCI_HOME=$ORACLE_HOME  
export OCI_LIB_DIR=$OCI_HOME  
export OCI_INCLUDE_DIR=$OCI_HOME/sdk/include  
export NLS_LANG=AMERICAN_AMERICA.UTF8  
export DYLD_LIBRARY_PATH=$OCI_LIB_DIR
```

* Last steps

```bash
cd $OCI_LIB_DIR  
ln -s libclntsh.dylib.11.1 libclntsh.dylib  
ln -s libocci.dylib.11.1 libocci.dylib
```
