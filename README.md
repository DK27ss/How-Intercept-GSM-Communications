# How-Intercept-GSM-Communications

![photo-1533664488202-6af66d26c44a](https://github.com/user-attachments/assets/efd149bb-99e4-4879-9503-fdbe5cb9f10c)

## Overview

Following on from my post on listening for ATC signals, I thought it would be interesting to also share with you this post on intercepting GSM signals.

Let me remind you that intercepting and decoding GSM signals that don't belong to you is strictly forbidden and illegal.

In mobile telephony, the equipment used by network users is called Mobile Station `(MS)`, and the equipment responsible for communicating directly with the `MS` is called Base Transceiver Station `(BTS)`.

In Europe, GSM communications are carried out in the `900 MHz` and `1800 MHz` bands. On the `900 MHz` band, in fact, they use frequencies between `880 MHz` and `915 MHz` for the uplink, i.e. from the MS to the BTS, and frequencies between `925 MHz` and `960 MHz` for the downlink, i.e. from the `BTS` to the `MS`.

Each of these two `35 MHz` bands is divided into `174 200 kHz` channels. They are called Absolute Radio-Frequency Channel Number `(ARFCN)` and, for the `900 MHz` band, are numbered from `0` to `125` and from `975` to `1023`.

The other `ARFCNs` (between 125 and 974) are not used, or are used in other frequency bands (GSM 450, GSM 480 and DSC 1800).

The `174 channels` each have eight time slots `(TS)` of around `577 μs`. They are numbered from zero to seven. These are the physical channels used to transmit voice or signaling. They contain data in three categories:

    Traffic CHannels (TCHs) 
    Full rate Traffic CHannel (TCH/F) 
    Half rate Traffic CHannel (TCH/H) 
    Dedicated Control CHannels (DCCHs) 
    Standalone Dedicated Control CHannel (SDCCH) 
    Fast Associated Control CHannel (FACCH) 
    Slow Associated Control CHannel (SACCH) 
    Common Control CHannels (CCCHs) 
    Broadcast Control CHannel (BCCH) 
    Access Grant CHannel (AGCH) 
    Random Access CHannel (RACH) 
    Paging CHannel (PCH) 
    Synchronization CHannel (SCH)
    Frequency Correction CHannel (FCCH)

To guarantee confidentiality, communications are normally encrypted. The `A5/1`, developed in 1987, is supposed to fulfill this role. In 2009, it was broken using resources accessible to all. In 1989, the `A5/2` was developed for countries where the law did not allow the use of the `A5/1`. Due to its weakness, it is not used. Released in 1999, the `A5/3`, used for `3G communications`, is intended to replace the `A5/1`. Although theoretical attacks on the `A5/3` exist, in practice they are not significant. In fact, in France, the `A5/1` is still in the majority.

I recommend installing the `DragonOS Focal` distribution, which simplifies the work with most of the tools already installed.

I recommend the gr-gsm [fork](https://github.com/bkerler/gr-gsm) for `GNU RADIO` versions `3.10-3.11`. I've encountered incompatibility problems, but the fork here works.

    git clone https://github.com/bkerler/gr-gsm
    cd gr-gsm
    mkdir build
    cd build
    cmake ..
    mkdir $HOME/.grc_gnuradio/ $HOME/.gnuradio/
    make

Once `gr-gsm` has been installed, simply browse the directories to launch the various tools:

    grgsm_scanner 
    grgsm_livemon 
    grgsm_channelize.py 
    grgsm_capture.py 
    grgsm_decode

Make sure your RF receiver is correctly configured, in the case of a `HackRF` we can use the command `hackrf_info`.

![hackrf_infp](https://github.com/user-attachments/assets/86776db4-cc6c-42fb-b36a-fe76ad0f392c)

Location Area Code `(LAC)` is used to group cells belonging to the same area, in order to optimize signaling. The Mobile Country Code `(MCC)` corresponds to the country code. In France, it is always `208`. The Mobile Network Code `(MNC)` identifies an operator's network. In the GSM band, `Orange` uses MNCs `1` and `2`, `SFR` uses MNCs `10` and `13`, `Free` uses MNC `15` and `Bouygues Telecom` uses MNCs `20` and `88`.

We can now use `grgsm_scanner` to analyze the various cells around us 

![grgsm_scanner](https://github.com/user-attachments/assets/e1cfe869-4766-413f-ab2f-4ef2a15d4eca)

You can also be more precise with the following command

    grgsm_scanner -s 2e6 -p 0 -g 50 --speed=3 -v

// Syntax

    sample rate with -s or --samp-rate=. The default value is 2000000.
    frequency correction with -p or --ppm=. The default value is zero. It can be used to compensate for receiver hardware faults.
    gain with -g or --gain=. The default value is 24, which can be increased in the event of poor signal reception.
    the speed at which the software scans with --speed= between zero and five. The default value is four. Zero corresponds to the slowest speed and five to the fastest.

This lets us know which `ARFCNs` or frequencies are being used by surrounding `BTSs`, and thus which frequency(s) it's worth trying to capture.

![found_ccch](https://github.com/user-attachments/assets/c01582a1-465c-4672-96ed-68d407a6fb1a)

Even without recognition, it is still possible to use freq-by-freq to find the frequencies at which data is exchanged.

Once this has been done, we can move on to using `grgsm_livemon`

Next, open `wireshark` with the following command, or manually if you have problems opening it in `CLI`

    sudo wireshark-gtk -k -f udp -Y gsmtap -i lo &
    
![run_wireshark_cli](https://github.com/user-attachments/assets/3a5d125e-a832-4620-bbe9-4b8e22329d78)

This will enable you to check that data is being intercepted. It can also be used if you need to search for frequencies manually to validate correct reception.

https://github.com/user-attachments/assets/1bd87b6c-6e73-40ef-9788-73c48bbe1dd8

