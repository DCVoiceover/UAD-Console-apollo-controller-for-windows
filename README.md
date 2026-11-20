Congratulations! Reading this, you've shown you're a voice actor who reads

the instructions first. Or you ignored the instructions, screwed up, and reluctantly

clicked on this file. If you're the latter, your father would be proud. 

Feel free to read the message from your father that is appropriate to your order 

of operations. Do not read the one that doesn't relate to your situation. I'll know.



| I see. You know, your sibling would have figured it out without a ReadMe. - Your dad |

| Nice job, son/daughter/offspring of your chosen title of truth. How about we throw the baseball around in the backyard? - Your dad |



The rest of this file was written by Claude. If that's not cool, please send all 

flogging hate mail to the senator of your choice. 

It's way more cathartic than flaming me.

# 

# Apollo Twin X Gain Hotkeys (Windows)

Jump your Apollo mic preamp gain to saved levels with one keypress or Stream Deck button, from any window, without opening UAD Console.

It includes a small settings window where you can see the live gain, capture the current level into a preset, and edit your preset values.

**Tested with:** Apollo Twin X USB, UAD Console, Windows, Stream Deck+. It may work with other Apollos that UAD Console controls, but the device and input numbers in step 4 may differ.

> \*\*Unofficial.\*\* This uses a local port that Universal Audio doesn't document. A future UAD update could change or break it. Use at your own risk. Not affiliated with or supported by Universal Audio.

## How it works

UAD Console starts a background program (the UA Mixer Engine) that listens for commands on a local port (`127.0.0.1`, port `4710`). These scripts send it commands, and Console and the hardware knob follow along automatically.

## Files

|File|What it does|
|-|-|
|`ua.ps1`|Sends one command to the engine (PowerShell helper)|
|`readgain.ps1`|Reads the current gain and saves it for the settings window|
|`apollo-gain.ahk`|The main program: hotkeys and settings window (AutoHotkey v2)|

## Setup

### 1\. Put the files in a folder

Create `C:\\ApolloGain` and copy `ua.ps1`, `readgain.ps1` and `apollo-gain.ahk` into it. Any folder works as long as all three files stay together.

### 2\. Install AutoHotkey v2

Download **v2** (not v1) from [autohotkey.com](https://www.autohotkey.com) and run the installer.

### 3\. Start UAD Console and test the connection

Open UAD Console and leave it running. Then open PowerShell and run:

```powershell
cd C:\\ApolloGain
powershell -ExecutionPolicy Bypass -File .\\ua.ps1 "get /devices"
```

You should get a short reply that lists devices, like:

```
{"path": "/devices", "data": {... "children": {"0": {}, "1": {}}}}
```

If you get nothing or an error, make sure UAD Console is open and your Apollo is powered on, then try again.

### 4\. Find your device and input numbers

The scripts assume your Apollo is device `0` and your mic is input `0` (Mic 1). Check:

```powershell
powershell -ExecutionPolicy Bypass -File .\\ua.ps1 "get /devices/0"
```

Look for `DeviceName` and `"DeviceOnline": true`. If device 0 is a different or offline unit, try `get /devices/1`, and so on.

Mic 2 is input `1`. To confirm a mic input, run:

```powershell
powershell -ExecutionPolicy Bypass -File .\\ua.ps1 "get /devices/0/inputs/0/preamps/0"
```

and look for a field called `GainTapered`.

If your numbers differ from `0` and `0`, edit the `/devices/0/inputs/0` part in two places:

* `apollo-gain.ahk`, the line starting `Prop := "/devices/0/inputs/0/..."`
* `readgain.ps1`, the path inside the quotes on the 3rd line

### 5\. Learn how the gain value works

Gain is controlled with a field called `GainTapered`, a number from 0 to 1 that matches the knob position. The plain `Gain` field does **not** follow the knob, so don't use it.

Try it with headphones at a comfortable level, since gain will jump:

```powershell
powershell -ExecutionPolicy Bypass -File .\\ua.ps1 "set /devices/0/inputs/0/preamps/0/GainTapered/value 0.5"
```

Console and the hardware knob should move. Set it back afterward.

What Console displays depends on your Unison preamp:

* With a Unison preamp active (for example VoxBox), Console's number is roughly the 0-1 value times 10 (0.70 shows 7.0).
* With no Unison preamp, Console shows dB. It's close to `10 + 55 x value` but not exactly linear near the top of the range, so measure instead of calculating (see step 8).

### 6\. Run the program

Double-click `apollo-gain.ahk`. A green **H** icon appears in the system tray (click the small up-arrow in the taskbar if you don't see it). On the first run, the settings window opens automatically.

### 7\. Hotkeys

|Hotkey|Action|
|-|-|
|`Ctrl+Alt+1` to `Ctrl+Alt+5`|Preset bank 1 (intended for a Unison preamp such as VoxBox)|
|`Ctrl+Alt+Shift+1` to `Ctrl+Alt+Shift+5`|Preset bank 2 (intended for no Unison preamp, plain dB)|
|`Ctrl+Alt+Up` / `Ctrl+Alt+Down`|Small increase / decrease (for a Stream Deck+ dial)|
|`Ctrl+Alt+G`|Open the settings window (or double-click the tray icon)|

The preset values in the file are examples from the original author's setup. Replace them with your own in the next step.

### 8\. Set your own preset values

Open the settings window (`Ctrl+Alt+G`). The big number is your live gain (0 to 1), updating about once a second while the window is open.

1. In UAD Console (or on the hardware knob), set the gain to a level you want.
2. In the settings window, click **Capture** next to the preset you want. The live value is copied into that row.
3. Repeat for each preset. You can also type values directly (0 to 1).
4. Click **Test** on a row to try it before saving.
5. Click **Apply and save**. Your values are saved to `apollo-gain.ini` next to the program and take effect immediately.

Tip: set bank 1 with your Unison preamp on and bank 2 with it off, so the Console readout matches the levels you want in each case.

**Dial step** is how much each nudge changes the gain (0.01 is a good default).

### 9\. Stream Deck

In the Stream Deck app, drag a **Hotkey** action (System category) onto a button, click the hotkey field, and press the combination (for example `Ctrl+Alt+1`). Label the button with the level. Repeat for each preset.

**Stream Deck+ dial:** add a Hotkey action to a dial. If it offers separate rotate-left and rotate-right fields, set rotate right to `Ctrl+Alt+Up` and rotate left to `Ctrl+Alt+Down`. Set the dial press to any preset hotkey if you want a "snap back" level.

### 10\. Start automatically at login

1. Press `Win+R`, type `shell:startup`, press Enter.
2. Put a shortcut to `apollo-gain.ahk` in that folder.
3. Put a shortcut to UAD Console in that folder too. The hotkeys only work once Console has started, because Console is what launches the engine.
4. In Stream Deck preferences, make sure **Launch at login** is on.

After the first save, the program starts quietly in the tray with no window. Restart once and press a button to confirm everything comes up on its own.

## Troubleshooting

**Nothing happens when I press a hotkey**

* Is UAD Console running? The engine only exists while Console is running.
* Is the green H icon in the tray? If not, run `apollo-gain.ahk`.
* Re-run the step 3 test. If that fails, the problem is the connection, not the hotkeys.
* Another program may be using the same key combination.

**The settings window shows `--` or "Is UAD Console running?"**

Run these and check that a reply prints:

```powershell
powershell -ExecutionPolicy Bypass -File .\\readgain.ps1 "$env:TEMP\\ua\_live.txt"
type "$env:TEMP\\ua\_live.txt"
```

**Gain moves but not on the right mic**

Mic 1 is input `0`, mic 2 is input `1`. Check the step 4 paths.

**PowerShell says scripts are disabled**

The commands above use `-ExecutionPolicy Bypass` for this reason. Include it when testing by hand.

**AutoHotkey shows an error on launch**

Confirm you installed AutoHotkey v2 (not v1) and that all three files are in the same folder. Note the exact error message and line number when reporting an issue.

**A UAD update broke it**

The engine's command format is unofficial and may change. Re-run steps 3 to 5 to see what the engine now reports.

## Notes

* Each press runs a short hidden PowerShell command, so there's about a quarter of a second between pressing and the gain moving.
* The engine port has no password, so any program on your PC can send it commands. That's how UAD's engine works and isn't specific to this project. To confirm it's local only, run `netstat -an | findstr 4710` while Console is open. You want to see `127.0.0.1:4710` or `\[::1]:4710`.
* Other parts of the preamp (48V, Pad, HiZ, Low Cut, Phase) live on the same path, but this program doesn't touch them. Be careful if you experiment, because switching 48V on with the wrong mic or cable can cause damage.
* This code was written with AI assistance and has had limited testing. Issues and pull requests are welcome.

## License

[MIT](LICENSE)

