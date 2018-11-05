

                 VMware OVF Tool 4.2.0

System Requirements
-----------------------
VMware OVF Tool is supported on the following operating systems:

Windows 32-bit (x86) and 64-bit (x86_x64)
   Windows Vista
   Windows 2008
   Windows 7
   Windows 8
   Windows 2012
   Windows 8.1
   Windows 2013 R2

Linux 32-bit (x86) and 64-bit (x86_x64)
   CentOS 6.x
   Fedora Core 14.x 15.x 16.x 17.x 18.x
   RedHat Enterprise Linux (RHEL) 6.x
   SUSE Enterprise Server 11.x
   Ubuntu Desktop 10.x 11.x 12.x

Mac OS X:
   Mac OS X 10.7
   Mac OS X 10.8
   Mac OS X 10.9

Changes since 4.1.0
-------------------
  . Added SHA256 and SHA512 support for both manifest validation and digital
    signing.
  . Improved security by disabling TLSv1.0 protocol. Added the --sslVersion
    command line option to allow users to specify SSL version for HTTPS
    connections.
  . Improved security by disabling Diffie Hellman cipher from the default cipher
    suite. Added a command line option --sslCipherList to allow user to override
    default cipher suite.
  . Bug fixes and feature enhancements.

Changes since 4.0.0
-------------------
  . Expose debug options to users
  . Bug fixes and feature enhancements.

Changes since 3.5.0
-------------------
  . Bug fixes and improvements

Changes since 3.0.2
-------------------
  . Support for vCloud Suite 5.5
  . Full IPv6 support against vCloud Director 5.5

Changes since 3.0.1
-------------------
  . Signed Mac OS X installer.
  . Fix bug on Mac OS to handle OVF Tool being in $PATH and resolve the install
    directory correctly.

Changes since 3.0
-------------------
  . Changed default to not export device subtypes for CDrom/floppy/parallel/serial
    ports. This improves portability between Workstation and ESX. This can be
    controlled with the -exportDeviceSubtypes option.
  . Fixed bug in handling of message bundles
  . Fixed bug in handling of misaligned memory sizes

Changes since 2.1.0
-------------------
  . Supported values for diskMode and ipAllocationPolicy have changed. See --help for more info.
  . Support for vSphere 5.1.
  . Lossless support when exporting OVFs/VMXs.
  . Support for reading/writing sha256 manifests. See --help for more info.
  . Improved security by verifying SSL thumbprints. This is controlled by the options
    --sourceSSLThumbprint, --targetSSLThumbprint, and --noSSLVerify. The default
    is to verify and prompt for acceptance.

Changes since 2.0.1
-------------------
  . Support for vCloud as both a source and a target.
  . Enhanced integration support (--machineMode).
  . Support for vSphere 5.0.
  . Improved lax mode support.
  . Improved error reports.

Changes since 2.0.0
-------------------
  . Fixed bug that caused OVF Tool not to work with the "Import from vSphere"
    feature on the VMware Virtual Appliance Marketplace.
  . Fixed launch issue on Linux/Mac if installed in a location where the path
    includes spaces.
  . Fixed issue with properties when importing to vApprun workspace.

Changes since 1.0.1
-------------------
  . Support for the OVF 1.1 specification.
  . Support for OVF packages that includes ISO and floppy images.
  . Support for vApprun as target and source
    (see http://labs.vmware.com/flings/vapprun).
  . Support for OVF Tool on the Mac OS X operating system (Intel-only).
  . A lax mode (--lax) that enables a best-effort conversion of OVF
    packages which do not fully conform to the OVF standard or include
    virtual hardware that is not supported by the destination. For example,
    VirtualBox (version 3.0 and 3.1) produced OVF packages can be converted in lax mode.
  . Bug fix for handling vmxnet3 targets.

FAQ
------------------
Q: How do I use OVF Tool with VMware Server 2.0.x?
A: VMware Server 2.0.x default port is 8333. To connect to a VMware Server 2.0
   host use vi://username:password@hostname:8333/ as target/source.

Q: What type of X.509 certificates does OVF Tool use?
A: X509 certicates used for OVF Tool must be properly formatted so that each
   line in the base64 encoding is 64 characters long except the last line which
   may be shorter. This conforms to RFC 1421(cf. page 12,
   http://www.ietf.org/rfc/rfc1421.txt).

Q: Can I use the OVF Tool on the ESX Service Console?
A: Yes. However, you must use a VI locator as a target (use localhost as the
   host), and specify the username/password for the host. Using a file
   target (i.e., a file path directly to a VMFS volume) will generate
   files that are incompatible with ESX.

Q: Can I use OVF tool on an ESX host to backup a VMX file?
A: Yes, but you have to use a VI source locator to export the virtual machine
   to an OVF package. Do not use OVF Tool directly on the VMX files on a VMFS
   filesystem.

Q: How do I encode special characters in locators?
A: The locators are URLs, so special characters must be escaped using % followed
   by their ASCII hex value. For instance, if you use a "@" in your password, it 
   must be escaped with %40 as in vi://foo:b%40r@hostname, and a slash in
   a Windows domain name (\) can be specified as %5c.

Q: Which versions of vSphere support OVF packages with ISO/floppy images?
A: vSphere 4.1 and later. You can use the flag --noImageFiles to ignore upload
   of ISO and floppy image files for older versions of vSphere.

Q: When using a windows network share as target filesystem it fails with "Failed to create disk:
   \\some_server\Public\test-disk1.vmdk. Reason: The destination file system does
   not support large files."
A: A workaround for this is to assign a drive letter for the network share or
   add support for "compression", "unicode on disk" and "persistent ACLs" as filesystem
   attributes on the shared filesystem.

For More Information
-----------------------
For more information about VMware OVF Tool, see
http://www.vmware.com/go/ovf/

Follow the vApp blog on http://blogs.vmware.com/vapp/
----------------------------------------------------------------------
Copyright 2012 VMware, Inc.  All rights reserved.
