# interactor-motion

The motion plane: it decides what a body does next and generates the reference pose a physics tracker then follows.

## What it is for

The generator it is designed around is ARDY, an autoregressive diffusion model that takes root paths, waypoints, full-body keyframes and sparse joint constraints. Its output is kinematic, so a tracker beside the crowd plane turns it into actuator commands against gravity and contact. The model needs a GPU, so a zone that hosts this plane needs one as well. Poses are on the engine's humanoid skeleton profile.

## Build and run

The repository holds the design and no code, so there is nothing to build.

## Licence

The licence is not stated.
