# <span style="color:rgb(112, 48, 160)">Basic Elements</span>

**<span style="color:rgb(255, 0, 0)">Mesh Filter</span>**  
Stores the 3D geometry of an object (its vertices, edges, and faces). It defines the shape, but by itself it is not visible.

**<span style="color:rgb(255, 192, 0)">Mesh Renderer</span>**  
Takes the mesh from the Mesh Filter and renders it on screen. It applies materials, textures, lighting, and shadows so the object becomes visible.

**<span style="color:rgb(0, 176, 80)">Collider</span>**  
Defines the physical boundaries of an object for collision detection. It does not need to match the visual mesh exactly and is often simplified for performance.

**<span style="color:rgb(0, 176, 80)">RigidBody</span>**  
Gives<span style="color:rgb(112, 48, 160)"> </span>the object physical properties like mass, gravity, and velocity. It allows the object to move and interact realistically with forces and collisions in the physics system.


- Enabling the **<mark style="background: #FF5582A6;">Convex</mark>** property simplifies the collider into a convex shape, almost as if you were wrapping the object in some hard material. Unity needs **Mesh** colliders to be set as **Convex** to allow them to interact with other objects.
  
  
<span style="color:rgb(0, 176, 240)">Prefab</span> 
A **Prefab** is a reusable template of a GameObject.  
You can create one object and reuse it multiple times in your scene.

All instances are linked to the original Prefab, so if you update it, all copies update too.

