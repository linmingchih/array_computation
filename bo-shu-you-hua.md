# 波束優化

{% code lineNumbers="true" %}
```python
import json, os
import numpy as np
import matplotlib.pyplot as plt
from win32com import client

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
    for i in get_excitations():
        if i not in sources:
            sources[i] = ('0W', '0deg')
    
    sources_list = []
    for name, (mag, phase) in sources.items():
        sources_list.append(["Name:=", name, "Magnitude:=", mag, "Phase:=", phase])
    
    para = [["IncludePortPostProcessing:=", True, "SpecifySystemPower:=", False]]
    
    oModule = oDesign.GetModule("Solutions")
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
    		"PhiStart:="		, "0deg",
    		"PhiStop:="		, "360deg",
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
    
    etheta_real = arr[0].GetRealDataValues("rETheta")
    etheta_imag = arr[0].GetImagDataValues("rETheta")
    ephi_real = arr[0].GetRealDataValues("rEPhi")
    ephi_imag = arr[0].GetImagDataValues("rEPhi")
    
    return [etheta_real, etheta_imag, ephi_real, ephi_imag]


class farfield():
    def __init__(self, etheta=np.zeros((361,181)), ephi=np.zeros((361,181))):
        self.etheta = etheta
        self.ephi = ephi
        
    def __add__(self, other):
        etheta = self.etheta + other.etheta
        ephi = self.ephi + other.ephi
        
        return farfield(etheta, ephi)
    
    def set(self, mag, phase):
        etheta = np.sqrt(mag)*np.exp(1j*np.radians(phase))*self.etheta
        ephi = np.sqrt(mag)*np.exp(1j*np.radians(phase))*self.ephi
        
        return farfield(etheta, ephi)
    
    def plot_rETotal(self):
        self.retotal = np.sqrt(np.power(np.absolute(self.etheta),2)+np.power(np.absolute(self.ephi),2))
  
        plt.figure(figsize=(8, 4))
        plt.title('rETotal')
        plt.xlabel('phi')
        plt.ylabel('theta')
        
        plt.imshow(self.retotal.T, cmap='jet')
        plt.colorbar()
        print(np.max(self.retotal))

class beam():        
    def __init__(self, field_json=''):
        self.excitations = {}
        
        if os.path.isfile(field_json):
            with open(field_json) as f:
                self.port_fields = json.load(f)
                self.ports = list(self.port_fields.keys())
        
        else:
            oModule = oDesign.GetModule("ReportSetup")
            oModule.DeleteAllReports()
            
            insert_sphere()
            self.ports = get_excitations()
            
            for port in self.ports:
                edit_sources({port:('1W', '0deg')})
            
                self.port_fields[port] = get_farfield()
                print(f'Set Port: {port}')
            
            oModule = oDesign.GetModule("RadField")
            if 'python' in oModule.GetChildNames():
                oModule.DeleteSetup(['python'])
            
            with open(field_json, 'w') as f:
                json.dump(self.port_fields, f, indent=4)

            
        for port, [etheta_real, etheta_imag, ephi_real, ephi_imag] in self.port_fields.items():
            etheta = [complex(i, j) for i, j in zip(etheta_real, etheta_imag)]
            etheta = np.array([etheta[i:i+181] for i in range(0, 181*361, 181)])
            
            ephi = [complex(i, j) for i, j in zip(ephi_real, ephi_imag)]
            ephi = np.array([ephi[i:i+181] for i in range(0, 181*361, 181)])
            self.port_fields[port] = farfield(etheta, ephi)

        for port in self.ports:
            self.excitations[port] = (0, 0)
        
        self.excitations[self.ports[0]] = (1, 0)
        self.calculate()
        
    def calculate(self):
        f0 = farfield()
        for port, (mag, phase) in self.excitations.items():
            f0 = f0 + self.port_fields[port].set(mag, phase)
        
        self.field = f0
    
    def edit_sources(self, sources):
        for i in self.excitations:
            if i not in sources:
                self.excitations[i] = (0, 0)
            else:
                mag, phase = sources[i]
                self.excitations[i] = (mag, phase)
    
    def push_excitation(self):
        pass

    def plot_realized_gain(self):
        pd = (np.power(np.absolute(self.field.etheta),2)
              +np.power(np.absolute(self.field.ephi),2))/377/2
        
        total_power = sum([mag for mag, phase in self.excitations.values()])
        print(total_power)
        realized_gain = pd/(total_power/4/np.pi)
        print(np.max(realized_gain))
        
        plt.figure(figsize=(8, 4))
        plt.title('Realized Gain')
        plt.xlabel('phi')
        plt.ylabel('theta')
        
        plt.imshow(realized_gain.T, cmap='jet')
        plt.colorbar()

b1 = beam('d:/demo/field.json')
b1.plot_realized_gain()
```
{% endcode %}
