# HFSS連結初始化

於AEDT2022R2開啟HFSS的陣列天線設計，在Spyder當中執行以下代碼即可連結到該設計：

```python
from win32com import client

oApp = client.Dispatch("Ansoft.ElectronicsDesktop.2022.2")
oDesktop = oApp.GetAppDesktop()
oDesktop.RestoreWindow()
```

接下來我們便可以用程式碼編輯3D模型、修改激發源、擷取遠場數值等。
