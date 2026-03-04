## Summary

**Challenge:** Shells Bells CLI  
**Level:** Intermediate  
**Skills:** Linux system enumeration, Shell & process environment analysis, Git forensic recovery, cryptographic file handling (OpenSSL, GPG), binary inspection (xxd, file), steganographic artifact detection, adversarial problem solving  
**Platform:** TryHackMe

## Overview

Last December, TryHackMe hosted a holiday-themed series of introductory challenges designed to help people get familiar with broad-strokes cybersecurity concepts. 

Predictably, there were more difficult challenges hidden away for the curious and determined to find, and this document is an overview of one such task.

**!!! THIS WRITEUP CONTAINS NO FLAGS !!!**

## Context

The narrative of the TryHackMe holiday challenges was centered around malicious bunnies attempting to turn the Christmas holidays into "Easter 2.0". One "rebel" bunny by the name of McSkidy had found evidence of this evil plot and left a series of clues behind in a compromised machine to help blue team save Christmas. 

Within the Shells Bells challenge, McSkidy's clues can be combined into a secret key, and the hunter (i.e. you) must use their sleuthing and cyber skills to unravel the mystery of what he was trying to communicate via the files he hid in the system.

## Writeup

### Known Information 

Through the Beginner-level Shells Bells challenge, we already have `sudo` access to the compromised machine and have undone some of the work of the evil bunny plotters.

The machine itself has several user accounts:
- `mcskidy`
- `eddi_knapp`
- `alice` (or something, I don't quite remember, irrelevant to this challenge)

The user <code>mcskidy</code> has a <code>readme</code> sitting in their <code>Documents</code> folder. Let's open it up.

```bash
From: mcskidy  
To: whoever finds this

I had a short second when no one was watching. I used it.

I've managed to plant a few clues around the account.
If you can get into the user below and look carefully,
those three little "easter eggs" will combine into a passcode
that unlocks a further message that I encrypted in the
/home/eddi_knapp/Documents/ directory.
I didn't want the wrong eyes to see it.

Access the user account:
username: eddi_knapp
password: S0mething1Sc0ming

There are three hidden easter eggs.
They combine to form the passcode to open my encrypted vault.

Clues (one for each egg):

1) I ride with your session, not with your chest of files.
Open the little bag your shell carries when you arrive.

2) The tree shows today; the rings remember yesterday.
Read the ledger’s older pages.

3) When pixels sleep, their tails sometimes whisper plain words.
Listen to the tail.

Find the fragments, join them in order, and use the resulting passcode
to decrypt the message I left. Be careful — I had to be quick,
and I left only enough to get help.
```

### First Steps

Broadly speaking, then, we're looking for:
- something shell/session-related (thematically appropriate for "Shells Bells")
- some form of archive or log
- an image with the flag appended to the end

The first day that I sat down and tackled this, the hints were slightly different and considerably more vague, which led me on a goose chase that I absolutely didn't need to go on through the bowels of `root` with my good ol' friend `grep`.

e.g.:

```bash
zgrep -R "frag" /var/log
zgrep -R "pass" /var/log
strings -a -n 4 file.bin | egrep -o 'FLAG\{[^}]+\}'
# ChatGPT also advised that I search for files with hex offsets and UTC-16 but my "effort" gut check started going off at that point.

grep -R -i --line-number "PASSFRAG" /home 2>/dev/null
# One of the searches I used later to come to the conclusion that the keys weren't grep-able.
```

While I was initially concerned that this was a total waste of time, **critically**, I did find a few interesting things in `.viminfo`:

```bash
# File marks: '0 178 0 ~/fix_passfrag.sh |4,48,178,0,1762878235,"~/fix_passfrag.sh" '1 178 0 /etc/fix_passfrag.sh |4,49,178,0,1762878202,"/etc/fix_passfrag.sh" '2 1 0 /home/socmas/2025/wishlist.txt |4,50,1,0,1762863956,"/home/socmas/2025/wishlist.txt" '3 144 0 ~/Documents/create_red_herrings.sh |4,51,144,0,1760120133,"~/Documents/create_red_herrings.sh" '4 13 36 ~/Documents/encrypted_note.txt |4,52,13,36,1760119877,"~/Documents/encrypted_note.txt"
```

Obviously, the scripts no longer existed, but this is still a big find.

**The Takeaways:** 
- There's a very high probability that the flags here are **not** in any way "grep-able". 
- There's at least one red herring somewhere! 
- We'll be working with a few encrypted files

A day after the challenge was released, McSkidy's note was updated to point the user specifically to `/home/eddi_knapp/Documents`. That streamlined things considerably.

### Flag 1

First things first, a classic check:

```bash
eddi_knapp@tbfc-web01:~$ ls
Desktop    Music     Templates                            wget-log
Documents  Pictures  Videos
Downloads  Public    fix_passfrag_backups_20251111162432
```
That `wget-log` reminded me that we can check the user's bash history, their `.profile`, and all sorts of things:

- `file.txt~`
- `.bash_history`
- `.viminfo` (already checked while log-scanning above)
- `./local/share/.../history*`
- `.swp.` 

Rather unexpectedly, `.bash_history` gives us something interesting:

```bash
# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
export PASSFRAG1="FLAG-NUMERO-UNO"
```

Flag 1 is out of the way! This is the obvious answer to **Hint Number 1** since this would be loaded when the shell is initiated by the user. 

**Lesson Learned:** As an embarrassing side note, this would also have been immediately visible had I just run `printenv`.

On to Flag 2!

### Further Exploration

We found the note in the McSkidy's `Documents` folder, so let's check some of `eddi_knapp`'s other "main" folders:

- `Desktop`: nothing of note
- `Pictures`: a whole pile of different photos
- `Videos`: empty (no hidden files either)
- `Downloads`: a mysterious text file! What's it say?

```bash
image download list:
- wallpaper_spring.png
- family_holiday.jpg
- profile_pic.png
- scenery_01.png
```

Could this be a pointer to Hint Number 3?

TL:DR - no.

### Red Herring 1

In the interests of 1) keeping my writeup as true-to-life as possible and 2) being thorough regarding the work I did, I'll briefly detail the methods and tools I used to come to the conclusion that I had indeed been red herring'd. 

If you'd rather skip straight to what was ultimately relevant to Flag 2, click [here](#flag-2).

As "luck" would have it, Eddi's `Documents` folder also had a note!

```bash
eddi_knapp@tbfc-web01:~/Documents$ ls
mcskidy_note.txt.gpg  notes_on_photos.txt
eddi_knapp@tbfc-web01:~/Documents$ cat notes_on_photos.txt 
Photo notes:
- backup all images weekly
- sync with phone when connected
- organize into 3 folders per year
```
We now have two notes that are pointing us toward photos, ostensibly the long list of them saved in the `Pictures` folder. Weird, but I have to be thorough.

So I headed back into `Pictures` to check those files out.

I used variations on `xxd image.png | head (and tail) -n 5` to confirm that the initial and final values for the images are all fine. 

Hm. 

I know there's a red herring lying around, could this be it?

One photo - `family_holiday.jpg` - has PNG hex values but otherwise nothing remarkable - Herring Lightbulb 1. 

Herring Lightbulb 2, `binwalk` and `exiftool` aren't on the machine, which means they aren't needed and I'm likely trying to dig too hard into image forensics to try and find a key.

The first flag came from a single, correctly executed command, and unlike in other CTFs I've done, there's no evidence here that there's extra information in these images that's been buried or sequentially encrypted.

Herring Lightbulb 3 - I could, feasibly, simply go up a few levels, run a combination xxd / grep search for anything flag-related, and "brute force" the key that way, but that also seems to contradict the "style" of this CTF.

Remember - it'd be very odd if the designers of this challenge just made the flag findable by grep search. This is **InTeRmEdIaTE** after all, and that mentality didn't work before.

Let's put a pin in the pictures for now and go up a level, back to home.

### Red Herring 2

We need to rerun the `ls` command and make sure we're actually seeing everything.

```bash
drwxrwxr-x  2 eddi_knapp eddi_knapp 4.0K Dec  1 08:32 .secret
drwx------  3 eddi_knapp eddi_knapp 4.0K Nov 11 12:07 .secret_git
drwx------  3 eddi_knapp eddi_knapp 4.0K Oct  9 17:20 .secret_git.bak
```

Well, well, well, what do we have here?

`.secret` contains a GPG-encrypted `.tar` file that I'm not able to decrypt. 

**Lesson Learned:** For CTFs, always run `-h` to show hidden files.

The private key folder in our `.gnupg` directory is EMPTY - which likely means that they removed the keys so I have to go herring-hunting once again.

A `.gpg` file can be decrypted if the key is right, so let's see what key we have:

```bash
eddi_knapp@tbfc-web01:~/Documents$ gpg -d mcskidy_note.txt.gpg 
gpg: AES256.CFB encrypted data
gpg: problem with the agent: Permission denied
gpg: encrypted with 1 passphrase
gpg: decryption failed: Bad session key
```
Mwop mwop, but not unexpected - we need the key. 

After quite a bit of chatting with ChatGPT and searching, I once again came to the conclusion that it'd be quite strange to have a genuine key that's easily searchable (by file format, for example).

Particularly so when there was one more place I could look into that was sitting right in front of me.

### Flag 2

Incidentally, ChatGPT was a very helpful tool for this challenge! It hallucinated like no one's business, of course, but it did give me a sense of what sorts of things I *could* be searching for based on my progress, and for someone with my limited experience, that was huge.

In this case, it reminded me that the `.secret_git` folder could have a commit history tucked away, and it did!

Specifically, the `.git_secret` folder had a `COMMITMSG` that shows there was an obviously secret text in the previous commits:

```bash
Changes to be committed: # deleted: secret_note.txt
```

So, let's restore it. Comments are mine!

```bash
# Git refuses to operate as root in a repo owned by another user unless you tell it the repo is safe!
root@tbfc-web01:/home/eddi_knapp/.secret_git$ git config --global --add safe.directory /home/eddi_knapp/.secret_git

# We need to see the commit hashes in order to review specific ones
root@tbfc-web01:/home/eddi_knapp/.secret_git$ git log --all --oneline --graph
* e924698 (HEAD -> master) remove sensitive note
* d12875c add private note

root@tbfc-web01:/home/eddi_knapp/.secret_git$ git ls-tree --name-only d12875c
secret_note.txt

# show the git log output
root@tbfc-web01:/home/eddi_knapp/.secret_git$ git show d12875c:secret_note.txt
========================================
Private note from McSkidy
========================================
We hid things to buy time.
PASSFRAG2: "FLAG_NUMERO_DOS"
```

**Lesson Learned:** Check what's in front of your face first. Also, I learned how to review and manipulate git logs!

Let's keep it moving!

## Flag 3

On a hunch, I went back to Eddi's `Pictures` folder and ran another `ls`, this time with `-ahl`.

```bash
-rw-rw-r--  1 eddi_knapp eddi_knapp 1.5K Oct  9 18:07 .easter_egg
-rw-r--r--  1 eddi_knapp eddi_knapp 5.6M Aug 13 18:15 .hidden_pic_1.png
-rw-r--r--  1 eddi_knapp eddi_knapp 5.6M Jul 20 18:15 .hidden_pic_2.png
-rw-r--r--  1 eddi_knapp eddi_knapp 5.6M Sep 11 18:15 .hidden_pic_3.png
-rw-r--r--  1 eddi_knapp eddi_knapp 5.6M Oct  2 18:15 .hidden_pic_4.png
-rw-r--r--  1 eddi_knapp eddi_knapp 5.6M Sep 28 18:15 .hidden_pic_5.png
```

........you've gotta be kidding me.

OKAY THEN, let's keep this quick and easy. 

```bash

root@tbfc-web01:/home/eddi_knapp/Pictures$ xxd .easter_egg | tail -n 4
00000570: 4040 4040 4040 4040 4040 0a0a 7e7e 2048  @@@@@@@@@@..~~ H
00000580: 4150 5059 2045 4153 5445 5220 7e7e 7e0a  APPY EASTER ~~~.
00000590: 5041 5353 4652 4147 333a 2063 304d 316e  PASSFRAG3: "FLAG_
000005a0: 470a                                     NUMERO_TRES"

```

## Cracking the File

After the mother of all facepalms, I assembled my three keys and made my way back to `/home/eddi_knapp/Documents/`, revealing the note's contents:

```bash
Congrats — you found all fragments and reached this file.

Below is the list that should be live on the site. If you replace the contents of
/home/socmas/2025/wishlist.txt with this exact list (one item per line, no numbering),
the site will recognise it and the takeover glitching will stop. Do it — it will save the site.

(Redacted flag info)

A final note — I don't know exactly where they have me, but there are *lots* of eggs and I can smell chocolate in the air. Something big is coming.  — McSkidy

When the wishlist is corrected, the site will show a block of ciphertext. This ciphertext can be decrypted with the following unlock key:

UNLOCK_KEY: (redacted)

To decode the ciphertext, use OpenSSL. For instance, if you copied the ciphertext into a file /tmp/website_output.txt you could decode using the following command:

cat > /tmp/website_output.txt
openssl enc -d -aes-256-cbc -pbkdf2 -iter 200000 -salt -base64 -in /tmp/website_output.txt -out /tmp/decoded_message.txt -pass pass:'(redacted)'
cat /tmp/decoded_message.txt

Sorry to be so convoluted, I couldn't risk making this easy while King Malhare watches. — McSkidy
```

This is now straight config, and it even comes with commands too! The newly restored Christmas website spat out yet another message:

```bash
Well done — the glitch is fixed. Amazing job going the extra mile and saving the site. Take this flag: (redacted)

NEXT STEP:
If you fancy something a little...spicier....use the FLAG you just obtained as the passphrase to unlock:
/home/eddi_knapp/.secret/dir

That hidden directory has been archived and encrypted with the FLAG.
Inside it you'll find the sidequest key.
```
## The Final Flag

This was it - one more flag stood between me and the Hard-level holiday challenge. Within `/home/eddi_knapp/.secret/dir` we get, as expected, an encrypted file.

Accessing that final flag required the following (comments mine again):

```bash

root@tbfc-web01:/home/eddi_knapp/.secret$ gpg --decrypt dir.tar.gz.gpg

# Entered the passkey, got a bunch of binary

root@tbfc-web01:/home/eddi_knapp/.secret$ gpg --decrypt dir.tar.gz.gpg > decrypted.tar.gz

# Let's see if it's real and decompress if it is!

root@tbfc-web01:/home/eddi_knapp/.secret$ file decrypted.tar.gz 
decrypted.tar.gz: gzip compressed data, from Unix, original size modulo 2^32 430080
root@tbfc-web01:/home/eddi_knapp/.secret$ tar -xzvf decrypted.tar.gz 
dir/
dir/sq1.png
```

The PNG, when opened, revealed the final flag.

![An edited picture of an Easter Egg with the CTF flag removed](../../assets/shells-bells/outro.png)

That's all folks! As I wrote in my draft writeup:

We're done here.