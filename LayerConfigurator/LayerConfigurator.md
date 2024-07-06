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

- Store existing boxes to an Array of Internal Symbol. This is for the `for loop` function later on the Add Layout Preset[🚀](LayerConfigurator/LayerConfigurator.md#pallet-height-box-height-and-preset-name)
	```Javascript
	var newArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal); 
	newArray.push(uniqueId);
	TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', newArray);
	```
	Read the current `RectIndex` value for box rotation using `TcHmi.Symbol.read()`
	
	Then Write the newly push array to the same Internal Symbol using `TcHmi.Symbol.writeEx()`

## Box Modifier
This section is use to modify the box further outside of the generation button above. With this, user may fine tuned their liking of how the box looks.

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
- Write the new Top value for `Move Up` and `Move Down` using `TcHmi.Controls.get()` with attribute `.setTop`

	```Javascript
 	var top1 = TcHmi.Controls.get(x).setTop();
 	```
- Write the new Left value for `Move Right` and `Move Left` using `TcHmi.Controls.get()` with attribute `.setLeft`

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
 	var BOX_NUM = TcHmi.Controls.get('activeRECT').getText();
 	```
- Store the current Width value in a temporary variable

	```Javascript
 	temp = TcHmi.Controls.get(BOX_NUM).getWidth();
 	```
- Set the Width value with the current Length value

	```Javascript
 	TcHmi.Controls.get(BOX_NUM).setWidth(TcHmi.Controls.get(BOX_NUM).getHeight());
 	```
- Set the Length value with the temporary variable

	```Javascript
 	TcHmi.Controls.get(BOX_NUM).setHeight(temp);
 	```
- Read the current `rotationStates` value of Box Rotation using `TcHmi.Symbol.read()` with symbol type `TcHmi.SymbolType.Internal`

	```Javascript
 	var rotationStates = JSON.parse(TcHmi.Symbol.read('rotationStates', TcHmi.SymbolType.Internal));
 	```
- Set the `rotationStates` the new value. Incremental of 90 degree with possible value of 0, 90, 180, and 270

	```Javascript
 	rotationStates[BOX_NUM] = (rotationStates[BOX_NUM] + 90) % 360;
 	```
- Write the new `rotationStates` value back to the Internal Symbol

	```Javascript
 	TcHmi.Symbol.writeEx('%i%rotationStates%/i%', JSON.stringify(rotationStates));
 	```

## 📏Pallet Height, Box Height, and Preset Name
This section is to input the Height value of the Box and Pallet and Name of the Preset. All boxes within the same Preset must have the same Height

![Add_Preset](https://github.com/BuDinamo/HMI-GantryPalletizerBeckhoff/assets/117176956/88dae1cb-e626-4539-9ab4-e601651bf612)

The code is pretty long as it contains the whole checking measure

`Add Preset` button code:
```Javascript
function savedPreset() {
	var rectanglesArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal);
	if (rectanglesArray.length <= 1) {
		    alert('WARNING!\nInvalid amount of BOX');
		    return;
	}
	
	var presetName = TcHmi.Controls.get('namaPreset').getText();
	if (!presetName) {
		    alert('WARNING!\nInvalid name for PRESET');
		    return;
	}
	
	palettall = TcHmi.Controls.get('PaletTall').getValue();
	boxtall = TcHmi.Controls.get('BoxTall').getValue();
	if (palettall <= 0 || boxtall <= 0) {
		    alert('WARNING!\nInvalid value for TALL');
		    return;
	}

	var CurrentPreset = TcHmi.Symbol.read('PLC_PresetSaver', TcHmi.SymbolType.Internal);
	var existingPreset = CurrentPreset.find(function(p) { return p.preset === presetName; });	        
	if (existingPreset)
	{
		alert('ERROR!\nPreset "' + presetName + '" already exists.');
		return;
	}	
	
	var scale = JSON.parse(TcHmi.Symbol.read('ScaleTrans', TcHmi.SymbolType.Internal));
	var origi JSON.parse(TcHmi.Symbol.read('OriginTrans', TcHmi.SymbolType.Internal));
	
	var rectanglesData = {};
	
	var specificRectangleId = "PALLETTE_FIX";
	var specificRectangle = TcHmi.Controls.get(specificRectangleId);
	if (specificRectangle) {
		rectanglesData[specificRectangleId] = {
			top: specificRectangle.getTop(),
			left: specificRectangle.getLeft(),
			width: specificRectangle.getWidth(),
			height: specificRectangle.getHeight(),
			tall: palettall
		};
	}
	
	for (var i = 0; i < rectanglesArray.length; i++) {
		var rectangleId = rectanglesArray[i];
		var rectangle = TcHmi.Controls.get(rectangleId);
		if (rectangle) {
			var transform = rectangle.getTransform();
			rectanglesData[rectangleId] = {
				top: rectangle.getTop(),
				left: rectangle.getLeft(),
				width: rectangle.getWidth(),
				height: rectangle.getHeight(),
				tall: boxtall
			};

			if (rectangle.getLeft() < specificRectangle.getLeft() || rectangle.getTop() < specificRectangle.getTop() || (rectangle.getLeft()+rectangle.getWidth()*0.75) > (specificRectangle.getLeft()+specificRectangle.getWidth()*0.75) || (rectangle.getTop()+rectangle.getHeight()*0.75) > (specificRectangle.getTop()+specificRectangle.getHeight()*0.75)) 
			{
				alert('WARNING!\none or more boxes exceed the palette limit');
				return;
			}
	    		
			for (var j = i + 1; j < rectanglesArray.length; j++) {
			        var rect1 = TcHmi.Controls.get(rectanglesArray[i]);
			        var rect2 = TcHmi.Controls.get(rectanglesArray[j]);
			        if (rect1.getLeft() < rect2.getLeft() + rect2.getWidth()*0.75 &&
			            rect1.getLeft() + rect1.getWidth()*0.75 > rect2.getLeft() && 
			            rect1.getTop() < rect2.getTop() + rect2.getHeight()*0.75 &&
			            rect1.getTop() + rect1.getHeight()*0.75 > rect2.getTop()) {
			            alert('WARNING!\nCollision detected between ' + rectanglesArray[i] + ' and ' + rectanglesArray[j]);
			            return;
			        }
		        }
		}		
	}

	var data = {name:presetName, data:rectanglesData};
	let arr = [];
	
	async function processBoxes(data) {
		var rotationStates = JSON.parse(TcHmi.Symbol.read('rotationStates', TcHmi.SymbolType.Internal));
		
		for (var boxName in data.data) {
			var box = data.data[boxName];
			
			let currentBox = [presetName, boxName, box.left, box.top, box.height, box.width, box.tall];
			arr.push(currentBox);
			console.log(arr);
		}
		
		for (var k = 0; k < arr.length; k++) {
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][0]%/s%', arr[k][0]);
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][1]%/s%', arr[k][1]);
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][2]%/s%', arr[k][2]);
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][3]%/s%', arr[k][3]);
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][4]%/s%', arr[k][4]);
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][5]%/s%', arr[k][5]);
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][6]%/s%', arr[k][6]);
			TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][7]%/s%', rotationStates[arr[k][1]]);
		}

		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.WriteFileName%/s%', presetName);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.bGenerate%/s%', true);

		await new Promise(resolve => {
			let intervalBuffer = setInterval(() => {
				TcHmi.Symbol.readEx2('%s%PLC1.CSV_WriteBOXToCSV.WriteDone%/s%', function (data7) {
					if (data7.value == true) {
						clearInterval(intervalBuffer);
						resolve();
					}
				});
			}, 100);
		});	
		
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WritePRESETToCSV.sPresetName%/s%', presetName);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WritePRESETToCSV.bGenerate%/s%', true, function (writeData) {
			if (writeData.error === TcHmi.Errors.NONE) {
				alert('SUCCESS!\nPreset "' + presetName + '" has been saved.');
				
				var myArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal);
				for (var i = 1; i < myArray.length; i++) {
					var elementToRemove = TcHmi.Controls.get(myArray[i]);
					if (elementToRemove) {
						elementToRemove.destroy();
					}
				}
				myArray = [myArray[0]];
				TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', myArray);
				TcHmi.Symbol.writeEx('%i%rotationStates%/i%', {"PALLETTE_FIX": 0});
				TcHmi.Controls.get('TcHmiCombobox').setSelectedIndex(0);
				TcHmi.Symbol.writeEx2('%i%BoxCounter%/i%', 0);
				TcHmi.Controls.get('panjangBOX').setValue(0);
				TcHmi.Controls.get('lebarBOX').setValue(0);
				TcHmi.Controls.get('namaPreset').setText('');
				TcHmi.View.load('LandingPage.view');
			} else {
				alert('ERROR!\nError writing preset "' + presetName + '" to csv', writeData.error);
			}						                     	
		});
    }
	processBoxes(data);
}
savedPreset();
```
### Condition Checking
- No Box available to be saved

	```Javascript
 	var rectanglesArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal);
	if (rectanglesArray.length <= 1) {
		    alert('WARNING!\nInvalid amount of BOX');
		    return;
	}
 	```
- Box or Pallet height is zero

	```Javascript
 	palettall = TcHmi.Controls.get('PaletTall').getValue();
	boxtall = TcHmi.Controls.get('BoxTall').getValue();
	if (palettall <= 0 || boxtall <= 0) {
		    alert('WARNING!\nInvalid value for TALL');
		    return;
	}
 	```
- Preset Name is empty

	```Javascript
 	var presetName = TcHmi.Controls.get('namaPreset').getText();
	if (!presetName) {
		    alert('WARNING!\nInvalid name for PRESET');
		    return;
	}
 	```
- Preset Name is already exist

	```Javascript
 	var CurrentPreset = TcHmi.Symbol.read('PLC_PresetSaver', TcHmi.SymbolType.Internal);
	var existingPreset = CurrentPreset.find(function(p) { return p.preset === presetName; });	        
	if (existingPreset)
	{
		alert('ERROR!\nPreset "' + presetName + '" already exists.');
		return;
	}
 	```

### Prepare Box and Pallet Data
- Fetch Pallet data

	```Javascript
 	var specificRectangleId = "PALLETTE_FIX";
	var specificRectangle = TcHmi.Controls.get(specificRectangleId);
	if (specificRectangle) {
		rectanglesData[specificRectangleId] = {
			top: specificRectangle.getTop(),
			left: specificRectangle.getLeft(),
			width: specificRectangle.getWidth(),
			height: specificRectangle.getHeight(),
			tall: palettall
		};
	}
 	```
- Fetch Box data

	```Javascript
 	for (var i = 0; i < rectanglesArray.length; i++) {
		var rectangleId = rectanglesArray[i];
		var rectangle = TcHmi.Controls.get(rectangleId);
		if (rectangle) {
			var transform = rectangle.getTransform();
			rectanglesData[rectangleId] = {
				top: rectangle.getTop(),
				left: rectangle.getLeft(),
				width: rectangle.getWidth(),
				height: rectangle.getHeight(),
				tall: boxtall
			};
 		}
	}
 	```
	- Check if Box overhang on top of Pallet

		```Javascript
  		if (rectangle.getLeft() < specificRectangle.getLeft() || rectangle.getTop() < specificRectangle.getTop() || (rectangle.getLeft()+rectangle.getWidth()*0.75) > (specificRectangle.getLeft()+specificRectangle.getWidth()*0.75) || (rectangle.getTop()+rectangle.getHeight()*0.75) > (specificRectangle.getTop()+specificRectangle.getHeight()*0.75)) 
			{
				alert('WARNING!\none or more boxes exceed the palette limit');
				return;
			}
	 	```
	- Check if Box have collision between other Boxes

		```Javascript
  		for (var j = i + 1; j < rectanglesArray.length; j++) {
			var rect1 = TcHmi.Controls.get(rectanglesArray[i]);
			var rect2 = TcHmi.Controls.get(rectanglesArray[j]);
			if (rect1.getLeft() < rect2.getLeft() + rect2.getWidth()*0.75 &&
			    rect1.getLeft() + rect1.getWidth()*0.75 > rect2.getLeft() && 
			    rect1.getTop() < rect2.getTop() + rect2.getHeight()*0.75 &&
			    rect1.getTop() + rect1.getHeight()*0.75 > rect2.getTop()) {
			    alert('WARNING!\nCollision detected between ' + rectanglesArray[i] + ' and ' + rectanglesArray[j]);
			    return;
			}
		}
		```
- Store Box and Pallet data in a JSON format

	```Javascript
 	var data = {name:presetName, data:rectanglesData};
 	```

### Save Preset Layer as a .CSV file
- Fetch `rotationStates` from Internal Symbol

	```Javascript
 	var rotationStates = JSON.parse(TcHmi.Symbol.read('rotationStates', TcHmi.SymbolType.Internal));
 	```
- Rearrange the data in a suitable format and then Store it in an Array variable

	```Javascript
 	for (var boxName in data.data) {
		var box = data.data[boxName];
		
		let currentBox = [presetName, boxName, box.left, box.top, box.height, box.width, box.tall];
		arr.push(currentBox);
		console.log(arr);
	}
 	```
- Transfer the Array and `rotationStates` to PLC variable

	```Javascript
 	for (var k = 0; k < arr.length; k++) {
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][0]%/s%', arr[k][0]);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][1]%/s%', arr[k][1]);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][2]%/s%', arr[k][2]);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][3]%/s%', arr[k][3]);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][4]%/s%', arr[k][4]);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][5]%/s%', arr[k][5]);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][6]%/s%', arr[k][6]);
		TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.Box_Data['+ k +'][7]%/s%', rotationStates[arr[k][1]]);
	}
 	```
- Transfer the Preset Name to PLC variable

	```Javascript
 	TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.WriteFileName%/s%', presetName);
 	```
- Execute the PLC Program to save the data in .CSV file

	```Javascript
 	TcHmi.Symbol.writeEx('%s%PLC1.CSV_WriteBOXToCSV.bGenerate%/s%', true);
	```
	- Wait for the process to finish

		```Javascript
  		await new Promise(resolve => {
			let intervalBuffer = setInterval(() => {
				TcHmi.Symbol.readEx2('%s%PLC1.CSV_WriteBOXToCSV.WriteDone%/s%', function (data7) {
					if (data7.value == true) {
						clearInterval(intervalBuffer);
						resolve();
					}
				});
			}, 100);
		});
	 	```

### Save Preset Name as a separate .CSV file
- Transfer the Preset Name to PLC variable

	```Javascript
 	TcHmi.Symbol.writeEx('%s%PLC1.CSV_WritePRESETToCSV.sPresetName%/s%', presetName);
 	```
- Execute the PLC Program to save Preset Name in .CSV file

	```Javascript
 	TcHmi.Symbol.writeEx('%s%PLC1.CSV_WritePRESETToCSV.bGenerate%/s%', true, function (writeData) {
			if (writeData.error === TcHmi.Errors.NONE) {
				alert('SUCCESS!\nPreset "' + presetName + '" has been saved.');
 			}
 	});
 	```

### Reset all
```Javascript
var myArray = TcHmi.Symbol.read('RectIndex', TcHmi.SymbolType.Internal);	// fetch RectIndex

for (var i = 1; i < myArray.length; i++) {					// destroy all current boxes
	var elementToRemove = TcHmi.Controls.get(myArray[i]);
	if (elementToRemove) {
		elementToRemove.destroy();
	}
}

myArray = [myArray[0]];								// set an empty Array
TcHmi.Symbol.writeEx2('%i%RectIndex%/i%', myArray);				// write the array back to RectIndex
TcHmi.Symbol.writeEx('%i%rotationStates%/i%', {"PALLETTE_FIX": 0});		// empty rotationStates to only contain Pallet data
TcHmi.Controls.get('TcHmiCombobox').setSelectedIndex(0);			// reset combobox to index zero
TcHmi.Symbol.writeEx2('%i%BoxCounter%/i%', 0);					// reset BoxCounter to zero
TcHmi.Controls.get('panjangBOX').setValue(0);					// reset textbox input to zero
TcHmi.Controls.get('lebarBOX').setValue(0);					// reset textbox input to zero
TcHmi.Controls.get('namaPreset').setText('');					// reset textbox input to empty
TcHmi.View.load('LandingPage.view');						// return the page to Landing Page
```
