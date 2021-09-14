# Gantry bowing-induced Z-offset correction through relative reference index
A guide on how to minimize inconsistent Z offset caused by gantry bowing on Klipper machines, particularly Voron 2.4. Are your first layers squished too much when the printer is too cold (first print of the day), and too far from the bed when it's nice and toasty? Or the other way around? Do you have to heatsoak for hours just to get that nice crisp first layer? Read on.

# What?
Does your printer have steel linear rails bolted to aluminium extrusions? Is the entire printer sealed up? Then you might be suffering from gantry bowing due to bimetallic thermal expansion. Read more in [whoppingpochard's excellent repo](https://github.com/tanaes/whopping_Voron_mods/tree/main/extrusion_backers). I'll keep it short here - if you have your rails mounted on the bottom of your extrusions, as the gantry member heats up, it bows upwards in the middle. If both your X and Y are set up this way, then the combined bowing is most prominent in the middle of the bed, and least prominent in the corners.
You might have noticed that when you take a bed mesh when the printer is hot, it has a distinct "bowl" shape. This is because the probe has to travel further down to trigger in the middle of the bed, and shorter in the corners - like this:

![hot_mesh](https://user-images.githubusercontent.com/61467766/132994563-c2806d2c-62d9-4998-b55d-57e2504ad0ca.JPG)

But when your printer is at ambient temp (say, cooled down overnight), the mesh looks more like this:

![cold_mesh](https://user-images.githubusercontent.com/61467766/132994585-c49b21d7-e49f-4aef-88dd-979ca8468449.JPG)

# How the hell does *that* have anything to do with Z offset?!

You have probably been told to set a thing called "relative\_reference\_index" in your bed_mesh config section to (x points * y points) - 1) / 2. This tells klipper that the middle point of the mesh (usually center of the bed) - exactly where you were probably doing your initial Z offset calibration (with a feeler gauge or a sheet of printer paper) - is Z=0. But in reality, this is the most thermally unstable point of the bed mesh with changing temperature of the gantry members, so your "Z=0" is now a moving target. For reference, on my 300mm V2.4, this difference in Z offset between "heatsoaking for 10 minutes" and "printing for 3 hours" is roughly 80 microns. Sounds little, but it is more than enough to give you a "perfect squish" for the first and "can see the second layer through the first one" for the second option.

Your bed mesh still works fine, you get a (hopefully) consistent first layer - but it will have a *slightly-to-moderately* different global Z offset every time. If you preheat for a long time, this effect is minimized, because you probably have everything set up to work at thermal equilibrium. But if you don't want to waste time heatsoaking for hours, there is a solution.

# What can I do about it?
You can get gantry backers, which can eliminate most (if not all) of the gantry bowing effects. That is the gucci solution, but costs money, and on large printers it might not be enough.
You can also set the relative reference index to a more thermally stable position. Any of the corners of your bed mesh should be fine. I personally use the last one, i.e. my RRI is set to (x points * y points) - 1, for a 5x5 mesh that is 24. This tells klipper that the far right corner is where Z = 0, and when the gantry bows, the middle is at a negative number. The mesh is effectively the same shape, but shifted in Z, and the reference for Z=0 is now much more stable (i.e., a not-that-much-moving target).

## That's it?
No! Remember that this directly affects your Z offset, which can cause a catastrophic nozzle strike if you are not careful. It will be best if you redo your Z offset calibration outright (remember to measure in the corner of your choice, not in the middle of the bed!)

There is another way which worked for me (but I am absolutely definitely not telling you to do the same, ever) - take your old bed mesh that works for you (i.e. all your Z offset setup works with *this one mesh*). It has a value of 0 in the middle. The corner of your choice will have a large value, say 0.15 mm. If you now move the RRI to that corner (and take the mesh again to update it!), your middle point will measure -0.15 mm. You need to *add* this value to your position_endstop, i. e. if your old position_endstop was -1.2, it will be -1.35 after changing RRI to the corner. **IF YOU USE A DIFFERENT ENDSTOP SYSTEM, THINK TWICE ABOUT WHAT YOU ARE DOING**. Whatever system you use, your end result needs to be that your nozzle is *further from the bed* than it was before. Be careful, if you are not sure, just redo the Z calibration. I don't want to be responsible for your PEI.

## But I load a previously-taken mesh instead of taking a fresh one before every print

You shouldn't do that unless you always print from the same start conditions. But if you are a rebel and want to do it anyway, you will have to take a new mesh, the old one is tied to the old RRI (has a 0 in the middle). Or just manually subtract the corner's value from all points, you renegade.

#### **WARNING**
If you set up your mesh at hot thermal equilibrium and then try to use it when the printer is cold, depending on the height delta this **MIGHT** result in a nozzle strike. I *strongly* recommend that you always take a fresh mesh before each print.

## There is always a catch

Yes. If your first layer takes a long time and you don't heatsoak long enough, the gantry bowing will increase *during* the first few layers, but your mesh will stay the same. What can you do about it? Ignore it, heatsoak longer or get gantry backers.

### Did you just invalidate this entire thing?

No, setting RRI to a corner might not be perfect, but it still helps a ton, especially for small footprint objects with short first layer time.

### But there has to be another way?!

Yes, there is. You can change the mesh dynamically based on the printer's temperature with [Virtual Gantry Backers](https://github.com/Deutherius/VGB)! :)

## But my rails are on the top, not on the bottom

The effects will be reversed and your bed mesh probably looks like an upside-down bowl. The main point still stands - the middle of the bed is the most thermally unstable reference for Z=0. Change it to a corner and redo the Z calibration.

# Results (yours may vary)

## RRI in the middle of the bed

Print started from cold printer state (heatsoak for ~10 minutes, calibrated for my squish preferences) - this one sparks joy

![20210822_163323](https://user-images.githubusercontent.com/61467766/132995533-47ec8428-063f-4c85-880f-78b2ce3d32a0.jpg)

Print started from a hot printer state (been printing for hours) - this one does not spark joy

![20210822_163518](https://user-images.githubusercontent.com/61467766/132995554-e72db181-300a-4c9b-8352-a351b62a164b.jpg)

## RRI far right corner

Print started from a cold state

![20210912_190246](https://user-images.githubusercontent.com/61467766/132996368-07eb553e-39a6-4a99-a5ae-365a7c92e510.jpg)

Print started from a hot state

![20210912_190219](https://user-images.githubusercontent.com/61467766/132996429-9d7d6c91-46dc-43c0-b77b-b980c4df47b8.jpg)

