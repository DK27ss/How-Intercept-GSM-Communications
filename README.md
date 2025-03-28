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

I recommend the [fork](https://github.com/bkerler/gr-gsm) for `GNU RADIO` versions `3.10-3.11`. I've encountered incompatibility problems, but the fork here works.

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

Location Area Code `(LAC)` is used to group cells belonging to the same area, in order to optimize signaling. The Mobile Country Code `(MCC)` corresponds to the country code. In France, it is always `208`. The Mobile Network Code `(MNC)` identifies an operator's network. In the GSM band, `Orange` uses MNCs `1` and `2`, `SFR` uses MNCs `10` and `13`, `Free` uses MNC `15` and `Bouygues Telecom` uses MNCs `20` and `88`.


    
