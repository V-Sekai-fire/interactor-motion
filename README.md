# fabric-motion-plane

The plane that decides what a body does next. It reads a controller command and the body's
current pose, searches a database of captured motion, and publishes the frame that matches.

## Why a search and not a network

A learned controller was tried first and did not converge in four sessions. The diagnosis is
in `fabric-crowd-plane`'s logbook and it is not the point here. The point is what replaced it:

**Motion matching does not train.** There is no convergence to wait for, no GPU in the
runtime, and no out-of-distribution behaviour, because the controller cannot output anything
that is not already a frame a person performed. A CPU is also the only runtime that fits the
budget.

The corpus answers the whole input surface on its own. 100STYLE's eight motion types are the
stick disc, decomposed: `ID` centred, `FW` and `FR` forward, `BW` and `BR` back, `SW` and `SR`
sideways, `TR` the transitions between them, sampled a hundred times over in different styles.
810 clips, **22.1 hours**.

So a body playing an idle clip stands, and standing is true by construction rather than by
convergence.

## Why a plane

It runs once per body per tick, reading the current pose and publishing a matched one. That is
per-tick data exchange with the data plane, so it shares a ring, so it is co-located. **A ring
forces co-location**, which puts this in the zone domain with the control plane, the data
plane, the crowd plane, the edge planes, the tool plane and the janet plane.

Being a plane rather than a library also decides the memory. The database is loaded **once per
domain**, not once per body: 810 clips shared by every body on the machine, which is the
zero-copy argument stated as a number rather than a preference.

## What a search costs

Two feature groups, and both are needed:

- **The trajectory**, where the body goes next. A query supplies this from the stick, so it is
  the half the player drives.
- **The pose**, where the feet and the hips are now. A query supplies this from the body that
  is already playing, so it is the half that keeps the motion continuous.

Matching only the trajectory gives the right direction with a foot skate at every jump.
Matching only the pose gives a smooth body that ignores the stick.

Every feature is divided by its own standard deviation before it is compared. Without that,
hip velocity in metres per second and foot position in metres are compared on one scale and
the search is decided by whichever has the larger range. That normalisation is the part a
reimplementation gets wrong, and it is taken from `godot-motion-matching`, MIT, Copyright 2024
Guilherme Sousa.

## The skeleton

Every clip is on `GeneralSkeleton`, which is Godot's `SkeletonProfileHumanoid`. A database
compares frames across clips, so a clip on a different rig is not a worse match. **It is a
meaningless one.**

The 810 clips here share one source and therefore one set of bone axes, so they are internally
consistent. Mixing in a clip from another rig requires Godot's humanoid retarget first, which
rectifies bone axes and standardises the silhouette, and a rename alone does not do that.

## State

The database builder and the search are here. The ring subscription and the harness subtree
come next, and until then it runs offline against the corpus.
