# Discovery report

## Problem statement

An animation studio needs a CPU ray tracer it can maintain and extend itself, because licences cost it over $2M last year and vendors take months to add features, but its last in-house renderer failed and nobody could debug it.

## Client interview summary

Interview held with the whole class on 16 November 2026. Quotes taken from shared class notes.

### Reliability

**Need:** same input, same image, and any defect explained within a day.
- **Q:** "What will show you that the software meets your needs?"
- **A:** "If I give it some inputs, I can always expect the same output. [...] If there is an anomaly in an image, we can work out within the same day why it happened."
- **Q:** "Walk me through the last time you rendered with your previous tool."
- **A:** "We kept finding black artifacts [...]. Nobody could tell us why. [...] We delivered 3 weeks late."
- **My conclusion:** this is their biggest pain. Rendering must be deterministic, and the code tested and documented enough to find a bug fast.

### Speed

**Need:** a 90-minute HD film rendered in 2 months max on 100 machines.
- **Q:** "How long can final rendering take?" **A:** "We don't want to spend more than 2 months on final rendering."
- **Q:** "How big is the farm?" **A:** "We have up to 100 machines."
- **Q:** "How long is the film?" **A:** "It is a 90-minute film" (HD, 24 fps in my notes).
- **My conclusion:** 129,600 frames for 144,000 machine-hours, so about 67 minutes max per HD frame on one machine.

**Need:** quick previews for designers, high quality for the final render.
- **Q:** "What is a reasonable render time for you?" **A:** "They need to get images really quickly."
- **Q:** "How many creative renders do you do?" **A:** "At least 10 different pre-renders per image."
- **My conclusion:** two modes, preview and final. No target time yet for the preview.

### Adding features

**Need:** add their own features without waiting for a vendor.
- **Q:** "What is the main thing you need?"
- **A:** "The ability to add our own features. With Maya [...] it takes 6 months or a year to add a feature."
- **Q:** "Which features do you need?" **A:** "For the moment, [...] spheres and planes."
- **My conclusion:** main need. A new shape should be added without rewriting existing code.

### Maintenance and documentation

**Need:** the team keeps working on the code when its author leaves.
- **Q:** "Is there anything we have missed?"
- **A:** "What happens when the person who wrote the software leaves. [...] Nobody could tell me why [these choices] had been made."
- **Q:** "What kind of documentation do you need?" **A:** "Technical people will be reading it. [...] We expect some kind of user guide for the artists."
- **My conclusion:** write down each decision and its reason (ADRs), plus a user guide for artists.

### Pipeline

**Need:** a renderer with no interface, fed by a scene file.
- **Q:** "Do you need an interface, or is a file enough?"
- **A:** "A standard file is created and saved by another tool, [...] put onto the render farm. No interface at this point."
- **My conclusion:** a command-line tool: scene file in, images out.

### Constraints

- CPU only: "We don't have graphics cards."
- Open source, with partner studios contributing.
- Delivery by 5 January.
- Budget: "no more than 500,000" euros, against $2M of licences last year.
- C++, imposed in the brief.

### Open questions

- Target time and resolution for a preview?
- Scene file format produced by their tool?
- HD = 1920 × 1080? (assumed)
- CPU cores and RAM per farm machine?
- Which licence suits the partner studios?

## State of the art

| Solution | Maker | Licence | Pricing (date checked) | Capabilities | Sources |
|---|---|---|---|---|---|
| Maya + Arnold | Autodesk | Commercial | $2,010/year per user (2026-11-16)\* | 3D suite, client's current tool | [1] |
| Arnold | Autodesk | Commercial | $430/year per user; one licence per render node (2026-11-16)\* | Film path tracer, CPU | [2], [3] |
| RenderMan 27 | Pixar | Commercial | $595 per licence + $250/year maintenance (2026-11-16) | Pixar's film renderer, CPU mode | [4] |
| V-Ray | Chaos | Commercial | €39.50 to €94.90/month (2026-11-16) | CPU or GPU, distributed rendering | [5] |
| Redshift | Maxon | Commercial | $289/year (2026-11-16)\* | GPU-first, CPU possible | [6] |
| Cycles | Blender | Apache 2.0 | Free (2026-11-16) | Production path tracer, CPU or GPU | [7] |
| Embree | Intel | Apache 2.0 | Free (2026-11-16) | CPU ray-intersection library | [8] |
| pbrt-v4 | Pharr, Jakob, Humphreys | Apache 2.0 | Free (2026-11-16) | Reference renderer, research and teaching | [9] |

\* Vendor page not readable; price from CG Channel.

Commercial tools are paid per user and per render node, so the bill grows with the farm. Open-source ones are free but follow their own roadmap.

1. https://www.cgchannel.com/2026/03/autodesk-releases-maya-2027/
2. https://www.cgchannel.com/2026/10/autodesk-releases-arnold-7-5-4/
3. https://help.autodesk.com/cloudhelp/ENU/AR-Core/files/arnold-setup/arnold_user_guide_arnold_setup_ac_licensing_home_html.html
4. https://renderman.pixar.com/store/
5. https://www.chaos.com/vray/buy-online
6. https://www.cgchannel.com/2026/09/maxon-releases-redshift-2026-9/
7. https://www.cycles-renderer.org/
8. https://github.com/RenderKit/embree
9. https://github.com/mmp/pbrt-v4

## What is ray tracing?

For each pixel, we send a ray from the camera into the scene and find the first object it hits. From that point, we send a second ray towards the light: if something blocks it, the point is in shadow, otherwise it is lit. That gives the pixel's colour. It is accurate but slow, and since each pixel is independent, the work can be split across many machines.

Sources: [Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html), [Scratchapixel](https://www.scratchapixel.com/), [Physically Based Rendering](https://pbr-book.org/).

## What is C++?

### Programming paradigms

A compiled language, fast, with no garbage collector. It mixes procedural code, object-oriented code (classes, virtual methods) and generic code (templates).

### What to watch out for

- Memory: each object needs one owner; use `std::unique_ptr` rather than `new`/`delete`.
- Never compare `double` values with `==`, use a tolerance.
- `5 / 2` gives `2` with integers.
- Undefined behaviour (e.g. reading outside an array) gives random results instead of a clean crash.

### Shaping the domain model

One class per client concept: `Ray`, `Vector`, `Camera`, `Sphere`, `Plane`, `Scene`, `Image`. A `Shape` interface with a virtual `hit` method lets us add a new shape without touching the others.

Sources: [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines), [cppreference](https://en.cppreference.com/), Bjarne Stroustrup, *A Tour of C++*, 3rd ed., 2022.
