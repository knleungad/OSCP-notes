Two broad categories:
- On-disk evasion
- In-memory evasion

## On-disk evasion

Packers
- Generates an executable which is smaller but functionally equivalent with a new binary structure (with a new hash signature)
- Alone not sufficient to evade modern AVs

Obfuscators
- Reorganize and mutate code in a way that makes it more difficult to reverse-engineer
- Marginally effective against signature-based detection
- Also have runtime in-memory capabilities

Crypter
- Cryptographically alters executable code, adding a decryption stub that restores the original code upon execution
- Decryption occurs in-memory (leaving only encrypted code on-disk)
- One of the most effective AV evasion techniques

Modern AV evasion requires combination of multiple techniques: anti-reversing, anti-debugging, virtual machine emulation detection, etc.

Tools:
- [The Enigma Protector](https://www.enigmaprotector.com/en/home.html) (commercial)

## In-memory evasion
In-memory injections (aka PE injection) is used to bypass AV products on Windows machines by manipulation of volatile memory. It does not write any files to disk.

Remote Process Memory Injection
- Injects the payload into another valid PE that is not malicious

Reflective DLL Injection
- Loads a DLL stored by the attacker in the process memory
- Unlike regular DLL injection, which loads a malicious DLL from the disk using LoadLibrary
- Attacker must write their own version of the API that does not rely on a disk-based DLL

Process Hollowing
- First launch a non-malicious process in a suspended state. Once launched, the image of the process is removed from memory and replaced with a malicious executable image. Finally, the process is then resumed and malicious code is executed instead of the legitimate process.

Inline Hooking
- Modifying memory and introducing a hook (an instruction that redirects the code execution) into a function to make it point to malicious code. Upon executing malicious code, the flow will return back to the modified function and resume execution, appearing as if only the original code had executed.
- Often employed by rootkits.



