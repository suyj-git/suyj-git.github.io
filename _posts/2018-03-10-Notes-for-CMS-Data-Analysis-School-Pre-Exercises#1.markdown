---
layout: note
---
Notes for [CMS Data Analysis School Pre-Exercises](https://twiki.cern.ch/twiki/bin/view/CMS/SWGuideCMSDataAnalysisSchoolPreExerciseFirstSet)

*Making sure you have a CERN Grid Account before you reading this!*

## Grid Certificate and CMS VO registration
Both of grid certificate and CMS VO registration is needed for visiting DAS(Data Aggregation System).

# Grid Certificate
Grid Certificate can be obtained in [CERN Certification Authority](https://ca.cern.ch/ca/user/Request.aspx?template=EE2User).
Then you need install the certificate on both your personal computer and remote host in CERN(lxplus.cern.ch).

For personal computer, just doule-click the certificate you downloaded and follow the instructions the installer shows.

For remote host, you should firstly upload your certificate to the host (e.g by FileZilla). Then 
{%highlight bash%}
$ cd ~
$ mkdir .globus
$ cd ~/.globus
$ mv /path/to/mycert.p12 .
$ rm -f usercert.pem
$ rm -f userkey.pem
$ openssl pkcs12 -in mycert.p12 -clcerts -nokeys -out usercert.pem
$ openssl pkcs12 -in mycert.p12 -nocerts -out userkey.pem
$ chmod 400 userkey.pem
$ chmod 400 usercert.pem
{%endhighlight%}

# CMS VO Registration
The registration can be done [here](https://voms2.cern.ch:8443/voms/cms/register).

After filling your information and submitting the request, it may take several days to receive the the consent mail.
--------------------------
## Github
Github account is needed for checkint out and developing CMSSW.If you dont have a Github account you can get one [here](https://github.com/).

After you have a Github account, fork [CMSSW](https://github.com/cms-sw/cmssw).

You shoule generate a ssh key for the remote host by following instructions [here](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/).Then register your ssh key in your [Github](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/#platform-linux) account.

In your terminal at lxplus.cern.ch, execute
{%highlight bash%}
$ git config --global user.name "Your name" 
$ git config --global user.email "Your email"
$ git config --global user.github "Your user name in Github"
{%endhighlight%}

## CMS Configuration
Put
{%highlight bash%}
$ source /cvmfs/cms.cern.ch/cmsset_default.csh #or .sh for bash
{%endhighlight%}
in your file ~/.bash_profile.

Then
{%highlight bash%}
cd ~/nobackup
mkdir YOURWORKINGAREA
cd YOURWORKINGAREA
cmsrel CMSSW_9_3_2
cd CMSSW_9_3_2/src
cmsenv
git cms-init
{%endhighlight%}

## DAS(Data Aggregation System)
You can find DAS [here](https://cmsweb.cern.ch/das/).

To use DAS in your terminal, you should firstly
{%highlight bash%}
$ voms-proxy-init --voms cms
{%endhighlight%}
Then you can use DAS like this
{%highlight bash%}
das_client.py --query="dataset=/DoubleMuon*/Run2017C-PromptReco-v3/MINIAOD" --format=plain
{%endhighlight%}

More about DAS can be found [here](https://twiki.cern.ch/twiki/bin/view/CMSPublic/WorkBookDataSamples).

##EDM
After my testing, the LFN(logical file name) 

root://eoscms.cern.ch//eos/cms/store/relval/CMSSW_9_3_0_pre5/RelValZMM_13/MINIAODSIM/93X_mc2017_realistic_v2-v1/00000/96FBB6F5-0E92-E711
-841B-0025905B85C0.root

returned by `edmFileUtil` command can *NOT* be used to dump event content by command
{%highlight bash%}
edmDumpEvent root://eoscms.cern.ch//eos/cms/store/relval/CMSSW_9_3_0_pre5/RelValZMM_13/MINIAODSIM/93X_mc2017_realistic_v2-v1/00000/96FBB6F5-0E92-E711
-841B-0025905B85C0.root
{%endhighlight%}
. The PFN(physical file name) 

/store/relval/CMSSW_9_3_0_pre5/RelValZMM_13/MINIAODSIM/93X_mc2017_realistic_v2-v1/00000/96FBB6F5-0E92-E711-841B-0025905B85C0.root

can not be used, either.

To dump event content in /store/relval/CMSSW_9_3_0_pre5/RelValZMM_13/MINIAODSIM/93X_mc2017_realistic_v2-v1/00000/96FBB6F5-0E92-E711-841B-0025905B85C0.root, you should firstly go to [DAS](https://cmsweb.cern.ch/das/) and search by the PFN.
Then clicking "Download" shown below,
![](https://github.com/WestRice/westrice.github.io/blob/master/_posts/f1.jpg)
and use the LFN shown below
![](https://github.com/WestRice/westrice.github.io/blob/master/_posts/f2.jpg)
