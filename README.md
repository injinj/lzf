# Readme for lzf

Originally from http://software.schmorp.de/pkg/liblzf

This provides Debian based systems compatibility with CentOS/Fedora based
systems, which have a package liblzf and liblzf-devel.  I have not found
a Debain liblzf package, so I created this.

The shared library that gets created here is liblzf.so.1, this the same that
the RPM system uses.  If you compile a program using this package, you should
be able to run the same binary on both systems using either the Fedora package
or this package.

The
[liblzf-devel](https://fedora.pkgs.org/31/fedora-x86_64/liblzf-devel-3.6-17.fc31.x86_64.rpm.html)
package also puts the lzf.h and lzfP.h into the /usr/include directory, this
package does that as well.

Here's how to build it and install it.

```console

# install building tools

$ sudo apt-get install make g++ gcc chrpath devscripts

# clone the lzf git

$ git clone https://github.com/injinj/lzf

# make the deb file

$ cd lzf
$ make dist_dpkg

# install the deb file

$ sudo dpkg -i dpkgbuild/lzf_1.0.0-1_amd64.deb

# compress and uncompress something

$ lzf README
$ unlzf README.lzf
$ ldd /usr/bin/lzf
        linux-vdso.so.1 (0x00007ffc2dff0000)
        liblzf.so.1 => /usr/lib/liblzf.so.1 (0x00007fd24c5b2000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fd24c213000)
        /lib64/ld-linux-x86-64.so.2 (0x00007fd24c9b9000)

# list the package contents

$ dpkg -L lzf
/.
/usr
/usr/bin
/usr/bin/lzf
/usr/bin/unlzf
/usr/include
/usr/include/lzf.h
/usr/include/lzfP.h
/usr/lib
/usr/lib/liblzf.a
/usr/lib/liblzf.so.1.0.0-1
/usr/share
/usr/share/doc
/usr/share/doc/lzf
/usr/share/doc/lzf/changelog.Debian.gz
/usr/share/doc/lzf/copyright
/usr/lib/liblzf.so
/usr/lib/liblzf.so.1

# remove the package

$ sudo dpkg -r lzf
```

For Ubuntu 18.04 under Windows 10, this build works for me:

```console
$ sudo apt-get update
$ sudo apt-get install make g++ gcc devscripts libpcre2-dev chrpath
$ sudo apt-get install debhelper
$ sudo update-alternatives --set fakeroot /usr/bin/fakeroot-tcp
```

Continue with above.

The original [README](README) and [LICENSE](LICENSE) are included here, but
the original configure and install scripts are not.

