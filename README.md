

# KRPano Developer Library
Version 2.5

This is a tiny library with powerful daily tools for developer. It helps to save time and produce simpler, more legible code.
Place it in your project folder and include in usual way.
## Overview
- **`invisible_content`** style for smart visibility management of any displayed object;
- **`console`** calls for easy debug output;
- **`asynccall`** shortcut (nice addition to **`callwhen`**);
- **`reliable_width`** and **`reliable_height`** styles to get and handle textfield dimensions when they are ready. The arbitary(unset) textfield dimensions are not available at the moment when textfield is created, so we need to wait until sizes are defined by krpano engine; 
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
1. **`tween_duration`** – sets duration of fade in and fade out;
2. **`target_alpha`** – sets **`alpha`** value when object is fully visible (NOTE that **`visible`** style sets **`alpha`** to 1);
3. **`tween_type`** – sets tween type (**`default`** by default);
4. **`show_delay`**, **`hide_delay`** – delays before showing or hiding;
5. **`allow_showing`** and **`allow_hiding`** – boolean variables that can be changed to prevent showing or hiding when respective methods are called;
* Getting this flag value:
1.  **`tween_in_progress`** – variable that states if object's **`alpha`** is being tweened at the moment;
* Defining custom code in these functions:
1.  **`show_precall`**, **`show_fast_precall`**, **`hide_precall`**, **`hide_fast_precall`** – these are called in the beginning of **`show`**(**`hide`**) and **`show_fast`**(**`hide_fast`**) calls respectively. They are always executed despite allow flags are set to **`false`**. These calls also ignore delays;
1. **`show_before`**, **`hide_before`** – these are called if allow flags are set to **`true`** and after delay but before actual changing of alpha has started;
2. **`show_after`**, **`hide_after`** – are called when  **`alpha`** tween completes and **`visible`** is set to **`false`**
3. **`stop_tween`** – stops currently working tween call;

NOTE: All parameters are already tuned to deliver nice experience. I change values almost never.

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
Add styles `reliable_width` or `reliable_height` in the end of style list for your object.

    <layer name="layer_name" style="invisible_content|some_text|reliable_height" keep="true"
    	type="textfield"
    	width="300"
    	align="lefttop"
    
    	onheight="
    	    calc(y, height + ...);
    	    show();
    	"
    />

This styles override **`onloaded`** event to it's own code
#### Status flags:
**`width_available`**, **`height_available`** – are set to true when respective dimension is initialised;

#### User-defined calls
* **`onloaded_override`** – here goes your onloaded code;
* **`onwidth`**, **`onheight`** – are called when **`width`** or **`height`** gets relevant value.

### V. New
This is a powerful tool to create new hotspots and layers in an 'object-oriented' way with inheritance management

#### Functions

* **`newhotspot(hotspot_name, styles);`**
* **`newlayer(layer_name, styles);`**
* **`new(style_1|style_2|..., new_object_name, parameter_1, parameter_2, parameter_3, ... parameter_n);`**

**`newhotspot`** and **`newlayer`** actions create new object with defined style set. Finally they copy a link to new object to **`this`** alias;

#### Usage

1. Define contructor method in your style. It should have the same name as a style itself. Constructor should call **`newhotspot`** or **`newlayer`** actions in first line (depending on what object you want to create). This actions inside constructor have one correct form to call: **`newhotspot(%1, %2);`** or **`newlayer(%1, %2);`**.
2. Inheritance works this way: if multiple styles are passed in first argument of a **`new`** call it will search for last style with defined constructor and call it with all passed arguments.

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

NOTE: if **`new`** calls are done in cycles or nested constructions you need to keep an eye on what is stored in **`this`** alias in each given moment of time.

An example with real code:

	new(invisible_content|visible|dot_spot,
        calc('dot_spot_' + dot_count),
        get(mouse_ath1),
        get(mouse_atv1),
        get(active_plane_spot.linked_plane)
	);

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

### Helpful snippets
Snippets and autocompletions are kept [here](https://github.com/apushkarev/Krpano-Markup-Language)

Check this ones:
* ic, sic, sicv, lsi, lsiv, lsikt, lsivkt, hsi, hsiv, hsikt, hsivkt, cwh, cwl, cwt, asynccall, nh, nl, new, cl, cm, cv, cd
