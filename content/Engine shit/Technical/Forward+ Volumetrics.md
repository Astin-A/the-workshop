


# Volumetrics in a Forward+ renderer

Volumetric fog, smoke, and light shafts in a Forward+ renderer. Assumes an existing depth prepass, clustered light culling, per-cluster light lists, and a transparent pass. Reuses those light lists instead of adding a separate lighting path.

## What a froxel is

A **froxel** is a *frustum-aligned voxel*: one cell in a 3D grid that subdivides the camera's view frustum. It is the volumetric cousin of the 2D screen tile used by tiled forward rendering.

- A **tile** is a 2D rectangle of screen pixels (e.g. 16x16 px).
- A **voxel** is a 3D cube in world space.
- A **froxel** is a 2D tile extruded along the view ray in *depth slices*, with slice boundaries chosen in view space (usually exponentially so near-camera slices are thinner).

Every pixel of the screen maps to exactly one froxel column per depth slice. Froxels let you compute lighting, density, and scattering **once per cell** instead of once per pixel, then sample the result per pixel. That is the performance trick that makes real-time volumetric fog feasible.

```mermaid
flowchart LR
    subgraph Screen["Screen space"]
        T["Tile<br/>16x16 px"]
    end
    subgraph Depth["Depth slicing"]
        D["Exponential slices<br/>e.g. 64 per column"]
    end
    subgraph Froxel["Froxel"]
        F["Tile x depth slice<br/>= frustum-aligned cell"]
    end
    T --> F
    D --> F

    classDef default fill:#303040,stroke:#7aa2f7,stroke-width:2px,color:#f5f5fa;
    classDef primary fill:#341660,stroke:#d8b4fe,stroke-width:2px,color:#f5f5fa;
    classDef secondary fill:#1e3244,stroke:#7ac7f7,stroke-width:2px,color:#f5f5fa;
    classDef tertiary fill:#1e3f2e,stroke:#7af7a2,stroke-width:2px,color:#f5f5fa;
    classDef neutral fill:#30303a,stroke:#9a9aa5,stroke-width:2px,color:#f5f5fa;
    classDef muted fill:#1a3538,stroke:#7ae2e2,stroke-width:2px,color:#f5f5fa;
    classDef accent fill:#3a1a2e,stroke:#f77ac7,stroke-width:2px,color:#f5f5fa;
    classDef alt fill:#3e2e1e,stroke:#f7c87a,stroke-width:2px,color:#f5f5fa;
    classDef strong fill:#3e2028,stroke:#f77a8a,stroke-width:2px,color:#f5f5fa;

    class T secondary;
    class D muted;
    class F primary;

    style Screen fill:#1e3244,stroke:#7ac7f7,stroke-width:2px,color:#f5f5fa
    style Depth fill:#1a3538,stroke:#7ae2e2,stroke-width:2px,color:#f5f5fa
    style Froxel fill:#341660,stroke:#d8b4fe,stroke-width:2px,color:#f5f5fa
```

A froxel volume is stored as a 3D texture, typically `RGBA16F`, where RGB holds accumulated in-scattered light and A holds extinction (or transmittance). Because froxel grids align with cluster grids in most Forward+ engines, the same light culling data can drive both.

## Scope

Every volumetric system answers one question per fragment or froxel:

> How much light is scattered toward the camera along the view ray through media, and how much is extinguished?

That splits into three modules that can ship independently.

<table>
  <thead>
    <tr>
      <th>Module</th>
      <th>Responsibility</th>
      <th>Depends on</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>A. Media representation</td><td>Where fog/smoke/dust exists and its density</td><td>Nothing</td></tr>
    <tr><td>B. Volumetric lighting</td><td>In-scattering and extinction inside media</td><td>Cluster light lists, shadow maps</td></tr>
    <tr><td>C. Compositing</td><td>Applying fog color and transmittance to surfaces</td><td>Opaque and transparent passes</td></tr>
  </tbody>
</table>

```mermaid
flowchart LR
    A[Media representation] --> B[Volumetric lighting]
    A --> C[Compositing]
    B --> C
    L[Cluster light lists] --> B
    L --> P[Lit particles / billboards]
    P --> C

    classDef default fill:#303040,stroke:#7aa2f7,stroke-width:2px,color:#f5f5fa;
    classDef primary fill:#341660,stroke:#d8b4fe,stroke-width:2px,color:#f5f5fa;
    classDef secondary fill:#1e3244,stroke:#7ac7f7,stroke-width:2px,color:#f5f5fa;
    classDef tertiary fill:#1e3f2e,stroke:#7af7a2,stroke-width:2px,color:#f5f5fa;
    classDef neutral fill:#30303a,stroke:#9a9aa5,stroke-width:2px,color:#f5f5fa;
    classDef muted fill:#1a3538,stroke:#7ae2e2,stroke-width:2px,color:#f5f5fa;
    classDef accent fill:#3a1a2e,stroke:#f77ac7,stroke-width:2px,color:#f5f5fa;
    classDef alt fill:#3e2e1e,stroke:#f7c87a,stroke-width:2px,color:#f5f5fa;
    classDef strong fill:#3e2028,stroke:#f77a8a,stroke-width:2px,color:#f5f5fa;

    class A primary;
    class B secondary;
    class C tertiary;
    class L muted;
    class P accent;
```

## Contracts

Define and freeze these before writing passes. Everything downstream uses them.

```text
ClusterIndexFromScreenUVAndDepth(uv, viewDepth) -> clusterIndex
GetClusterLightList(clusterIndex) -> lightList
SampleMediaDensity(worldPos) -> density
SampleVolumetricLighting(worldPos) -> {inscatter, transmittance}
ApplyVolumetricFog(surfaceColor, worldPos) -> color
```

Rules:

- `SampleMediaDensity` is the only place density is defined.
- `ApplyVolumetricFog` is the only place fog is composited.
- Particles and froxels use the same cluster lookup function.
- Transparent fragments use depth-prepass depth for cluster lookup when needed.

## Render graph placement

```mermaid
flowchart TD
    DP[Depth prepass] --> LC[Light culling]
    LC --> FI[Froxel injection]
    FI --> FT[Temporal smoothing]
    FT --> FG[Froxel integration]
    FG --> OP[Opaque pass]
    OP --> TP[Transparent pass]
    TP --> PP[Post process]
    LC --> OP
    LC --> TP
    FG --> TP

    classDef default fill:#303040,stroke:#7aa2f7,stroke-width:2px,color:#f5f5fa;
    classDef primary fill:#341660,stroke:#d8b4fe,stroke-width:2px,color:#f5f5fa;
    classDef secondary fill:#1e3244,stroke:#7ac7f7,stroke-width:2px,color:#f5f5fa;
    classDef tertiary fill:#1e3f2e,stroke:#7af7a2,stroke-width:2px,color:#f5f5fa;
    classDef neutral fill:#30303a,stroke:#9a9aa5,stroke-width:2px,color:#f5f5fa;
    classDef muted fill:#1a3538,stroke:#7ae2e2,stroke-width:2px,color:#f5f5fa;
    classDef accent fill:#3a1a2e,stroke:#f77ac7,stroke-width:2px,color:#f5f5fa;
    classDef alt fill:#3e2e1e,stroke:#f7c87a,stroke-width:2px,color:#f5f5fa;
    classDef strong fill:#3e2028,stroke:#f77a8a,stroke-width:2px,color:#f5f5fa;

    class DP neutral;
    class LC secondary;
    class FI primary;
    class FT accent;
    class FG primary;
    class OP default;
    class TP tertiary;
    class PP muted;
```

## Phase 0: Audit

Goal: identify insertion points and lock the contracts.

Tasks:

- Map render graph: depth prepass, culling, opaque, transparent, post, debug views.
- Document light list format: SSBO/texture layout, max lights per cluster.
- Document transparent sorting and blending rules.
- Add test scenes:
  - height fog with a moving spotlight
  - local smoke volume with a flashlight
  - dust particles lit by emergency lights
  - transparent particles inside fog

Deliverables: integration map, shared shader header with contracts, baseline GPU timings.

Done when: opaque rendering is unchanged and every future pass has a named insertion point.

## Phase 1: Media representation

Goal: define where media exists, independent of lighting.

Tasks:

- Analytic media: height fog, optional distance fog, local sphere/box volumes with soft falloff.
- Volume-texture media: 3D density texture with tiling, scrolling, noise modulation, fixed world-to-UV mapping.
- Particle-derived density: written into the same representation, behind a flag.
- Implement `SampleMediaDensity` combining all three.

Deliverables: media authoring components, density function, density debug view.

Done when: density can be inspected in isolation and does not change when lights change.

## Phase 2: Volumetric lighting

Goal: compute in-scattering and extinction using the existing cluster light data. Two options, can coexist.

### B1: Froxel fog (default)

```mermaid
flowchart LR
    D[SampleMediaDensity] --> INJ[Injection]
    L[Cluster light list] --> INJ
    INJ --> TMP[Temporal smoothing]
    TMP --> INT[Integration]
    INT --> S[SampleVolumetricLighting]

    classDef default fill:#303040,stroke:#7aa2f7,stroke-width:2px,color:#f5f5fa;
    classDef primary fill:#341660,stroke:#d8b4fe,stroke-width:2px,color:#f5f5fa;
    classDef secondary fill:#1e3244,stroke:#7ac7f7,stroke-width:2px,color:#f5f5fa;
    classDef tertiary fill:#1e3f2e,stroke:#7af7a2,stroke-width:2px,color:#f5f5fa;
    classDef neutral fill:#30303a,stroke:#9a9aa5,stroke-width:2px,color:#f5f5fa;
    classDef muted fill:#1a3538,stroke:#7ae2e2,stroke-width:2px,color:#f5f5fa;
    classDef accent fill:#3a1a2e,stroke:#f77ac7,stroke-width:2px,color:#f5f5fa;
    classDef alt fill:#3e2e1e,stroke:#f7c87a,stroke-width:2px,color:#f5f5fa;
    classDef strong fill:#3e2028,stroke:#f77a8a,stroke-width:2px,color:#f5f5fa;

    class D tertiary;
    class L secondary;
    class INJ primary;
    class TMP accent;
    class INT primary;
    class S muted;
```

- Froxel grid matches or derives from cluster grid.
- Exponential depth slicing aligned with cluster slicing.
- 3D texture stores scattering (RGB) and extinction (A), e.g. `RGBA16F`.
- Injection: sample density, read cluster light list, accumulate in-scattering per light.
- Integration: march each froxel column, output fog color and transmittance.
- Temporal smoothing: reproject previous volume, clamp blend.

### B2: Ray march (fallback or high quality)

- March from camera to depth-prepass depth.
- Sample density and cluster light list at each step.
- Accumulate scattering and transmittance.
- Jitter with blue noise and temporally reproject.
- Optional half-resolution march with bilateral upsample.

Shared lighting rules:

- Use only the cluster light list for the fragment or froxel.
- Shadow sampling is optional per light; document which light types support shadows in media.
- Phase function: isotropic by default; Henyey-Greenstein optional per light.

Deliverables: froxel pipeline and/or ray-march pipeline, `SampleVolumetricLighting`.

Done when: a moving spotlight produces stable light shafts, and cost scales with froxel resolution or step count rather than total light count.

## Phase 3: Lit particles and billboard media

Goal: render transparent particles and large smoke volumes lit by the same cluster light lists, and optionally let them contribute density.

```mermaid
flowchart LR
    UV["Screen UV + view depth"] --> CI[Cluster index]
    CI --> LL[Cluster light list]
    LL --> PL[Particle lighting]
    PL --> TP[Transparent pass]
    PL --> MD["Optional density injection"]
    MD --> FI[Froxel injection]

    classDef default fill:#303040,stroke:#7aa2f7,stroke-width:2px,color:#f5f5fa;
    classDef primary fill:#341660,stroke:#d8b4fe,stroke-width:2px,color:#f5f5fa;
    classDef secondary fill:#1e3244,stroke:#7ac7f7,stroke-width:2px,color:#f5f5fa;
    classDef tertiary fill:#1e3f2e,stroke:#7af7a2,stroke-width:2px,color:#f5f5fa;
    classDef neutral fill:#30303a,stroke:#9a9aa5,stroke-width:2px,color:#f5f5fa;
    classDef muted fill:#1a3538,stroke:#7ae2e2,stroke-width:2px,color:#f5f5fa;
    classDef accent fill:#3a1a2e,stroke:#f77ac7,stroke-width:2px,color:#f5f5fa;
    classDef alt fill:#3e2e1e,stroke:#f7c87a,stroke-width:2px,color:#f5f5fa;
    classDef strong fill:#3e2028,stroke:#f77a8a,stroke-width:2px,color:#f5f5fa;

    class UV neutral;
    class CI secondary;
    class LL secondary;
    class PL primary;
    class TP tertiary;
    class MD accent;
    class FI primary;
```

Tasks:

- Particle pixel shader computes screen UV and view depth, calls `ClusterIndexFromScreenUVAndDepth`, fetches `GetClusterLightList`, loops over only those lights.
- Render as transparent, depth-tested, depth-write-off.
- Billboard/impostor smoke volumes use the same lighting path; optional proximity fade when the camera enters the volume.
- Optional particle-to-media bridge: particles inject density into the froxel media behind a flag.

Deliverables: lit transparent shader path, optional particle density injection pass.

Done when: particles and billboards respond to dynamic lights, are fogged exactly once, and do not depend on a deferred G-buffer for transparent lighting.

## Phase 4: Compositing

Goal: apply volumetric lighting to opaque and transparent surfaces without double-fogging.

```mermaid
flowchart TD
    S[Surface shading] --> F[ApplyVolumetricFog]
    F --> T[Tone map]
    P[Lit particle color] --> F

    classDef default fill:#303040,stroke:#7aa2f7,stroke-width:2px,color:#f5f5fa;
    classDef primary fill:#341660,stroke:#d8b4fe,stroke-width:2px,color:#f5f5fa;
    classDef secondary fill:#1e3244,stroke:#7ac7f7,stroke-width:2px,color:#f5f5fa;
    classDef tertiary fill:#1e3f2e,stroke:#7af7a2,stroke-width:2px,color:#f5f5fa;
    classDef neutral fill:#30303a,stroke:#9a9aa5,stroke-width:2px,color:#f5f5fa;
    classDef muted fill:#1a3538,stroke:#7ae2e2,stroke-width:2px,color:#f5f5fa;
    classDef accent fill:#3a1a2e,stroke:#f77ac7,stroke-width:2px,color:#f5f5fa;
    classDef alt fill:#3e2e1e,stroke:#f7c87a,stroke-width:2px,color:#f5f5fa;
    classDef strong fill:#3e2028,stroke:#f77a8a,stroke-width:2px,color:#f5f5fa;

    class S neutral;
    class P accent;
    class F primary;
    class T muted;
```

Tasks:

- Opaque pass: after shading, call `ApplyVolumetricFog(surfaceColor, worldPos)`.
- Transparent pass: particles and billboards compute their own lit color, then apply the same fog function once.
- Do not apply fog to already-fogged post-process effects.
- Fog is applied after surface lighting and before tone mapping.

Deliverables: updated opaque/transparent shader includes, render graph wiring.

Done when: opaque and transparent objects are fogged consistently and there are no double-fog artifacts on particles inside smoke.

## Phase 5: Fallbacks

Ship only what your targets need. Each fallback preserves the `ApplyVolumetricFog` contract so opaque and transparent passes do not change.

<table>
  <thead>
    <tr>
      <th>Technique</th>
      <th>When to use</th>
      <th>Notes</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Analytic fog only</td><td>Minimum spec / mobile</td><td>No froxel or ray march; <code>SampleVolumetricLighting</code> returns analytic result</td></tr>
    <tr><td>Half-res froxel + bilateral upsample</td><td>Mid-range GPUs</td><td>Cheaper injection and integration</td></tr>
    <tr><td>Ray march low step count</td><td>GPU without froxel support</td><td>Fewer steps, temporal accumulation</td></tr>
    <tr><td>Screen-space light shafts</td><td>Stylized look or very low end</td><td>Post-process radial blur from bright pixels; not physically grounded</td></tr>
  </tbody>
</table>

Done when: switching quality mode requires no shader rewrites in opaque or transparent passes.

## Phase 6: Quality scaling, profiling, rollout

Tasks:

- Quality presets: froxel resolution, ray-march step count, temporal on/off, shadow sampling in media, particle density injection.
- Fallback chain: froxel -> half-res froxel -> ray march low -> analytic fog.
- Profiling: GPU timings for injection, integration, temporal, ray march, transparent pass; light-list occupancy per cluster; froxel bandwidth.
- Feature flags, validated independently on target hardware:
  - Flag 1: lit transparent particles
  - Flag 2: froxel volumetric fog
  - Flag 3: particle density injection
  - Flag 4: temporal smoothing

Done when: every module can be disabled without breaking rendering, the low-end path meets frame budget, and the Phase 0 test scenes pass visual validation.

## Risks and mitigations

<table>
  <thead>
    <tr>
      <th>Risk</th>
      <th>Mitigation</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Transparent fragments lack reliable cluster depth</td><td>Use depth-prepass depth for cluster lookup; validate with debug view</td></tr>
    <tr><td>Froxel flicker with moving lights</td><td>Temporal reprojection with clamping; reduce injection noise</td></tr>
    <tr><td>Double fog on transparents</td><td>Single <code>ApplyVolumetricFog</code> function, applied once per fragment</td></tr>
    <tr><td>Light-list format too limited</td><td>Extend SSBO layout minimally; avoid rewriting culling</td></tr>
    <tr><td>Cost blowup on high light counts</td><td>Rely on cluster culling; cap max lights per cluster</td></tr>
    <tr><td>Particle density injection causes feedback</td><td>Inject only from a subset of particles; clamp density</td></tr>
  </tbody>
</table>

## Summary

Three decoupled modules: media representation, volumetric lighting via existing cluster light lists, and single-path compositing. Froxel fog is the default lighting method, ray marching is an optional high-quality path, and analytic and screen-space fallbacks are available. No technique is tied to any specific game or engine feature beyond the Forward+ cluster light data already present.