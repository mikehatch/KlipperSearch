The goal is to create a solution for Klipper that allows the user to do a global search for configuration parameters regardless of which file it is to find the settings they are looking for and allow them to open the file for editing.  

The ecosystem of software components includes:
- Klipper the printer control software that takes gcode files and drives the printer.
- Moonraker is a middleware component that talks to Klipper APIs 
- Mainsail is a web UI for interacting with the printer, through Moonraker 

The global search tool should appear in the web ui as a banner search box or a search widget (may be easier?) that allows the user to search for their setting, view a list of results, and click on the one they want to open the file for editing.  

Before starting, I want to explore the approaches for creating the solution:
Simple: Use the current file APIs to perform the search, or use an operating system call like grep to do the search
Complex: Create an index using an library as an extension to moonraker to provide more rapid search capabilities.

