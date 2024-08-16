# HVAC Heatmap UI

Created by Lucas Xu, highschool intern at the University of Alberta: July 2 - August 9

A python tkinter user interface designed to visualize the number of people in each room. The purpose is so that energy can then be more efficiently allocated and saved based off this data. Floorplans can be uploaded and rooms can be drawn out and customized, data regarding the number of people in each room can be uploaded. This application is used in conjunction with a program that can detect the number of people in ecah room based off their current wifi strength of one or more access points

Due to time restrictions, some featuers have not been implemented that would increase user friendliness, as well as a lack of aestheticness

## Changelog
- changed roomdata/savestate format from `[ id, coords ]` to  `[ id, name, coords ]`
- added helper function for asking user input repeatetly until its valid
- standardized that all coordinates are in json format (0,0) topleft and (1,1) bottomright of the image.
- fixed snapping
- broke hovering (who actually cares though lmao)
- removed savestate and now only use rooms (pd.DataFrame). No need for both, they hold the same state
- changed vertices to be a np.array
- added Enums for ShiftState and DrawingState

### TODO soon
- visualize the snapping radius. Maybe draw circles on each vertex corner. When shifting (not snapping), turn them off. Or maybe make a second hotkey to show, such as alt.
- Show the name of the floor you are currently editing somewhere (titlebar?)
- fix the hourly occupancy visualization part.

## Usage

You can upload a series of floorplans all under a directory with `Upload Directory`, a single image with `Upload Image`, and as well as data that can be matched with room IDs with `Upload Data`
You can select an image to be displayed onto the pannable canvas with the listbox on the right, as well as search for a specific file using the search bar above
Paint mode will set you into paint mode, Erase mode into erase, and Save and exit will save the rooms you draw for the next time you open them
Hovering over a room will display its room ID and the number of people
`+` and `-` buttons to zoom in and out respectively

**__Formatting__**: The images uploaded must be PDF or PNG. The room data uploaded must be a csv file with two columns 'id' and 'people', 'people' must be a list, with each index representing each hour, 0 indexed. 0-23 will represent 0:00-23:00, respectively.

**Pan Mode**
Left Click and Drag = pan around canvas

**Paint Mode**
Right Click - Create a vertex for your room, autosnap is enabled by default
Left Shift Hold - Holding left shift will disable autosnap, releasing shift will re-enable autosnap
Enter - Finish creating your room, vertexes become a shape and represent an overlay of a room. Enter room ID that will be matched in the data you upload

**Erase Mode**
Right Click - Erase a room you are currently hovering over

## How it works

The application is meant to be used with a program that autmatically detect the number of people in each room in real time.

A canvas is established to create a working space for uploading images that can be then panned around.

Zoom in and out enlarge and minimize the image, giving the illusion of zooming in and out. The max width/height must be adjusted as this happens.

Panning around uses the inbuilt functions of a tkinter canvas `tkinter.scan_mark()` and `tkinter.scan_dragto()`

A listbox is created to display a set of image paths. The searchbox can then easily loop through the set to see if the current value typed in the searchbox is found in each item of the set.

Uploading images simply uses PIL to open and display the image on the canvas, pdfs are converted into images saved onto a temporary folder that is deleted on close, and the newly converted images can be displayed like normal images.

Uploading directories utlize `os.walk()` method that can "walk" through all sub-directories and access files, which each file detected as png or pdf is dealt the same way as a normal single image

Creating vertexes simply take the canvas-relative coordinates of the mouse position and push it into a list, this list is then looped through to draw lines

The last index of the list and the current mouse position(canvas-relative coords) is taken to create a thinner line that will move on mouse motion, to display a shadow of the next line to be created

Creating a shape then takes all the items of the list (coords of each vertex) and use it to create a shape that will be linked to a room ID, later used to display a colour based on how many people reside in that room.

Saving will take the current list of all shapes and save and push into a file, that can be read when the image is reopened so that the shapes can be saved. Vertices list is for the currently drawing shape, shapes list is for the already drawn shapes.

Hovering uses a raycasting algorithm to detect whether the current position of the mouse is within a closed polygon, with this algorithm we can display the info of the room if a mouse is hovering over a room.

Erasing also utilizes the raycasting algorithm, if a right click is detected while the mouse is hovering over a current room, that current room is deleted off the shapes list. Saving will update this new list into the csv file.

To display the current hour, the uploaded csv data file via `Upload Data` will have a room ID and a list of integers representing people, each index of the list will represent an hour (from 0:00 - 23:00)


The current hour can be selected using the slider or played using the playback button by the slider, which this data is sent to a function that will take the data of the room data csv file of that current hour and pass it to the draw shapes function.
