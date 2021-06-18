---
layout: post
hero-bg-color: "#000"
title: "How to get the Asus mic mute key working in Linux on Zephyrus G14 (2021)"
date: 2021-06-15
category: Misc
subtitle: "Get the Mic Mute key working on the Asus Zephyrus series laptops running linux!"
thumb: "g14-2021.png"
summary: "The mic mute key that ships with the Asus Zephyrus G14 functions normally on Windows, but on Linux it is not recognised by most distros. The fix listed on the Asus-Linux’s FAQ page did not work for me after multiple tries, so I had to dig around a bit and finally found a way using acpi. It took a while, but the mic mute key is finally working normally on Zephyrus G14 2021 loaded with Kubuntu 21.04!"

# github:
# worktype: ""
# progress: 100
---

## Preamble

The mic mute key that ships with the Asus Zephyrus G14 functions normally on Windows, but on Linux it is not recognised by most distros. The fix listed on the [Asus-Linux's FAQ page](https://asus-linux.org/faq/#mic-mute-doesn-t-work){:target="blank"} did not work for me after multiple tries, so I had to dig around a bit and finally found a way using `acpi`. It took a while, but the mic mute key is finally working normally on Zephyrus G14 2021 loaded with Kubuntu 21.04!

Read on to get it working yourself as well! I have included all the scripts I used towards the end as well.

## Update [19-06-2021]

After some more hours of digging through the internet, I managed to find a way to find out the correct device identification for the version of Asus N-key keyboard installed on 2021 G14s, allowing for the `hwdb` method listed on the Asus-Linux's website to work.

**So, if you have a 2021 Rog Zephyrus G14, all you need to do is this**:

1. Create a file named `/etc/udev/hwdb.d/90-nkey.hwdb` with: 
   ```bash
   evdev:input:b0003v0B05p19B6*
     KEYBOARD_KEY_ff31007c=f20 # x11 mic-mute
   ```
2. Then update hwdb with:

   ```bash
   sudo systemd-hwdb update
   sudo udevadm trigger
   ```

This should immediately make the mic mute key usable. The same device identification number (the part after evdev:) might also apply to 2021 G15. 

### If the above does not work for you:

You will need to do the following to find out the device identification number for your device: 

> *Adapted from https://askubuntu.com/a/743291/904665*

1. Run `usb-devices` on a terminal. This will output a large chunk of text. Look for something that says `Product=N-KEY Device`.  The output should look something like this:

   ![image-20210619012303999](/assets/n-key.png)

   Note the vendor ID. For example, in the above screenshot, the vendor ID is `0b05`.

2. Run `sudo find /sys -name *modalias | xargs grep -i 0b05`. Replace `0b05` with whatever you got. This output should be fairly large as well. 

   ![image-20210619012911830](/assets/modalias.png)

3. Now, we are looking for a line that goes something like `xxx:modalias:input:xxx`. From the above output, we can spot two such lines:

   `.../input/input25/modalias:input:b0003v0B05p19B6e0110-e0...`

   `.../input/input24/modalias:input:b0003v0B05p19B6e0110-e0...`

   You will notice the part after `input:` is the same in both. This part is exactly what we need. Even though, the part after `input:` is exactly same for me, it is not necessary that all of it will be same for everyone, so no need to worry if you got it slightly different. Some of it is always going to be same though (more on that [here](https://wiki.archlinux.org/title/Map_scancodes_to_keycodes#:~:text=Generic%20input%20devices,the%20device%20capabilities.)).

4. Grab the part after `input:` and just before the first `e` from the previous output. This is the part we'll need. In the above output, that part is: `b0003v0B05p19B6`.

5. Now, create a file named `/etc/udev/hwdb.d/90-nkey.hwdb` with: 

   ```bash
   evdev:input:b0003v0B05p19B6*
     KEYBOARD_KEY_ff31007c=f20 # x11 mic-mute
   ```

   Make sure to replace `b0003v0B05p19B6` with what you got from step 3.

6. Then update hwdb with:

   ```bash
   sudo systemd-hwdb update
   sudo udevadm trigger
   ```

This should immediately make the mic mute key usable without a system restart.

If even this does not work, there is yet another, somewhat "hacky" option for you:

## Fallback Option

These are the steps you need to follow:

1. Figure out the key event for the mic mute key. Execute `acpi_listen` on the terminal and press the mic mute key. The event identifier will be printed on the terminal. If nothing is printed on the terminal when you press the mic mute key, this option will not work you.

|     ![image-20210615153633960](/assets/acpi-listen.png)      |
| :----------------------------------------------------------: |
| <font size="2">It was <i>button/micmute MICMUTE 00000080 00000000 K</i> for me. It will probably be the same for everyone.</font> |

2. Configure PulseAudio to run system-wide using [this helpful blog](https://fhackts.wordpress.com/2017/07/01/running-pulseaudio-system-wide-with-pacmd-on-arch/){:target="blank"} (reason is explained at the end). Restart the system.

3. (**Optional**) For some reason, after following the last step and restarting my system, my user got removed from the `sudoers` list and I couldn't use `sudo` anymore. If this happens with you, follow [this blog](https://www.tecmint.com/fix-user-is-not-in-the-sudoers-file-the-incident-will-be-reported-ubuntu/){:target="blank"} and it should be fixed.

4. Once this is done, run `sudo PULSE_RUNTIME_PATH=/var/run/pulse -u pulse pacmd list-sources | grep index` . This will give you a list of indices for all the input sources connected to your system. If you want to use a USB microphone or something else, attach that too so that you can note all the indices.

5. For me, the indices were 0, 1 and 2. Create a file at `/etc/acpi/asus-mutemic.sh` and put the following bit of code inside it. As you can observe there is a recurring piece of code for every index. Add more indices or reduce the existing ones as per your setup. This script will mute *all* these indices when the mute mic button is pressed. 

    <script src="https://gist.github.com/IAmSuyogJadhav/3b265936e4f8fb8e144802d4dcf086e6.js"></script>
    
    Make the script executable: `sudo chmod +x /etc/acpi/asus-mutemic.sh`

6. Create another file  at `/etc/acpi/events/asus-mutemic` and put the following code inside it:

    <script src="https://gist.github.com/IAmSuyogJadhav/2b954ebfd052c21fd1e408311845dd6e.js"></script>
    This file will be responsible for listening to the keypress event and invoking the script we wrote earlier. Note that, if you got a different event code in the first step, put that after `event=` in the above file.

7. Finally, restart acpid: `sudo /etc/init.d/acpid reload`. You may need to restart the computer if the key does not start working right away.

&#x200B;

|      ![img](/assets/qcq6ozh268571.gif){:width="250"}      |
| :-------------------------------------------------------: |
| <font size="2">A GIF showing the script in action.</font> |

### Scripts

- `/etc/acpi/asus-mutemic.sh`: [Github Gist](https://gist.github.com/IAmSuyogJadhav/3b265936e4f8fb8e144802d4dcf086e6){:target="blank"}
- `/etc/acpi/events/asus-mutemic`: [Github Gist](https://gist.github.com/IAmSuyogJadhav/2b954ebfd052c21fd1e408311845dd6e){:target="blank"}

### Also note

- The step 2 is needed because the action triggered by acpi events run as the root user, and the way PulseAudio is setup by default, it does not allow anyone (not even root) to modify the PulseAudio settings. The PulseAudio system mode was made for making that possible.
- In the step 5, the commented-out line of code was meant to automate the process of gathering indices and muting all of them, but that didn't work for some unknown reason. If you figure out why, please let me know in the comments.
- Note that doing this makes it so that you won't be able to adjust the volume through PulseAudio's volume control GUI. I am pretty sure it will require some additional config from the user's end to get it functioning, but I am using Plasma DE which has a nice little applet for volume control that functions well despite this so it didn't create a big issue for me. If you absolutely need to use the `pavucontrol` GUI, there might be some tweaking required get the GUI functioning. 

## Footnotes

And that should be it! Do let me know if any of these options work for you. If you know an even better way of doing this, be sure advise in the comments below. Also, getting the device identification numbers is a slightly tricky process many people new to linux might struggle with. If you happen to find the same for your device, kindly leave it in the comments below along with your model name and year so that others can benefit from it.

Peace!

> *The thumbnail is borrowed from Asus' official product description page for Zephyrus G14 (2021) available [here](https://rog.asus.com/laptops/rog-zephyrus/2021-rog-zephyrus-g14-series/){:target="blank"}.*
