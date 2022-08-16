# Crank Development Guide Book

> @DAF201
> 
> [Email me](mailto:DAF201@blink-in.com)
>
> Find me in the UTD library 3rd-floor self-study section near the network bookshelves if you need (I am a college student)

# if you are here to find how to use my modules such as Modbus or keypad, go to the last section directly.

# foreword

----

I am not here to teach you Lua, and I will assume you are familiar with Lua syntax, keywords, style, and grammar... Below, I will only go over things I found important, for other parts like metrics, connectors, and two searches in SDK, I will not explain how to use them. Also, before you start reading the remaining parts, you need to know:

----

1. The case sensitive is the difference between the simulator and the actual machine, don't name something like "variable = 0, VARIABLE = 1". It will not run correctly on an actual machine, and your co-works will want to punch you for sure.
2. When you found you are copying and pasting a chunk of code, stop and think about it.
3. When you found your function is longer than a screen, stop and think about it.
4. House is made out of frames and bricks, not decorations. Build necessary framework first then implements functions.
5. Don't copy variables around unless necessary.
6. Try to avoid using some APIs, they are slow (namely, gre.timer_set_interval).
7. It cannot print to the serial ports or ssh, you need to make your own print function.
8. unpack does not work correctly on crank Lua, which means function(...) will not work correctly
9. I cannot guarantee you will know how to make things work after reading, but this will help with making the project more flexible and stronger if you need to add new features in the future.
10. Try to make all the UI changes on the UI side using variable flags and binding, it is really complex on the Lua side to make UI changes.

# table of contents:

1. [some thing general](#something_general)
2. [UI structure and some important concepts](#UI_structure_and_important_concepts)
3. [system init](#system_init)
4. [customized functions](#customized_functions)
5. [task table](#task_table)
6. [data_structure](#data_structure)
7. [data_IO](#data_IO)
8. [some components](#free_components)
9. [how to use my modules](#modules)
# something_general

----

First of all, Crank is an embedded UI SDK. It allows you to make embedded projects much easier than hand writes every line of code. However, even if you don't have to hand write every line of code, it is still very important to keep the remaining parts "modular", "abstract", and "low coupling".

## keep things separate

For example, you have an update everything function that reads data from an external source, saves those data to variables, and displays those data on the screen. This function will be called every second.

Assume for some reason, a piece of data replay took very long. In this case, the function will be stuck on this piece of data, and the remaining data and updates will be stuck too.

A better solution is to separate fetch data, save data, and update screens into three functions, and use a timer to regulate them. (which is pretty much impossible in Lua cause Lua does not support threading. luckily, the event callback trigger is by the UI engine, you don't need to worry about when is the event going to come back).

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/module.png"> -->
![image](./src/module.png)

(this may not be a good example but just an general idea about what to do and what not to do)

## don't repeat yourself

Additionally, there is no sense to make a function for every button with similar outputs. For example, there are six values on the screen, and each of them has an add/sub button near it. There is no sense to write 12 functions such as value_1_add, value_1_sub, and value_2_add... instead, just make value_change(value_path, operation, amount), where value_path represents which value you want to change, and operation represents add/subtract, the amount represents how many you want to add/subtract to it.

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/generalize.png"> -->
![image](./src/generalize.png)

(no one wants to see 12 functions doing the same thing with different "GLOBAL VARIABLES" within them)

Trust me, no one wants to read such a thing. You are not here to enum out all possible results

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(33).png"> -->
![image](./src/Screenshot%20(33).png)

## clarity is everything

Naming is a super important part of software programming. A well-named variable/function should be clear and short if the condition allows. The long long name is acceptable as long as the purpose is clear and easy to understand. Also, it is okay to name something pathlike if necessary.

```lua
-- this function will be called when the application starts
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

Some bad naming examples, which will cause your co-workers' blood pressure to raise sky high

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
-- I prefer something like this
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

Also, for those really cost tons of times task, I will suggest pass it to backend and let backend handle that (task time >=3s in my opinion, except internet requests or violent break into a hash). Because backend use a task-inquester

# UI_structure_and_important_concepts

----

Before moving on, there is another very important part you need to be very familiar with, the structure of the UI.

A basic view of the structure of the UI should be just like below

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/UI_structure.png"> -->
![image](./src/UI_structure.png)

It looks very complex, but we don't need to get into every single part of it, only those important parts.

### screen

Yes, a screen is a screen, a whole screen. Your full screen will be filled with the items on the screen you are at.

### layer 

A section on the screen, and you can have as many layers as you need on the same screen. However, layers will overlap with each other. The layer on the top will cover the layer at the bottom, which will result in you can only see the layer on top. In addition, the layer can be larger than the screen size, but only the section within the screen will be displayed. (also the layers are shared between screens, so you can't have two layers with the same name but doing different things)
the larger part is the screen, the smaller part is the layer belong to this screen

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(34).png"> -->
![image](./src/Screenshot%20(34).png)

### control

Then here comes the important part, that controls, the smallest unit you must have to display something or do something by touching the screen (you cannot just have an image on screen).

just like layers belong to the screen, and control must have a layer, and it will follow the display status of the layer (which means if you hide the layer, all controls belonging to this layer will be "hide" even if they are set to "show").

For each control, to display something, you will need to add "render extension", then we can start inserting elements (mostly images or text).

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(37).png"> -->
![image](./src/Screenshot%20(37).png)

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(38).png"> -->
![image](./src/Screenshot%20(38).png)

### variable

As it's named, it is a variable, and it can store a value of its type. However, this is one of the most important parts of the whole article, because it is the channel connecting the UI display, data, and external source. (it may belong to the application itself also rather than control only)

For each image or text in a control, we can assign a variable to it, and its display will automatically change with the variable. Each element can only be linked to one variable, but the same variable may be linked to many different elements (just like a function, each x only has one and only one y, but a y can have many x).

Below is how to link a variable to an element

Click on the eye-like button, click create a variable

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(39).png"> -->
![image](./src/Screenshot%20(39).png)

Assign a value to it (optional, but suggested cause you want something like default value to display when the connection is down)

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(40).png"> -->
![image](./src/Screenshot%20(40).png)

Then you will see a variable under the value of the element and within the control.

<!-- <img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(42).png"> -->
![image](./src/Screenshot%20(42).png)

Why is the UI variable so important? Because you can get/modify this variable using Lua APIs and a varible path, the UI display will also change at the same time.

```lua
-- get value from this path
gre.get_value(string_value_path)
-- change a value at this path to something...
gre.set_value(string_value_path, any_type_value_that_match)
-- image value is a string relative path that points to an image in the current project's images folder 
-- ./images/image_name.img_type (project_folder/images/image_you_want)
```

Also, trying to display nil (Null, None, nptr, just such thing in other languages) will cause the system to crash, so value check before the pass in.

### action

An action is something to do when an event happens (On the UI side, mostly touch, press, release, the application started, or such internal events. We will talk about the custom events in the data IO section).

When an action is triggered by an event, it will do something. It can play an animation, change a variable, jump to another screen, call a Lua function, and many others things. Most of the time, we will only use those I mentioned.

below is an example of a press action, it will call the Lua function hello_world when I click on the control (which is the image after the application start)

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(43).png">

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(44).png">

<img src="https://github.com/DAF201/Crank_DEV_Guide/blob/main/src/Screenshot%20(45).png">

you can also make it trigger by other events or doing other things

### event

The event is a concept of the happening of something. An event will have its name, data type, and data. Event has three types, self event, incoming event, and outgoing events.
 
A self event is something that happened to the application or screen. An example is the press event. In most cases, we only need small parts of the self-events, and mostly for buttons. commonly used self events include :"touch", "press", "release", "application start", and "screen show".

An incoming event is an event that comes from outside of the application. The main purpose of this type of event is to update data or execute external commands.

An outgoing event is an event that sends out from the application. The main purpose of this type of event is to make changes to an external source or let an external source execute commands.

we will talk about incoming and outgoing events in detail in data_IO.

add custom event

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(46).png">

# system_init

----

Now we are moving on to the system initialization section, below is the process of the system initialization I redesigned for [brix](https://github.com/DAF201/intern_2022/tree/main/brix).

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/system_init.png">

The main purpose of the system initialization is to ensure the UI is displaying correctly, and the services are started.

As people always say, "a good beginning is half done", as the entrance of the application, the system initialization is one of the most important parts. The structure of the initialization directly influences the efficiency of the application when running.

The initialization can be split into two parts, the data update and the start-up of the service. 

Date updates include screen updates and variable updates. Screen update is updating the status of the screen, such as which screen should we go to when the application starts, which part of the screen should be shown, and which parts of the screen should be hidden (even we can set those in UI, sometimes we still need some curtain like things to show on application start, but we don't want curtain to block our eyesight when developing). The variable update is changing the variables stored in the controls or application, which will change the display on the screen or let Lua get different data later (which usually makes no sense, except for some flag variables stored in the application. Cause mostly those variables are used and manipulated by services).

Then we have the service start up. Service is not a concept defined by the crank, instead, it is my personal defined concept. According to my definition, a service is a separately running function from the main thread that provides a service for the main thread. Crank Lua does not support coroutine or threading, but it has something similar Actually Lua itself does not support threading at all. Why do I mention this? Because the crank provides some C APIs to do that (Looks like C but I can not guarantee how the crank made those).

Like before, I am going to provide some examples of the services. 

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/service.drawio.png">

The timer service is a function that runs separately from the main thread. It will fetch all the tasks being registered in the task table from the main thread or other services, and execute the task by the time it reaches the task's schedule. After each execution, it will check if the task is a repeating task. If not, it will unregister this task from the task table.

A funny fact, the crank has build in timer and threading by it self, which are: 

``` lua
-- make an function execute every interval ms
id = gre.timer_set_interval(interval,function)
-- stop the repeating of function executation
gre.timer_clear_interval(id)
-- create a thread
gre.thread_create(function)
```

However, I will say use those APIs as less as possible. Those built-in threading will slow down the application. The creation of new thread costs, the switches between threads costs, and the running of threading costs. There are costs everywhere when you have too many threads. When I got the Brix, every timed function was called by the "gre.timer_set_interval", and the system was quite slow (the communication part took very long to react, the events were lost some time, and the clock was not loaded correctly sometimes). Later, I decided to rewrite the structure, then I have the structure above (But I will still say keep only necessary parts, I intergraded other services such as heartbeat service which check communication status as tasks into registered tasks to speed it up). 

# customized_functions

----

Finally, we reached the coding part, so let's start with a hello world.

Suddenly, you realize: "where in the world am I going to put my functions?" The SDK doesn't look like a text editor at all.

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(47).png">

So let's start from zero to one.

Firstly, we always want to have a function that starts when the application starts. So we go to the application, right-click the project, click add action

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(50).png">

Then, we select event: application start, action: Lua script, and type in "app_start" as function name, then click "edit"

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(51).png">

It will say something like the function does not exist, do you want to create it? click yes, then you will see it create a "callback.lua" with a

```lua
--- @param gre#context mapargs
function app_start(mapargs)
--TODO: Your code goes here...
end
```

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(53).png">

we change it to

```lua
function app_start(mapargs)
    print("hello world")
end
```

Now we can test the code, after we got an entrance. It will print a "Hello world" on console when application starts (right bottom).

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(52).png">

And we can start with building out system initialization and services with this entrance. You will start your system initialization and services startup from this app_start function (Still structure is important, don't put everything together [[to my system init]](#system_init)).

For those custom functions, you can put them all in this "call_back.lua" (which you shouldn't), or right-click on the script section, new to create a new script. However, you need to add a requirement in "callback.lua" to access them since "callback.lua" is the only entrance.

```lua
-- I created a test.lua with a test_function_from_test_dot_lua function inside it
test=require("test")
--- @param gre#context mapargs
function app_start(mapargs)
    print("hello world")
end
```

Then you can access your function via events such as button touch manually to interact with the user.

you can go to the project path, scripts folder, and create scripts manually but what is the point of doing that...

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(55).png">

# task_table

----

I pulled this part out cause I think it is important. STOP adding a bunch of gre.timer_set_interval or gre.thread_create (this thing doesn't even support arguments)

You need to come up with a task management service yourself. The quality of this part will directly influence the overall premormance.

Here is an example of task timer

```lua
-- this is a time based task table. you can make a id based table too, that will be easier but a litt bit slower than time based
--time counter
local timer_counter = 0;
Task_table = {}
--print table
function dump(o)
    if type(o) == 'table' then
        local s = '{ '
        for k, v in pairs(o) do
            if type(k) ~= 'number' then
                k = '"' .. k .. '"'
            end
            s = s .. '[' .. k .. '] = ' .. dump(v) .. ','
        end
        return s .. '} '
    else
        return tostring(o)
    end
end
--let this run in a while true in seprate thread or use gre set an interval to it
function Clock()
    if timer_counter > 65536 then
        timer_counter = 0
    else
        timer_counter = timer_counter + 1
        if #Task_table == 0 then
            return
        else
            for interval, function_table in pairs(Task_table) do
                if timer_counter % interval == 0 then
                    for index, functions_value in pairs(function_table) do
                        if not pcall(functions_value[1], functions_value[3], functions_value[4], functions_value[5]) then
                            print("error when running: " .. functions_value[1])
                        end
                    end
                end
            end
        end
    end
end
function Task_register(func, id, interval, args1, args2, args3)
    if Task_table[interval] == nil then
        Task_table[interval] = { { func, id, args1, args2, args3 } }
    else
        table.insert(Task_table[interval], #Task_table, { func, id, args1, args2, args3 })
    end
end
function Task_unregister(id)
    for interval, function_table in pairs(Task_table) do
        for index, functions_value in pairs(function_table) do
            if functions_value[2] == id then
                table.remove(function_table, index)
            end
        end
    end
end
--add tasks
Task_register(print, "print", 1, "test ", "1", " 2")
Task_register(print, "print2", 1, "test ", "3", " 4")
--windows debug
while true do
    Clock()
    if timer_counter > 10 then
    
    --remove tasks
        Task_unregister("print")
        
    end
    os.execute("powershell sleep 1")
end
--end of debug
```
output
```lua
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4
test    1        2
test    3        4 --I unregistered print("test", 1, 2) after a certain times of calling
test    3        4
test    3        4
test    3        4
test    3        4
test    3        4
test    3        4
test    3        4
```

So basically, you will need to make such a thing somewhere to make a separate task calling system from the main thread, and register tasks and unregistered tasks from the main base on conditions. This will make life much easier than waiting for a slow/blocking task to go off in the main or make mutiple threads to handle tasks.

Also, for those extremely complex tasks which take tons of sources and time(>=3s), I will suggest passing them to the backend. Because the backend is created in C language, it will be faster and allows you to use the real threading. With threading, you can create a suspect-inspector model (which is basically a function timeout system. I have trouble remembering things so I prefer a weird name, it helps me distinguish different terms).

Everything below runs in a separate thread from the main thread, and the timer is another separate thread.

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/timeout.png">

# data_structure

----

let me draw a conclusion here. I don't see there is any point to save multiple copies of the same variables around unless you are making a backup or so. For the keypad, if the keypad is designed well, it should have a keypad buffer to store temporary data instead of directly making changes to real value.

now continues with preach, avoid storing global variables unless necessary.

the current data structure of Brix when initialize is just like

current design          |  my opinion
:-------------------------:|:-------------------------:
Why are we having a Lua "proxy"? | Just directly read everything to UI
<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/current_data_structure.drawio.png"> |  <img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/ideal_data_structure.drawio.png">

and the current data structure at the data update parts is like

current design          |  my opinion
:-------------------------:|:-------------------------:
<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/current_requests_data_structure.drawio.png"> |  <img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/ideal_requests_data_structure.drawio.png">

I am really confused about why are we having global tables everywhere, and the over brief naming brought me lots of trouble.

```lua
data_barrel_1["temp"] = 0
--okay, what is this temp for? We have like 3 or 4 temperatures that need to be displayed on different sections on UI for barrel 1
```

Things will be much cleaner if we just store variables straight into the UI variable because the UI variable is stored in the path.

```lua
-- this is where the data_barrel_1['temp'] goes to. As a path, you can see where the data is being used in UI.
-- And we can just change the value of the UI variable using gre.set_value(path, value)
gre.set_value(current_conditions_layer.b1card_group.Temp_Measure.text, 0)
```

It is just much much better for me to have a path-like thing to let me know what is this variable means to use for.

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(60).png">

So, just store everything you need in UI application screen, and get and change via

```lua
--set value
        gre.set_value(path,value)
--get value
        value = gre.get_value(path)
```

But for table, just try to reduce usage of global tables and name them clearly, save yourself and others time.

Yes, it can save table too, but save table in lua is much easier.

Also, use a table for variables highly associated to save yourself and others time, because the table is more organized and easier to use when looping.

```lua
-- use table 
barrel_data = {
    { 0, 0, 0 }, --barrel_1 : {co2 , h2o , pressure}
    { 0, 0, 0 }, --barrel_2 : {co2 , h2o , pressure}
    { 0, 0, 0 }, --...
    { 0, 0, 0 }
}
-- instead of 
barrel_1_co2 = 0
barrel_1_h2o = 0
barrel_1_pressure = 0
barrel_2_co2 = 0
barrel_2_h2o = 0
barrel_2_pressure = 0
barrel_3_co2 = 0
barrel_3_h2o = 0
barrel_3_pressure = 0
barrel_4_co2 = 0
barrel_4_h2o = 0
barrel_4_pressure = 0
-- to make things more organized and easier to use for looping
```

<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(61).png">

However, I don't have time to change those cause it is too messy and I am about to return to school soon. So just start to change the structure at the next project.
A short example of what I am talking about:

1. I have a variable Backend version I need to store a "08/15/2022" to it when initializing
2. I create a string variable in UI called Backend_version
<img src="https://github.com/DAF201/Crank-Development-Guide-Book/blob/main/src/Screenshot%20(62).png">

3. I copy the path of the UI variable and store the value to it using

```lua
gre.set_value("backend_version", "08/15/2022" )
```

4. Next time, when I need to use the backend version data, such as update screen display, I use

```lua
gre.get_value("backend_version")
```

to take variables out from UI. And if we just directly bind this UI variable to display, we can directly change the value displayed on screen which should be 08/15/2022 now.

# data_IO
----

The IO system of the Crank is just like a 

# free_components
----


# modules
----
