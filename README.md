This is a patch for the game client "Metal Rage Online" to get it to run on Windows 11, written in C & Python.

The resource protection system Y0da causes compatability issues on Win11 that are not present on previous versions. 
This patch circumvents that as well as resolves other issues with the game on modern windows versions.

Thanks to Zephos for helping to co-create this fix!

Client & Installer files avaliable [Here](https://archive.org/download/metal-rage-online-client/MetalRage%20Online/).

Documentation for the fixes is in the Wiki [Here](https://github.com/shanzenos/Metal-Rage-Online-Win11-Fix/wiki)

The following links were also very useful for working on this:
* [Y0da soruce code](https://sourceforge.net/projects/yodap/)
* [Y0da unpacking scripts](https://github.com/dubuqingfeng/ollydbg-script/tree/master/Yodas%20Protector)

#### Why patch 5 (TLS callbacks)

The y0da shell registers two TLS callbacks (0x1095112D and 0x10951159). Both end with a plain `ret` (`c3`), not the `ret 0xC` that the Windows TLS callback convention (`stdcall`, three arguments) needs. On Windows 11 (observed on build 26200), the WoW64 ntdll helper that calls TLS callbacks does not restore `esp` afterwards, so the return lands on the `Reason` argument (0 to 3) and the process takes an access violation at address 0x0 to 0x3. That happens on every thread create and thread exit, and every one of these exceptions is reported to Windows Error Reporting, which causes the very slow startup and the short stalls during play. Patch 5 sets the TLS directory's `AddressOfCallBacks` to the terminating 0 of the callback list, so the loader calls no callback, and stores callback 2 (0x5A bytes at file offset 0x19159) already decoded, which is what callback 1 would have done at process attach. The entry point decodes the rest of the payload itself (patch 2), so nothing else changes.
