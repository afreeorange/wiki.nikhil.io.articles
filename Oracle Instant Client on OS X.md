* Head to [the download page](http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html). 
    Login required.
* Download these files:

        instantclient-basic-macos.x64-11.2.0.3.0.zip  
        instantclient-sdk-macos.x64-11.2.0.3.0.zip  
        instantclient-sqlplus-macos.x64-11.2.0.3.0.zip  
        instantclient-jdbc-macos.x64-11.2.0.3.0.zip

* Make sure that you get the 64-bit versions (for 64-bit Pythons),
    else you'll see something like this at import

        ImportError: dlopen(/Users/nikhil/.pyenv/versions/manifold-api/lib/python2.7/site-packages/cx_Oracle.so, 2): Symbol not found: _OCIAttrGet  
          Referenced from: /Users/nikhil/.pyenv/versions/manifold-api/lib/python2.7/site-packages/cx_Oracle.so  
          Expected in: flat namespace

* Extract them at the command line. You'll get a folder 
    called "`instantclient_11_2`"

* Move it to `/usr/local`

        sudo mkdir /usr/local/oracle  
        sudo mv instantclient_11_2 /usr/local/oracle/instantclient  
        sudo chown user:group /usr/local/oracle

* Set some environment variables

        export ORACLE_HOME=/usr/local/oracle/instantclient  
        export OCI_HOME=$ORACLE_HOME  
        export OCI_LIB_DIR=$OCI_HOME  
        export OCI_INCLUDE_DIR=$OCI_HOME/sdk/include  
        export NLS_LANG=AMERICAN_AMERICA.UTF8  
        export DYLD_LIBRARY_PATH=$OCI_LIB_DIR

* Last steps

        cd $OCI_LIB_DIR  
        ln -s libclntsh.dylib.11.1 libclntsh.dylib  
        ln -s libocci.dylib.11.1 libocci.dylib

Other Notes
-----------

* Had to copy `tnsnames.ora` to `/etc` for things to work properly.
