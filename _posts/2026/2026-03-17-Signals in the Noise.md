---
title: "Signals in the Noise"
layout: post
categories: [writeups, CTF]
description: Write-up for the TCM's CTF Project Helix.
image:
  path: /assets/2026/tcmctf/helix.png
  alt: helix
---

TCM created a CTF named Project Helix that "_takes you on a mission to recover deleted notes and identify the RNA based key required to unlock a missing geneticist’s encrypted specimen archive_". This sounded like a fun challenge, so I decided to give it a try. This is a write up of how I solved the CTF.

## Project Helix: CTF Scenario

Three days ago, lead geneticist Dr. Owens sent a final frantic transmission claiming he had intercepted a "biological broadcast." He described it as a repeating signal he believed to be a synthetic RNA sequence manifesting as radio frequency interference.

In violation of Protocol 4, Owens began unauthorized testing on a high-risk specimen. All communications stopped soon after. A security sweep of Owen's laboratory confirmed he is missing. His local workstation was found powered on, but all of his research has been deleted. Internal sensors indicate the "broadcast" is still active on a localized frequency, but the specific coordinates were deleted along with Owen's research notes.

You have been provided a forensic triage image of Owens' workstation. Your objective is to recover the deleted notes and identify the RNA-based key required to unlock his encrypted specimen archive.


## Initial Triage
The challenges start with downloading a KAPE triage. I used my [bash script](https://www.mennovanveenendaal.com/posts/Forensic-Investigation-setup-script/) to process the data into csv files and began searching.

## Finding Deleted Artifacts
Since the scenario mentioned notes, I started by looking at Prefetch, Link files, Jump Lists, and Amcache output. In the Link files I found an entry for `freq.lnk` pointing to `\Downloads\freq.txt.`. The other artifacts did not contain relevant data.

Next, I reviewed the output from MFTECmd. Because the scenario mentioned deleted files, I filtered the `In Use` column to off and file extension to `.txt`. This returned one entry: `.\Users\drowens\Downloads\freq.txt`.

I then removed the file extension filter and searched for File Name entries containing `freq` with `In Use` off. This revealed a second file: `freq.txt:Zone.Identifier`, with the following Zone Id Contents:
```
freq.txt:Zone.Identifier
\[ZoneTransfer]
ZoneId=3
HostUrl=https://ctf.tcmsecurity.com/3c7d7997c1a7/freq.txt
```

## Recovered Notes
Navigating to this Url gave me the following page:

```txt
---------------------------------------

LOG_086:
Found a hum. It's too constant, an- did I just discover the next WOW signal??     Not a machine. Too rhythmic. Too... wet. Like a heartbeat in the wires.

---------------------------------------

LOG_087:
It's getting louder... at first it would go days with silence. but now it repeats. |it repeats|it repeats|it repeats|it repeats|it repeats|

It seems to be ... every 78 seconds.

---------------------------------------

LOG_088:
It speaks in strands. A four-fold tongue. I saw it on the waterfall today. tHEY are sending a message... quaternary? every pulse is part of a base... A...C...G...U... but what is the charcode? They called me CRAZY!

If I start counting from 0 it starts to make sense.

---------------------------------------

LOG_089:
Black van in the driveway again. Tinted. Staring.... They know what i found. they know i can decode it. It's not safe here. Deleting it all. Pulling the copper out of the walls. THEY ARE TRANSCRIBING ME. STAY OFF THE LADDER.

---------------------------------------

1000.0 (X)
1000.1 (X)
1000.2 (X)
1000.3 (X)
1000.4 (X)
[...]
4999.7 (X)
4999.8 (X)
4999.9 (X)
```

## Exploring the Signal Source

Removing /freq.txt from the URL led to the main page.

![View](/assets/2026/tcmctf/radio.png)
_Fig.1 Radio Page_

Clicking the "Acquire Specimen" button downloaded a file named flag.zip, which was password protected. The password was not `ACGU`.

The tuning controls adjusted the spectrum and volume. When the power button was pressed, radio static could be heard.

Clicking the REC button started a recording. Pressing it again downloaded a file named `SIG_INT_[UNIX_TIMESTAMP].webm`.

Leaving the REC button active for approximately 2.35 minutes also triggered a download. The audiofile contained the same radio static, and the metadata did not provide additional clues.

### Identifying the Correct Frequency
Looking at the freq.txt file, the number sequences appeared to represent frequencies. Assuming all tested frequencies were marked with `(X)`, I searched for lines without this marker using the regex `^((?!(X)).)*$`. This returned one result: `3817.3 (?)`.

The Radio Freq button did not allow acquired input to this frequency, so I wanted to changed the HTML to this. In the browser console I found the function `Use RADIO_SYSTEM.update(freq) for direct tuning.` and used it to set the frequency to 3817.3. After enabling the radio, the static now included a repeating sequence of tones.

![RADIO_SYSTEM.update](/assets/2026/tcmctf/freq.png)
_Fig.2 RADIO_SYSTEM.update_

I recorded the signal again. Based on the logs, the signal repeated every 78 seconds, so I captured a full cycle.

I clicked `rec` again and waited for the download of the .webm file to start. Based on the logs, the signal repeated every 78 seconds, with a full audio file of 2.35 minutes I would capture that sequence.

## Extracting the Signal
Listening to the recording revealed four distinct tones within the static. To extract these I used ChatGPT to Vibe code a Python script to processes the audio the audio. First, I converted the file from webm to wav:
```
ffmpeg -i SIG_INT_1773410816575.webm -ac 1 -ar 44100  SIG_INT_1773410816575.wav
```

```
$ python3 spectrogram_decoder.py SIG_INT_1773410816575.wav

Detected tone centers (Hz):
[ 997.98114483 2002.58789062 2994.83789062 4005.17578125]

Symbol stream:
[2, 1, 3, 0, 2, 1, 2, 0, 1, 3, 0, 3, 1, 2, 1, 0, 3, 1, 2, 3, 0, 1, 2, 1, 3, 0, 1, 3, 0, 1, 3, 0, 1, 0, 2, 1, 0, 3, 2, 1, 0, 1, 0, 1, 2, 3, 1, 2, 3, 0, 1, 3, 2, 1, 2, 3, 1, 2, 1, 3, 0, 2, 1, 2, 0, 1, 3, 0, 3, 1, 2, 1, 0, 3, 1, 2, 3, 0, 1, 2, 1, 3, 0, 1, 3, 0, 1, 3, 0, 1, 0, 2, 1, 0, 3, 2, 1, 0, 1, 0, 1, 2, 3, 1, 2, 3, 0, 1, 3, 2, 1, 2, 3, 1, 2]
```

To verify the results, I loaded the wav file into Sonic Visualiser and inspected the spectrogram. The script output did not fully match, as repeated tones were not always captured correctly by the script.

![Spectrogram](/assets/2026/tcmctf/spectrogram.png)
_Fig.3 Spectrogram_

I then used Claude to extract the tones, which resulted in a sequence such as `CGGCCUAACUUACUUA[...]`, and this matched the Spectrogram. Claude also suggested a password, `eraseSlip||RNAPolym`. However, this password was incorrect and lacked explanation.

## Interpreting the Sequence
I explored RNA encoding further to understand how the sequence could be decoded.

*The second and third N-terminal helices of CNOT3 contact the arginyl-tRNA D-loop and D-stem, while a short element named the tRNA clamp motif (tCM)*
https://pmc.ncbi.nlm.nih.gov/articles/PMC11583848/

This led to several unsuccessful attempts, including converting the sequence into hexadecimal values and experimenting with different decoding approaches. None produced meaningful results.

To confirm the sequence of tones, I recorded a second sample on the radio webpage and compared it again using Sonic Visualiser. Using Claude I extracted the complete tone sequence from this second audio file and checked this with the Spectrogram in Sonic Visualiser. 

The same shorter sequence was present in both audio files.

## Decoding the Key
While researching RNA based encoding, I found a write up of another [CTF](https://ctftime.org/writeup/19105)that used DNA encoding techniques. 
```
mappings: A(0) – 00 | T(1) – 01 | C(2) – 10 | G(3) – 11` and `mapping A(0) – 00 | C(1) – 01 | G(2) – 10 | T(3) – 11
```

Using a similar approach, I mapped the four tones to binary values:

- Lowest frequency 1000Hz = A = `00`
- frequency 2000Hz = C = `01`
- frequency 3000Hz = G = `10`
- Highest frequency 4000Hz = U = `11`

Using notepad++ search and Replace All I converted this the tone sequence from the sound files to binary.
```
01101001011100000111110001111100010100100100111001000001010100000110111101101100011110010110110101100101011100100110000101110011011001010101001101101100
```

Using CyberChef, I converted this binary into ASCII: `ip||RNAPolymeraseSl`.

This closely resembled the earlier password suggestion by Claude. Then the two pipes in the ASCII text and the hint in the log file hit me, `|it repeats|it repeats|it repeats|it repeats|it repeats|`. I tried the password `|RNAPolymeraseSlip|`:

```shell
$ 7z x flag.zip -p'|RNAPolymeraseSlip|'

7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03
 64-bit locale=en_US.UTF-8 Threads:6 OPEN_MAX:1024, ASM

Scanning the drive for archives:
1 file, 231 bytes (1 KiB)

Extracting archive: flag.zip
--
Path = flag.zip
Type = zip
Physical Size = 231

Everything is Ok

Size:       31
Compressed: 231


$ cat flag.txt 
TCM{V01D_S1GN4L_4U7H3N71C473D}
```

This successfully extracted the flag:

![Flag](/assets/2026/tcmctf/flag.png)
_Fig.4 Flag_

## 10-7A
This was a fun and interesting CTF. What started as a forensic investigation quickly expanded into SIGINT analysis, data decryption, and even elements of biology.

AI was helpful during the process, but its output needed to be verified. I prefer to understand the solution rather than rely on AI to solve the challenge. In this case, the incorrect password suggestion actually made me try harder.

Looking back, the hints were clear and useful. I should have paid closer attention to them earlier in the process.