The following was written for VMWare 3.0 on OS X, but applies to Windows
and Linux as well.

Virtual Machine Structure
-------------------------

Although a Virtual Machine *appears* to be a file, it's actually a
folder composed of many files. Expand the table below if you're
interested in learning about them.

*   `<vmname>.log or vmware.log`.
:   This is the file that keeps a log of key VMware Workstation activity. This 
    file can be useful in troubleshooting if you encounter problems. This file 
    is stored in the directory that holds the configuration (`.vmx`) file of 
    the virtual machine.
*   `<vmname>.nvram or nvram`
:   This is the file that stores the state of the virtual machine's BIOS.
*   `<vmname>.vmdk`
:   This is a virtual disk file, which stores the contents of the virtual 
    machine's hard disk drive. A virtual disk is made up of one or more `.vmdk`
    files. If you have specified that the virtual disk should be split into 2GB 
    chunks, the number of `.vmdk` files 
    depends on the size of the virtual disk. As data is added to a virtual 
    disk, the `.vmdk` files grow in size, to a maximum of 2GB each. (If you 
    specify that all space should be allocated when you create the disk, these 
    files start at the maximum size and do not grow.) Almost all of a `.vmdk` 
    file's content is the virtual machine's data, with a small portion allotted 
    to virtual machine overhead. If the virtual machine is connected directly 
    to a physical disk, rather than to a virtual disk, the `.vmdk` file stores 
    information about the partitions the virtual machine is allowed to access. 
    Earlier VMware products used the extension `.dsk` for virtual disk files.
*   `<diskname>-<###>.vmdk`
:   This is a redo-log file, created automatically when a virtual machine has 
    one or more snapshots. This file stores changes made to a virtual disk 
    while the virtual machine is running. There may be more than one such file. 
    The `###` indicates a unique suffix added automatically by VMware
    Workstation to avoid duplicate file names.
*   `<vmname>.vmsd`
:   This is a centralized file for storing information and metadata 
    about snapshots.
*   `<vmname>-Snapshot.vmsn`
:   This is the snapshot state file, which stores the running state of a 
    virtual machine at the time you take that snapshot
*   `<vmname>-Snapshot<###>.vmsn`
:   This is the file which stores the state of a snapshot
*   `<vmname>.vmss`
:   This is the suspended state file, which stores the state of a suspended 
    virtual machine .Some earlier VMware products used the extension `.std`
    for suspended state files
*   `<vmname>.vmtm`
:   This is the configuration file containing team data.
*   `<vmname>.vmx`
:   This is the primary configuration file, which stores settings chosen in the 
    New Virtual Machine Wizard or virtual machine settings editor. If you 
    created the virtual machine under an earlier version of VMware Workstation 
    on a Linux host, this file may have a `.cfg` extension
*   `<vmname>.vmxf`
:   This is a supplemental configuration file for virtual machines that are 
    in a team. Note that the<tt> .vmxf </tt>file remains if a virtual machine 
    is removed from the team.


For instance, I have a Virtual Machine I called "**Windows XP
Professional**". It's stored as a directory called
`Windows XP Professional.vmwarevm`. The VMDK images are:

    Windows XP Professional.vmdk  
    Windows XP Professional-s001.vmdk  
    Windows XP Professional-s002.vmdk  
    Windows XP Professional-s003.vmdk  
    Windows XP Professional-s004.vmdk  
    Windows XP Professional-s005.vmdk  
    Windows XP Professional-s006.vmdk  
    Windows XP Professional-s007.vmdk  
    Windows XP Professional-s008.vmdk  
    Windows XP Professional-s009.vmdk  
    Windows XP Professional-s010.vmdk  
    Windows XP Professional-s011.vmdk

**Note:** I'm going to call the topmost the 'primary' VMDK for the
Virtual Machine for the following sections.

So many VMDK's!
---------------

Yep. They're all in 2GB chunks, presumably to get over the [volume size
limitations](http://en.wikipedia.org/wiki/File_Allocation_Table) of
FAT16, in case you choose to format your Virtual Machine as such.

### I want one VMDK to rule them all

Fine. Fire up a terminal (or a "PowerShell" if you dream of Ballmer
every night), and navigate to your virtual machine's folder. There are
three things you'll be doing here:

1.  Move all your fragmented VMDK's to a temporary folder
2.  Run a program that will 'stitch' them to a big file
3.  Move this resultant file to the virtual machine's folder
4.  Give this big file the same name as your virtual machine folder

#### Example

    cd /Users/tech/Documents/Virtual Machines.localized/Windows XP Professional.vmwarevm  
    mkdir ~/temp  
    mv *.vmdk ~/temp  
    cd ~/temp  
    Library/Application\ Support/VMware\ Fusion/vmware-vdiskmanager -r "Windows XP Professional.vmdk" -t 0 "XP Temp Image.vmdk"  
    mv "XP Temp Image.vmdk" /Users/tech/Documents/Virtual Machines.localized/Windows XP Professional.vmwarevm/"Windows XP Professional.vmdk"

On Windows, this would be located at `C:\Program Files\VMware\VMware Workstation`

Sources
-------

-   [Running the VMware Virtual Disk Manager Utility (reference to CLI
    options](http://www.vmware.com/support/ws45/doc/disks_vdiskmanager_run_ws.html)
-   [Virtual Disk Manager - PDF
    Manual](http://www.vmware.com/pdf/VirtualDiskManager.pdf)
-   [Virtual Disk Manager CLI
    examples](http://www.vmware.com/support/ws45/doc/disks_vdiskmanager_eg_ws.html)
