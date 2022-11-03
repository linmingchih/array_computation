---
description: 將HFSS遠場數值資料抓到Python當中並儲存起來
---

# 資料擷取

以下函數取得遠場資料並返回

{% code lineNumbers="true" %}
```python
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
```
{% endcode %}
