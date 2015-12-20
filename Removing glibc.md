Tried to upgrade in an altroot. Compilation failed, which meant that
commands like "ls" and "cp" would segfault. Found these via mtimes.
D'oh.

`   lib/`  
`   libBrokenLocale-2.19.so libBrokenLocale.so libBrokenLocale.a gconv libm-2.19.so \`  
`   libm.so libieee.a libm.a libdl-2.19.so libdl.so libdl.a libmemusage.so libmcheck.a \`  
`   libg.a libc_nonshared.a libSegFault.so libpcprofile.so libcidn-2.19.so libcidn.so \`  
`   libthread_db-1.0.so libthread_db.so libresolv-2.19.so libnss_dns-2.19.so libnss_dns.so \`  
`   libanl-2.19.so libanl.so libresolv.a libanl.a libnss_files-2.19.so libnss_files.so \`  
`   libnss_db-2.19.so libnss_db.so libnss_hesiod-2.19.so libnss_hesiod.so librpcsvc.a \`  
`   libnss_nis-2.19.so libnss_nis.so libnss_nisplus-2.19.so libnss_nisplus.so libnss_compat-2.19.so \`  
`   libnss_compat.so libutil-2.19.so libutil.so libutil.a ld-2.19.so audit libBrokenLocale.so.1 \`  
`   libm.so.6 libdl.so.2 libcidn.so.1 libthread_db.so.1 libnss_dns.so.2 libanl.so.1 \`  
`   libnss_files.so.2 libnss_db.so.2 libnss_hesiod.so.2 libnss_nis.so.2 libnss_nisplus.so.2 \`  
`   libnss_compat.so.2 libutil.so.1 ld-linux-x86-64.so.2 \`  
`   crt1.o gcrt1.o Mcrt1.o Scrt1.o crti.o crtn.o`

`   sbin/`  
`   zic zdump nscd sln ldconfig iconvconfig`

`   bin/ `  
`   iconv localedef locale gencat mtrace tzselect getconf pcprofiledump xtrace catchsegv \`  
`   getent makedb rpcgen sprof pldd ldd sotruss`

`   include/`  
`   limits.h values.h features.h gnu-versions.h stdc-predef.h iconv.h gconv.h locale.h langinfo.h \`  
`   xlocale.h assert.h ctype.h libintl.h nl_types.h math.h fpu_control.h complex.h fenv.h tgmath.h \`  
`   ieee754.h setjmp.h signal.h stdlib.h monetary.h inttypes.h stdint.h errno.h ucontext.h alloca.h \`  
`   fmtmsg.h stdio_ext.h printf.h stdio.h libio.h _G_config.h dlfcn.h malloc.h obstack.h mcheck.h \`  
`   string.h strings.h memory.h endian.h argz.h envz.h byteswap.h wchar.h uchar.h time.h dirent.h \`  
`   grp.h pwd.h unistd.h glob.h regex.h wordexp.h fnmatch.h getopt.h tar.h sched.h re_comp.h wait.h \`  
`   cpio.h spawn.h fcntl.h poll.h utime.h ftw.h fts.h termios.h termio.h ulimit.h ar.h a.out.h libgen.h \`  
`   stab.h sgtty.h ttyent.h paths.h fstab.h mntent.h search.h err.h error.h sysexits.h syscall.h ustat.h \`  
`   regexp.h syslog.h scsi net wctype.h shadow.h gshadow.h argp.h crypt.h pthread.h semaphore.h aio.h \`  
`   mqueue.h execinfo.h thread_db.h protocols aliases.h ifaddrs.h netipx netash netax25 netatalk netrom \`  
`   netpacket netrose neteconet netiucv netinet resolv.h netdb.h arpa nss.h rpc rpcsvc nfs stropts.h sys \`  
`   utmp.h lastlog.h pty.h utmpx.h elf.h link.h bits gnu `

`   var/`  
`   db`

`   libexec/`  
`   getconf`

[Category: Nikhil's Notes](Category:_Nikhil's_Notes "wikilink")
