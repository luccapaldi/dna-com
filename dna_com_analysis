"""
This script performs a simple analysis on a single DNA moving through a
microfluidic channel.

Input background-subtracted tiff image stack of DNA optained using an Andor
camera and associated metadata file. This analysis calculates the DNA's center
of mass for each frame in the stack, the displacement (step distance) for x-
and y-pixel coordinates and timestamps extracted from the metadata file, and
the instantaneous velocity of the DNA. It outputs histograms of x- (due to the
surface roughness as the DNA jumps around) and y- (due to the flow)
instantaneous velocites, a looping grayscale video, and a looping color video
with the center of mass overlaid. 
"""

import numpy as np
import tifffile as tiff
from scipy import ndimage
import pickle
import matplotlib.pyplot as plt
import matplotlib.animation as animation
from conflux import extractmetadata  #  3rd party module

def main():
    #  Print welcome message
    print("""
          Welcome! This script performs analysis on a tiff image stack of a 
          single DNA moving through a microfluidic channel. To begin, place
          the background subtracted tiff image stack, metadata file, and
          conflux module (now Tondu, available on Github at
          https://github.com/xcapaldi/tondu) in the same directory as this
          file.
          """)
    
    #  Get filenames from user
    tiff_filename = input("Enter tiff filename: ")
    metadata_filename = input("Enter metadata filename: ")
    
    #  Create help message
    help_message = """
          Having trouble? Try the following troubleshooting steps:
              1. Verify that the tiff image stack, metadata file and Tondu
                 (formerly Conflux) files are all in the same folder as this
                 program.
              2. Make sure you are entering the filenames for the tiff image
                 stack and metadata file correctly, without quotes and
                 including the file extension.
                   """
    
    #  Load data as an array
    file_present = False
    #  Counter to display help message
    attempts = 0
    while file_present == False:  
        try:
            array_tiff = tiff.imread(tiff_filename)
            timestamps = extract_timestamps(metadata_filename)
            file_present = True
        except IOError:
            print("One or more files not found. Please try again.")
            tiff_filename = input("Enter tiff filename: ")
            metadata_filename = input("Enter metadata filename: ")
            attempts += 1
        if attempts > 2:
            print(help_message)
    options_message = """
          Type the letter followed by ENTER:
          g -- grayscale video
          c -- color video with center of mass overlay
          p -- x- and y- histograms of instanteous velocities
          o -- view this options menu
          q -- quit
          """
    print(options_message)
    
    #  Calculate center of mass for each frame
    cmx_coords = []
    cmy_coords = []
    for frame in range(len(array_tiff)):
        cmx, cmy = calculate_center_of_mass(array_tiff[frame])
        cmx_coords.append(cmx)
        cmy_coords.append(cmy)
    #test_cm_calc(array_tiff)  #  Test accuracy of cm calculation
        
    #  Convert coordinates to floats
    cmx_coords_floats = [float(coord) for coord in cmx_coords]
    cmy_coords_floats = [float(coord) for coord in cmy_coords]
    
    #  Calculate displacement of cm coordinates
    x_disp,y_disp = calculate_displacement(cmx_coords_floats,cmy_coords_floats)
    
    #  Calculate time steps
    time_steps = calculate_time_steps(timestamps)
    
    #  Calculate instantaneous velocity for x- and y-coordinates
    x_inst_vel, y_inst_vel = calculate_inst_vel(x_disp, y_disp, time_steps)
    
    #  Round cmx and cmy coordinate lists
    cmx_rounded = [round(coord) for coord in cmx_coords_floats]
    cmy_rounded = [round(coord) for coord in cmy_coords_floats]
    
    #  Get user options
    user_choice = get_options()
    display_data(user_choice, array_tiff, x_inst_vel, y_inst_vel, cmx_rounded,
                 cmy_rounded, options_message)

#------------------------------------------------------------------------------
#  USER INTERFACE

def get_options():
    """
    Displays program options, gets user choice and validates the input.
    """
    
    valid_inputs = ["g","c","p","o","h","q"]
    user_input = input("Choice: ").lower()
    
    if user_input not in valid_inputs:
        valid = False
        while valid == False:
            user_input = input("Invalid. Pick a valid option: ").lower()
            if user_input in valid_inputs:
                valid = True
                
    return user_input

def display_data(user_choice, array_tiff, x_inst_vel, y_inst_vel,
                 cmx_rounded, cmy_rounded, options_message):
    """
    Displays data based on user input.
    
    Keyword arguments:
    user_choice -- user choice for options menu
    array_tiff -- multidimensional array of tiff images
    x_inst_vel -- list of instantaneous velocities for x-coordinates
    y_inst_vel -- list of instantaneous velocities for y-coordinates
    cmx_rounded -- list of rounded x coordinates for center of mass of 3D array
    cmy_rounded -- list of rounded y coordinates for center of mass of 3D array
    """
    
    quit_program = False
    while quit_program == False:
        if user_choice == 'g':
            play_image_stack(array_tiff)
            user_choice = get_options()
        elif user_choice == 'c':
            stack_overlaid = display_cm_overlay(array_tiff,
                                            cmx_rounded, cmy_rounded)
            play_image_stack(stack_overlaid)
            print(options_message)
            user_choice = get_options()
        elif user_choice == 'p':
            plot_xy_inst_vel(x_inst_vel, y_inst_vel)
            user_choice = get_options()
        elif user_choice == 'o':
            print(options_message)
            user_choice = get_options()
        else:
            quit_program = True
  
#------------------------------------------------------------------------------
#  PROGRAM EXECUTION

def test_cm_calc(array_tiff):
    """
    Test accuracy of center of mass function by comparing to skikit version.

    Keyword arguments:
    array_tiff -- multidimensional array of tiff images
    """
    
    img_grayscale = array_tiff[:,:,0]  #  Converts color (3 dimensional) array
                                       #  to grayscale (2 dimensional) array.
    scipy_cm = ndimage.measurements.center_of_mass(img_grayscale)
    hwritten_cm = calculate_center_of_mass(img_grayscale)
    
    print("Scipy calculation: ", scipy_cm)
    print("Personal calculation: ", hwritten_cm)

def extract_timestamps(metadata):
    """
    Extract timestamps from metadata file. Function ("extractmetadata") used
    from conflux module (available at https://github.com/xcapaldi/conflux).

    Keyword arguments:
    metadata -- filename of metadata (*.txt) file from an Andor camera
    """
    
    #  Extract metadata in pickle format
    extractmetadata(metadata, log = False)
    
    #  Open and unpickle file
    pickle_file = open('channel-0_time-series.pickle', 'rb')
    timestamps = pickle.load(pickle_file)
    
    return timestamps

#------------------------------------------------------------------------------
#  DATA ANALYSIS

def calculate_center_of_mass(array_tiff):
    """
    Calculate center of mass for each slice of a 3D array.
    
    Keyword arguments:
    array_tiff -- multidimensional array of tiff images
    """
    
    #  Sum x(column) and y(row) intensity values
    m_x = array_tiff.sum(axis=1)
    m_y = array_tiff.sum(axis=0)
    
    #  cm = sum(m*r)/sum(m), where r is the arbitrary distance from the origin
    cmx = (np.sum(m_x * (np.arange(m_x.size))) / np.sum(m_x))
    cmy = (np.sum(m_y * (np.arange(m_y.size))) / np.sum(m_y))
    
    return(cmx, cmy)

def calculate_displacement(cmx,cmy):
    """
    Calculate displacement (aka. step values) of (x,y) coordinates.

    Keyword arguments:
    cmx -- list of floating x coordinates for center of mass of 3D array
    cmy -- list of floating y coordinates for center of mass of 3D array
    """
    
    x_disp = []
    y_disp = []
    
    #  Create lists to multiply through
    x_init = cmx[:]
    x_final = cmx[1:]
    y_init = cmy[:]
    y_final = cmy[1:]
    
    #  Calculate displacement
    for (x_init_value, x_final_value) in zip(x_init, x_final):
        disp_x_value = x_final_value - x_init_value
        x_disp.append(disp_x_value)
        
    for (y_init_value, y_final_value) in zip(y_init, y_final):
        disp_y_value = y_final_value - y_init_value
        y_disp.append(disp_y_value)
        
    return x_disp, y_disp

def calculate_time_steps(timestamps):
    """
    Calculate time steps for individual frames of a tiff stack.

    Keyword arguments:
    timestamps -- list of timestamps for tiff stack
    """
    
    time_steps = []
    
    #  Create lists to multiply through
    time_init = timestamps[:]
    time_final = timestamps[1:]
    
    #  Calculate time steps
    for (time_init_value, time_final_value) in zip(time_init, time_final):
        frame_time_value = time_final_value - time_init_value
        time_steps.append(frame_time_value)
        
    return time_steps

def calculate_inst_vel(x_disp, y_disp, time_steps):
    """
    Calculate instantaneous velocities of (x,y) coordinates.

    Keyword arguments:
    x_disp -- list of step values of x-coordinates
    y_disp -- list of step values of y-coordinates
    time_steps -- list of time step values for frames of a tiff stack
    """
    
    x_inst_vel = []
    y_inst_vel = []
    
    #  Calculate instantaneous velocities
    for (x_disp_value, time_step) in zip(x_disp, time_steps):
        x_inst_vel_value = x_disp_value/time_step
        x_inst_vel.append(x_inst_vel_value)
        
    for (y_disp_value, time_step) in zip(y_disp, time_steps):
        y_inst_vel_value = y_disp_value/time_step
        y_inst_vel.append(y_inst_vel_value)
        
    return x_inst_vel, y_inst_vel

#------------------------------------------------------------------------------
#  DATA VISUALIZATION

def plot_xy_inst_vel(x_inst_vel, y_inst_vel):
    """
    Plot x and y instantaneous velocities as separate histograms.

    x_inst_vel -- list of instantaneous velocities for x-coordinates
    y_inst_vel -- list of instantaneous velocities for y-coordinates
    """

    plt.figure(1)
    plt.hist(x_inst_vel, bins='auto', rwidth = 0.85, color='g')
    plt.title("Instantaneous velocity, x-coordinates")
    
    plt.figure(2)
    plt.hist(y_inst_vel, bins = 'auto', rwidth = 0.85)
    plt.title("Instantaneous velocity, y-coordinates")
    plt.show()
    
def play_image_stack(array_tiff):
    """
    Display tiff image stack as a looping video.

    Keyword arguments:
    array_tiff -- multidimensional array of tiff images
    """
    
    fig = plt.figure()
    images = []
    
    #  Loop through each slice of 3D array, animate and store in list
    for frame in range(len(array_tiff)):
        image = plt.imshow(array_tiff[frame], animated = True)
        images.append([image])
        
    #  Initialize animation
    ani = animation.ArtistAnimation(fig, images, interval=50)
    plt.show()

def display_cm_overlay(array_tiff, cmx_rounded, cmy_rounded):
    """
    Convert grayscale tiff image stack to RGB and overlay center of mass.

    Keyword arguments:
    array_tiff -- multidimensional array of tiff images
    cmx_rounded -- list of rounded x coordinates for center of mass of 3D array
    cmy_rounded -- list of rounded y coordinates for center of mass of 3D array
    """
    
    list_rgb = []
    coord_count = 0
    
    #  Iterate through frames and convert grayscale images to RGB images.
    for image in range(len(array_tiff)):
        #  Create index values for center of mass
        x_index = (cmx_rounded[coord_count])
        y_index = (cmy_rounded[coord_count])
        coord_count += 1
        
        #  Convert each frame to RGB
        rbg_channel = array_tiff[image]
        image_rgb = np.stack((rbg_channel, rbg_channel, rbg_channel), axis=2)
        
        #  Add red pixel at center of mass cordinates
        #  To change color of cm pixel, change RGB intensity values below
        image_rgb[x_index, y_index, :] = [255, 0, 0]
        list_rgb.append(image_rgb)
        
    cm_overlay = np.asarray(list_rgb)
    
    return cm_overlay

#  Initialize program with error handling
try:
    main()
except KeyboardInterrupt:
    print("Program terminated due to keyboard interrupt.")

