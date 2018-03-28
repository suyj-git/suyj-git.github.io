---
layout: note
---
Notes for [CMS Data Analysis School Pre-Exercises](https://twiki.cern.ch/twiki/bin/view/CMS/SWGuideCMSDataAnalysisSchoolPreExerciseFirstSet)

*Making sure you have a CERN Grid Account before you reading this!*

## Slim MiniOAD
An example script for sliming MiniOAD:
{%highlight python%}
## import skeleton process
import FWCore.ParameterSet.Config as cms

process = cms.Process("DAS")

process.load("FWCore.MessageService.MessageLogger_cfi")

process.maxEvents = cms.untracked.PSet( input = cms.untracked.int32(1000) )

process.source = cms.Source("PoolSource",
    fileNames = cms.untracked.vstring(
      'root://cmseos.fnal.gov//store/user/cmsdas/2017/pre_exercises/DYJetsToLL.root'
    )
)

process.out = cms.OutputModule("PoolOutputModule",
    fileName = cms.untracked.string('slimMiniAOD_MC_MuEle.root'),
    outputCommands = cms.untracked.vstring(['drop *', 'keep *_slimmedMuons__*', 'keep *_slimmedElectrons__*'])
)

process.end = cms.EndPath(process.out)
{%endhighlight%}

## Use FWLite on The FWLite
Use following commands to checkout FWLite and build it:
{%highlight bash%}
git cms-addpkg PhysicsTools/FWLite
scram b # or scram b -j 8 to speed up the compiling
cmsenv
{%endhighlight%}
**It is necessary to call cmsenv again after compiling this package because it adds executables in the $CMSSW_BASE/bin area. **

To make a ZPeak from this executable, using the MC MiniAOD, run the following command (which will not work out of the box, see below):

`FWLiteHistograms inputFiles=slimMiniAOD_MC_MuEle.root outputFile=ZPeak_MC.root maxEvents=-1 outputEvery=100`

You can see that you will get the following error:
{%highlight bash%}
terminate called after throwing an instance of 'cms::Exception'
  what():  An exception of category 'ProductNotFound' occurred.
Exception Message:
getByLabel: Found zero products matching all criteria
Looking for type: edm::Wrapper<std::vector<reco::Muon> >
Looking for module label: muons
Looking for productInstanceName: 

The data is registered in the file but is not available for this event
{%endhighlight%}
This error occurs because your input files slimMiniAOD_MC_MuEle.root is a MiniAOD and does *NOT* contain *reco::Muon* whose label is muons. 
It contains, however, *slimmedMuons*.

Open the code `$CMSSW_BASE/src/PhysicsTools/FWLite/bin/FWLiteHistograms.cc` to replace `reco::Muon` by `reco::slimmedMuons`.
