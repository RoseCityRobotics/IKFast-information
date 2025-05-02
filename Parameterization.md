# IKFast Inverse Kinematics Parameterizations Documentation
# *AI GENERATED*
IKFast is a powerful tool capable of generating efficient, analytical Inverse Kinematics (IK) solvers for robotic manipulators. A key aspect of using IKFast is selecting the correct *parameterization type*, which defines the specific kinematic problem the solver will address based on the constraints placed on the end-effector's pose. This document details the various IK parameterization types available in IKFast, helping developers choose the most suitable one for their application.

---

## Core IK Parameterization Types (from Thesis)

These types form the foundation and are described in detail in the original IKFast research.

**1. Transform 6D (Translation + Rotation)**

* **DOF Constrained:** 6
* **Description:** This is the most common and comprehensive IK type. It finds joint configurations (`q_arm`) that place the robot's end-effector at a precise 6D pose, defined by both a target position (`t`) and a target orientation (`R`) in the world frame relative to the robot's base.
* **Use Cases:**
    * Standard pick-and-place tasks requiring specific gripper orientation.
    * Precise tool positioning and orientation (e.g., welding, drilling, assembly).
    * Positioning and aiming cameras or sensors accurately.
    * Any task where the full 6D pose of the end-effector is critical.
* **Target Specification:** A full 4x4 transformation matrix representing the desired end-effector pose relative to the robot's base.

**2. Translation 3D**

* **DOF Constrained:** 3
* **Description:** Finds joint configurations that place the end-effector's *origin* at a specific 3D position (`t`) in the world, leaving the end-effector's orientation (`R`) unconstrained. Any orientation that allows the arm to reach the target point is considered a valid solution.
* **Use Cases:**
    * Reaching a specific point in space (e.g., touching a location, pointing with a fingertip).
    * Pushing buttons or objects where only the contact point matters, not the tool's angle.
    * Positioning the *base* of a tool (like a drill tip) at a location before orienting it separately using another method or joint.
* **Target Specification:** A 3D vector representing the desired position (`px`, `py`, `pz`) of the end-effector origin.

**3. Rotation 3D**

* **DOF Constrained:** 3
* **Description:** Finds joint configurations that align the end-effector's *orientation* (`R`) with a specific target orientation, leaving the end-effector's position (`t`) unconstrained. The solver will find solutions that achieve the desired orientation at any reachable point in the workspace.
* **Use Cases:**
    * Orienting a camera or sensor in a specific direction when its exact position isn't critical.
    * Aligning a tool (like a screwdriver or wrench) with a target orientation before moving into the final position.
    * Pointing tasks where only the direction the end-effector faces is important.
* **Target Specification:** A 3x3 Rotation Matrix or Quaternion representing the desired end-effector orientation relative to the robot's base.

**4. Ray 4D**

* **DOF Constrained:** 4
* **Description:** Finds joint configurations such that a ray defined in the end-effector's frame (origin `p_n`, direction `d_n`) aligns with a target ray defined in the world frame (`p_ee`, `d_ee`). The key feature is that the solution allows the end-effector to be positioned *anywhere along the target world ray*, as long as the alignment is correct. This constrains 3D position relative to the ray direction (effectively 2 constraints) and the direction itself (2 constraints).
* **Use Cases:**
    * Positioning a camera so its line-of-sight passes through a specific point or aligns with a specific line in space (e.g., looking at an object from a distance).
    * Aligning a tool (like a laser pointer, spray nozzle, or sensor beam) along a target line without needing to be at a specific point on that line.
* **Target Specification:** Two 3D vectors: the origin of the world ray (`p_ee`) and the direction of the world ray (`d_ee`).

---

## Additional IK Parameterization Types (from IKFast Website/Usage)

These types offer more specialized constraints, often useful for specific tasks.

**5. Direction 3D**

* **DOF Constrained:** 2 (out of 3 rotational DOF)
* **Description:** Finds joint configurations that point a specific axis of the end-effector frame (typically the Z-axis, often representing the tool's primary direction or approach vector) along a desired direction vector in the world. The position of the end-effector and the rotation *around* the specified axis (the "roll" angle) remain unconstrained.
* **Use Cases:**
    * Aiming a directional antenna, sensor, or light source.
    * Pointing a tool in a general direction without needing a specific position or precise roll angle.
    * A simpler version of `Rotation3D` when only the pointing direction of one primary axis matters.
* **Target Specification:** A 3D unit vector representing the desired world direction for the end-effector's specified axis (e.g., Z-axis).

**6. Lookat 3D**

* **DOF Constrained:** 2 (out of 3 rotational DOF)
* **Description:** Finds joint configurations that point a specific axis of the end-effector frame (typically the Z-axis) towards a specific target point in the world. Similar to `Direction3D`, the position of the end-effector and the rotation *around* the specified axis (roll) remain unconstrained. The direction is implicitly defined by the vector from the current (or any reachable) end-effector origin to the target point.
* **Use Cases:**
    * Pointing a camera or sensor *at* a target object or location from any reachable position, maintaining focus on the target.
    * Keeping a tool aimed towards a point of interest while the robot base or other joints move.
* **Target Specification:** A 3D vector representing the world coordinates of the point to look at.

**7. Translation Direction 5D**

* **DOF Constrained:** 5 (3 translational, 2 rotational)
* **Description:** Finds joint configurations that place the end-effector's origin at a specific 3D position *and* point a specific axis of the end-effector frame (typically Z-axis) along a desired direction vector in the world. The rotation *around* the specified axis (roll) remains unconstrained. This combines `Translation3D` and `Direction3D`.
* **Use Cases:**
    * Surface processing tasks like grinding, polishing, painting, inspection, or applying sealant, where the tool must be at a specific location and oriented normal (or tangent) to the surface, but the rotation around that normal doesn't matter.
    * Insertion tasks where the approach direction and final position are critical, but the roll angle isn't (e.g., inserting a round peg).
    * Welding along a path requiring specific tool orientation relative to the weld direction at each point.
* **Target Specification:** Two 3D vectors: the desired end-effector position (`px`, `py`, `pz`) and the desired world direction for the end-effector's specified axis.

**8. Translation X/Y/Z Axis Angle 4D**

* **DOF Constrained:** 4 (3 translational, 1 rotational)
* **Description:** Finds joint configurations that place the end-effector's origin at a specific 3D position *and* align one of the end-effector's primary axes (X, Y, or Z, specified in the type name, e.g., `TranslationZAxisAngle4D`) with a desired axis vector in the world. The rotation *around* this aligned axis remains unconstrained, while the other two rotational degrees of freedom are fixed by the alignment constraint.
* **Use Cases:**
    * Placing an object flat onto a surface (e.g., use `TranslationZAxisAngle4D`, align the end-effector's Z-axis with the surface normal, position at the target location; roll around Z is free).
    * Inserting a non-rotationally symmetric key or part where one axis must be aligned correctly (e.g., use `TranslationZAxisAngle4D` to align the insertion axis Z, position correctly; roll around Z is free).
    * Reaching a point while ensuring a specific part of the tool (e.g., the flat side represented by the XY plane) faces a certain direction (e.g., use `TranslationZAxisAngle4D` to align Z).
* **Target Specification:** Two 3D vectors: the desired end-effector position (`px`, `py`, `pz`) and the desired world axis vector that the specified end-effector axis should align with.

**9. Translation XY 2D**

* **DOF Constrained:** 2 (translational)
* **Description:** Finds joint configurations that place the end-effector's origin at a specific (`x`,`y`) position *relative to the robot's base frame*, ignoring the Z coordinate and all orientation constraints. This is primarily useful for planar robots or tasks restricted to a 2D plane.
* **Use Cases:**
    * Simple positioning tasks on a 2D plane (e.g., SCARA-like robots or tasks where Z is fixed or irrelevant).
    * Quickly checking reachability in the XY plane without considering Z or orientation.
    * Drawing or plotting on a flat surface.
* **Target Specification:** A 2D vector representing the desired (`x`,`y`) position relative to the robot base.

---

## Less Common / Specialized Types

These types appear less frequently or might be specific implementations. Detailed documentation should be confirmed from official IKFast sources if needed.

* **TranslationXAxisAngleZNorm4D / TranslationYAxisAngleXNorm4D / TranslationZAxisAngleYNorm4D:** These appear to be variants of `TranslationAxisAngle4D`. They likely constrain 3D position and align one primary axis (X, Y, or Z) like the `TranslationAxisAngle4D` types, but add a *fourth* constraint related to the orientation around that primary axis. The `ZNorm`, `XNorm`, `YNorm` might imply aligning a *secondary* end-effector axis (e.g., Y when Z is primary) with a *secondary* world reference vector (e.g., a surface normal or tangent). DOF constrained is likely 4, but the exact nature of the 4 constraints differs from `TranslationAxisAngle4D`.
* **TranslationLocalGlobal6D:** This naming suggests a scenario involving multiple coordinate frames rather than a fundamental IK parameterization type. It might refer to a higher-level function that uses a standard IKFast solver (like `Transform6D`) where the target 6D pose is dynamically calculated based on transformations between local (e.g., object) and global (e.g., world) frames.

---

Choosing the correct IKFast parameterization type is crucial for efficient and effective robot motion planning. By selecting the type that most closely matches the task's constraints, you ensure the solver focuses on the relevant degrees of freedom, leading to faster computation and more appropriate solutions.
