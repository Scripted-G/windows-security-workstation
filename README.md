# Windows Security Workstation
Decisions and architecture behind a hardened Windows 11 IoT LTSC daily driver built for security work, learning, and homelab.

About This Document
This is not a tutorial. It does not list commands to copy-paste. It documents the reasoning behind a specific platform choice — running a hardened Windows install as a daily driver for security study, systems-level programming, and offensive lab work — and the architectural decisions that follow from that choice.
The audience is people considering similar paths: those interested in security careers but unsure whether they need to be on Linux full-time, those who want to be on Windows but are unhappy with what consumer Windows has become, and those who want to think about platform tradeoffs more carefully than influencer culture tends to allow.
If you want command-by-command instructions, this is the wrong document. If you want to understand why someone would choose this architecture and how the pieces fit together, read on.
The Platform Question
Most online discussion of operating systems for security work skews toward Linux. The community signal is loud: real practitioners use Linux, real hackers run Arch, anyone serious about systems work does it on a Unix-flavored OS. This is partly true and largely overstated.
The honest version: the Linux/Unix culture is where C, the C development toolchain, and most security tooling were born. The terminal-native workflows that mature security professionals develop — tmux, command-line everything, configuration as code — are easier on Linux because Linux was built around them. None of this is propaganda. The cultural fit is real.
What's less acknowledged is that this culture is loudest in places where hobbyists and students gather, not where professionals work. Practitioners doing offensive security in government, in industry, and in well-known consultancies often run Windows or macOS as their daily driver and isolate their offensive tooling to specialized Linux distributions in VMs. The pattern is consistent across multiple sources: Windows as daily driver for normal computing (email, communications, documents, gaming, general use), Kali or Parrot in VMs for the actual security work, sometimes custom Linux spins on dedicated hardware for specific operations.
This separation isn't compromise. It's the right architecture. Daily-driver concerns and offensive tooling concerns are genuinely different — mixing them creates risk (offensive tools running alongside personal data) and friction (offensive distros are not optimized for daily use). The mature setup acknowledges this and provisions appropriate environments for each layer.
The implication: the right question isn't "which OS is best for security work?" The right question is "which OS makes the most sense as my daily driver, given that my actual security work will live in VMs anyway?" Once framed that way, the answer becomes about your own life, your own tooling needs, and your own preferences — not about ideological alignment with any community.
Constraints I Was Designing Around
The architecture below is shaped by specific constraints. Other people with different constraints would arrive at different answers.

What I needed:
A daily driver suitable for sustained study, focused work, and gaming
Privacy and control over what software runs and what data leaves the machine
A real offensive lab with proper network isolation
A development environment for learning C and eventually Go
Hardware support for current GPUs and audio equipment
Integration with software I actually use (Office, specific creative tools, games)

What I was unwilling to accept:
Microsoft account requirement
Telemetry, Recall, or Copilot integration
Forced updates that change behavior or remove control
AI features integrated into the OS regardless of consent
Loss of gaming compatibility
Loss of audio/video stack quality

What I was willing to give up:
Some out-of-box convenience (the system requires real configuration)
Access to certain mainstream Windows features that conflict with the privacy posture
The "everything is one neat ecosystem" feeling that consumer Windows or macOS provides

These constraints rule out several options. Consumer Windows 11 fails on the privacy and control axis. macOS fails on hardware flexibility and gaming. Stock Linux distributions fail on gaming compatibility, certain professional software, and some audio/video workflow polish. The remaining option that meets these constraints is a hardened Windows install — specifically, an edition of Windows where Microsoft has already removed most of what I'd want to remove anyway.
Why Windows 11 IoT Enterprise LTSC
Windows 11 IoT Enterprise LTSC 2024 is the edition I run. It is genuinely different from consumer Windows 11 in ways that matter.
LTSC stands for Long-Term Servicing Channel. It's an enterprise edition designed for environments where stability matters more than new features — industrial systems, ATMs, medical devices, and similar deployments that can't tolerate disruptive updates. Microsoft maintains it on a separate update track that doesn't introduce feature changes mid-cycle.
The IoT designation extends LTSC to embedded and specialized deployment scenarios. Functionally, it's the same kernel and core OS as enterprise LTSC with a longer support lifecycle (typically ten years).
What makes IoT LTSC the right base for a privacy-conscious daily driver:

No Cortana, no Copilot, no Recall. These are not stripped out by user action — they're not present in the edition at all. There is no AI assistant integrated into the OS. There is no Recall feature taking screenshots of your activity. There is no Copilot suggesting actions or processing your usage.
No Microsoft Store or built-in consumer apps. The bloat that ships with consumer Windows 11 (Candy Crush, news apps, weather apps, prompts to try OneDrive, prompts to try Microsoft 365) is absent. The system boots into a clean desktop with only what you choose to install.
No forced feature updates. Updates are security-only on the LTSC track. Microsoft does not push interface changes or new features to LTSC machines mid-cycle.
Group Policy support. The full range of Group Policy controls is available, which matters for hardening — many consumer-edition Windows installations cannot use Group Policy to disable telemetry or enforce privacy settings because Home edition doesn't support it.
Designed for environments that need stability and control. The whole edition is built around the assumption that the operator wants to make decisions about what runs and what doesn't, rather than having Microsoft make those decisions on their behalf.

The tradeoff is that LTSC is harder to obtain than consumer Windows. It's licensed for specific deployment scenarios and isn't sold through normal retail channels. People who want it have to find it through legitimate volume licensing or other appropriate paths. This is by design — Microsoft doesn't want consumer users on LTSC because LTSC reduces the data collection and ad surface that consumer Windows is built around.
For a daily driver where the priority is privacy and stability over new features, this is the right edition. For someone who wants the latest UI experiments and AI integrations, it's the wrong edition.
Hardening Approach
Even with LTSC as the base, additional hardening was applied. The approach was philosophical rather than checklist-driven: assume Microsoft's defaults are wrong for my use case unless proven otherwise, and verify each setting reflects intentional choice rather than ignorance.
The categories that mattered:
No Microsoft account. The install was completed using a local account. This is increasingly difficult on consumer Windows but remains supported on LTSC. The result is that no aspect of my daily computing is tied to a cloud identity.
Telemetry disabled. Both Group Policy settings and service-level disabling were used to ensure that diagnostic data and telemetry don't leave the machine.
OneDrive prevented. Group Policy blocks OneDrive integration regardless of whether the client is installed. This matters because some Windows components (notably Microsoft Photos) try to integrate with OneDrive at the API level even when the OneDrive client itself isn't present.
Bloat removed. Several legacy or duplicate components that ship in LTSC were removed via DISM (Deployment Image Servicing and Management) and registry modifications. The principle: nothing should be on the machine that I don't actively use.
File Explorer simplified. Several default UI elements that consumer Windows added in recent versions were removed at the registry level. The goal is a file manager that does what file managers do, without integration prompts for cloud services or AI features.
The end result is a Windows install that boots into a clean, fast, controlled environment that does what I tell it to and nothing else. It's the Windows experience that consumer Windows used to be before Microsoft began monetizing the operating system itself.
Toolchain Decisions
With the base OS settled, the next decisions were about development and security tooling.
C development: MSYS2 + native compilation, not WSL2. When learning C, the question of WSL2 versus native Windows tooling matters. WSL2 makes sense if your target is Linux — you're building software that will run on Linux servers, and you want to develop in the target environment. My target is the language itself, with eventual goals that include Windows-native tooling. MSYS2 provides GCC and the standard Unix-style C development environment as native Windows binaries. Code written this way produces real Windows executables. The compiler invocation, debugger workflow, and standard library are functionally identical to Linux for the kind of work a textbook covers.
The tradeoff: MSYS2 is cosmetically more involved than apt install build-essential would be on Debian. Once configured and added to PATH, the daily experience is equivalent.
Virtualization: VMware Workstation Pro, not Hyper-V. Windows includes Hyper-V, and Microsoft pushes it as the default virtualization layer. I chose VMware Workstation Pro instead. Reasons:

VMware's UX for individual VM management is significantly more polished
Cross-platform familiarity — Workstation Pro behaves the same on Windows and Linux, and VMs created in one can be opened in the other
TPM passthrough, UEFI/Secure Boot configuration, and isolated network types are first-class features rather than enterprise-only options
Snapshot management is genuinely good, which matters for offensive lab work where you want to roll back state frequently

VMware Workstation Pro became free for personal use in 2024, which removed the previous cost barrier. For someone running a serious homelab, this is now the default rational choice on Windows.
Shell: PowerShell 7, not PowerShell 5.1. Windows ships with PowerShell 5.1 (the legacy "Windows PowerShell"). PowerShell 7 is the modern, cross-platform, actively developed version and must be installed separately. The default integration experience treats PowerShell 7 as a first-class option, but you have to do the install yourself.

For someone serious about Windows administration as a skill, PowerShell 7 is the version to invest in. It's faster, has better cross-platform compatibility, and is where new features land. Sticking with 5.1 because it's the default would be like sticking with Python 2 in 2026 — possible but contraindicated.
Editor: VS Code, not full Visual Studio or terminal-native editor. For C learning specifically, VS Code with the C/C++ extension provides enough integration to be productive without abstracting away the compilation process the way a full IDE does. The pedagogical value of typing gcc file.c -o file and seeing the output is real — IDEs that hide this make learning the language harder, not easier.
Terminal-native editors (Neovim, Emacs) are legitimate options with genuine advantages, particularly for remote development. The learning curve is real, though, and competing with cert study and language acquisition for attention budget. They're worth investigating later, not adopting now.
Architecture of the Result
The system is designed around clear separation of concerns:
The host (Windows 11 IoT LTSC): Runs daily computing tasks. Web browsing, email, document work, study, code editing, gaming. No security tooling lives here. The host has access to the internet but is hardened against telemetry and unauthorized data egress.
The offensive lab (Kali Linux + Metasploitable 2 in VMs): Runs in VMware on a NAT network. The attack platform (Kali) and intentionally vulnerable target (Metasploitable 2) communicate with each other and have outbound internet for tool updates, but are isolated from the host's normal network presence.
The Active Directory lab (Windows Server 2022 + Windows 11 Pro in VMs): Runs on a separate isolated virtual network (host-only). The Domain Controller serves the domain-joined client. The client's internet access routes through the DC, mirroring the network topology of an actual enterprise environment. This network has no direct connection to the offensive lab.
The result: Three distinct environments on one machine, each appropriate to its purpose, none compromising the others. The host stays clean. The offensive lab stays isolated from personal data. The AD lab stays isolated from offensive activity. This is the architecture professionals use, and it's achievable on a single workstation with proper virtualization.
What This Approach Demonstrates
The choices documented here aren't right for everyone. They're right for someone with a specific set of constraints and priorities. The value of writing them down isn't to argue that this is the universal answer — it's to demonstrate that platform choices can be made deliberately, with reasoning, rather than inherited from defaults or community pressure.
Specifically, this architecture shows that:

Privacy-conscious computing is achievable on Windows without abandoning the platform's strengths
Security work doesn't require Linux as a daily driver — proper isolation via VMs is the more rigorous approach anyway
Influencer culture's "real practitioners use X" framings are usually identity claims, not engineering arguments
Platform choice should serve the work, not the other way around

For someone evaluating their own setup, the takeaway is to ask what you actually need from your daily driver, what you actually need from your specialized environments, and whether your current architecture provides each appropriately. The right answer for you may be different from what's documented here. The point is to make the choice deliberately.
Closing Note
This document will evolve as my use of the system evolves. If I add tools, refine the lab, or revise decisions, I'll update the relevant sections. The architecture is stable, but the specifics of what runs where will change as my work changes.
If you're building something similar and have questions, reach out via the contact information on my profile.
