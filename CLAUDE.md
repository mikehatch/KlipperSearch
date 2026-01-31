# Project Instructions

Objectives of the project are listed in objectives.md

Although there are two UIs for Klipper (Mainsail and Fluidd) I want to focus on Mainsail first, don't try to create the Fluidd.  

The Klipper ecosystem for this includes multiple components:

- Klipper (Klipper3d.org) - the firmware that controls the printer and uses the configuration files to control its behavior

- Moonraker (moonraker.readthedocs.io) - a python 3 based web server with APIs to interact with Klipper

- Mainsail (docs.mainsail.xyz) - a user interface for controling and monitoring printers that interfaces with Moonraker

- Fluidd (docs.fluidd.xyz) - an alternative user interface to mainsail, also interfaces with Moonraker

Challenges: 
- Connecting to a Klipper instance remotely for development or alternatively host a virtual Klipper printer using docker


## Key resources


## Project management
Initialize the git if not already done and configure a remote github repository