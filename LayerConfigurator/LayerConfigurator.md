# Layer Configurator Documentation
This page will focus on all of the components used within the Layer Configurator page
![Layout_Editor](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/c761c32c-bb6b-4f6c-8e7b-72252cfee335)


## Pallet Builder 
![Palet_Builder](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/61f69728-f892-47b8-981a-d312eebd1ed0)

Source code:
```
var lebar = TcHmi.Controls.get('lebarPALET').getValue();
var panjang = TcHmi.Controls.get('panjangPALET').getValue();

if (lebar > 0 && panjang > 0) {
		TcHmi.Controls.get('PALLETTE_FIX').setWidth(lebar);
		TcHmi.Controls.get('PALLETTE_FIX').setHeight(panjang);
	} else {
	    alert('WARNING!\nInvalid value for PANJANG or LEBAR');
	}
```
## 📦Box Builder 
### ❌Delete Box 
### ⬆Move Box 
### 🔁Rotate Box 
## 📏Pallet and Box Height 
## Preset Name
