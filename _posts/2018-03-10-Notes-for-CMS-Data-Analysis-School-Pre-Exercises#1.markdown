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
$ git config --global user.name “Your name" 
$ git config --global user.email “Your email”
$ git config --global user.github “Your user name in Github”
{%endhighlight%}
