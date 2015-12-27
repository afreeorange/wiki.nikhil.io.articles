### View contents of RPM

    rpm -qlp /path/to/file.rpm

### Extract without installing

    rpm2cpio /path/to/file.rpm | cpio -idmv

Files are then extracted to the directory this command is issued in.
