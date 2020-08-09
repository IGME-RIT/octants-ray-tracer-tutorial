Documentation Author: Niko Procopi 2020

This tutorial was designed for Visual Studio 2019
If the solution does not compile, retarget the solution
to a different version of the Windows SDK. If you do not
have any version of the Windows SDK, it can be installed
from the Visual Studio Installer Tool

Welcome to the Ray Tracing Octants Tutorial!
Prerequesites: 
	Ray Tracing Multi-OBJ
	Oct-Tree

In the previous tutorial, every ray had to check for collision with all 4450 triangles.

Now, every model with more than 50 triangles will have
a collision box around the entire model
	Car wheels, 
	car body, 
	cat, 
	dog

Additionally, every model with more than 350 polygons will be chopped into 8 octants,
each octant will have it's own box within the box that surrounds the entire model
	car body
	cat
	dog

This is hard-coded so that you can only have one level of octant division, which
means you cannot have octants inside of octants. This will change in a future 
tutorial

Rough overview of structures
New struct "chunk", which is an octant
	number of triangles in the octant
	12 triangles to form a collision box
	triangles indices, NOT copies of triangle data
Mesh has a few new variables
	12 triangles to form a collision box
	booleans for which optimizations are used
	8 "chunks" for octants, which might be empty depending on mesh

Rough overview of C++
Meshes are automatically chopped up into smaller pieces if needed, 
no modification to any OBJ files is needed

Rough overview of Shaders
Rather than checking every polygon, it checks for collision with boxes (12 triangles each),
before checking collision for collision with the hundreds of polygons stored
within each box

Specific C++ explanation
We make a new function OptimizeMesh that can automatically detect meshes that need 
optimization, based on how many triangles the mesh has. 

Level1:
If the mesh has more than 50 triangles, then we loop through all triangles to find the "min" 
and "max", then we call MakeBox which builds 12 triangles to form a box around the object.

Level2:
If the mesh has more than 350 triangles, then we take the existing "min" and "max" of the
object to find "min" and "max" of 8 divisions, each with 1/2 width, 1/2 height, and 1/2 length.
Then, we call a new function GetTrianglesInChunk to get which triangles are in each octant.
Instead of copying the "triangle" data into the octant, we simply record which index of the 
"triangle" array, to save memory and processing

Specific Compute Shader explanation
Full honesty, this was messy, I'll improve this later
The shader is dispatched for the number of triangles in all meshes combined,
plus, the number of triangles in all collision boxes, because those need to be 
adjusted for position, rotation, and scale, just like the geometry that is visible.
After manipulation of the collision box geometry, it gets sent to a buffer for the
fragment shader to use

Specific Fragment Shader explanation

bool intersectTriangles loops through all meshes, and handles meshes
differently depending on their level of optimization

When a ray launches into the scene, it first checks if a mesh has a box around it.
If there is no box, then it checks the polygons (the floor and the cube). If there is a 
box, then it checks for collision with the box (12 triangles). If a ray goes into an 
object's box, then it checks if the mesh is divided into octants. If not (car wheels), 
then it checks the polygons. If the mesh has octants, then the ray checks for collision
with all 8 octants, and then checks for collision with polygons within octants that 
the ray passes through

New function intersectMeshBox checks for collision with the box that surrounds
the entire mesh, and new function intersectChunkBox checks the octants within
the mesh box

When handling polygons within the octants, we do not loop through an array of
triangles, we loop through an array of indices, that coorespond to the position
in the Mesh's triangle array