# Task Plan: Stabilize Hand Rendering

## Goal
Improve the gesture-driven 3D hand so the interaction still follows the user's hand, but the visible model keeps realistic proportions and avoids severe stretching.

## Current Phase
Complete

## Phases

### Phase 1: Discovery
- [x] Inspect app structure and current hand-rendering pipeline
- [x] Identify where gesture landmarks drive mesh scale, pose, or deformation
- [x] Record relevant findings in findings.md
- **Status:** complete

### Phase 2: Fix Design
- [x] Choose a constrained mapping that preserves hand proportions
- [x] Keep existing gesture logic intact where possible
- [x] Note the decision and rationale
- **Status:** complete

### Phase 3: Implementation
- [x] Apply targeted code changes
- [x] Avoid unrelated visual or architectural churn
- **Status:** complete

### Phase 4: Verification
- [x] Run build or type checks
- [x] Verify the local app visually in browser
- [x] Record test results in progress.md
- **Status:** complete

### Phase 5: Delivery
- [x] Summarize changed files and verification
- [x] Call out any remaining caveats
- **Status:** complete

### Phase 6: Four-Pose GLB Hand
- [x] Replace live per-bone landmark direction tracking with stable pose templates
- [x] Add open, pinch, rotate, and fist visual poses
- [x] Verify build and browser runtime logs
- **Status:** complete

### Phase 7: High-Precision Rig Path
- [x] Add a standard parent-child right-hand rig asset
- [x] Load and validate the high-precision rig before falling back to the WebXR hand
- [x] Drive finger chains continuously from MediaPipe landmarks
- [x] Verify static rig hierarchy, build, and browser runtime logs
- **Status:** complete

### Phase 8: Valve OpenXR Dual-Hand Rig
- [x] Extract Valve OpenXR left/right glove assets and license notes
- [x] Replace the temporary high-precision rig path with Valve bone alias mapping
- [x] Render left and right rigged hands with original Valve materials
- [x] Verify static hierarchy, build, and local page availability
- **Status:** complete

### Phase 9: Valve Palm-Facing Calibration
- [x] Add rest-basis palm normal calibration for Valve glove models
- [x] Keep target camera basis, handedness, and gesture semantics unchanged
- [x] Verify build, static GLB hierarchy, and local page availability
- **Status:** complete

## Key Questions
1. Is the visible hand a rigged model, procedural mesh, or landmark surface?
2. Is deformation caused by non-uniform scale, landmark smoothing, camera aspect, or mesh construction?
3. Can the fix preserve gesture alignment while using a stable hand silhouette?

## Decisions Made
| Decision | Rationale |
|----------|-----------|
| Keep changes scoped to the rendering/mapping path | User says gesture logic is mostly correct; the urgent issue is visual deformation. |
| Use root transform plus constrained bone rotations | The GLB bones are independent under the armature, so moving every bone to landmark coordinates stretches the skin; preserving rest offsets keeps proportions stable. |
| Use four precomputed GLB pose templates | The current GLB cannot support reliable IK-like finger tracking; stable pose blending preserves the GLB look while matching interaction states. |
| Prefer `high-precision-right-hand.glb` and keep WebXR hand as fallback | The generated rig has real parent-child finger chains for continuous landmark driving; the WebXR hand remains a safe fallback if a future asset is missing or invalid. |
| Use Valve OpenXR left/right glove models as the final high-precision rig target | The downloaded Valve assets have standard wrist-to-fingertip parent-child chains and original glove materials, matching the user's requested two-hand realistic display. |
| Flip only the Valve rest-basis palm normal for palm-facing correction | The displayed glove showed the back side when the real palm faced the camera; calibrating the model rest basis fixes visual orientation without changing MediaPipe handedness or interaction logic. |

## Errors Encountered
| Error | Attempt | Resolution |
|-------|---------|------------|
| `Start-Process -FilePath npm` failed on Windows with `%1 is not a valid Win32 application` | 1 | Use `npm.cmd` for the dev server process. |

## Notes
- Treat existing code and uncommitted changes as user-owned unless clearly created during this task.
