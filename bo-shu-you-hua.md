# 波束優化

{% code lineNumbers="true" %}
```python
import numpy as np
from win32com import client
from math import sin, cos, atan, radians, degrees, sqrt, pi

oApp = client.Dispatch("Ansoft.ElectronicsDesktop.2022.2")
oDesktop = oApp.GetAppDesktop()
oDesktop.RestoreWindow()

oProject = oDesktop.GetActiveProject()
oDesign = oProject.GetActiveDesign()
oEditor = oDesign.SetActiveEditor("3D Modeler")

def get_excitations():
    oModule = oDesign.GetModule("BoundarySetup")
    return oModule.GetExcitations()[::2]

def edit_sources(sources):
    oModule = oDesign.GetModule("Solutions")
    
    sources_list = [] 
    for name, (mag, phase) in sources.items():
        sources_list.append(["Name:=", name, "Magnitude:=", mag, "Phase:=", phase])
    
    para = [["IncludePortPostProcessing:=", True, "SpecifySystemPower:=", False]]
    oModule.EditSources(para + sources_list)

def insert_sphere():
    oModule = oDesign.GetModule("RadField")
    if 'python' in oModule.GetChildNames():
        oModule.DeleteSetup(['python'])

    oModule.InsertInfiniteSphereSetup(
    	[
    		"NAME:python",
    		"UseCustomRadiationSurface:=", False,
    		"CSDefinition:="	, "Theta-Phi",
    		"Polarization:="	, "Linear",
    		"ThetaStart:="		, "0deg",
    		"ThetaStop:="		, "180deg",
    		"ThetaStep:="		, "1deg",
    		"PhiStart:="		, "-180deg",
    		"PhiStop:="		, "180deg",
    		"PhiStep:="		, "1deg",
    		"UseLocalCS:="		, False
    	])

def get_farfield(solution = "Setup1 : LastAdaptive"):    
    oModule = oDesign.GetModule("ReportSetup")
    arr = oModule.GetSolutionDataPerVariation(  
    "Far Fields", 
    solution, 
    ["Context:=", "python"],
    	[
    		"Theta:="		, ["All"],
    		"Phi:="			, ["All"],
    	], 
    [["rETheta", 'rEphi']])
    
    result = [arr[0].GetRealDataValues("rETheta"),
              arr[0].GetRealDataValues("rEPhi")]
    
    return result

class farfield():
    def __init__(self, field_data):
        pass
    
    def __add__(self):
        pass
    
    def __mul__(self):
        pass
    
    def eirp_cdf(self):
        pass
    
    def plot():
        pass
    

class beam():
    def __init__(self):
        self.port_fields = None
        
        self.excitations = {}
        self.field
        self.information = {}
        
    def load(self, field_npy, reload=False, version='2022.2'):
        pass
    
    def edit_sources(self, sources):
        pass
    
    def push_excitation(self):
        pass
    
```
{% endcode %}
