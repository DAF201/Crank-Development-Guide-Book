# Crank_DEV_Guide

## table on contents:
1. [some thing general](#something_general)
2. [system init](#system_init)
3. [system running](#system_running)
4. [system_structure](#system_structure)
5. [task table](#task_table)
6. [data_structure](#data_structure)
7. [data_IO](#data_IO)


### something_general

----

First of all, Crank is an embedded UI SDK. It allows you to make embedded projects much easier than hand writes every line of code. However, even if you don't have to hand write every line of code, it is still very important to keep the remaining parts "modular", "abstract", and "low coupling".

For example, you have an update everything function that reads data from an external source, saves those data to variables, and displays those data on the screen. This function will be called every second.

Assume for some reason, a piece of data replay took very long. In this case, the function will be stuck on this piece of data, and the remaining data and updates will be stuck too.

A better solution is to separate fetch data, save data, and update screens into three functions, and use a timer to regulate them. (which is pretty much impossible in Lua cause Lua does not support threading. luckily, the event callback trigger is by the UI engine, you don't need to worry about when is the event going to come back).


### system_init

----

### system_running

----


### system_structure

----

### task_table

----

### data_structure

----

### data_IO

----
