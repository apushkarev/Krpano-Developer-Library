# KRPano Developer Library
This is a tiny library with powerful daily tools for developer. It helps to save time and produce simpler, more legible code.
Place it in your project folder and include in usual way.
## Overview
- **`invisible_content`** style for smart visibility management of any displayed object;
- **`console`** calls for easy debug output;
- **`asynccall`** shortcut (nice addition to **`callwhen`**);
- **`reliable_width`** and **`reliable_height`** styles to obtain and handle textfield dimensions when they are ready;
- **`new`**, **`newhotspot`** and **`newlayer`** – calls to support object-oriented style of code. They make code A LOT shorter and manage inheritance in a distinct way;
- **`this`** – just a tag to make aliases.
## More details
### I. Invisible Content
This style helps to show or hide objects with smooth fades.
#### Usage
* Styles `invisible_content` and `visible` should be applied first in list of object styles.

		<layer name="layer_name" style="invisible_content|other_styles" keep="true"
		...
		/>
		<layer name="layer_name" style="invisible_content|visible|other_styles" keep="true"
		...
	/>

* `invisible_content` makes object invisible (`visible="false" alpha="0"`) by default;
* adding `visible` overrides visibility options to `visible="true" alpha="1"` 
* Primary methods: **`show`**, **`hide`**, **`show_fast`**, **`hide_fast`**:

		callwith(layer[layer_name], show);
		callwith(layer[layer_name], show_fast);
		callwith(layer[layer_name], hide);
		callwith(layer[layer_name], hide_fast);
	The difference between ordinary and fast calls is: fast ones work immediately, ordinary use tweens to fade in or out.
	 Hide method sets `visible="false"` after `alpha` is tweened to 0.
* Developer can manage show and hide processes by setting these variables:
-	**`tween_duration`** – sets duration of fade in and fade out;
-	**`target_alpha`** – sets **`alpha`** value when object is fully visible (NOTE that **`visible`** style sets **`alpha`** to 1);
-	**`tween_type`** – sets tween type (**`default`** by default);
-	**`show_delay`**, **`hide_delay`** – delays before showing or hiding;
-	**`allow_showing`** and **`allow_hiding`** – boolean variables that can be changed to prevent showing or hiding when respective methods are called;
* Getting this flag value:
-	**`tween_in_progress`** – variable that states if object's **`alpha`** is being tweened at the moment;
* Defining custom code in these functions:
-	**`show_precall`**, **`show_fast_precall`**, **`hide_precall`**, **`hide_fast_precall`** – theese are called in the beginning of **`show`**(**`hide`**) and **`show_fast`**(**`hide_fast`**) calls respectively. They are always executed despite allow flags are set to **`false`** and ignore delays;
-	**`show_before`**, **`hide_before`** – these are called if allow flags are set to **`true`** and after delay but before actual changing of alpha has started;
-	**`show_after`**, **`hide_after`** – are called when  **`alpha`** tween completes and **`visible`** is set to **`false`**
-	**`stop_tween`** – stops currently working tween call;
### II. Console
#### Usage
* **`console.log(expression);`** – calcs expression value and shows it in console;
* **`console.divider();`** – prints divider line;
* **`console.msg('message');`** – displays text message;
* **`console.var(variable_name);`** – displays variable with nice formatting;

See [krpano documentation on expressions](https://krpano.com/docu/actions/#expressions)
### III. Async call
#### Usage
    asynccall(expression,
    	actions;
    );

Actions will be executed as soon as expression resolves to **`false`**
(NOTE: **`callwhen`** does the same but when expression resolves to **`true`**)
### IV. Reliable width and height
This is a style that overrides **`onloaded`** event to it's own code
#### Status flags:
**`width_available`**, **`height_available`** – are set to true when respective dimension is initialised;

#### User-defined calls
**`onloaded_override`** – overrides onloaded event;
**`onwidth`**, **`onheight`** – are called when **`width`** or **`height`** gets relevant value.

### V. New
This is a powerful tool to create new hotspots and layers in an 'object-oriented' way with inheritance management

#### Functions
**`newhotspot(hotspot_name, styles);`**
**`newlayer(hotspot_name, styles);`**
**`new(style_1|style_2|..., new_object_name, parameter_1, parameter_2, parameter_3, ... parameter_n);`**

#### Usage

1. **`newhotspot`** and **`newlayer`** create new object with defined style set  and copy a link to it to **`this`** alias;
2. Define contructor method in your style. It should have the same name as a style itself:
	
		<style name="style_1"
			...
			style_1="
				newhotspot(%1, %2);
				...
				// typical calls to initialise parameters
				// %2 and %3 will be parameter_1 and parameter_2 values from new call respectievly
				set(this.variable1, %2);
				set(this.variable2, %3);
				callwith(this, your_code);
				...
			"
		/>
3. Inheritance works this way: if multiple styles are passed in first argument the **`new`** call it will search for last style with defined constructor and call it it with all passed arguments.

NOTE: if **new** calls are done in cycles or nested constructions you need to keep an eye on what is stored in **`this`** alias in each given moment of time.

An example with real code:

	new(invisible_content|visible|dot_spot, calc('dot_spot_' + dot_count), get(mouse_ath1), get(mouse_atv1), get(active_plane_spot.linked_plane) );
	
	<style name="dot_spot"
		url="../img/dot_spot.png"
		...
		...
		linked_plane=""
		...
		dot_spot="
			newhotspot(%1, %2);

			set(this.ath, %3);
			set(this.atv, %4);

			set(this.linked_plane, %5);
			callwith(this, detect_coordinates);

			inc(dot_count);
		"
		detect_coordinates="
			...
		"
	/>