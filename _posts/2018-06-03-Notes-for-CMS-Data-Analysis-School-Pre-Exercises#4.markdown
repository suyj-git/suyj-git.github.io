---
layout: note
---
Notes for [CMS Data Analysis School Pre-Exercises](https://twiki.cern.ch/twiki/bin/view/CMS/SWGuideCMSDataAnalysisSchoolPreExerciseFirstSet)

**Making sure you have a CERN Grid Account before you reading this!**

# EDAnalyzer
In your CMSSW/src path, execute
{%highlight bash%}
git cms-addpkg PhysicsTools/PatExamples
{%endhighlight%}

Save the following code to MyZPeakAnalyzer.cc in PhysicsTools/PatExamples/src/
{%highlight C++%}
#include <map>
#include <string>

#include "TH1.h"

#include "FWCore/Framework/interface/Frameworkfwd.h"
#include "FWCore/Framework/interface/Event.h"
#include "FWCore/Framework/interface/EDAnalyzer.h"
#include "FWCore/ParameterSet/interface/ParameterSet.h"
#include "FWCore/ServiceRegistry/interface/Service.h"
#include "CommonTools/UtilAlgos/interface/TFileService.h"
#include "DataFormats/PatCandidates/interface/Muon.h"

#include "DataFormats/PatCandidates/interface/Electron.h"
#include "DataFormats/PatCandidates/interface/Muon.h"

class MyZPeakAnalyzer : public edm::EDAnalyzer {

public:
  explicit MyZPeakAnalyzer(const edm::ParameterSet&);
  ~MyZPeakAnalyzer();
  
private:

  virtual void beginJob() ;
  virtual void analyze(const edm::Event&, const edm::EventSetup&);
  virtual void endJob() ;
  
  // simple map to contain all histograms; 
  // histograms are booked in the beginJob() 
  // method
  std::map<std::string,TH1F*> histContainer_; 
  // ----------member data ---------------------------     
  edm::EDGetTokenT<pat::MuonCollection> muonCollToken;
  edm::EDGetTokenT<pat::ElectronCollection> elecCollToken;
 
  // input tags 
  edm::InputTag muonSrc_;
  edm::InputTag elecSrc_;
};


MyZPeakAnalyzer::MyZPeakAnalyzer(const edm::ParameterSet& iConfig):

  histContainer_(),
  muonSrc_(iConfig.getUntrackedParameter<edm::InputTag>("muonSrc")),
  elecSrc_(iConfig.getUntrackedParameter<edm::InputTag>("elecSrc")){

  muonCollToken = consumes<pat::MuonCollection>(muonSrc_);
  elecCollToken = consumes<pat::ElectronCollection>(elecSrc_);

}

MyZPeakAnalyzer::~MyZPeakAnalyzer(){
}

void
MyZPeakAnalyzer::analyze(const edm::Event& iEvent, 
			 const edm::EventSetup& iSetup){

  // get pat muon collection 
  edm::Handle< std::vector<pat::Muon>> muons;
  iEvent.getByToken(muonCollToken, muons);

  // fill pat muon histograms
  for (auto it = muons->cbegin(); it != muons->cend(); ++it) {
    histContainer_["muonPt"] ->Fill(it->pt());
    histContainer_["muonEta"]->Fill(it->eta());
    histContainer_["muonPhi"]->Fill(it->phi());

    if( it->pt()>20 && fabs(it->eta())<2.1 ){
      for (auto it2 = muons->cbegin(); it2 != muons->cend(); ++it2){
	if (it2 > it){
	  // check only muon pairs of unequal charge 
	  if( it->charge()*it2->charge()<0){
	    histContainer_["mumuMass"]->Fill((it->p4()+it2->p4()).mass());
	  }
	}
      }
    }
  }

  // get pat electron collection 
  edm::Handle< std::vector<pat::Electron>> electrons;
  iEvent.getByToken(elecCollToken, electrons);

  // loop over electrons
  for (auto it = electrons->cbegin(); it != electrons->cend(); ++it) {
    histContainer_["elePt"] ->Fill(it->pt());
    histContainer_["eleEta"]->Fill(it->eta());
    histContainer_["elePhi"]->Fill(it->phi());    
  }

  // Multiplicity
  histContainer_["eleMult" ]->Fill(electrons->size());
  histContainer_["muonMult"]->Fill(muons->size() );
}

void 
MyZPeakAnalyzer::beginJob()
{
  // register to the TFileService
  edm::Service<TFileService> fs;


  histContainer_["mumuMass"]=fs->make<TH1F>("mumuMass", "mass",    90,   30., 120.);
  
  // book histograms for Multiplicity:

  histContainer_["eleMult"]=fs->make<TH1F>("eleMult",   "electron multiplicity", 100, 0,  50);
  histContainer_["muonMult"]=fs->make<TH1F>("muonMult",   "muon multiplicity",     100, 0,  50);

  // book histograms for Pt:

  histContainer_["elePt"]=fs->make<TH1F>("elePt",   "electron Pt", 100, 0,  200);
  histContainer_["muonPt"]=fs->make<TH1F>("muonPt",   "muon Pt", 100, 0, 200);

 // book histograms for Eta:
  histContainer_["eleEta"]=fs->make<TH1F>("eleEta",   "electron Eta",100, -5,  5);
  histContainer_["muonEta"]=fs->make<TH1F>("muonEta",   "muon Eta",  100, -5,  5);


 // book histograms for Phi:
  histContainer_["elePhi"]=fs->make<TH1F>("elePhi",   "electron Phi", 100, -3.5, 3.5);
  histContainer_["muonPhi"]=fs->make<TH1F>("muonPhi",   "muon Phi",     100, -3.5, 3.5);
    
}

void 
MyZPeakAnalyzer::endJob() 
{
}

#include "FWCore/Framework/interface/MakerMacros.h"
DEFINE_FWK_MODULE(MyZPeakAnalyzer);

{%endhighlight%}

And save the following code to MyZPeak_cfg.py in CMSSW_9_3_2/src/
{%highlight python%}
import FWCore.ParameterSet.Config as cms

process = cms.Process("Test")

process.source = cms.Source("PoolSource",
  fileNames = cms.untracked.vstring(
  		'root://cmseos.fnal.gov//store/user/cmsdas/2018/pre_exercises/fourth_set/slimMiniAOD_data_MuEle_1.root',
#      	'file:slimMiniAOD_data_MuEle_1.root'
  )
)

process.MessageLogger = cms.Service("MessageLogger")
process.maxEvents = cms.untracked.PSet( 
    input = cms.untracked.int32(10000) 
)

process.analyzeBasicPat = cms.EDAnalyzer("MyZPeakAnalyzer",
  muonSrc = cms.untracked.InputTag("slimmedMuons"),                  
  elecSrc = cms.untracked.InputTag("slimmedElectrons"),
)

process.TFileService = cms.Service("TFileService",
                                   fileName = cms.string('myZPeakCRAB.root')
                                   )

process.p = cms.Path(process.analyzeBasicPat)
{%endhighlight%}

Use `scram b` and `cmsRun MyZPeak_cfg.py` to produce myZPeakCRAB.root 

# Fit myZPeakCRAB.root
Add the following lines to your `rootlogon.C` in your home directory
{%highlight python%}
{
 gSystem->Load("libFWCoreFWLite.so");
 AutoLibraryLoader::enable();
 gSystem->Load("libDataFormatsFWLite.so");
 gROOT->SetStyle ("Plain");
 gSystem->Load("libRooFit") ;
 using namespace RooFit ;
 cout << "loaded" << endl;
}
{%endhighlight%}


The different distribution that we would fit to the Z mass peak are:

* Gaussian 
$$G(x;\mu,\sigma)=\frac{1}{\sqrt{2\pi}\sigma}\exp\left[-\frac{(x-\mu)^2}{2\sigma^2}\right]$$

* Relativistic Breit-Wigner
$$B(m;M,\Gamma)=N\frac{2}{\pi}\frac{\Gamma^2M^2}{(m^2-M^2)^2+m^4(\Gamma^2/M^2)}$$

* Convolution of relativistic Breit-Wigner plus interference term with a Gaussian
$$P(m)=\int B(m';M,\Gamma)G(m-m';\mu,\sigma)d m'$$

Some general remarks about fitting a Z peak:

To fit a generator-level Z peak a Breit-Wigner fit makes sense. However, reconstructed-level Z peaks have many detector resolutions that smear the Z mass peak. 

* If the detector resolution is relatively poor, then it is usually good enough to fit a gaussian (since the gaussian detector resolution will overwhelm the inherent Briet-Wigner shape of the peak). 
* If the detector resolution is fairly good, then another option is to fit a Breit-Wigner (for the inherent shape) convoluted with a gaussian (to describe the detector effects).This is in the "no-background" case. 
* If you have backgrounds in your sample (Drell-Yan, cosmics, etc...), and you want to do the fit over a large mass range, then another function needs to be included to take care of this - an exponential is commonly used.

# Use a macro in RooFit
You can find more about RooFit in ![twiki](https://twiki.cern.ch/twiki/bin/view/CMS/RooFit).

Add the following line to you rootlogon.C:
{%highlight C++%}
 gROOT->ProcessLine(".include /cvmfs/cms.cern.ch/slc6_amd64_gcc491/lcg/roofit/5.34.18-cms3/include/");
{%endhighlight%}

Here is an example of a macro using RooFit:
{%highlight C++%}
#include "RooGlobalFunc.h"
#include "RooRealVar.h"
#include "RooDataSet.h"
#include "RooDataHist.h"
#include "RooGaussian.h"
#include "TCanvas.h"
#include "RooPlot.h"
#include "TTree.h"
#include "TH1D.h"
#include "TRandom.h"
using namespace RooFit ;

void RooFit2(){
  gROOT->ProcessLine(".x ~/rootlogon.C");
  gStyle->SetOptFit(0);

  TFile *f = new TFile("myZPeakCRAB.root","READ");
  //f.cd("analyzeBasicPat");
  TH1F *Z_mass=(TH1F*)f->Get("analyzeBasicPat/mumuMass");

  Z_mass->Draw();

  // double hmin = Z_mass->GetXaxis()->GetXmin();
  // double hmax = Z_mass->GetXaxis()->GetXmax();

  float hmin = 80.0;
  float hmax = 100.0;


  // Declare observable x
  RooRealVar x("x","x",hmin,hmax) ;
  RooDataHist dh("dh","dh",x,Import(*Z_mass)) ;

  RooPlot* frame = x.frame(Title("Z mass")) ;
  dh.plotOn(frame,MarkerColor(2),MarkerSize(0.9),MarkerStyle(21));  //this will show histogram data points on canvas 
  dh.statOn(frame);  //this will display hist stat on canvas

  RooRealVar mean("mean","mean",95.0, 70.0, 120.0);
  RooRealVar width("width","width",5.0, 0.0, 120.0);
  RooRealVar sigma("sigma","sigma",5.0, 0.0, 120.0);
  //RooGaussian gauss("gauss","gauss",x,mean,sigma);
  //RooBreitWigner gauss("gauss","gauss",x,mean,sigma);
  RooVoigtian gauss("gauss","gauss",x,mean,width,sigma);

  RooFitResult* filters = gauss.fitTo(dh,"qr");
  gauss.plotOn(frame,LineColor(4));//this will show fit overlay on canvas 
  gauss.paramOn(frame); //this will display the fit parameters on canvas
  //filters->Print("v");

  // Draw all frames on a canvas
  TCanvas* c = new TCanvas("ZmassHisto","ZmassHisto",770,700) ;
  c->cd() ; gPad->SetLeftMargin(0.15);

  frame->GetXaxis()->SetTitle("Z mass (in GeV/c^{2})");  frame->GetXaxis()->SetTitleOffset(1.2);
  frame->Draw() ;
  c->SaveAs("myZmaa.png");  
  //float binsize = Z_mass->GetBinWidth(1); char Bsize[50]; 
  /*
  //sprintf(Bsize,"Events per %2.2f",binsize);
  // frame->GetYaxis()->SetTitle(Bsize);  
  //frame->GetYaxis()->SetTitleOffset(1.2);
  frame->Draw() ;
  //c->SaveAs("myZmaa.png");  
  */

}
{%endhighlight%}