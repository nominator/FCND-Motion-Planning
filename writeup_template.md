### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
This provides a basic path planning solution. The main script connects with the simulator via Udacidrone API and controls the drone. The script is basically a state-machine that transitions between different states for a drone flight mission.

It is quite similar to the backyard flyer state-machine with a new Planning state. After drone is armed it transitions to this state by calling plan_path(). This function 

- loads the obstacle map data from colliders.csv
- creats a grid using the map data, it returns north and east offsets as well. These are required to transform positions and waypoints between the map coordinates and grid coordinates. There is one major distinction between the two coordinate systems. The map has its origin in the center of the map, whereas grid has its origin in the lower left corner. Therefore it is important to tranform coordinates when using map or grid
- sets start and goal positions in grids coordinate frame. Since A* will be executed on the grid, therefore start and goal positions need to be set in grid coordinate frame.
- sets start position at the center of grid and map
- set a goal position relative to start position, i.e. 10m north and 10m east of start position
- searches a path using A*, the method a_star() takes in a grid alongwith start and goal cells and returns a list of cells representing the path from start to goal.
- the path cells are transformed to map waypoints using north and east offsets and are stored in waypoints list
- script then transitions to waypoint state that sets each waypoint as target position for drone to follow one by one

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
##### motion_planning.py: Line# 125
wrote a function called get_global_home_from_file() in moition planning class that loads the geodetic coordinates from the first line of colliders.csv file and sets it as the global home position of the drone using the set_home_position() of the drone class. This position can then be retrieved using self.global_home property of the drone class.

#### 2. Set your current local position
##### motion_planning.py: Line# 128
Next I set the local position of the drone relative to global_home by calling the global_to_local() function. I supplied the current global postion of the drone and its global home position to this method. Both of these are in geodetic frame. The output of the function is the current position of the drone relative to drone's home position in local ECEF frame. This essentially puts the current position of the drone in the map's coordinate frame provided in colliders.csv

#### 3. Set grid start position from local position
##### motion_planning.py: Line# 139
Now that the current position is in map's coordinate frame, I initialize grid_start position by transforming the local_position to grid coordinate frame using the north and east offsets returned by create_grid() method. The transformation is made by subtracting the offsets from local_position.

#### 4. Set grid goal position from geodetic coords
##### motion_planning.py: Line# 143 to 145
First I manually flew the drone to an arbitrary location on the map and noted the geodetic position displayed top left of simulator window. Initialized goal_geo using these geodectic coords. Then I called global_to_local() function passing it the goal_geo and global_home to get the local_goal position expressed relative to global_home position in map's coordinate frame. Third I transform this local_goal position to grid_goal by substracting grid offsets from it. At this point I have both start and goal positions expressed in grid frame alongwith the grid itself, all set for applying A*

#### 5. Modify A* to include diagonal motion (or replace A* altogether)
##### planning_utils.py: Line# 59 to 62 and 84 to 103
Added diagonal actions NW,NE,SW and SE to enum class Action with sqrt(2) as cost and modified valid_actions method to check for out-of-bounds and obstacle collision when applying these actions in path searching.

#### 6. Cull waypoints 
I implemented both collinearity and bresenham methods to prune the calculated path. For my chosen start and goal positions I tried different combinations, here are the results  A* calculated path as follows

A* Type | Path Length | Culled Path Length (Collinearity Test) | Culled Path Length (Bresenham)
--- | --- | --- | ---
A* without diagonal actions | 749 | c | 11
A* with diagonal actions | 579 | c | 15


### Execute the flight
#### 1. Does it work?
It works!

### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.
  
# Extra Challenges: Real World Planning

For an extra challenge, consider implementing some of the techniques described in the "Real World Planning" lesson. You could try implementing a vehicle model to take dynamic constraints into account, or implement a replanning method to invoke if you get off course or encounter unexpected obstacles.


