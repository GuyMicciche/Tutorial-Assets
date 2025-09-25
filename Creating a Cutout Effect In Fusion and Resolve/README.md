# Creating a Cutout Effect In Fusion and Resolve

This tutorial explains how to create a dynamic DaVinci Resolve/Fusion macro that creates a smooth cutout and zoom effect with background blur. This effect is perfect for highlighting specific areas of your footage while maintaining visual interest.

## What This Effect Does

- Creates a rectangular cutout mask on your footage
- Smoothly zooms into the cutout area
- Applies blur to the background while keeping the cutout sharp
- Provides customizable animation curves and timing controls

## Prerequisites

- DaVinci Resolve (free or Studio version)
- Basic understanding of Fusion page in Resolve
- Familiarity with nodes and connections in Fusion

## Tutorial Overview

In this tutorial, we'll build this effect step by step by creating:

1. **Basic Setup**
   - Input routing and background preparation
   - Rectangle mask for the cutout area

2. **Animation System**
   - Transform nodes for positioning and scaling
   - Bezier splines for smooth animation curves
   - LUT lookup nodes for easing controls

3. **Background Blur**
   - Gaussian blur effect with animated intensity
   - Merge operations for combining elements

4. **Keyframe Stretcher**
   - Timing controls for animation duration
   - In/out frame management

5. **Macro Creation**
   - Grouping all nodes into a single macro
   - Exposing user-friendly controls
   - Installation in DaVinci Resolve

## Key Features

- **Customizable Cutout**: Adjust size and position of the cutout area
- **Smooth Zoom**: Control zoom amount and easing curves
- **Background Blur**: Variable blur intensity with animation
- **Flexible Timing**: Adjustable in/out durations and frame stretching
- **Easy Controls**: All parameters accessible from Resolve's inspector

## Installation

Once created, the macro can be installed by:
1. Copying the `.setting` file to your Fusion macros folder (ex: `C:\Users\USER_NAME\AppData\Roaming\Blackmagic Design\DaVinci Resolve\Support\Fusion\Macros\`)
2. Restarting DaVinci Resolve
3. Finding the effect in the Effects Library under "User" category

## Usage

Simply drag the macro onto your timeline clip and adjust the parameters in the inspector to customize the effect for your footage.

---

*This tutorial will guide you through each step of creating this professional-grade effect from scratch in Fusion.*
