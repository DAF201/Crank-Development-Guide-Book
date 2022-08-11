# Crank_DEV_Guide

## table on contents:
1. [some thing general](#something_general)
2. [UI structure](#UI_structure)
3. [system init](#system_init)
4. [system running](#system_running)
5. [system_structure](#system_structure)
6. [task table](#task_table)
7. [data_structure](#data_structure)
8. [data_IO](#data_IO)


## something_general

----

First of all, Crank is an embedded UI SDK. It allows you to make embedded projects much easier than hand writes every line of code. However, even if you don't have to hand write every line of code, it is still very important to keep the remaining parts "modular", "abstract", and "low coupling".

> ## keep things separate

For example, you have an update everything function that reads data from an external source, saves those data to variables, and displays those data on the screen. This function will be called every second.

Assume for some reason, a piece of data replay took very long. In this case, the function will be stuck on this piece of data, and the remaining data and updates will be stuck too.

A better solution is to separate fetch data, save data, and update screens into three functions, and use a timer to regulate them. (which is pretty much impossible in Lua cause Lua does not support threading. luckily, the event callback trigger is by the UI engine, you don't need to worry about when is the event going to come back).

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/module.png">

(this may not be a good example but just for an general idea about what to do what not to do)

> ## don't repeat yourself

Additionally, there is not sense to make a function for every button with similar outputs. For example, there are six values on the screen, and each of them has an add/sub button near it. There is no sense to write 12 functions such as value_1_add, value_1_sub, and value_2_add... instead, just make value_change(value_path, operation, amount), where value_path represents which value you want to change, and operation represents add/subtract, the amount represents how many you want to add/subtract to it.

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/generalize.png">

(no one want to see 12 functions doing the same thing with different "GLOBAL VARIABLES" with in them)

Trust me, no one wants to read such a thing. You are not here to enum out all possible results

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(33).png">

> ## clarity is everything

Naming is a super important part of software programming. A well-named variable/function should be clear and short if the condition allows. The long long name is acceptable as long as the purpose is clear and easy to understand. Also, it is okay to name something pathlike if necessary.

```lua
-- this function will be called when application starts
function AppStart(mpargs)
end

-- path-like keys, not recommended but only when necessary, as they are from the UI path

action_register_map = {
    ["viscosity_layer.b1_buttons_group1.input_hysteresis_control.text"] = {40022, 1}, -- visc and pressure of the barrels number
    ["viscosity_layer.b1_buttons_group1.b1_input_cutout_control.text"] = {40023, 1},
    ["pressure_layer.b1_buttons_group1.input_hysteresis_control.text"] = {40024, 1},
    ["pressure_layer.b1_buttons_group1.b1_input_cutout_control.text"] = {40025, 1},

    ["viscosity_layer.b2_buttons_group1.input_hysteresis_control.text"] = {40026, 1},
    ["viscosity_layer.b2_buttons_group1.input_Cutout_control.text"] = {40027, 1},
    ["pressure_layer.b2_buttons_group1.input_hysteresis_control.text"] = {40028, 1},
    ["pressure_layer.b2_buttons_group1.input_Cutout_control.text"] = {40029, 1},

    ["viscosity_layer.b3_buttons_group1.input_hysteresis_control.text"] = {40030, 1},
    ["viscosity_layer.b3_buttons_group1.input_Cutout_control.text"] = {40031, 1},
    ["pressure_layer.b3_buttons_group1.input_hysteresis_control.text"] = {40032, 1},
    ["pressure_layer.b3_buttons_group1.input_Cutout_control.text"] = {40033, 1},

    ["viscosity_layer.b4_buttons_group1.input_hysteresis_control.text"] = {40034, 1},
    ["viscosity_layer.b4_buttons_group1.input_Cutout_control.text"] = {40035, 1},
    ["pressure_layer.b4_buttons_group1.input_hysteresis_control.text"] = {40036, 1},
    ["pressure_layer.b4_buttons_group1.input_Cutout_control.text"] = {40037, 1},

    ["Configuration_layer.BOM_control.text"] = {40063, 5}, -- bom string
    ["Configuration_layer.SerialNumber_control.text"] = {40068, 4}, -- serial number string
    ["Configuration_layer.StoreID_control.text"] = {40072, 3}, -- store ID string

    ["Compressor_layer.OnDelay_control.text"] = {40042, 1}, -- compressor on delay time number
    ["Compressor_layer.HoldOff_control.text"] = {40043, 1}, -- compressor hold off time number

    ["fan_hold_layer.Time_control.text"] = {40044, 1} -- fan hold off time number
}

-- check external source communication by heartbeat packages
function heartbeat_status_check()
end

```

Some bad naming examples, which will cause your co-workers blood pressure to raise sky high

```lua
-- okay what is CB? your naming is not a standard or universal common sense
function CBInit(mapargs)
end

-- don't leave space in any type of naming even string key
data_barrel_1["beater motor"] = 0


data_app["powerSaverStart1"] = 0
data_app["powerSaverStart2"] = 0
data_app["powerSaverStart3"] = 0
data_app["powerSaverStart4"] = 0
data_app["powerSaverStart5"] = 0
data_app["powerSaverStart6"] = 0
data_app["powerSaverStart7"] = 0
data_app["powerSaverStart8"] = 0
data_app["powerSaverEnd1"] = 0
data_app["powerSaverEnd2"] = 0
data_app["powerSaverEnd3"] = 0
data_app["powerSaverEnd4"] = 0
data_app["powerSaverEnd5"] = 0
data_app["powerSaverEnd6"] = 0
data_app["powerSaverEnd7"] = 0
data_app["powerSaverEnd8"] = 0

---why don't make something like
data_app["power_saver"] = {
    { 0, 0 }, --slot 1, first value is start and second value is end
    { 0, 0 }, --slot 2...
    { 0, 0 },
    { 0, 0 },
    { 0, 0 },
    { 0, 0 },
    { 0, 0 },
    { 0, 0 }
}

-- for those highly related things, just put them in an array or table

```


## UI_structure

----

Before moving on, there is another very important part you need to be very familiar with, the structure of the UI.

A basic view of the structure of the UI should be just like below

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/UI_structure.png">

It looks very complex, but we don't need to get into every single part of it, only those important parts.

1. ### screen
> Yes, a screen is a screen, a whole screen. Your full screen will be filled by the items on the screen you are at.

2. ### layer 
> A section on the screen, and you can have as many layers as you need on the same screen. However, layers will overlap with each other. The layer on the top will cover the layer at the bottom, which will result in you can only see the layer on top. In addition, the layer can be larger than the screen size, but only the section within the screen will be displayed. (also the layers are shared between screens, so you can't have two layers with the same name but doing different things)

the larger part is screen, the smaller part is the layer belone to this screen
<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(34).png">

3. ### controls
> then here comes the important part, that controls, the smallest unit you must have to display something or do something by touching the screen (you cannot just have an image on screen).
> just like layers belong to the screen, and control must have a layer, and it will follow the display status of the layer (which means if you hide the layer, all controls belonging to this layer will be "hide" even if they are set to "show").
> For each control, in order to display something, you will need to add "render extension", then we can start inserting elements (mostly images or text).
<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(37).png">
<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(38).png">

## system_init

----

## system_running

----


## system_structure

----

## task_table

----

## data_structure

----

## data_IO

----
