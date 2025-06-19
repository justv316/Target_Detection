# Target Detection Documentation
 Purpose: This is a document detailing everything you need to know to integrate Target Detection into your own mod. It is encouraged to read this entire document at least once, referring to the example plugin and template script files, before starting to integrate this into your mod. 
 Languages Used: ObScript
 Estimated Difficulty: Intermediate
 Required Knowledge: Existing Oblivion/Remastered modding & scripting experience. 
	Familiarity with Global Variables and how to create them; types of spells and effect scripts; Quests and quest scripts; XMarker References.
 License and Usage: This has been given the MIT License. Please refer to the license document for full information, but basically, you can use this and change it and do whatever you want with it so long as you properly credit me. This resource is exclusively being distributed via its GitHub repository, and is being posted on Nexus linking to the repository. 
	You must link in your mods credits to either the GitHub repository (https://github.com/justv316/Target_Detection) or the Nexus Mod Page (Insert Mod Page once its created) Failure to do so is cringe, uncool, and plagiarism, so please don't do that.  

	
## Description and Disclaimer
 This is a modders resource to dynamically collect and manage nearby actor references based on modder defined conditionals. This is accomplished by a spell script storing a reference as a temporary reference in the quest script, storing that temporary reference as a numbered reference, and then clearing the temporary reference variable. Once the reference is managed, the modder can do whatever they wish to it. This template is taken from my own version of this where I am using it to determine valid targets for an aura effect applied by wearing 6 pieces of matching gear. The script that handles the set bonuses sets a global variable (!bDebuffConditional) that tells the TargetDetectionSpellQuestScript to start looking for targets. 
 NOTE: While you are free to change anything, this is being published with the intention that you only change the "A. ScriptVariables to be set by Modder" to match your mod. Changing the structure of the script outside of what is advised can lead to unpredicatable results. 
 
## Plugin Setup
 Before you begin scripting you must create a few things in your plugin. These will be used later in the script.
   ScriptVariable These are Variables to be set by the modder in the template script files. These tables associates the ScriptVariable with what they are in your mod.
	
###	Cell: 
		
#### Description: This cell is used to initially place the XMarker References that will be used to manage effects.
#### Instructions: 
1. Create new cell, set the EditorID to HoldingCell or etc.
2. Inside the cell, place 4 XMarkerHeading references. 
3. Name each XMarker to suit your mod.
| ScriptVariable | Example | Description |
| ------------- | ------------- |:-------------:|
| !rCasterRef1 | TECasterRef | Used to apply the Target Detection AOE to the player. |
| !rCasterRef2 | TECasterRef2 | Used to apply the Target Detected spell effect to detected actors. |
| !rDebuffCaster | TEAuraRef | Used to apply debuffs to the managed references. |
| !rDispelCaster | TEDispelRef | Used to dispel applied effects. |
			
###	Quests:
	
#### Description: These quests are used to run the Target Detection scripts
#### Instructions: 
1. Create two new quests. 
2. Name each to suit your mod. 
3. Set them to Start Game Enabled.
| ScriptVariable | Example | Description |
| ------------- | ------------- |:-------------:|
| !TargetDetectionQuest | TEQQTargetDetectionQuest | Runs Target Detection and the management of references. This is the Quest that is running TargetDetectionQuestScript. |
| !TargetDetectionSpellQuest | TEQQTargetDetectionSpellQuest | Used to determine whether or not Target Detection Should be running. This is the Quest that is running TargetDetectionSpellQuestScript.
	
	
###	Spells: 
	
#### Description: These spells are used to facilitate various parts of Target Detection/Management.
#### Instructions: 
1. Create the necessary spells. 
2. Name them to suit your mod - Each spell has its own details.
| ScriptVariable | Example | Description | Details | Flags
|-------------|:-------------:|:-------------:|:-------------:|:-------------:|
| !SpTargetDetection | TESpTargetDetection | Used to apply the Script Effect 'TargetDetectionEffectScript' to all actors in the spells distance | Touch; 0 Mag; 750 Area; 3 Seconds; Script Effect (Will be: TargetDetectionSpellQuestScript) | Touch Spell Explodes w/ no Target; Immune to Silence; Area Effect Ignores LOS; Script Effect Always Applies; Disallow Spell Absorb/Reflect
| !SpTargetDetected | TESpTargetDetected | Used to determine whether or not a target is being managed or not | Touch; 0 Mag; 0 Area; 600 Seconds; Script Effect (Will be: TargetDetectedEffectScript) | Immune to Silence; Script Effect Always Applies; Disallow Spell Absorb/Reflect
| !SpDispel | TESpAuraDispelDebuff |  Used to dispel applied effects. | Touch; 1000 Mag; 0 Area; 0 Seconds; Dispel | Immune to Silence; Disallow Spell Absorb/Reflect
| !SpDebuffEffect | TESpAuraCursedDebuff | This is the actual effect you are applying. It doesn't necessarily need to be a spell effect. | Touch; Variable Magnitude; 0 Area; 600 Seconds; Variable Effect | (Variable) Immune to Silence; Disallow Spell Absorb/Reflect
	
###	Global Variables: 
		
#### Description: These are conditionals for whether or not Target Detection should be running, or is running. 
#### Instructions: 
1. Create two new Global Variables. 
2. Name them to suit your mod. Each are shorts. 
| ScriptVariable | Example | Description |
| ------------- | ------------- |:-------------:|
| !bTargetDetection | TEbTargetDetection | Determines whether or not Target Detection is running
| !bDebuffConditional | TEbAuraCursed | Determines whether or not Target Detection should be running. This can be whatever you need it to be. 

In the template example, a script that tracks how many of the same set item the player is wearing, and gives them a buff in the form of an ability carrying an effect script that does various things, and sets this Global Variable to 1. The TargetDetectionSpellQuestScript tracks this global to turn on target detection. 

## C) Scripts
Now that the plugin is setup, we can write our scripts and attach them where needed. 

NOTE: I encourage you to write your scripts in Notepad++ (or your preferred IDE) first before adding them to your esp. I also urge you to use the Construction Set Extender https://www.nexusmods.com/oblivion/mods/36370 
I do all of my testing in Remaster exclusively. 

There are 4 scripts that need to be setup. I've included templates of these scripts. You will need to Find and Replace the ScriptVariables to match what you've created above. Also be sure to change the ScriptName to suit your mod.

###	1. TargetDetectionSpellQuestScript
This is the QuestScript for !TargetDetectionSpellQuest

#### ScriptVariables to be set by Modder:
1. !bDebuffConditional
2. !bTargetDetection
3. !rCasterRef1
4. !SpTargetDetection

#### Explanation:
Initially, this runs once per second to see if Target Detection should be running, then every 3 seconds (The duration of !SpTargetDetection) to check for new targets. 
!rCasterRef1 is used to cast !SpTargetDetection on the player, applying TargetDetectionEffectScript to all actors in its range (750 by default) and then immediately dispel the effect on the player so they do not see the Script Effect in their magic effects.


###	2. TargetDetectionEffectScript
This is the script magic effect for the !SpTargetDetection spell that is ran on nearby actors. 

#### ScriptVariables to be set by Modder:
1. !TemplateTargetDetectionQuest
2. !SpTargetDetected

#### Explanation:
The ScriptEffectStart block is ran when the Spell effect is first applied. It sets the default variables. 
The ScriptEffectUpdate block starts to run and starts a timer NOTE: Without this timer check this block takes upwards of 5 seconds to update, for probably Remastered reasons. After 0.3 seconds it will check to see if TemplateTargetDetectionQuest.rIncomingRef is available to store a reference. TargetDetectionQuestScript stores that rIncomingRef as the next available numbered Reference and clears the variable for re-use. Another reference can be stored in the following frame. 
If rIncomingRef is full, the script needs to wait and then try, and so sets the bMustWait variable to 1, causing the script to loop after another 0.2 seconds until the reference is stored
The Conditional on Line 21 (if Player.GetDistance rSelf <= 750 && rSelf.GetCombatTarget == Player) defines what a valid target is in your mod. The template dictates that a valid target for storage is one within 750 units and is in combat with the player. You can change this conditional to be quite literally anything to suit your needs. This distance matches the range of the !SpTargetDetection spell. 
The ScriptEffectFinish block exists to force the script to stop after the spell effects duration (3 seconds) has expired. NOTE: Without this it just doesn't work, for probably Remastered reasons.


###	3. TargetDetectedEffectScript
This is a blank script attached to the !SpTargetDetected spell that tells us if a target has already been detected.

###	4. TargetDetectionQuestScript
This is the QuestScript for !TargetDetectionQuest 
	 
#### ScriptVariables to be set by Modder:
1. !bDebuffConditional
2. !rDebuffCaster
3. !rCasterRef2
4. !rDispelCaster
5. !SpDebuffEffect
6. !SpTargetDetected
7. !SpDispel
	 
#### Explanation:
This script manages references and does things to them. This runs every 0.05 seconds. 
The first block (if rIncomingRef != 0) stores references that are given to it from the TargetDetectionEffectScript. Every script tick it will check to see if a reference needs stored, stores it, and clears the incoming reference variable.

The following blocks are where things are done to references based on conditionals.
The first conditional (if rRef1.IsSpellTarget !SpTargetDetected == 0) checks if the stored reference is affected by the !SpTargetDetected spell and moves our 2nd CasterRef to it then applies that effect to it. Now that the effect is active we know that specific actor is being tracked and to not track it again, and we move on to if the actor is affected by !SpTargetDetected, we track if the reference is dead or alive (if rRef1.GetDead == 0 elseif rRef1.GetDead == 1) and we track if we've applied a Debuff to the reference (if bRef1Debuff == 0 elseif bRef1Debuff == 1). 
Within the Debuff block, we define what we want to do to the reference and conditionals to do those things. (if !bDebuffConditional == 1) If we want to apply a effect to the reference, we must use a CasterRef to do so, as AddSpell applies the spell to the base actor, effecting every version of that actor until removed. Once the Debuff is applied, we track whether or not the actor should still have the debuff. For example, if the reference leaves a certain range of the player, remove the debuff. We must again use a CasterRef to dispel the debuff (When I tried using rRef1.Dispel SpDebuffEffect it just didn't work?) This block can be expanded to fit your specific situation as when whether or not a target should be effected by the debuff is entirely up to you.
The 2 conditionals I have in the template are (if Player.GetDistance rRef1 > 150) and (if rRef1.GetCombatTarget != Player). These are the opposite of the conditionals established in TargetDetectionEffectScript


## Final Steps
### Now that everything is created:
 
 1. Assign !SpTargetDetection the Scripted Effect TargetDetectionEffectScript. 
 2. Assign !SpTargetDetected the Scripted Effect TargetDetectedEffectScript. 
 3. Assign !TargetDetectionQuest the QuestScript TargetDetectionQuestScript. 
 4. Assign !TargetDetectionSpellQuest the QuestScript TargetDetectionSpellQuestScript.
 
 Target Detection is now set up to work in your mod. 

##Troubleshooting!


Changing the effect on !SpTargetDetected to a visual effect can help you visually confirm whether or not an actor is being tracked.

If an XMarker is tasked to do 2 or more different things at the exact same frame, the game will crash, therefore I use 4 distinct XMarker caster's to cast spells to avoid that.

##Final Thoughts
 
If you require support with this, you can find me in the Oblivion Remastered Modding Community Discord or the /r/OblivionMods Discord as Fox/TheFoxOfKvatch. You can also send me a message on Nexus and I will eventually respond to it. I prefer to not be added in Discord until after we chat. 