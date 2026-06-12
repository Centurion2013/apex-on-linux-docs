# Oracle Linux 9 installation steps

## Download Oracle Linux 9
You can download installation images from: https://yum.oracle.com/oracle-linux-isos.html

This documentation was prepared using:
https://yum.oracle.com/ISOS/OracleLinux/OL9/u7/x86_64/OracleLinux-R9-U7-x86_64-dvd.iso (Full Image - 14 Gb)


## Install Oracle Linux 9 
Follow the installation instructions during installation. No additional steps are required.


## Post-install
Do update OS software after installation:

```console
sudo dnf update -y
```

## Minimal Linux configuration
To optimize resource consumption, consider choosing a **Minimal** configuration during Linux setup. Refer to the [manual components installation guide](minimal_linux_installation.md) for additional details.
