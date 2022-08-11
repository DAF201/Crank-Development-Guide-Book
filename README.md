# Crank_DEV_Guide

## table on contents:
1. [some thing general](#something_general)
2. [system init](#system_init)
3. [system running](#system_running)
4. [system_structure](#system_structure)
5. [task table](#task_table)
6. [data_structure](#data_structure)
7. [data_IO](#data_IO)


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

trust me, no one wants to read such a thing. You are not here to enum out all possible results

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(33).png">

> ## clarity is everything

naming is a super important part of software programming. A well-named variable/function should be clear and short if the condition allows. The long long name is acceptable as long as the purpose is clear and easy to understand. Also, it is okay to name something pathlike if necessary.

```lua
-- this function will be called when app starts
function AppStart(mpargs)
end

-- path-like keys, not recommended but only when necessary, as they are from the UI path.

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

-- check external source communication by heart beat packages
function heartbeat_status_check()
end

```

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
