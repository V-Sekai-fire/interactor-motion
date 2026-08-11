# fabric-motion-plane

The active plane. It decides what a body does next, generates the motion, and publishes the
reference the physics tracker follows.

## What it runs

[ARDY](https://github.com/v-sekai-multiplayer-fabric/fabric-ardy), autoregressive diffusion
with a hybrid representation, Apache-2.0, from SIGGRAPH 2026. It generates interactively
rather than replaying, and it takes the constraints a controller needs to give it: root paths
and waypoints, full-body keyframes, and sparse joint positions and rotations.

That constraint interface is why it is here rather than an earlier model. The previous
generator honoured the endpoints of a full-body keyframe constraint and lay flat for
85 per cent of the clip between them, and it refused four behaviours outright: getting up by
three separate routes, sitting, and crouching. A model that takes a waypoint and a keyframe is
the difference between asking for motion and hoping for it.

## What it does not do

**It is not the physics.** It produces a reference pose, and a reference pose is kinematic. A
tracker turns that into actuator commands against gravity and contact, and without one this
plane drives an animation rather than a body. The tracker is the other half and it lives with
the crowd plane.

So the loop is: a command and the body's current state reach this plane, ARDY generates the
next motion, and the tracker makes a physical body follow it.

## Why a plane

It runs per body per tick and it exchanges that with the data plane, so it shares a ring, so
it is co-located. **A ring forces co-location**, which puts this in the zone domain beside the
control plane, the data plane, the crowd plane, the edge planes, the tool plane and the janet
plane.

Being a plane also decides the model. It is loaded **once per domain** rather than once per
body, and every body on the machine generates against the same weights.

## The cost, stated

ARDY is a diffusion model and it wants a GPU. Every other member of the zone domain runs on a
CPU inside the Fly budget, so this plane is the one that changes what a zone costs to host.
That is a real consequence and not a detail: a domain containing this plane needs a machine
with a GPU, and a domain without it does not.

The alternative is to run ARDY offline and ship what it produced, which costs nothing at
runtime and gives up the interactivity that made it the choice. Both are legitimate and they
are different products.

## The skeleton

`GeneralSkeleton`, which is Godot's `SkeletonProfileHumanoid`, because a reference pose is
compared against and driven onto one rig. A pose on a different rig is not a worse reference.
**It is a meaningless one.**

## State

**Not built.** This holds the decision. The harness subtree, the ring subscription and the
ARDY call come next.
