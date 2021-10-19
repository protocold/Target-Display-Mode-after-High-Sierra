Hey guys, I have some good news regarding Target Display Mode (TDM).



I have successfully triggered TDM manually under newer MacOS (tested in Big Sur). This is only for iMac 27" 2010 (iMac 11,3) and perhaps 2009(untested) [2011 wont work. Please read note 7 below].



This is made possible because of prior work done by:

A) Reverse engineering by Florian Echtler (which I am not going to repeat here):

https://floe.butterbrot.org/matrix/hacking/tdm/

B) SMC read/write utility under Hendrik Holtmann's smcFanControl project
https://github.com/hholtmann/smcFanControl/tree/master/smc-command



In summary, these are my findings:

1) under High Sierra(or older), TDM is handled by Display Port Daemon (dpd) which listen to Cmd F2 key pressing to trigger TDM. It is unclear what dpd completely does but at the minimum it writes 2 keys to SMC which triggers the TDM switching and launches dpaudiothru which handle the audio routing.

https://www.unix.com/man-page/osx/8/dpd/



2) Florian figured out:

- To turn on TDM, all we need is to write 2 SMC keys: 1 to MVHR, sleep for 1 second, then write 2 to MVMR.

- To turn off TDM, all we need is to write 2 SMC keys: 0 to MVHR, sleep for 1 second, then write 2 to MVMR.



3) It seems like this requires no additional software as Florian did this under linux and as soon as the keys are written into the SMC the switching will occur.



4) However Florian's util in his github only works for linux but luckily I've found out the util from Hendrik can do the same job under Mac OS



5) so this is what I did(tested on both High Sierra and Big Sur, must run smc util as root or else it wont work):

- To turn on TDM:

    a) sudo ./smc -k MVHR -w 01

    b) wait 1 second

    c) sudo ./smc -k MVMR -w 02



- To turn off TDM:

    a) sudo ./smc -k MVHR -w 00

    b) wait 1 second

    c) sudo ./smc -k MVMR -w 02



But this only handled the display, audio routing didn't work as dpaudiothru was not loaded as dpd was completely bypassed here.



6) Now, I think I have only solved part of the puzzle from this research and testing. We all know CMD-F2 doesnt work in newer OS to trigger TDM. In order to complete this exercise, I am going to need all the possible help from you guys:

  a) dpd and dpaudiothru actually exist in all the Mac OS version (e.g. Catalina, Big Sur etc etc). For some reason on newer Mac OS dpd doesnt respond to Cmd F2 (maybe it wasnt loaded at all.. how do we find out?!)

  b) I tried to run dpd manually but it would just quit and exit without doing the switching under Big Sur. I also tried to run dpaudiothru but it thrown errors. I have no idea on how to trace this.

  c) even tho dpd/dpaudiothru existed in the newer MacOS version but we dont know if these binary are doing the same thing as those found under High Sierra (or older).

  d) maybe there is an alternative way that can intercept Cmd-F2 and then run some script to call smc util? We still need to figure out how to do the audio routing though but this could be another direction to achieve the same end result.



7) This only works for 2010 (and possibly 2009) as it supports display port only and finding from this post suggests TDM is handled by Thunderbolt on the 2011 so additional and separate stream of research required because 2011 will not respond to the 2 smc keys:

https://github.com/floe/smc_util/issues/8
