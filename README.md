


# KRPano Developer Library
Version 4.0.0 (September 8, 2024)

This is a tiny library with powerful daily tools for krpano developer. It helps save time and produce more legible code.
devlib2.xml is now legacy, devlib4.kml is current version.

## Installation
Place it in your project folder and include in usual way.

## Overview
- `Invisible_Content` style for smart visibility management of any displayed object;
- anonymous function to set `window.krpano` global variable in window object so all krpano functionality becomes available anywhere in JS code
- `window._k` object with a lot of useful abbreviations for window.krpano object internal methods
- `console` calls for easy debug output;
- *(removed from v4)* `asynccall` shortcut (in addition to `callwhen`);
- *(removed from v4)* `reliable_width` and `reliable_height` styles to asynchronously obtain and handle textfield dimensions when they are rendered. The arbitary(unset) textfield dimensions are not available at the moment when textfield is created. That's why code needs to wait until sizes are defined by krpano engine; 
- `new`, `newhotspot`, `newlayer` and `newplugin` – calls to support object-oriented style of code. They make code A LOT shorter and manage inheritance in a distinct way;
- `this` – just a tag to make aliases;
- `get_this` – action to copy object link to `this` ;
- `array_push` – action to add new item to array or create new array with specified name;
- `remove_array` – action to remove all array items.

## More details

### I. Invisible Content
This style helps show or hide objects with smooth fades.
*Invisible_Content and Visible prototypes should go first in prototype list*

#### Usage
* Styles `invisible_content` and `visible` should be written first in list of object styles.

		<layer name="layer_name" style="Invisible_Content|Other_Styles" keep="true"
		  ...
		/>
		<layer name="layer_name" style="Invisible_Content|Visible|Other_Styles" keep="true"
		  ...
		/>

* `Invisible_Content` prototype makes object invisible (`visible="false" alpha="0"`) by default;
* adding `Visible` prototype overrides visibility options to `visible="true"` and `alpha="target_alpha"` 
* Primary methods: `show`, `hide`, `show_fast`, `hide_fast`, `update_alpha`:

		callwith(layer[layer_name], show);
		callwith(layer[layer_name], show_fast);
		callwith(layer[layer_name], hide);
		callwith(layer[layer_name], hide_fast);
		callwith(layer[layer_name], update_alpha);

	The difference between ordinary and fast calls is: fast ones work immediately, ordinary use tweens to fade in or out.
	 Hide method sets `visible="false"` after `alpha` is tweened to 0.

Developer can
* manage show and hide processes by setting variables:
1. `tween_duration_show`, `tween_duration_hide` – sets duration of fade in or fade out;
2. `target_alpha` – sets `alpha` value when object is fully visible (NOTE that `visible` style sets `alpha` to 1);
3. `tween_type` – sets tween type (`default` by default);
4. `show_delay`, `hide_delay` – delays before showing or hiding;
5. `allow_showing` and `allow_hiding` – boolean variables that can be changed to prevent showing or hiding when respective methods are called;

* get flag values:
7.  `is_showing`, `is_hiding`– variables that state if object's `alpha` is being tweened at the moment;
* add custom code in actions:
8.  `show_precall`, `show_fast_precall`, `hide_precall`, `hide_fast_precall` – these are called in the beginning of `show`(`hide`) and `show_fast`(`hide_fast`) calls respectively. They are always executed despite allow flags are set to `false`. These calls also ignore delays;
9. `show_before`, `hide_before` – these are called if allow flags are set to `true` and after delay but before actual changing of alpha has started;
10. `show_after`, `hide_after` – are called when  `alpha` tween completes and `visible` is set to `false`
11. `stop_hiding`, `stop_showing` – stop active tween call;
12. `update_alpha` – if `target_alpha` value has changed this call will update alpha to new target_value. 

NOTE: All parameters are already tuned to deliver nice experience. I change values almost never.

### II. Console
#### Usage
* `console.log(expression);` – calcs expression value and shows it in console;
* `console.divider();` – prints divider line;
* `console.msg('message');` – displays text message;
* `console.var(variable_name);` – displays variable with nice formatting;

See [krpano documentation on expressions](https://krpano.com/docu/actions/#expressions)

### V. New

This is a powerful tool to create new hotspots and layers in an 'object-oriented' way with inheritance management.
See [krpano bundler project](https://github.com/apushkarev/krpano-bundler) for reference

#### Functions

* `newhotspot(hotspot_name, styles);`
* `newlayer(layer_name, styles);`
* `new(style_1|style_2|..., new_object_name, parameter_1, parameter_2, parameter_3, ... parameter_n);`

`newhotspot` and `newlayer` actions create new object with defined style set. Finally they copy a link to new object to `this` alias;

#### Usage

1. Define constructor method in your style. It should have the same name as a style itself. Constructor should call `newhotspot` or `newlayer` actions in first line (depending on what object you want to create). This actions inside constructor have one correct form to call: `newhotspot(%1, %2);` or `newlayer(%1, %2);`.
2. Inheritance works this way: if multiple styles are passed in first argument of a `new` call it will search for last style with defined constructor and call it with all passed arguments.

		<style name="Style_1"
	      ...
	      Style_1="
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

NOTE: if `new` calls are done in cycles or nested constructions you need to keep an eye on what is stored in `this` alias in each given moment of time.

An example with real code:

	new(Invisible_Content|Visible|Dot_Spot,
        calc('dot_spot_' + dot_count),
        get(mouse_ath1),
        get(mouse_atv1),
        get(active_plane_spot.linked_plane)
	);

	<style name="Dot_Spot"
	  url="../img/dot_spot.png"
	  ...
	  ...
	  linked_plane=""
	  ...
	  Dot_Spot="
	    newhotspot(%1, %2);

	    set(this.ath, %3);
	    set(this.atv, %4);

	    set(this.linked_plane, %5);
	    callwith(this, detect_coordinates);

	    inc(dot_count);
	    add_dot_text();
	  "
	  detect_coordinates="
	    ...
	  "
	  add_dot_text="
		  calc(linked_text, name + '_t');
		  new(dot_text,
			  get(linked_text),
			  ...,
			  ...
			);
	  "
	/>
3. `new` call will create an alias `parent` where a link to a caller object will be stored. In a code sample above `new` is called from `add_dot_text` action. So the `parent` alias will store link to newly created `dot_spot` object whether it was a hotspot or a layer.

An example how `parent` alias can be used:

		<style name="Dot_Text"
			parent_spot=""
			
			Dot_Text="
				newhotspot(%1, %2);
			
				copy(this.parent_spot, parent.name);
				copy(this.ath, parent.ath);
				copy(this.atv, parent.atv);
			"
		/>

### VI. this, get_this
`this` alias is handy to save caller object when it calls another object method with `callwith` operator and passes it's own parameters.

		<layer name="some_layer"
		  property1="false"
		  some_action="
		  	...
		  	get_this();
		  	callwith(layer[some_layer2],
			  another_action(get(this.property1));
		  	);
		  "
		/>

### VII. array_push, remove_array actions

	array_push(new_array, 'item_name'); 
	
	is equal to:

	set(new_array['item_name'].name, 'item_name');

To remove array:

	remove_array(new_array);


### Helpful snippets
Snippets and autocompletions are kept [here](https://github.com/apushkarev/Krpano-Markup-Language)

Check this ones:
* ic, sic, sicv, lsi, lsiv, lsikt, lsivkt, hsi, hsiv, hsikt, hsivkt, cwh, cwl, cwt, asynccall, nh, nl, new, cl, cm, cv, cd
