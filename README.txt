
***Maya Outliner Organizer***
by Maysam Al-Ani

Github: https://github.com/malaniwork/Maya-Outliner-Organizer
Tool overview: https://www.malani.work/outliner-organizer
Tool demo (video): https://vimeo.com/726198391

***DESCRIPTION

This tool launches a window to help artists organize scene objects (non-cameras,non-groups,and non-joints) by putting them into groups matched by name. 
The artist can choose to select childless meshes, sort selected meshes alphabetically, and create groups based on unique names indicated by the user defined split value.

***HOW TO SETUP AND USE

1. Place the file Outliner_Organizer.py in this Maya file directory: ~\Documents\maya\2022\scripts

2. In Maya, make a new shelf item and name it. In the video, I named it "Outliner Organizer". 
Copy this code block from the file Outliner_Organizer.py or simply copy and paste the below under Command in the Shelf Editor window:

###### This script will show a window to help user organize scene objects (non-cameras,non-groups,and non-joints) by putting them into groups matched by name

import maya.cmds as cmds
import maya.mel as mm

def alphabet(*args):
    # Reorder meshes into alphabetical order
    mm.eval('string $sel[] = `ls -sl`; string $resorted[] = sort($sel); for ($each in $resorted){reorder -back $each;}')

def organizer(*args):
    # List all assemblies, name cameras and var excludeType them
    assemblies = cmds.ls(sl=True)
    if not assemblies:
        errorMessage1 = cmds.confirmDialog(title='Error', message="No mesh is selected. Please click on the 'Select childless meshes' button", button=['OK'], defaultButton='OK', cancelButton='OK', dismissString='OK' )
        cmds.error("No mesh is selected. Please click on the 'Select childless meshes' button")
        
    excludeType = ['persp', 'top', 'side', 'front']
    
    finalAssemblies = []
    
   # Add non-camera assemblies to the finalAssemblies list
    for each in assemblies:
        if each not in excludeType:
                finalAssemblies.append(each)

    category = []
    
    # Query OBJsplitValue input and add to category list based on split
    for each in finalAssemblies:
        splitter = cmds.textFieldGrp( 'OBJsplitValue', query = True, text = True)
        if splitter:
            typ = each.split(splitter)
            test = typ[0]
            category.append(test)
            finalCategory = list(set(category))
        else:
            errorMessage2 = cmds.confirmDialog(title='Error', message="Please define a split value in the 'Split Value' text input", button=['OK'], defaultButton='OK', cancelButton='OK', dismissString='OK' )
            cmds.error("Please define a split value")
    
    # Format group name based on each unique finalCategory name    
    for eachCate in finalCategory:
        cateGRP = cmds.group(name='{}_GRP'.format(eachCate), empty = True)
        
        # If the assembly name matches the group name, parent the assembly
        for eachAssmb in finalAssemblies:
            if eachAssmb.startswith(eachCate):
                cmds.parent(eachAssmb, cateGRP)
               

                
def queryTextField(*args):
    
    # Store splitter inputs for OBJ and GROUP
    
    objSplitInput = cmds.textFieldGrp( 'OBJsplitValue', query = True, text = True)
    
    print (objSplitInput)

def selectAll(*args):
    
    # List all top nodes/assemblies
    top_nodes = cmds.ls(assemblies=True)
    
    # Select the top nodes
    cmds.select(top_nodes, replace=True)
    
    # For all camera, transform, and joint types, deselect
    for node in top_nodes:
        # Check shape of each top level node
        children = cmds.listRelatives(node, shapes=True) or {}
        # Loop through child shapes
        for shape in children:
            #If the node has a child shape that is a camera, transform, or joint, deselect the node
            if cmds.nodeType(shape)=='camera':
                cmds.select(node, deselect=True)
                break
                
        if not children:
            if cmds.nodeType(node) == 'transform':
                cmds.select(node, deselect=True)
            
            if cmds.nodeType(node) == 'joint':
                cmds.select(node, deselect=True)

# UI Window
title = "Mesh Outliner Organizer"

window = cmds.window( title="Outliner Organizer", widthHeight=(300, 180), sizeable=False )
cmds.columnLayout( adjustableColumn=True)
cmds.separator( h=10, style='none', bgc=[0.33,0.5,0.6])
cmds.text(title, font="boldLabelFont", bgc=[0.33,0.5,0.6])
cmds.separator( h=5, style='none', bgc=[0.33,0.5,0.6])
cmds.separator( h=10,style='out')
cmds.button( label='Select childless meshes', command=selectAll) # Run the def select all when button pressed to organize into groups
cmds.separator( h=5, style='none')
cmds.button( label='Sort meshes alphabetically', command=alphabet) # Run the def alphabet when button pressed to organize into groups
cmds.separator( h=5, style='none')
cmds.textFieldGrp('OBJsplitValue', label = 'Split Value:', columnWidth=((1,118),(2,110))) # Text input for OBJ split string value
cmds.rowColumnLayout (numberOfColumns=2)
cmds.separator( w=60, style='none')
cmds.button(label='Organize meshes into groups', command=organizer, width=170) # Run the def organizer when button pressed to organize into groups
cmds.separator( h=10, style='none')
cmds.separator( h=10, style='out')
cmds.separator( h=10, style='none')
cmds.rowColumnLayout (numberOfColumns=1)
cmds.button( label='Close', command=('cmds.deleteUI(\"' + window + '\", window=True)') ) # Closes the window
cmds.setParent( '..' )
cmds.showWindow( window )

3. To launch this tool, simply click on the button on the shelf you made in step #2

Done!

*** For any issues, questions, or contributions, please post on the Github page.