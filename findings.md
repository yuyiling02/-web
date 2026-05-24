# Findings & Decisions

## Requirements
- Fix the ugly, severely deformed hand visualization.
- Preserve the roughly correct gesture interaction and alignment.
- Make the 3D output look more stable and intentional.

## Research Findings
- `components/ModelViewer.tsx` currently replaces the old `VirtualHand` skeleton overlay with `RiggedVirtualHand`.
- `components/RiggedVirtualHand.tsx` loads `/models/right.glb`, clones its skeleton, and attempts to drive bones from MediaPipe landmarks.
- The current rigging code writes `bone.position.copy(targetPosition)` for every bound bone. Because Three.js bone positions are local to their parent, feeding model-space landmark targets into every bone can compound transforms through the hierarchy and severely stretch the skinned mesh.
- The original `VirtualHand` fallback still exists in `ModelViewer.tsx`, but the active render path now uses the rigged GLB hand.
- `public/models/right.glb` has one skinned mesh and 25 bones. The bones are mostly direct children of `Armature`, so preserving rest offsets is important; translating each bone independently turns the hand mesh into a stretched landmark surface.

## Technical Decisions
| Decision | Rationale |
|----------|-----------|
| Start by finding the hand model/rendering path | Screenshots show deformation in the translucent cyan hand, not in labels or Earth layers. |
| Preserve bone rest offsets and constrain pose updates to rotation/root transform | This keeps the skinned mesh proportions stable while retaining gesture-following orientation. |
| Base visual scale on palm width and wrist-to-middle-MCP distance | Palm points are more stable than fingertip distance, especially during pinch or fist gestures. |
| Switch to four stable visual poses | The GLB's flat bone hierarchy makes live per-joint landmark driving unreliable; open/pinch/rotate/fist templates avoid small-finger twisting and mesh tearing. |
| Add a generated standard rig for high precision | The WebXR source does not include a hand-tracking solver, so true continuous motion needs a valid parent-child finger rig. |

## Issues Encountered
| Issue | Resolution |
|-------|------------|

## Resources
- Project root: C:\Users\yuyiling\Desktop\可视化交互\第七版本 展示
- `components/RiggedVirtualHand.tsx`
- `components/ModelViewer.tsx`

## External Rig Candidates
- Valve OpenXR hand assets are the strongest fit found so far. `openxr_glove_right_model_slim.glb` has one skinned mesh, one skin, 26 joints, and real parent-child chains named `Wrist_R`, `Thumb_*_R`, `Index_*_R`, `Middle_*_R`, `Ring_*_R`, `Little_*_R`, plus `Palm_R`.
- Valve OpenXR hand asset folder includes left/right `.glb`, `.fbx`, `.blend`, and `.ma` files and a BSD-3-like license attached directly to the asset folder.
- Valve SteamVR Unity Plugin includes `vr_glove_right_model_slim.fbx` and left-hand equivalent. It is also BSD-3-Clause and is a good fallback if the OpenXR GLB is too large for the web build.
- `imadeddinedjekoune/Hand-Detection-3D` includes `RightHand_CV.fbx` and is MIT according to its README, but local binary string inspection did not confirm a clean per-finger hierarchy; treat it as reference or third-choice until opened in Blender/Unity.
- CGTrader and Sketchfab have rigged-hand options, but their skeleton naming/hierarchy is not inspectable before download and licenses vary. They are backup sources, not first pick for the current system.
- Implemented the Valve OpenXR path using the original left/right glove GLBs and a canonical bone alias layer. Static validation confirms both extracted GLBs have 1 skin, 1 mesh, 26 joints, no missing aliases, and valid wrist-to-fingertip parent-child chains.
- User validation showed the Valve glove back side when the real palm faced the laptop camera. The fix is a rest-basis palm normal flip for Valve models only; MediaPipe handedness and gesture semantics stay unchanged.

## Visual/Browser Findings
- User screenshots show a translucent cyan 3D hand with fingers and palm stretched into broad triangular sheets.
- The interaction labels and Earth layers appear usable; the major defect is the hand mesh proportions during gesture tracking.
- The hand appears to track orientation and gesture state, but the palm/finger geometry is being over-deformed or non-uniformly scaled.
- Browser verification loaded the local app successfully and showed no console errors or warnings after the fix. No live camera hand feed was available in the verification environment.
- Added visual `rotate` pose for index-middle contact so full-screen rotation has a matching hand state.
- `public/models/high-precision-right-hand.glb` validates as 25 bones with five direct wrist-to-fingertip parent-child chains.
