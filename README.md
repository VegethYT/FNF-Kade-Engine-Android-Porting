# FNF-Kade-Engine-Android-Porting:

The things I use when porting a Kade Engine mod

**This should be used for every Kade Engine version except Kade Engine 1.6 and above**

### PC compile instructions For Android:

1. Download
* [JDK](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html) - Download the version `19` of it
* [Android Studio](https://developer.android.com/studio) - I recomend you to download the latest version
* [NDK](https://developer.android.com/ndk/downloads/older_releases?hl=fi) - Download the version `r21e` (This is the version recomended by Lime)

2. Install JDK, Android Studio 
Unzip the NDK (the NDK does not need to be installed because its a zip archive)

3. We need to set up Android Studio for this go to android studio and find android sdk (in settings -> Appearance & Behavior -> system settings -> android sdk)
![first pic](https://user-images.githubusercontent.com/59097731/104179652-44346000-541d-11eb-8ad1-1e4dfae304a8.PNG)
![second pic](https://user-images.githubusercontent.com/59097731/104179943-a9885100-541d-11eb-8f69-7fb5a4bfdd37.PNG)

4. Run command `lime setup android` in CMD/PowerShell (You need to insert the program paths)

5. Open project in CMD/PowerShell `cd (path to fnf source)`
And run command `lime build android -final`
The apk will be generated in this path (path to source)\export\release\android\bin\app\build\outputs\apk\debug
_____________________________________

## Instructions:

1. You Need to install extension-androidtools

To Install it You Need To Open Command prompt/PowerShell And Type
```cmd
haxelib git extension-androidtools https://github.com/MAJigsaw77/extension-androidtools.git
```

2. Download the repository code and paste it in your source code folder

3. You Need to add these things in project.xml

On This Line
```xml
	<!--Mobile-specific-->
	<window if="mobile" orientation="landscape" fullscreen="true" width="0" height="0" resizable="false" />
```

Replace It With
```xml
	<!--Mobile-specific-->
	<window if="mobile" orientation="landscape" fullscreen="true" resizable="false" allow-shaders="true" require-shaders="true" allow-high-dpi="true" />
```

Add
```xml
	<assets path="mobile" rename="assets/mobile" if="mobile" /> 
```

Then, After the Libraries, or where the packeges are located add
```xml
	<haxelib name="extension-androidtools" if="android" />
```

Add
```xml

	<!--Allow working memory greater than 1 Gig-->
	<haxedef name="HXCPP_GC_BIG_BLOCKS" />

	<!--Always enable Null Object Reference check-->
	<haxedef name="HXCPP_CHECK_POINTER" />
	<haxedef name="HXCPP_STACK_LINE" />
	<haxedef name="HXCPP_STACK_TRACE"/>

	<section if="android">
		<!--Permissions-->
		<config:android permission="android.permission.ACCESS_NETWORK_STATE" />
		<config:android permission="android.permission.ACCESS_WIFI_STATE" />
		<config:android permission="android.permission.INTERNET" />
		<config:android permission="android.permission.WRITE_EXTERNAL_STORAGE" />
		<config:android permission="android.permission.READ_EXTERNAL_STORAGE" />
		<config:android permission="android.permission.VIBRATE" />
		<config:android target-sdk-version="29" if="${lime <= 8.0.0}" />

		<!--Gradle-->
		<config:android gradle-version="7.6" gradle-plugin="7.3.1" />
	</section>

	<section if="ios">
		<!--Dependency--> 
		<dependency name="Metal.framework" />
		<dependency name="WebKit.framework" />
	</section>
```

4. Setup Controls.hx

After these lines
```haxe
import flixel.input.actions.FlxActionSet;
import flixel.input.keyboard.FlxKey;
```

Add
```haxe
#if mobile
import mobile.flixel.FlxButton;
import mobile.flixel.FlxHitbox;
import mobile.flixel.FlxVirtualPad;
#end
```

Before these lines
```haxe
override function update()
{
	super.update();
}
```

Add
```haxe
	#if mobile
	public var trackedInputsUI:Array<FlxActionInput> = [];
	public var trackedInputsNOTES:Array<FlxActionInput> = [];

	public function addButtonNOTES(action:FlxActionDigital, button:FlxButton, state:FlxInputState)
	{
		var input:FlxActionInputDigitalIFlxInput = new FlxActionInputDigitalIFlxInput(button, state);
		trackedInputsNOTES.push(input);
		action.add(input);
	}

	public function addButtonUI(action:FlxActionDigital, button:FlxButton, state:FlxInputState)
	{
		var input:FlxActionInputDigitalIFlxInput = new FlxActionInputDigitalIFlxInput(button, state);
		trackedInputsUI.push(input);
		action.add(input);
	}

	public function setHitBox(Hitbox:FlxHitbox)
	{
		inline forEachBound(Control.UP, (action, state) -> addButtonNOTES(action, Hitbox.buttonUp, state));
		inline forEachBound(Control.DOWN, (action, state) -> addButtonNOTES(action, Hitbox.buttonDown, state));
		inline forEachBound(Control.LEFT, (action, state) -> addButtonNOTES(action, Hitbox.buttonLeft, state));
		inline forEachBound(Control.RIGHT, (action, state) -> addButtonNOTES(action, Hitbox.buttonRight, state));
	}

	public function setVirtualPadUI(VirtualPad:FlxVirtualPad, DPad:FlxDPadMode, Action:FlxActionMode)
	{
		switch (DPad)
		{
			case UP_DOWN:
				inline forEachBound(Control.UP, (action, state) -> addButtonUI(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonUI(action, VirtualPad.buttonDown, state));
			case LEFT_RIGHT:
				inline forEachBound(Control.LEFT, (action, state) -> addButtonUI(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonUI(action, VirtualPad.buttonRight, state));
			case UP_LEFT_RIGHT:
				inline forEachBound(Control.UP, (action, state) -> addButtonUI(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonUI(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonUI(action, VirtualPad.buttonRight, state));
			case LEFT_FULL | RIGHT_FULL:
				inline forEachBound(Control.UP, (action, state) -> addButtonUI(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonUI(action, VirtualPad.buttonDown, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonUI(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonUI(action, VirtualPad.buttonRight, state));
			case BOTH_FULL:
				inline forEachBound(Control.UP, (action, state) -> addButtonUI(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonUI(action, VirtualPad.buttonDown, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonUI(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonUI(action, VirtualPad.buttonRight, state));
				inline forEachBound(Control.UP, (action, state) -> addButtonUI(action, VirtualPad.buttonUp2, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonUI(action, VirtualPad.buttonDown2, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonUI(action, VirtualPad.buttonLeft2, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonUI(action, VirtualPad.buttonRight2, state));
			case NONE: // do nothing
		}

		switch (Action)
		{
			case A:
				inline forEachBound(Control.ACCEPT, (action, state) -> addButtonUI(action, VirtualPad.buttonA, state));
			case B:
				inline forEachBound(Control.BACK, (action, state) -> addButtonUI(action, VirtualPad.buttonB, state));
			case A_B | A_B_C | A_B_E | A_B_X_Y | A_B_C_X_Y | A_B_C_X_Y_Z | A_B_C_D_V_X_Y_Z:
				inline forEachBound(Control.ACCEPT, (action, state) -> addButtonUI(action, VirtualPad.buttonA, state));
				inline forEachBound(Control.BACK, (action, state) -> addButtonUI(action, VirtualPad.buttonB, state));
			case NONE: // do nothing
		}
	}

	public function setVirtualPadNOTES(VirtualPad:FlxVirtualPad, DPad:FlxDPadMode, Action:FlxActionMode)
	{
		switch (DPad)
		{
			case UP_DOWN:
				inline forEachBound(Control.UP, (action, state) -> addButtonNOTES(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonNOTES(action, VirtualPad.buttonDown, state));
			case LEFT_RIGHT:
				inline forEachBound(Control.LEFT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonRight, state));
			case UP_LEFT_RIGHT:
				inline forEachBound(Control.UP, (action, state) -> addButtonNOTES(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonRight, state));
			case LEFT_FULL | RIGHT_FULL:
				inline forEachBound(Control.UP, (action, state) -> addButtonNOTES(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonNOTES(action, VirtualPad.buttonDown, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonRight, state));
			case BOTH_FULL:
				inline forEachBound(Control.UP, (action, state) -> addButtonNOTES(action, VirtualPad.buttonUp, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonNOTES(action, VirtualPad.buttonDown, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonLeft, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonRight, state));
				inline forEachBound(Control.UP, (action, state) -> addButtonNOTES(action, VirtualPad.buttonUp2, state));
				inline forEachBound(Control.DOWN, (action, state) -> addButtonNOTES(action, VirtualPad.buttonDown2, state));
				inline forEachBound(Control.LEFT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonLeft2, state));
				inline forEachBound(Control.RIGHT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonRight2, state));
			case NONE: // do nothing
		}

		switch (Action)
		{
			case A:
				inline forEachBound(Control.ACCEPT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonA, state));
			case B:
				inline forEachBound(Control.BACK, (action, state) -> addButtonNOTES(action, VirtualPad.buttonB, state));
			case A_B | A_B_C | A_B_E | A_B_X_Y | A_B_C_X_Y | A_B_C_X_Y_Z | A_B_C_D_V_X_Y_Z:
				inline forEachBound(Control.ACCEPT, (action, state) -> addButtonNOTES(action, VirtualPad.buttonA, state));
				inline forEachBound(Control.BACK, (action, state) -> addButtonNOTES(action, VirtualPad.buttonB, state));
			case NONE: // do nothing
		}
	}

	public function removeVirtualControlsInput(Tinputs:Array<FlxActionInput>)
	{
		for (action in this.digitalActions)
		{
			var i = action.inputs.length;
			while (i-- > 0)
			{
				var x = Tinputs.length;
				while (x-- > 0)
				{
					if (Tinputs[x] == action.inputs[i])
						action.remove(action.inputs[i]);
				}
			}
		}
	}
	#end
```

5. Setup MusicBeatState.hx and MusicBeatSubstate.hx.

In the lines you import things add
```haxe
#if mobile
import mobile.flixel.FlxHitbox;
import mobile.flixel.FlxVirtualPad;
import flixel.FlxCamera;
import flixel.input.actions.FlxActionInput;
import flixel.util.FlxDestroyUtil;
#end
```

After these lines
```haxe
inline function get_controls():Controls
	return PlayerSettings.player1.controls;
```

Add
```haxe
	#if mobile
	var hitbox:FlxHitbox;
	var virtualPad:FlxVirtualPad;
	var trackedInputsHitbox:Array<FlxActionInput> = [];
	var trackedInputsVirtualPad:Array<FlxActionInput> = [];

	public function addVirtualPad(DPad:FlxDPadMode, Action:FlxActionMode, ?visible = true):Void
	{
		if (virtualPad != null)
			removeVirtualPad();

		virtualPad = new FlxVirtualPad(DPad, Action);
		virtualPad.visible = visible;
		add(virtualPad);

		controls.setVirtualPadUI(virtualPad, DPad, Action);
		trackedInputsVirtualPad = controls.trackedInputsUI;
		controls.trackedInputsUI = [];
	}

	public function addVirtualPadCamera(DefaultDrawTarget:Bool = true):Void
	{
		if (virtualPad != null)
		{
			var camControls:FlxCamera = new FlxCamera();
			FlxG.cameras.add(camControls, DefaultDrawTarget);
			camControls.bgColor.alpha = 0;
			virtualPad.cameras = [camControls];
		}
	}

	public function removeVirtualPad():Void
	{
		if (trackedInputsVirtualPad.length > 0)
			controls.removeVirtualControlsInput(trackedInputsVirtualPad);

		if (virtualPad != null)
			remove(virtualPad);
	}

	public function addHitbox(?visible = true):Void
	{
		if (hitbox != null)
			removeHitbox();

		hitbox = new FlxHitbox();
		hitbox.visible = visible;
		add(hitbox);

		controls.setHitBox(hitbox);
		trackedInputsHitbox = controls.trackedInputsNOTES;
		controls.trackedInputsNOTES = [];
	}

	public function addHitboxCamera(DefaultDrawTarget:Bool = true):Void
	{
		if (hitbox != null)
		{
			var camControls:FlxCamera = new FlxCamera();
			FlxG.cameras.add(camControls, DefaultDrawTarget);
			camControls.bgColor.alpha = 0;
			hitbox.cameras = [camControls];
		}
	}

	public function removeHitbox():Void
	{
		if (trackedInputsHitbox.length > 0)
			controls.removeVirtualControlsInput(trackedInputsHitbox);

		if (hitbox != null)
			remove(hitbox);
	}
	#end

	override function destroy()
	{
		#if mobile
		if (trackedInputsHitbox.length > 0)
			controls.removeVirtualControlsInput(trackedInputsHitbox);

		if (trackedInputsVirtualPad.length > 0)
			controls.removeVirtualControlsInput(trackedInputsVirtualPad);
		#end

		super.destroy();

		#if mobile
		if (virtualPad != null)
			virtualPad = FlxDestroyUtil.destroy(virtualPad);

		if (hitbox != null)
			hitbox = FlxDestroyUtil.destroy(hitbox);
		#end
	}
```

And somehow you finished adding the android controls to your mod.

6. Adding VPAD in the correct states/substates
```haxe
#if mobile
addVirtualPad(LEFT_FULL, A_B);
#end
```
In ChartingState.hx before super.create()

Then in FreeplayState.hx before super.create()

Then in GameOverState.hx before super.create()

Then in GameOverSubstate.hx you have to put
```haxe
#if mobile
addVirtualPad(NONE, A_B);
addVirtualPadCamera();
#end
```
After bf.playAnim('firstDeath');

Then in GitarooPause.hx you have to put
```haxe
#if mobile
addVirtualPad(LEFT_RIGHT, A_B);
#end
```
Before super.create()

Then in MainMenuState.hx you have to put
```haxe
#if mobile
addVirtualPad(UP_DOWN, A_B);
#end
```
Before super.create()

Then in OptionsMenu.hx you have to put
```haxe
#if mobile
addVirtualPad(LEFT_FULL, A_B_C);
#end
```
Before super.create()

Then in Playstate.hx you have to put
```haxe
#if mobile
addHitbox();
#end
```
After scoreTxt.cameras = [camHUD];

Then in StoryMenuState.hx you have to put
```haxe
#if mobile
addVirtualPad(LEFT_FULL, A_B);
#end 
```
Before super.create()

7. Prevent the Android BACK Button

in TitleState.hx

After
```haxe
override public function create():Void
```

Add
```haxe
#if mobile
FlxG.android.preventDefaultKeys = [BACK];
#end
```

8. Set An action to the BACK Button

You can set one with
```haxe
#if mobile || FlxG.android.justReleased.BACK #end
```

9. On `sys.FileSystem`, `sys.io.File` Stuff

This is not working with app storage but on phone storage it will work with this
```haxe
SUtil.getStorageDirectory() + 
```
This will make the game use the phone storage but you will have to add one thing in your source

In Main.hx before 
```haxe
addChild(new FlxGame(gameWidth, gameHeight, initialState, zoom, framerate, framerate, skipSplash, startFullscreen));
```

Add
```haxe
SUtil.checkPermissions();
```

This will check for android storage permisions and the `assets` or `mods` directories

10. On Crash Application Alert

On Main.hx after
```haxe
public function new()
{
	super();
```

Add
```haxe
SUtil.uncaughtErrorHandler();
```

11. File Saver

This is a feature to save files with sys.io.File in phone storage in `saves` directory

```haxe
SUtil.saveContent("your file name", ".txt", "lololol");
```

12. Do an action when you press on the screen

```haxe
#if mobile
var justTouched:Bool = false;

for (touch in FlxG.touches.list)
	if (touch.justPressed)
		justTouched = true;

if (justTouched)
	//Your code
#end
```

## Credits:
* Vegeth me - Porting the code for Kade Engine
* Saw (M.A. JIGSAW) - Doing the rest of the code, utils, pad buttons and other things
* luckydog7 - Original code for mobile controls and hitbox graphic shape original code.
* Goldie - Pad designer.
