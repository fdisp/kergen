# kergen
Linux kernel config generator

## What's it all about
Building your kernel from source, while practical, it is also a long and tedious task. 
Looking up hardware information, searching for needed drivers, enabling each driver separately along with options each driver depends on, is a very long and boring task. However, like any other boring human work, it will eventually be automated. Kergen is a step in that direction.

Note that this application does not intend to completely replace the configuration process, but to greatly simplify it.
There are many different ways in which a user would want to configure their kernel, and this application does not favor either.
It will only add kernel options that it's 100% sure you will need. So mainly it will support all of your plugged in hardware, mounted file systems, etc. etc.
Currently it takes care of all PCI, USB, SCSI, devices, and file system support, and works only on x86 architectures. In future versions, other types of hardware and architectures will be supported as well.

#  Setup

## 0. Install Dependencies
Open terminal and type:

`sudo apt-get install python git`

## 1. Get Kergen

   `git clone https://github.com/nichoski/kergen.git`

   `cd kergen`

## 2. Install

`sudo python setup.py build`
        
`sudo python setup.py install`

For a list of setup parameters type:

`sudo python setup.py -h`

the following output will be displayed:


    user@debian:/home/user/kergen$ python setup.py --help
    Common commands: (see '--help-commands' for more)

      setup.py build      will build the package underneath 'build/'
      setup.py install    will install the package

    Global options:
      --verbose (-v)      run verbosely (default)
      --quiet (-q)        run quietly (turns verbosity off)
      --dry-run (-n)      don't actually do anything
      --help (-h)         show detailed help message
      --no-user-cfg       ignore pydistutils.cfg in your home directory
      --command-packages  list of packages that provide distutils commands

    Information display options (just display information, ignore any commands)
      --help-commands     list all available commands
      --name              print package name
      --version (-V)      print package version
      --fullname          print <package name>-<version>
      --author            print the author's name
      --author-email      print the author's email address
      --maintainer        print the maintainer's name
      --maintainer-email  print the maintainer's email address
      --contact           print the maintainer's name if known, else the author's
      --contact-email     print the maintainer's email address if known, else the
                          author's
      --url               print the URL for this package
      --license           print the license of the package
      --licence           alias for --license
      --description       print the package description
      --long-description  print the long package description
      --platforms         print the list of platforms
      --classifiers       print the list of classifiers
      --keywords          print the list of keywords
      --provides          print the list of packages/modules provided
      --requires          print the list of packages/modules required
      --obsoletes         print the list of packages/modules made obsolete

    usage: setup.py [global_opts] cmd1 [cmd1_opts] [cmd2 [cmd2_opts] ...]
       or: setup.py --help [cmd1 cmd2 ...]
       or: setup.py --help-commands
       or: setup.py cmd --help


## Usage
kergen is called as root, and without arguments it does nothing. Here is a list of arguments and their functions:  
-g, --generate   - Generate optimized kernel options for your hardware and add the non existing ones to .config  
-n, --new        - Start from scratch without using any existing config files (runs 'make mrproper' and 'make defconfig')  
-u, --upgrade    - Select the newest installed kernel version and if --new is not used copy the old .config file and run 'make olddefconfig'.  
-m, --menuconfig - Opens a kernel configuration menu at the end of all other operations, but prior build  
-b, --build      - Build a new kernel and copy it in /boot

Optionally depgen and kergen-map can be used as standalone tools with the following functions:  
kergen-map is called without arguments and returns a list of kernel options needed to satisfy the current hardware (at the moment it only works for PCI, USB, and SCSI hardware) and mounted filesystems.

depgen accepts a list of kernel config options and returns a list of needed dependencies for those options. It also prompts the user asking whether or not to add those options along with the dependencies to the kernel .config file.

## Structure and functions
Kergen is split into three tools: kergen, kergen-map, and depgen
    
kergen is the tool that interacts with the user, and uses both kergen-map and depgen to satisfy the user's needs. It provides interface for generating new configurations, and it simplifies the usage of many functions that are already provided by the kernel, such as opening the menuconfig, building and installing a new kernel, and upgrading a kernel, using the adapted options of the old one's .config.

kergen-map looks for hardware information in the /sys filesystem, operates over that information and finds the needed drivers using the modules.alias database, and then searches the kernel source code to get the proper kernel config option for each driver.

depgen receives a list of kernel options as arguments. Then it reads the info from all the Kconfig files in the kernel sources, from which it gets dependency expressions for every kernel option, and the dependency expressions are parsed, resulting with possible lists of dependency kernel options that would satisfy each option. The possible lists are verified against the .config file, and against each other, and only the non conflicting ones are left out, trimmed of any possible options that are already satisfied. Dependencies of each dependency option are also calculated, and added to the lists. Then every possible combination of dependency options for each kernel option is made, and only the one having the smallest amount of additional dependencies is added to the .config file.

## Technologies used:
All three tools are written in Python 3.
Information is gathered from the kernel source code, the /sys filesystem, and the modules.alias file which is provided by the project.
modules.alias gets generated automatically whenever a kernel is built, and contains info for all the built modules. For users of this app not to have to build all the kernel modules to generate modules.alias, the project will provide a copy.
