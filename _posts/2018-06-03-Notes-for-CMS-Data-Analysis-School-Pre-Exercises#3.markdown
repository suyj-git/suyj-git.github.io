---
layout: note
---
Notes for [CMS Data Analysis School Pre-Exercises](https://twiki.cern.ch/twiki/bin/view/CMS/SWGuideCMSDataAnalysisSchoolPreExerciseFirstSet)

**Making sure you have a CERN Grid Account before you reading this!**

# Setup CRAB
Add
{%highlight bash%}
source /cvmfs/cms.cern.ch/crab3/crab.sh
{%endhighlight%}
to you bash_profile.

Then use 
{%highlight bash%}
crab checkwrite --site=T2_CN_Beijing
{%endhighlight%}
to check whether you can operate on servers in Beijing.

# Use CRAB to submit you job(s)
This is an example of configuration file used by crab:
{%highlight python%}
from WMCore.Configuration import Configuration
config = Configuration()

config.section_("General")
config.General.requestName = 'CMSDAS_MC_generation_test0'
config.General.workArea = 'crab_projects'

config.section_("JobType")
config.JobType.pluginName = 'PrivateMC'
config.JobType.psetName = 'CMSDAS_MC_generation.py'
config.JobType.allowUndistributedCMSSW = True

config.section_("Data")
config.Data.outputPrimaryDataset = 'MinBias'
config.Data.splitting = 'EventBased'
config.Data.unitsPerJob = 10
NJOBS = 10  # This is not a configuration parameter, but an auxiliary variable that we use in the next line.
config.Data.totalUnits = config.Data.unitsPerJob * NJOBS
config.Data.publication = True
config.Data.outputDatasetTag = 'CMSDAS2018_CRAB3_MC_generation_test0'

config.section_("Site")
config.Site.storageSite = 'T2_CN_Beijing'
{%endhighlight%}

Then execute
{%highlight bash%}
crab submit -c crabConfig_MC_generation.py
{%endhighlight%}

You can use `crab status -d <CRAB-project-directory> --long` to check if there are any errors in you jobs.

Once you find any failed jobs, you can use `crab resubmit -d <CRAB-project-directory> --jobids <jobids,jobidA-jobidB>` to resubmit failed jobs.

# How to fetch your data 
1. DAS

You will find your data published in DAS with name the same as "Output dataset".Then you can refer to your data by URL provided by DAS.

2. IHEP

You can find your data in the path `/pnfs/ihep.ac.cn/data/cms/store/user/your_user_name` in servers in IHEP.
