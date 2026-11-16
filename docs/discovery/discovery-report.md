# Discovery report

## Problem statement

An animation studio needs an in-house, CPU-only ray tracer that it can extend and maintain itself, because commercial licences cost it over $2 million last year and new features take 6 to 12 months to obtain, but its previous in-house renderer produced unexplained black spots and nobody left in the team could debug it.

## Client interview summary

The interview was held with the whole class on 16 November 2026. The quotes below are rebuilt from shared class notes and lightly condensed.

### 1. Trust in the image

**Need:** the same input always produces the same image.
- **Q:** "What will show you that the software meets your needs?"
- **A:** "It produces a reliable image: if I give it some inputs, I can always expect the same output."
- **My conclusion:** rendering must be deterministic. It can be checked by rendering the same scene twice and comparing the images pixel by pixel.

**Need:** when an image has a defect, the team finds the cause within the same day.
- **Q:** "Walk me through the last time you rendered with your previous tool."
- **A:** "We kept finding black artifacts in the images [...] and the rendering time doubled. [...] Nobody could tell us why. [...] We delivered 3 weeks late."
- **Q:** "What will show you that the software meets your needs?"
- **A:** "If there is an anomaly in an image, we can work out within the same day why it happened and what steps we can take to correct it."
- **My conclusion:** this is the client's biggest frustration. Diagnosis in under one working day requires tests, clear code and documentation, not only correct images.

### 2. Speed

**Need:** a full film is rendered in 2 months or less.
- **Q:** "How long can final rendering take?"
- **A:** "Once the creative side is done, we don't want to spend more than 2 months on final rendering."
- **Q:** "How big is the farm?" **A:** "We have up to 100 machines."
- **Q:** "What is the size of the target? How long is the film?" **A:** "It is a 90-minute film", in HD at 24 frames per second (from my own notes).
- **My conclusion:** 90 min × 60 s × 24 fps = 129,600 frames. 60 days × 24 h × 100 machines = 144,000 machine-hours. **One HD frame must render in about 67 minutes or less on one farm machine.**

**Need:** designers see a quick preview of their work.
- **Q:** "What is a reasonable render time for you?"
- **A:** "For my team to work correctly, they need to get images really quickly."
- **Q:** "How many creative renders do you do?" **A:** "At least 10 different pre-renders per image."
- **A (on image size):** "For the designer, a fast preview is very important [...]. For the final scene, high image quality is very important."
- **My conclusion:** two modes are needed, a fast, smaller preview and a high-quality final render. The preview has no target time yet (see open questions).

### 3. Adding their own features

**Need:** the studio can add features itself, without waiting for a vendor.
- **Q:** "What is the main thing you need?"
- **A:** "The main thing is the ability to add our own features. With Maya [...] either they say no, or it takes 6 months or a year to add a feature."
- **Q:** "Which features do you need?"
- **A:** "The requirements change constantly with every new project [...]. For the moment, let's say we need to be able to render spheres and planes."
- **My conclusion:** this is the main need. Adding a new shape or feature should mean adding code, not rewriting existing code (the open/closed principle). Scope for now: spheres and planes.

### 4. Maintainability and documentation

**Need:** the team can maintain the software after its author leaves.
- **Q:** "Is there anything we have missed?"
- **A:** "What happens when the person who wrote the software leaves. [...] We found these architectural choices, and nobody could tell me why they had been made."
- **Q:** "What kind of documentation do you need?"
- **A:** "Technical people will be reading it. However, we expect some kind of user guide for the artists."
- **My conclusion:** every important decision must be written down with its reason (architecture decision records). Two audiences: developers, and artists who set up scenes. Measurable check: a new developer builds and tests the project using only the README.

### 5. How the tool fits in the pipeline

**Need:** a renderer with no user interface, driven by a scene file.
- **Q:** "Do you need an interface, or is a file enough?"
- **A:** "A standard file is created and saved by another tool, and this file is put onto the render farm [...]. No interface at this point."
- **Q:** "Do you want us to build an editing tool, or do you just want to render?"
- **A:** "We just want to render. It's a black box: it takes a description of the scene and produces images."
- **My conclusion:** a command-line program: scene file in, image files out.

### 6. Constraints

| Constraint | Client's words |
|---|---|
| CPU only | "We don't have graphics cards, so everything must be rendered by the CPU." |
| Open source, open to partners | "Currently, everything is open source." Partners should be able "to contribute their changes correctly." |
| Delivery date | "The software must be delivered by January 5th." |
| Budget | "No more than 500,000 [€]. [...] We expect the investment to be cheaper." Last year: "over $2 million on licences", paid per artist and per render node. |
| Language | C++, imposed in the project brief. |

### Requirements at a glance

| Requirement | Type | How to check it |
|---|---|---|
| Render spheres and planes from a scene file | Functional | Reference scenes render correctly |
| Preview mode and final mode | Functional | Both modes exist |
| User guide for artists | Functional | Guide exists and covers scene setup |
| Same input, same image | Non-functional | Two renders are identical |
| ≤ 67 min per HD frame per machine | Non-functional | Timed on a farm-like machine |
| Defect diagnosed within a day | Non-functional | Tests, logs and docs point to the cause |
| New feature without rewriting code | Non-functional | New shape added without editing existing shapes |
| Maintainable without its author | Non-functional | Build and test from the README alone |
| CPU only, open source, C++, 5 Jan, ≤ €500k | Constraint | Imposed |

### Open questions

- What is the target time and resolution for a preview?
- Which file format does the scene tool produce? (The tool's name was unclear in the notes.)
- Is HD 1920 × 1080? (Assumed.)
- How many CPU cores and how much RAM does each farm machine have?
- Which licence suits the partner studios (permissive or copyleft)?
- What information does the team need to diagnose a defect (logs, intermediate images)?

## State of the art

_To be completed: prices are being verified._

## What is ray tracing?

Ray tracing builds an image by following light backwards. For each pixel, a ray is traced from the camera through that pixel into the scene, and the first object it hits is found. From that point, a second ray is traced towards the light source: if something blocks it, the point is in shadow, otherwise it is lit. The result gives the colour of the pixel. More rays can follow reflections or refractions for more realism, which makes the method accurate but slow. Every pixel is computed independently, so the work can be split across many machines.

Sources:
- Peter Shirley, *Ray Tracing in One Weekend*: https://raytracing.github.io/books/RayTracingInOneWeekend.html
- Scratchapixel, *An Overview of the Ray-Tracing Rendering Technique*: https://www.scratchapixel.com/
- Pharr, Jakob, Humphreys, *Physically Based Rendering* (4th ed., free online): https://pbr-book.org/

## What is C++?

### Programming paradigms

C++ is a compiled language: the code is translated into machine instructions, with no virtual machine and no garbage collector, which makes it fast. It supports several styles: procedural (functions), object-oriented (classes, inheritance, virtual methods) and generic (templates). It also has lambdas, borrowed from functional programming.

### What to watch out for

- **Memory ownership:** each object needs one clear owner. Modern C++ uses smart pointers (`std::unique_ptr`) instead of manual `new` and `delete`.
- **Decimal numbers:** never compare `double` values with `==`; use a tolerance.
- **Integer division:** `5 / 2` gives `2`, which can silently lose half a pixel.
- **Implicit conversions:** the compiler can turn one type into another without being asked; `explicit` prevents it.
- **Undefined behaviour:** some mistakes (reading outside an array, for example) do not crash reliably and give random results, the kind of bug behind "black spots nobody can explain".

### Shaping the domain model

Each concept of the client's domain becomes a class named in the client's words: `Ray`, `Vector`, `Camera`, `Sphere`, `Plane`, `Scene`, `Image`. A common `Shape` interface, with a virtual `hit` method, lets the renderer treat every shape the same way: a new shape is a new class, with no change to existing code, which answers the client's main need. `std::optional` expresses "the ray hits nothing" without magic values, and the scene owns its shapes through `std::unique_ptr`.

Sources:
- Bjarne Stroustrup, *A Tour of C++* (3rd ed.), Addison-Wesley, 2022.
- C++ Core Guidelines: https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
- cppreference: https://en.cppreference.com/
