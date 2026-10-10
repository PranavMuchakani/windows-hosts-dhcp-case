# Windows troubleshooting: an oversized HOSTS file
A real troubleshooting case involving a Windows HOSTS file with hundreds of thousands of entries. This write-up separates what was observed from what can only be inferred.
What was observed
The Windows HOSTS file at C:\Windows\System32\drivers\etc\hosts was 10,793,413 bytes (about 10.8 MB).
Inspection showed approximately 419,580 active, non-comment lines.
The Windows machine was also experiencing connectivity trouble. Earlier diagnostics showed an IPv4 self-assigned address in the 169.254.x.x range, which indicates that a normal DHCP IPv4 lease had not been obtained.
Replacing the file while Windows was running failed with an access/lock problem.
Resource Monitor's CPU → Associated Handles search showed svchost.exe (PID 2268, running as NetworkService at the time of inspection) with a handle referring to the HOSTS file. Do not terminate an arbitrary svchost.exe process: it can host critical Windows services.
In Windows Recovery Command Prompt, the Windows installation was confirmed on C:\. The file was overwritten directly using:
echo # Standard Windows HOSTS file > C:\Windows\System32\drivers\etc\hosts
The file size then measured 32 bytes. No backup was created.
After returning to Windows, a website (the KSRTC site) loaded in the browser.
What this does—and does not—prove
The sequence is consistent with the HOSTS file contents contributing to hostname-resolution or website-access problems. HOSTS entries can override normal DNS results: if a domain is mapped to a loopback or other incorrect address, that hostname may fail to load even when the network link is up.
However, this case does not prove that the HOSTS file caused the DHCP failure. DHCP assigns an IP address before ordinary hostname lookup; a HOSTS file normally does not determine the DHCP lease. In this case, connectivity problems may have had more than one cause, and a website loading after cleanup does not by itself confirm that IPv4 DHCP was repaired (the browser may use IPv6).
Likewise, the earlier DNS Client memory measurement was only a snapshot and did not prove that all measured memory was caused by the HOSTS file.
Why an enormous HOSTS file can be a problem
Hundreds of thousands of mappings are far beyond what most users need in the system HOSTS file.
Blocking lists may accidentally include legitimate domains or map them to blocking addresses.
A large local mapping list adds data to hostname-resolution handling and can complicate troubleshooting, although the performance impact depends on the Windows version and workload.
Filtering products designed for large domain lists may be easier to maintain than placing an enormous list directly in the Windows HOSTS file.
How to inspect a HOSTS file lock
Run Resource Monitor (resmon).
Open CPU.
Expand Associated Handles.
Search for hosts.
Inspect the process name and full path. Do not kill a system process just because it has a handle open.
If Windows will not let you replace the file, a controlled Windows Recovery Environment can provide a way to edit it while the normal Windows services are not running. First verify the actual Windows drive letter in Recovery; it may not match the drive letter seen during a normal boot.
Restore a minimal HOSTS file (destructive)
Warning: This replaces the current HOSTS contents. It does not make a backup. Any blocklist entries in the file will be permanently removed unless they exist elsewhere.
From an elevated or Recovery Command Prompt, after verifying the Windows installation is actually on C:\Windows\, the minimal replacement used in this case was:
echo # Standard Windows HOSTS file > C:\Windows\System32\drivers\etc\hosts
Then verify the result:
dir C:\Windows\System32\drivers\etc\hosts
type C:\Windows\System32\drivers\etc\hosts
A typical Windows HOSTS file may also contain commented examples for localhost; those lines are not required to be active entries. Use the correct Windows drive letter for your own system.
Takeaway
A 10.8 MB HOSTS file with roughly 419,580 active entries is an important clue worth checking when investigating name-resolution problems. Replacing it reduced the file to 32 bytes, after which a website loaded in this case. Treat that as an observed before/after result—not proof that a HOSTS file explains every network symptom, especially DHCP failures.
