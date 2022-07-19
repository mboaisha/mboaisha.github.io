---
layout: post
title: lolminer, an interesting sample
---
I got that malware analysis itch recently and decided to work on a sample I came across. Haven't touched coinminer-style malware in awhile.

## A background on coinminer malware
In 2017, many things happened in the cyber world, WannaCry was unleashed upon the world allegedly by the Lazarus Group, Equifax was breached and NotPetya caused havoc in Ukraine and spilled over globally.

![btcpricechart2017](/assets/images/btcchart1.png)
In the sidelines however, Bitcoin was gaining momentum by breaking the $1000 mark eventually skyrocking to one of many all time highs Bitcoin would eventually have: $20,000. The cryptocurrency's insane rally likely spurred cybercrime elements to think outside the box, especially since demand for GPUs soared during the crypto market fever. 

The earliest coinminer-style malware that I could remember is Adylkuzz, a malware that rode the ETERNALBLUE (the exploit used to propogate WannaCry) in bandwagon to mine secretly mine Monero. Before 2017, coinminer malware were much rarer in the wild and relied on a skilled adversary to pull off as illustrated by [the incident uncovered by a certain security researcher](https://threatpost.com/dvr-infected-with-bitcoin-mining-malware/105167/) which found an infected digital video recorder device that was an infected with a Bitcoin Miner (In 2014, when Bitcoin was still trading in three digits!)


## Basic sample facts and triage
**Filename:** s.exe
**Size:** 4.77 mb (4773888 bytes)
**MD5:** 601ccdad5d43290b18ce9c0728e52d38
**SHA1:** d6a142337788d09e98af6665ea44899b248e46fd
**SHA256:** f78aa003a899db2d88065eefcad78377325e31bc2f7c4d6ce19e21773cd27d23
**Imphash:** f34d5f2d4577ed6d9ceec516c1f5a744
**SSDEEP:** 	98304:+XjW2zyivc/oHruK4uOQRKRvIgfxgGtaeOwvPDUG2WImwOsxJ:+XjWtivIoHStuOFlIogGc7wvAepwTxJ
**Magic: ** application/vnd.microsoft.portable-executable, (4D 5A)
**Sandbox Results:**
- [VirusTotal](https://www.virustotal.com/gui/file/f78aa003a899db2d88065eefcad78377325e31bc2f7c4d6ce19e21773cd27d23/detection)
- [Any.Run](https://app.any.run/tasks/62c04e96-3540-4a76-9739-456a27b78602/)
- [Hybrid-Analysis](https://www.hybrid-analysis.com/sample/f78aa003a899db2d88065eefcad78377325e31bc2f7c4d6ce19e21773cd27d23/62a07e8694c1667f4c5a4ac3)
- [JoeSandbox](https://www.joesandbox.com/analysis/1009230) 
- [Filescan.io](https://www.filescan.io/uploads/62a0c5116aa17a21fc2cba22/reports/2adf557d-f249-45a1-9021-abdb2a332ba0/overview)
- [Intezer](https://analyze.intezer.com/analyses/281eccd9-7e33-4ddb-90ed-bc1d4f994bfa)

## Hands-on analysis and notes
### Strings analysis
I like to start with a simple string analysis of the binary, as it can give clues on what we are dealing with here. There are couple of interesting observations:
- Indications of the program's name. Consistent with Intezer's verdict of LOLMiner
	- `lolMiner`
	- `get_lolMiner`
	- `lolliedieb@gmail.com`
- Possible capability of detecting virtualization, which explains why the sample did not run fully in sandboxes
	- `DetectVirtualMachine`
	- `DetectSandboxie`
	- `VirtualBox`
	- `vmware`
- List of GPU card model names
	- `nvidia`
	- `geforce`
	- `radeon`
	- `quadro`
- Fragments of a scheduled task command
	- `schtasks.exe`
	- `/create /sc MINUTE /mo 3 /tn "MicrosoftEdgeUpdate" /tr "`
- Bunch of base64 encoded strings, we will get to those later. It's worth noting the last string was preceeded by `powershell.exe` which may or may not be relevant later in the analysis.
	- `Q29uc2VudFByb21wdEJlaGF2aW9yQWRtaW4=`
		- `ConsentPromptBehaviorAdmin`
	- `UHJvbXB0T25TZWN1cmVEZXNrdG9w`
		- `PromptOnSecureDesktop`

	- `U2V0LUl0ZW1Qcm9wZXJ0eSAtUGF0aCAnSEtMTTpcXFNPRlRXQVJFXFxNaWNyb3NvZnRcXFdpbmRvd3MgRGVmZW5kZXIgU2VjdXJpdHkgQ2VudGVyXFxOb3RpZmljYXRpb25zJyAtTmFtZSBEaXNhYmxlTm90aWZpY2F0aW9ucyAtVmFsdWUgMQ==`
		- `Set-ItemProperty -Path 'HKLM:\\SOFTWARE\\Microsoft\\Windows Defender Security Center\\Notifications' -Name DisableNotifications -Value 1`
- Mining pool addresses
	- `asia1-etc.ethermine.org:14444`
	- `asia2.ethermine.org:14444`

- WMI queries
	- `SELECT * FROM Win32_VideoController`
	- `Select * from Win32_ComputerSystem`
- Indication of Anti-Debugging techniques being used... Maybe UPX packing too?
	- `CheckRemoteDebuggerPresent`
	- `antiDebugger`
	- `isDebuggerPresent`
	- `UPX0`
	- `UPX1`

### Decompilation and cursory static analysis
Since the UPX string was present, I checked whether the binary is packed:
![detectiteasyrun](/assets/images/die1.png)

Detect It Easy does not show any packers. It does show that the binary uses .NET, looks like a good time to throw it in dnSpy and take a closer look. (Disclaimer: I am not a reverse engineering expert by any means. Expect *amateur hour* galore from now on!)

The binary got decompiled with no issues, upon taking a cursory look of the code, it seems the author of this program did not employ any obvious tricks to hinder analysis (re: obfuscation), it's routine from what I understand roughly goes is as follows:
- Check for signs of virtualization, if detected immediately exit.
- Check for an existing mutex of `JWSFRGFQXQ`, if it exists, immediately exit.
- Execute `schtasks.exe` with the following arguments: `/create /sc MINUTE /mo 3 /tn \"MicrosoftEdgeUpdate\" /tr \%APPDATA% \\Z.exe \ /f`
	- Essentially the command creates a scheduled task called `MicrosoftEdgeUpdate` that launches `Z.exe` in the Application Data directory in three minutes.
- Import and execute the `lolminer` module and it's dependencies with arguments found in the config object.
	- Looks like it's attempting to mine Ethereum Classic.
	- This is where it does collection of system information to feed `lolminer` arguments.
- Continuously look for the following processes and kill them:
	- `mmc`: Microsoft Management Console
	- `ProcessHacker`: An attempt to make dynamic analysis harder
	- `taskmgr`: task manager
	- `Диспетчер задач`: task manager but in Russian
- Attempt to become Administrator by bypassing UAC.
- Disable Microsoft Defender notifications.

The config class contains juicy information:
![lolMinerconfig1](/assets/images/minerconfig1.png)
- Ethereum Classic Wallet: `0x4dd10a91e43bc7761e56da692471cd38c4aaa426`
- Mining Pool: `asia1-etc.ethermine.org:14444`
- The mutex we saw earlier: `JWSFRGFQXQ`

##  OSINT research notes
### On-chain analysis
Considering that Ethereum Classic is a [hard fork](https://help.coinbase.com/en/coinbase/getting-started/crypto-education/eth-hard-fork) of Ethereum, it is safe to assume that both addresses on ETH and ETC chains are owned by the same person.

I found [Tokenview](https://etc.tokenview.com/en/) to be a good ETC blockchain explorer for my needs here. Upon taking a look of the address, it seems the earliest recorded transaction in the wallet happened in June 6, 2022  as seen [here.](https://etc.tokenview.com/en/tx/0xc2ec1a07b14b51d175d0eb0a1fd1641f1247e6a0ef72267466e5c713ee5fffb3)

The transaction originated from 0xdf7d7e053933b5cc24372f878c90e62dadad5d42, an Ethermine address. [Ethermine](https://ethermine.org/) is a mining pool where cryptocurrency miners would "pool" their computational resources to improve their chances of earning new tokens.

The transaction flow on the wallet appear to be "linear" with no diverging flow, just sending it to [0x216D44960291E4129435c719217a7ECAe8c29927](https://etc.tokenview.com/en/address/0x216d44960291e4129435c719217a7ecae8c29927) which has a lot of $ETC... Which leads me to believe it's an address of an Exchange.

![etctransactions1](/assets/images/etc-visual-transactions1.png)

Now we can start looking at the address on the Ethereum blockchain..... I tried to use [Breadcrumbs.app](https://www.breadcrumbs.app/home) but then you get into a situation like this:
![breadcrumbsMess1](/assets/images/breadcrumbs1.jpg)

A graph that's bit difficult to read and make sense of. I was unable to make the graph clearer to read, though it did allow me to make some key observations and assesments:
- The malware wallet is funded by a mix of crypto originating from centralized exchanges and Tornado.Cash, a crypto tumbler. Crypto tumblers can be used to obfuscate the origin of crypto.
- The malware wallet owner made use of Remitano for Peer-to-Peer purchase of crypto.
- The user also likely engages in Peer-to-Peer exchange of cryptocurrency in the real world.

All in all... I am sure someone with better cryptocurrency investigation skills can carry on from here. Please do let me know if you publish a blog with findings!

### Pivoting
Based on the findings above, I made an experimental Yara rule to help me pivot
```
rule coinminer_lolminer_variant_experimental 
{
    meta:
        author = "Mohammad Boaisha"
        description = "Detection of malware using lolminer"
        created = "07-13-2022"
        last_modified = "07-18-2022"
		
    strings:
		// References to system tools
		$system1 = "powershell" nocase
		$system2 = "schtasks.exe" nocase
		$system3 = "mmc" nocase
		
		// miner artifacts
		$crypto1 = "cryptoWallet"
		$crypto2 = "etcWallet"
		$crypto3 = "HWID" nocase
		$crypto4 = "nvidia" nocase
		$crypto5 = "radeon" nocase
		$crypto6 = "lolMiner" nocase
		$crypto7 = "get_lolMiner" nocase
		$crypto8 = "lolliedieb@gmail.com" nocase
		
		// Malicious Indicators nocase
		$mal1 = "CheckRemoteDebuggerPresent" nocase
		$mal2 = "isDebuggerPresent" nocase
		$mal3 = "DetectVirtualMachine" nocase
		$mal4 = "DetectSandboxie" nocase
	
    condition:
        2 of ($system1, $system2, $system3) and 4 of ($crypto1, $crypto2, $crypto3, $crypto4, $crypto5, $crypto6, $crypto7, $crypto8) and 3 of ($mal1, $mal2, $mal3, $mal4)
}
```

Retrohunting in VirusTotal seemed to match bunch of other coinminers that utilize xmrig instead, which makes sense because lolMiner happens to support the mining of other cryptocurrencies.

Upon inspecting binaries with similar imphash and content, a recuring theme was revealed, a reference to something called [VolVeRMiner](https://teletype.in/@volver/VolVeRMINER). It's very likely that the author of the malware followed the tutorial provided and added specific modifications. 


## References and further reading
- [Malpedia: win.coinminer](https://malpedia.caad.fkie.fraunhofer.de/details/win.coinminer)
- [Investopedia: Bitcon's Price History](https://www.investopedia.com/articles/forex/121815/bitcoins-price-history.asp)
- [Proofpoint: Adylkuzz Cryptocurrency Mining Malware Spreading for Weeks Via EternalBlue/DoublePulsar](https://www.proofpoint.com/us/blog/threat-insight/adylkuzz-cryptocurrency-mining-malware-spreading-for-weeks-eternalblue-doublepulsar)
- [PCWorld: Why AMD’s Radeon graphics cards are almost impossible to buy right now](https://www.pcworld.com/article/406925/why-amds-radeon-graphics-cards-are-almost-impossible-to-buy-right-now.html)
- [Microsoft: schtasks.exe](https://docs.microsoft.com/en-us/windows/win32/taskschd/schtasks)
- [Investopedia: Mining Pool](https://www.investopedia.com/terms/m/mining-pool.asp)
- [Wikipedia: Cryptocurrency tumbler](https://en.wikipedia.org/wiki/Cryptocurrency_tumbler)
- [SANS Internet Storm Center:  Johannes Ullrich: More Device Malware: This is why your DVR attacked my Synology Disk Station (and now with Bitcoin Miner!)](https://isc.sans.edu/diary/More+Device+Malware%3A+This+is+why+your+DVR+attacked+my+Synology+Disk+Station+%28and+now+with+Bitcoin+Miner%21%29/17879)