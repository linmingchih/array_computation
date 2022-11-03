# 遠場物件與方法

遠場物件紀錄電磁場的矩陣資料並支援以下幾種方法

* \+運算，可以兩個場相加並返回新的場，例如C=A+B
* set()函數，設定功率與相位並返回新的場
* plot\_rEtotal()，=畫出電場強度分布圖

{% code lineNumbers="true" %}
```python
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
```
{% endcode %}
