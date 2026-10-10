# interactor-motion

The motion plane: it decides what a body does next and generates the reference pose a physics tracker then follows.

## What it is for

The repository holds no code, so this README is the design record.

**The generator.** The plane is designed around ARDY (Apache-2.0), an autoregressive diffusion model that generates interactively rather than replaying. It takes the constraints a controller gives: root paths and waypoints, full-body keyframes, and sparse joint positions and rotations. That constraint interface is why it was chosen. The earlier generator honoured only the endpoints of a full-body keyframe and lay flat for most of the clip between them, and it refused to get up, sit or crouch. A model that takes a waypoint and a keyframe can be asked for a motion instead of hoped to produce one.

**Not the physics.** A reference pose is kinematic. A tracker beside the crowd plane turns it into actuator commands against gravity and contact; without one, this plane drives an animation rather than a body. The loop is: a command and the body's state reach this plane, the model generates the next motion, and the tracker makes a physical body follow it.

**Why a plane.** It runs per body per tick and exchanges that with the data plane over a shared ring, and a ring forces co-location, so it sits in the zone domain beside the other planes. As a plane it loads the model once per domain, and every body on the machine generates against the same weights.

**The cost.** The model needs a GPU, and the rest of the zone domain runs on CPUs, so a domain that contains this plane needs a machine with a GPU and one without it does not. The alternative is to run the model offline and ship its output: no runtime cost, and no interactivity. They are different products.

**The skeleton.** Poses are on the engine's humanoid skeleton profile, because a reference pose is compared against and driven onto one rig, and a pose on a different rig is meaningless as a reference.

## Build and run

There is nothing to build.

## Licence

MIT. See [LICENSE](LICENSE).
