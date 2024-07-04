# Layer Configurator Documentation
This page will focus on all of the components used within the Layer Configurator page
![Layout_Editor](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/c761c32c-bb6b-4f6c-8e7b-72252cfee335)


## Pallet Builder 
![Palet_Builder](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/61f69728-f892-47b8-981a-d312eebd1ed0)

`Modify Pallet` button source code:
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
This function uses `TcHmi.Controls.get()` with attribute `.get` and `.set` based on [TcHmi Controls](https://infosys.beckhoff.com/english.php?content=../content/1033/te2000_tc3_hmi_engineering/3728912139.html&id=4044065286845169179) with usage as follow:
- Fetch length and width value from input textbox
```
var lebar = TcHmi.Controls.get('lebarPALET').getValue();
var panjang = TcHmi.Controls.get('panjangPALET').getValue();
```
- Set length and width value to the Pallet rectangle
```
TcHmi.Controls.get('PALLETTE_FIX').setWidth(lebar);
TcHmi.Controls.get('PALLETTE_FIX').setHeight(panjang);
```
## 📦Box Builder 
![Box_Builder](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/43f592f2-8ad8-41e5-a36c-d38f8270d4aa)

`Generate Box` button source code:
```
var increment = TcHmi.Symbol.read('BoxCounter', TcHmi.SymbolType.Internal); 
var uniqueId = 'BOX_' + increment;

var lebar = TcHmi.Controls.get('lebarBOX').getValue();
var panjang = TcHmi.Controls.get('panjangBOX').getValue();

var myRect = TcHmi.ControlFactory.createEx(
    'TcHmi.Controls.Beckhoff.TcHmiTextblock',
    uniqueId,
    {
        'data-tchmi-top': 220, 
        'data-tchmi-left': 370, 
        'data-tchmi-width': lebar, 
        'data-tchmi-height': panjang,
        'data-tchmi-border-style': {
			"left": "Dashed",
			"right": "Dashed",
			"top": "Solid",
			"bottom": "Dashed"
        },
        'data-tchmi-border-width': {
			"left": 1,
			"right": 1,
			"top": 5,
			"bottom": 1,
			"leftUnit": "px",
			"rightUnit": "px",
			"topUnit": "px",
			"bottomUnit": "px"
        },
        'data-tchmi-border-color': {
			"color": "rgba(255, 0, 0, 1)",
        },
        'data-tchmi-text-horizontal-alignment' : "Center",
        'data-tchmi-text-font-size':20,
        'data-tchmi-text' : uniqueId,
   }
);

var rotationStates = JSON.parse(TcHmi.Symbol.read('rotationStates', TcHmi.SymbolType.Internal));
rotationStates[uniqueId] = 0;
TcHmi.Symbol.writeEx('%i%rotationStates%/i%', JSON.stringify(rotationStates));
console.log(rotationStates);

var scaleTransformation = JSON.parse(TcHmi.Symbol.read('ScaleTrans', TcHmi.SymbolType.Internal));
var originTransformation = JSON.parse(TcHmi.Symbol.read('OriginTrans', TcHmi.SymbolType.Internal));
myRect.setTransform([scaleTransformation, originTransformation]);

var desktop = TcHmi.Controls.get('Desktop'); 
	if (desktop && myRect) 
	{
    	desktop.addChild(myRect); 
	}
	
TcHmi.Symbol.writeEx('%i%RectID%/i%', uniqueId);

var newArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal); 
newArray.push(uniqueId);
TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', newArray, function(writeResult) {});

var newIndex = newArray.indexOf(uniqueId);
TcHmi.Controls.get('TcHmiCombobox').setSelectedIndex(newIndex);
TcHmi.Controls.get('TcHmiCombobox_1').setSelectedIndex(10);
```
### ❌Delete Box 
### ⬆Move Box 
### 🔁Rotate Box 
## 📏Pallet and Box Height 
## Preset Name
