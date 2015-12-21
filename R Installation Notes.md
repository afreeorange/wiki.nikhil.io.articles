Set up the repository
---------------------

I used this [repository for the
installation](http://rm.mirror.garr.it/mirrors/CRAN/bin/linux/redhat/el5/i386/).
In `In /etc/yum.repos.d/r-project.repo`:

` [r-project]`  
` name=R-Statistics`  
` baseurl=`[`http://rm.mirror.garr.it/mirrors/CRAN/bin/linux/redhat/el5/$basearch`](http://rm.mirror.garr.it/mirrors/CRAN/bin/linux/redhat/el5/$basearch)  
` enabled=0`  
` includepkgs=R-core R-devel libRmath libRmath-devel`  
` gpgcheck=0`

Resolve the Perl dependency
---------------------------

R needs Perl(File::Copy::Recursive) to install properly. [Get it at the
DAG repo](http://dag.wieers.com/rpm/packages/perl-File-Copy-Recursive/).
Then:

` rpm -ivH path.to.perl.rpm`

Install R and related libraries
-------------------------------

` yum install R-core R-devel libRmath libRmath-devel --enablerepo=r-project`

The binary should be in `/usr/bin/R`. Fin!




