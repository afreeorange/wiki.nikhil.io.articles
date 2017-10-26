This document describes the bare minimum you'll need to use `cx_oracle` on a Lambda. Works for both Pythons 2 and 3. This is a bare Lambda and has nothing to do with Zappa or Chalice or Serverless or anything else all the cool kids seem to be using these days.

### Shared Objects

Go [here](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html) and download `instantclient-basic-linux.x64-12.2.0.1.0.zip`. That's all you'll need. Extract it. Then copy these files to a separate folder called "lib".

* `libclntshcore.so.12.1`
* `libclntsh.so.12.1`
* `libipc1.so`
* `libmql1.so`
* `libnnz12.so`
* `libociei.so`
* `libons.so`

Then, you'll have to install `libaio` on your system and copy `libaio.so.1` into "lib". 

### But I'm Lazy

OK. I've [cached all that here](http://public.nikhil.io/oracle_client_minimum.tgz).
 
### App Structure

```
/my_lambda
├── test.py
└── lib
    ├── libaio.so.1
    ├── libclntshcore.so.12.1
    ├── libclntsh.so.12.1
    ├── libipc1.so
    ├── libmql1.so
    ├── libnnz12.so
    ├── libociei.so
    └── libons.so
```

Here's `test.py`

```python
from __future__ import print_function
import cx_Oracle

# Yeah, you need this. 
with open('/tmp/HOSTALIASES', 'w') as f: f.write(f'{os.uname()[1]} localhost\n')

# Oracle away!
def handler(event, context):
    return str(
        cx_Oracle.connect(
            'username',
            'password',
            cx_Oracle.makedsn(
                'rds.amazonaws.com', 1521, 'SOME_SID',
            )
        ).cursor().execute('SELECT 42 FROM DUAL').fetchone()
    )
```

Now `pip install cx_oracle -t my_lambda/` to install `cx_oracle` into that folder. Zip it up and make a Lambda with `test.handler` as the... handler.

### Environment Variables

You'll need to set these before you execute your Lambda

* `ORACLE_HOME` to `/var/task`
* `HOSTALIASES` to `/tmp/HOSTALIASES`

You're all set!

### Notes

* Lambdas are not allowed to modify `/etc/hosts`
* I used [this AMI](https://console.aws.amazon.com/ec2/v2/home#Images:visibility=public-images;search=amzn-ami-hvm-2017.03.1.20170812-x86_64-gp2) to do all testing. See [this page](https://docs.aws.amazon.com/lambda/latest/dg/current-supported-versions.html) for more information.
* You don't need to set `LD_LIBRARY_PATH` since it already includes `/var/task/lib` (and your Lambda is executed from `/var/task`)
* Uncompressed, you're looking at ~200MB for all those files (~65MB compressed). 
