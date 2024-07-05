# Layer Configurator Documentation
This page will focus on all of the components used within the Layer Configurator page

![Layout_Editor](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/c761c32c-bb6b-4f6c-8e7b-72252cfee335)


## Pallet Builder 
This section is for modify the size of the Pallet being used in the real world.

![Palet_Builder](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/61f69728-f892-47b8-981a-d312eebd1ed0)

`Modify Pallet` button code:
```Javascript
var lebar = TcHmi.Controls.get('lebarPALET').getValue();
var panjang = TcHmi.Controls.get('panjangPALET').getValue();

if (lebar > 0 && panjang > 0) {
		TcHmi.Controls.get('PALLETTE_FIX').setWidth(lebar);
		TcHmi.Controls.get('PALLETTE_FIX').setHeight(panjang);
	} else {
	    alert('WARNING!\nInvalid value for PANJANG or LEBAR');
	}
```

- Fetch length and width value from input textbox using `TcHmi.Controls.get()` with attribute `.get`
	```Javascript
	var lebar = TcHmi.Controls.get('lebarPALET').getValue();
	var panjang = TcHmi.Controls.get('panjangPALET').getValue();
	```
 
- Set length and width value to the Pallet rectangle using `TcHmi.Controls.get()` with attribute `.set`
	```Javascript
	TcHmi.Controls.get('PALLETTE_FIX').setWidth(lebar);
	TcHmi.Controls.get('PALLETTE_FIX').setHeight(panjang);
	```
 
## 📦Box Builder 
This section is for generating rectangle shape on screen to visualize boxes on top of the Pallet.

![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/e09b350d-a604-4059-ac1a-2eada3ff66fc)

`Generate Box` button code:
```Javascript
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

var scaleTransformation = JSON.parse(TcHmi.Symbol.read('ScaleTrans', TcHmi.SymbolType.Internal));
var originTransformation = JSON.parse(TcHmi.Symbol.read('OriginTrans', TcHmi.SymbolType.Internal));
myRect.setTransform([scaleTransformation, originTransformation]);

var desktop = TcHmi.Controls.get('Desktop'); 
	if (desktop && myRect) 
	{
    	desktop.addChild(myRect); 
	}

var rotationStates = JSON.parse(TcHmi.Symbol.read('rotationStates', TcHmi.SymbolType.Internal));
rotationStates[uniqueId] = 0;
TcHmi.Symbol.writeEx('%i%rotationStates%/i%', JSON.stringify(rotationStates));

TcHmi.Symbol.writeEx('%i%RectID%/i%', uniqueId);

var newArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal); 
newArray.push(uniqueId);
TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', newArray);

var newIndex = newArray.indexOf(uniqueId);
TcHmi.Controls.get('TcHmiCombobox').setSelectedIndex(newIndex);
TcHmi.Controls.get('TcHmiCombobox_1').setSelectedIndex(10);
```

The source code is put inside this condition where `BoxCounter` is an incremental variable of Internal Symbol[^1] for every click of the button:

![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/9612b122-5160-4ee7-aaf8-f6fbb07c1590)

- Read the current `BoxCounter` value for box identifier using `TcHmi.Symbol.read()` with symbol type `TcHmi.SymbolType.Internal`
	```Javascript
	var increment = TcHmi.Symbol.read('BoxCounter', TcHmi.SymbolType.Internal); 
	var uniqueId = 'BOX_' + increment;
	```

- Fetch length and width value from input textbox using `TcHmi.Controls.get()` with attribute `.get`
	```Javascript
	var lebar = TcHmi.Controls.get('lebarBOX').getValue();
	var panjang = TcHmi.Controls.get('panjangBOX').getValue();
	```

- Set up the box properties and attributes in a variable using `TcHmi.ControlFactory.createEx` with **Textblock** instance `TcHmi.Controls.Beckhoff.TcHmiTextblock`
	```Javascript
	var myRect = TcHmi.ControlFactory.createEx(
	    'TcHmi.Controls.Beckhoff.TcHmiTextblock',
	    uniqueId,
	    {data-tchmi}
	);
	```
> [!Important]
> Each instance needs the exact type, identifier and additional attributes ( in this case is *data-tchmi* ). Make sure `uniqueId` as identifier is unique for every box instance, if not error may occur
	
- Customize the look of the box using `data-tchmi` attributes

	By using the Textblock rather than a regular rectangle, you can customize the look to your liking. What I did here is adding a _front face_ for the box by using a different `border-style` and `border-width` to one of the vertices
	```Javascript
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
	```

- Set the rotation state of the newly generated box in a separated data array. This is done to avoid problems if the rotation is set using the built-in rotation transformation
	```Javascript
	var rotationStates = JSON.parse(TcHmi.Symbol.read('rotationStates', TcHmi.SymbolType.Internal));
	rotationStates[uniqueId] = 0;
	TcHmi.Symbol.writeEx('%i%rotationStates%/i%', JSON.stringify(rotationStates));
	```
	Here I set up a JSON Array of Internal Symbol named `rotationStates` to put the rotation state of each boxes, this will become handy later on Rotate Box[🚀](LayerConfigurator/LayerConfigurator.md#rotate-box)
	
	Read the current `rotationStates` value for box rotation using `TcHmi.Symbol.read()`
	
	Then Write the newly push array to the same Internal Symbol using `TcHmi.Symbol.writeEx()`

- Store existing boxes to an Array of Internal Symbol. This is for the `for loop` function later on the Add Layout Preset[🚀](LayerConfigurator/LayerConfigurator.md#add-preset)
	```Javascript
	var newArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal); 
	newArray.push(uniqueId);
	TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', newArray);
	```
	Read the current `RectIndex` value for box rotation using `TcHmi.Symbol.read()`
	
	Then Write the newly push array to the same Internal Symbol using `TcHmi.Symbol.writeEx()`

## Box Modifier
This is use to modify the box further outside of the generation button above. With this, user may fine tuned their liking of how the box looks.

### Prerequisites
All off the Box Modifier are based on the Combobox of `Select Active ID`. And `Move Step` for Rotate.

![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/a43b02a6-ffcc-46a0-acd3-95f69354ba88)

- `Select Active ID` have a data source of `RectIndex` from Internal Symbol as mention above
- `Move Step` have its own data source of `StepIndex` from Internal Symbol which is an Array consist of value 1 to 10
  ```Javascript
  ["Move Step (mm)",
  "1",
  "2",
  "3",
  "4",
  "5",
  "6",
  "7",
  "8",
  "9",
  "10"]
  ```
- As the box selected, it will show up in the `Active Rect` place holder

	![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/4c2326d1-1c8c-4b7c-9a57-ffe245ac3bb9)

### ❌Delete Box 
This modifier use to Delete box from the page. Can only be done per box. To delete all of the box, please refresh the whole page.

![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/58853527-3004-499f-9905-584d60e668d7)

`Delete Box` button code:
```Javascript
var myArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal);

var myIndex = TcHmi.Controls.get('TcHmiCombobox').getSelectedIndex();

var myElement = TcHmi.Controls.get('TcHmiCombobox').getSelectedText();
var myElement2 = TcHmi.Controls.get(myElement);

if (confirm("Delete " + myElement + " ? ")){
	if (myElement2) {
		myElement2.destroy();
		myArray.splice(myIndex, 1);
		TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', myArray);
		TcHmi.Controls.get('TcHmiCombobox').setSelectedIndex(0);
	}
}
```

- Fetch the Array of boxes from `RectIndex` of Internal Symbol
	```Javascript
	var myArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal);
	```

- Fetch the Box Index of the selected box from `Select Active ID` Combobox using `TcHmi.Controls.get()` with attribute `.getSelectedIndex()`
	```Javascript
	var myIndex = TcHmi.Controls.get('TcHmiCombobox').getSelectedIndex();
	```

- Fetch the Identifier of the selected box from `Select Active ID` Combobox using `TcHmi.Controls.get()` with attribute `.getSelectedText()`

  Use this to `.get` the box directly
	```Javascript
	var myElement = TcHmi.Controls.get('TcHmiCombobox').getSelectedText();
	var myElement2 = TcHmi.Controls.get(myElement);
	```

- Using the built in confirmation dialog box from Javascript, include the following for Confirm:
	- Use `.destroy()` to completely remove the box and its identifier from the environment. After doing this, the Identifier can be use again
	```Javascript
	myElement2.destroy();
	```
	
	- Rearrange the `RectIndex` Array by `.splice` it directly using Box Index earlier
	```Javascript
	myArray.splice(myIndex, 1);
	```
	
	- Write the new `RectIndex` to the Internal Symbol
	```Javascript
	TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', myArray);
	```

### ⬆Move Box 
This modifier use to Move boxes accross the screen and arrange them inside the bounding box of Pallet. 

![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/7b9b6e60-66ef-47d3-9cea-5a7c8bd672e4)

All of the move button function was placed on the built in Event Handler in TwinCAT HMI. The two Event Handler I used are `.onStatePressed` and `onStateReleased`.

![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/1d710503-7d8c-4a88-b0d4-ab21269bf8f4)

The four move button have the same working principle, only differ in the arithmetic operation to the current position.

`Move Up` and `Move Down` button code:
```Javascript
// Move Up
MoveUp = setInterval(function() {
	var x = TcHmi.Controls.get('activeRECT').getText();
	var top1 = TcHmi.Controls.get(x).getTop();
	var step = parseInt(TcHmi.Controls.get('TcHmiCombobox_1').getSelectedValue());
	top1 -= (step*0.75);
	TcHmi.Controls.get(x).setTop(top1)
}, 75);

// Move Down
MoveDown = setInterval(function() {
	var x = TcHmi.Controls.get('activeRECT').getText();
	var top1 = TcHmi.Controls.get(x).getTop();
	var step = parseInt(TcHmi.Controls.get('TcHmiCombobox_1').getSelectedValue());
	top1 += (step*0.75);
	TcHmi.Controls.get(x).setTop(top1)
}, 75);
```

`Move Right` and `Move Left` button code:
```Javascript
// Move Right
MoveRight = setInterval(function() {
	var x = TcHmi.Controls.get('activeRECT').getText();
	var top1 = TcHmi.Controls.get(x).getLeft();
	var step = parseInt(TcHmi.Controls.get('TcHmiCombobox_1').getSelectedValue());
	top1 += (step*0.75);
	TcHmi.Controls.get(x).setLeft(top1);
}, 75);

// Move Left
MoveLeft= setInterval(function() {
	var x = TcHmi.Controls.get('activeRECT').getText();
	var top1 = TcHmi.Controls.get(x).getLeft();
	var step = parseInt(TcHmi.Controls.get('TcHmiCombobox_1').getSelectedValue());
	top1 -= (step*0.75);
	TcHmi.Controls.get(x).setLeft(top1);
}, 75);
```

- Fetch the Identifier of the selected box from `Active Rect` placeholder using `TcHmi.Controls.get()` with attribute `.getText()`

	```Javascript
	var x = TcHmi.Controls.get('activeRECT').getText();
	```
- Fetch the Step Size for selected box from `Move Step` Combobox using `TcHmi.Controls.get()` with attribute `.getSelectedValue()`

	```Javascript
 	var step = parseInt(TcHmi.Controls.get('TcHmiCombobox_1').getSelectedValue());
 	```
- Fetch current Top value for `Move Up` and `Move Down` using `TcHmi.Controls.get()` with attribute `.getTop`

	```Javascript
  	var top1 = TcHmi.Controls.get(x).getTop();
 	```
- Fetch current Left value for `Move Right` and `Move Left` using `TcHmi.Controls.get()` with attribute `.getLeft`

	```Javascript
 	var top1 = TcHmi.Controls.get(x).getLeft();
 	```
- Arithmetic operation for `Move Down` and `Move Right`

	```Javascript
	top1 += (step*0.75);
 	```
- Arithmetic operation for `Move Up` and `Move Left`

	```Javascript
	top1 -= (step*0.75);
 	```
- Write the new value to Top value for `Move Up` and `Move Down` using `TcHmi.Controls.get()` with attribute `.setLeft`

	```Javascript
 	var top1 = TcHmi.Controls.get(x).setTop();
 	```
- Write the new value to Left value for `Move Right` and `Move Left` using `TcHmi.Controls.get()` with attribute `.setLeft`

	```Javascript
 	var top1 = TcHmi.Controls.get(x).setLeft();
 	```

### 🔁Rotate Box 
This modifier use to Rotate boxes by its origin by the increment of 90 degree. 

![image](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/7b9b6e60-66ef-47d3-9cea-5a7c8bd672e4)

The rotate function is a mere switching the Width and Length value with addition of `rotationStates` data for the real rotation in the Gantry.

`Rotate` button code:
```Javascript
var BOX_NUM = TcHmi.Controls.get('activeRECT').getText();
temp = TcHmi.Controls.get(BOX_NUM).getWidth();
TcHmi.Controls.get(BOX_NUM).setWidth(TcHmi.Controls.get(BOX_NUM).getHeight());
TcHmi.Controls.get(BOX_NUM).setHeight(temp);

var rotationStates = JSON.parse(TcHmi.Symbol.read('rotationStates', TcHmi.SymbolType.Internal));
rotationStates[BOX_NUM] = (rotationStates[BOX_NUM] + 90) % 360;
TcHmi.Symbol.writeEx('%i%rotationStates%/i%', JSON.stringify(rotationStates));
```

- Fetch the Identifier of the selected box from `Active Rect` placeholder using `TcHmi.Controls.get()` with attribute `.getText()`

	```Javascript
 	```
- Store the current Width value in a temporary variable

	```Javascript
 	```
- Set the Width value with the current Length value
- Set the Length value with the temporary variable
- Read the current `rotationStates` value of Box Rotation using `TcHmi.Symbol.read()` with symbol type `TcHmi.SymbolType.Internal`
- Set the `rotationStates` the new value. Incremental of 90 degree with possible value of 0, 90, 180, and 270
- Write the new `rotationStates` 
## 📏Pallet and Box Height 
## Preset Name
