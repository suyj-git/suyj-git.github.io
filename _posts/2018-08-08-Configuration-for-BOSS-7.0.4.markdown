---
layout: note
---

*This article is for configuring BOSS v7.0.4, but the analogous instructions can be applied to BOSS of another version.
BOSS of lower version may caouse errors when you are trying example project named `TestRelease`*

# Basical konwledge about BOSS

After you are successfully registered as BES3 member, assuming your name is alice, you will find directories listed blow

* `/afs/ihep.ac.cn/users/alice` as the home directory, whose quota is 500 MB
* `/workfs/bes/alice` for storing algorithm, whose quata is 5 GB
* `/besfs/users/alice` where you usually sumbit jobs, simulate events, reconstruct events and analyze events, whose quota is 50 GB
* `/ihepbatch/besd13/alice` for temporary storage where your filse will be deleted after having left files untouched for 2 weeks, whose quota is 50 GB.
 The same rule applies to `/scratchfs/bes/alice` whose quato is 500 GB.

# Configuration for the environment
Make a directory named `cmthome` in your home directory(in this case, it is `/afs/ihep.ac.cn/users/alice`) 
then 
{%highlight bash%}
cp -r /afs/ihep.ac.cn/bes3/offline/Boss/cmthome/cmthome-7.0.4/ /afs/ihep.ac.cn/users/alice/cmthome/
{%endhightlight%}

You need modify the file `requirements` in the `cmthome-7.0.4` by deleting the symbol `#` ahead of the three lines listed below
{%highlight bash%}
#macro WorkArea "/ihepbatch/bes/maqm/workarea"
#path_remove CMTPATH  "${WorkArea}"
#path_prepend CMTPATH "${WorkArea}"
{%endhighlight%}
and change "/ihepbatch/bes/maqm/workarea" to "/workfs/bes/alice/7.0.4" (It is a really good idea that you operate in the directory indicating the version of BOSS).

Change `maqm` in `setupCVS.sh` to 'alice' then
{%highlight bash%}
source setupCMT.sh
cmt config
source setup.sh
source setupCVS.sh
{%endhightlight%}

Use `echo $CMTPATH` to check if your configuration is sucessful. If your work area is printed, the configuration is done.

# An example use of boss.exe
Copy `/afs/ihep.ac.cn/bes3/offline/Boss/7.0.4/TestRelease` to your `/workfs/bes/alice/7.0.4`.
Change directory(`cd`) to `TestRelease/TestRelease-00-00-78/cmt` then
{%highlight bash%}
cmt config
source setup.sh
{%endhightlight%}

Use `echo $TESTRELEASEROOT` to check if the environment is OK.

`cd` to `TestRelease/TestRelease-00-00-78/run`.

You simulate events by `boss.exe jobOptions_sim.txt`.Then recontruct the events by `boss.exe jobOptions_rec.txt`.
Finally you will get a root file named `rhopi_ana.root` after analyzing the reconstruction by `boss.exe jobOptions_ana_rhopi.txt`.

