# How-Intercept-GSM-Communications

![photo-1533664488202-6af66d26c44a](https://github.com/user-attachments/assets/efd149bb-99e4-4879-9503-fdbe5cb9f10c)

## Overview

Following on from my post on listening for `ATC` signals, I thought it would be interesting to also share with you this post on intercepting `GSM` signals.

Let me remind you that intercepting and decoding `GSM` signals that don't belong to you is strictly forbidden and illegal.

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

## GR-GSM & Dragon

// DragonOS Focal

I recommend installing the `DragonOS Focal` distribution, which simplifies the work with most of the tools already installed.

// GR-GSM Forked for GNU RADIO 3.10-3.11

I recommend the gr-gsm [fork](https://github.com/bkerler/gr-gsm) for `GNU RADIO` versions `3.10-3.11`. I've encountered incompatibility problems, but the fork here works.

    git clone https://github.com/bkerler/gr-gsm
    cd gr-gsm
    mkdir build
    cd build
    cmake ..
    mkdir $HOME/.grc_gnuradio/ $HOME/.gnuradio/
    make

![Capture](https://github.com/user-attachments/assets/4159b9b2-6837-4d70-8e64-43ee380d693d)
![Capture](https://github.com/user-attachments/assets/954e4797-5c52-4ca1-b329-c96ea4938392)

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

Next, open `wireshark`, grgsm_livemon, it sends data in GSMTAP format on the loopback interface on `UDP` port `4729`. So, to see the data, before running `grgsm_decode`, we launch `wireshark` with the following command, or manually if you have problems opening it in `CLI`.

    sudo wireshark -k -f udp -Y gsmtap -i lo &
    
![run_wireshark_cli](https://github.com/user-attachments/assets/3a5d125e-a832-4620-bbe9-4b8e22329d78)

This will enable you to check that data is being intercepted. It can also be used if you need to search for frequencies manually to validate correct reception.

https://github.com/user-attachments/assets/1bd87b6c-6e73-40ef-9788-73c48bbe1dd8

https://github.com/user-attachments/assets/e1a3189a-e07d-4051-975f-107e53540b3e

Congratulations, you've just intercepted GSM communications. :p

We can now analyze the results in `wireshark`

![datas_gsm_lapdm_gsmtap](https://github.com/user-attachments/assets/6f4f2185-9b96-432e-b6ec-1df6898e56d9)
![lapdm](https://github.com/user-attachments/assets/5978c8f1-84e3-4266-a860-d7c69b19ed4d)

We can now identify the two distinct protocols `LAPDm` and `GSMTAP`

There are mainly `Paging Request Type 1` (RR) packets among which an `Immediate Assignment` (RR) packet must be found. This will make it possible to find the `SDCCH` time slot in packets containing a `Channel Description`.

Now let's filter with `lapdm` in wireshark.

You need to find a `Ciphering Mode Command` (RR) packet. This determines the cipher mode `(A5/1 or A5/3)` in `Cipher Mode Setting`. If it's `A5/3`, although `grgsm_decode` supports it, as it can't be `broken`, we won't be able to read the `SMS content` without knowing the key. On the other hand, if it's `A5/1`, we can break the key with `kraken` and read the `SMS content`. The following screenshot shows that, in our case, the key is `A5/1` with an `SDCCH/8` channel.

    grgsm_decode -c capture.cfile -a 5 -m SDCCH8 -t 1

![a51](https://github.com/user-attachments/assets/882d424a-2449-4bf4-86f4-fbb37b80c885)

To decode `SDCCH/8`, on `time slot 1`, even though in my case I didn't even need it, a simple filter in wireshark was enough.

If you want to go further, you can use the following command, but I won't go any further than decrypting GSM communications protocols in this write-up

    grgsm_decode -c capture.cfile -a 5 -m SDCCH8 -t 1 -e 1 -k KEY (Ex: F5C55DB5E6E8B694)

This allows you to search for a `CP-DATA` packet with `gsm_sms` filter and unfold `TP-User-Data` to read the contents of the `SMS` :p

But it is not currently possible to decode the uplink alone with `grgsm_decode`. However, when the `MS` sends an `SMS`, the `BTS` acknowledges it with a `CP-ACK` `SMS` packet on the downlink.

`DTAP` packets are particularly interesting because they contain important information such as `LAI` or `IMSI`.

![imsii](https://github.com/user-attachments/assets/25cd2053-f3a4-4665-a21a-fc4700217761)
![orange](https://github.com/user-attachments/assets/d2c78f32-329f-44d5-898a-5fb10a589394)

these are also the ones we will look for in order to read the decrypted `SMS` with `grgsm_decode`

Open-source tools such as [IMSI-catcher](https://github.com/Oros42/IMSI-catcher), [GsmEvil2](https://github.com/ninjhacks/gsmevil2) and [Airprobe](https://github.com/iamckn/airprobe) can also produce very interesting results. 

A quick look at the results of these tools ^^

## IMSI-Catcher

![imsicatch](https://github.com/user-attachments/assets/8168e20e-8f96-4b68-bb6c-430f60fef75c)

    git clone https://github.com/Oros42/IMSI-catcher
    cd IMSI-catcher
    sudo python3 simple_IMSI-catcher.py -h

`Dragon` saves us time with the prerequisites already natively installed in the `OS`. All we have to do is run our commands.

![requirements](https://github.com/user-attachments/assets/1f75a914-0f0f-4297-b304-6c0993793f5c)

We can use the `-h` command to `list` the `available options`, as with any other `CLI` program.

Now, if we want to run an interception, all we have to do is use the following command with the `-s` argument.

    sudo python3 simple_IMSI-catcher.py -s

We also need to start listening with `grgsm_livemon`

![livemon_2](https://github.com/user-attachments/assets/924da950-0a4d-4431-b928-c6f47dfbb4e0)

https://github.com/user-attachments/assets/a1724378-6bd8-41aa-aecf-aae8555a5e9a

## GSM Evil 2

`Evil2` gives the same information, except that it has a web interface, a `buggy`, with the possibility of intercepting `SMS`. I tested it but no conclusive results. It would be better to refer to `Kraken` or consider write a `PoC`.

    git clone https://github.com/ninjhacks/gsmevil2
    pip3 install -r requirements.txt
    python GsmEvil.py

// Kalibrate (find the gsm frequency on which you capture sms or imsi. Using `kalibrate`, you'll get all your frequencies from nearby gsm base stations.)

    sudo apt install build-essential libtool automake autoconf librtlsdr-dev libfftw3-dev
    git clone https://github.com/steve-m/kalibrate-rtl
    cd kalibrate-rtl
    ./bootstrap && CXXFLAGS='-W -Wall -O3'
    ./configure
    make
    sudo make install

// Quick scan with Kalibrate

    kal -s GSM900

// Output (Example)

    kal: Scanning for GSM-900 base stations.
    GSM-900:
    	chan: 4 (935.8MHz + 320Hz)	power: 1829406.95
    	chan: 11 (937.2MHz + 308Hz)	power: 4540354.88
    ...

Then run with: (also with grgsm_livemon)

    python3 GsmEvil.py --host=localhost 

![evill](https://github.com/user-attachments/assets/f4cab33a-0bce-486f-b314-035361fa3890)

You can then access the web interface on `127.0.0.1`

![webapp](https://github.com/user-attachments/assets/81761556-83c1-4fd4-aca2-374f30c1cad4)

## Some useful resources

https://lemnet.fr/blog/post/interception-passive-sans-imsi-catcher-et-decodage-de-flux-gsm-avec-gr-gsm

https://connect.ed-diamond.com/MISC/mischs-016/interception-passive-et-decodage-de-flux-gsm-avec-gr-gsm

https://github.com/bkerler/gr-gsm

https://osmocom.org/projects/gr-gsm/wiki/Installation

https://github.com/ptrkrysik/gr-gsm/wiki/Usage

https://github.com/joswr1ght/kraken

https://payatu.com/blog/passive-gsm-sniffing-with-software-defined-radio/

https://www.rtl-sdr.com/rtl-sdr-tutorial-analyzing-gsm-with-airprobe-and-wireshark/

https://github.com/Scrut1ny/GSM-Sniffing-Guide

https://github.com/ninjhacks/gsmevil2

https://github.com/Oros42/IMSI-catcher
