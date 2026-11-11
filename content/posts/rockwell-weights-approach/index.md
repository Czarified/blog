+++
date = '2025-05-12T12:00:18Z'
draft = false
title = 'The Rockwell Weights Approach'
summary = 'This is Part 2 in a series I’m doing on Airframe Weight Estimation. In part 1, I simply introduced the topic and discussed why tools like this matter to me.'
tags = ['weight-estimation', 'aircraft-design', 'sweep']
featured_image = 'cover.png'
+++

*Originally published on [LinkedIn](https://www.linkedin.com/pulse/rockwell-weights-approach-benjamin-crews-eofse/).*

This is Part 2 in a series I'm doing on Airframe Weight Estimation.

In part 1, I simply introduced the topic and discussed why tools like this matter to me. Now, let's look into how Rockwell's implementation actually worked.

All the documentation for this is saved and stored by the [Defense Technical Information Center](https://www.linkedin.com/company/defense-technical-information-center). I'm compiling a full bibliography listing of all documents and source information (OCR doesn't get 100% so I have to manually fix some listings), but if you can't wait you can easily find these volumes online. The only small caveats are a couple of niche topics that either aren't available, or DTIC has informed me they had bad original scans and are working to get better copies available. Fingers crossed we can get those copies digitally archived!

Shout out to the archivists and professionals at DTIC; preserving works like these gives people like myself a huge step up in advancing the industry/governments we ultimately work for. Thank you!!

---

So, how do we accomplish airframe conceptual design and weight prediction with an analytical/physics-based system? You must simplify the design cycle at every point, breaking down each part and baking-in those simplifying assumptions. We'll break down an airframe analysis into multiple different areas. First, Weight Modules can be considered the physical "building blocks" of the airframe structure. Rockwell uses the same terminology as [SAWE: Society of Allied Weight Engineers, Inc.](https://www.linkedin.com/company/sawe) professionals do in their "Group Weight Statements" (Reference SAWE RP-7 or -8). Let's make a list:

1. Fuselage
2. Wing
3. Empennage
4. Air Induction System
5. Nacelle/Pylon
6. Landing Gear

These are the basic manufacturing items that form a typical aircraft, and already we can see some assumptions baked in. The Propulsion system is lumped inside the Air Induction System. The Control surfaces are assumed to be part of the Wing/Empennage, and there's no distinction between a Horizontal and Vertical Stabilizer at this level. Keep these in mind, but let's move on.

In addition to the physical building blocks, we need the building blocks for analysis. These include Mission Requirements, Material Systems, and Layout. Mission Requirements define what flight and thermal conditions the air vehicle needs to sustain. Material Systems define all the detailed material allowable data. The Layout is essentially a 3-view drawing of the aircraft, with overall configuration and structural arrangement. Rockwell's engineers describe these as "A set of design assumptions to describe fully the performance, operating envelope, mission, and structure design concept of the configuration."

That's a lot of inputs! Changing any of these can drastically affect the weight predictions, so a toolchain that can iterate on these is highly beneficial, even if significant input data is required. Further, many of the structural design criteria can be based on predetermined values of parameters in accordance with requirements specified in MIL 8860-8870. In modern times, requirements have evolved to be more flexible, but they're also less descriptive because of this! If you compare [JSSG-2006](https://quicksearch.dla.mil/qsDocDetails.aspx?ident_number=98713) ("Joint Service Specifications Guide") to MIL-STD-886X series, you'll see that the "bones" are all the same, but JSSG is mostly a fill-in-the-blank exercise instead of numbers to target. More simplifications we can use, even for the inputs. I digress...

With these basic building blocks, we can start to lay out how the control flow needs to be structured. Rockwell drew this up in the figure below. Remember, computing resources at the time were the primary constraint. Therefore, there's a lot of emphasis on outputs and the detailed coding paradigms passing data back and forth (I had never heard of an "[overlay](https://en.wikipedia.org/wiki/Overlay_(programming))" before).

{{< figure src="image-01.png" alt="Source: L. Ascani et al., &quot;SWEEP Executive Summary&quot;" caption="Source: L. Ascani et al., &quot;SWEEP Executive Summary&quot;" >}}

So L. Ascani *et al.* construct four categorical operations in the flow diagram: Input, Data Development, Weight Analysis, and Output. Input and Output are pretty self-explanatory, and follow everything I discussed above. The Data Development category is primarily estimating mass properties and inertial loads. These loads are the input for the Weights modules.

Data Development processes all the "boiler plate" stuff. It consumes the Mission Requirements, Material Systems, and Layout, and produces more detailed "component" loads for sizing estimates. Since structural weight estimation modules are multi-station analysis routines, loads are calculated at multiple discrete points ("stations"). You might be thinking "How do we know the mass properties of inertia if that's exactly what we're trying to solve for in the first place?" Yes! It's an iterative process. Initial estimates are very crude, and with each iteration the solution converges. A good starting point is a guess at weight and CG, and then an initial sizing iteration can kick off from there. This is the same way that most aircraft are designed, even at the detailed level: You guess, then do the math/tests, update and repeat; The classical Airframe Design Cycle.

{{< figure src="image-02.png" alt="Source: Niu, &quot;Airframe Stress Analysis&quot;" caption="Source: Niu, &quot;Airframe Stress Analysis&quot;" >}}

The Weight Analysis category (from Ascani) is where sizing occurs. The Weights Modules take component load inputs and solve for required material thickness and dimensions. Inside these modules are things like search routines for optimum stiffener spacing, failure mode considerations, and additional geometry/loadpath assumptions. The Weights Modules are perhaps the most complex part (we'll see... I'm still learning all the details as I write up my own version).

---

It's important to note here that certain sizing assumptions are made which negatively impact the results quality. For a weights estimate - and a good place to start for final fatigue/stress/DaDT analysis - these assumptions are fine. However, there are many places where the loadpath might not be exactly correct, or the attaching members (shear clips, struts, and fittings) are not sized, or stress concentrations are not included. Again, from a weights perspective, this is acceptable for this stage of the process. In fact, this extra coverage is a great opportunity to be addressed with the empirical relations. We can't get away from statistics completely! Reducing the empirical relations to only these "extra bits" minimizes the impact and limitations of statistics, while still getting us close to the final fly-away configuration.

Could we include all these details and have our "weights sizing" just be the final sizing? Skip the middle man! Well... yes...? Maybe...? I don't know man... We're essentially doing an entire engineering design process instead of this simplified one. The time scales of iteration loops are inherently still much longer than what I am (and Rockwell were) interested in, but there's room in this space to improve. It's just even harder to execute properly! For prototype builds, one-offs, and especially unmanned systems, the weights sizing is probably fine for an initial build/test anyway. For "real" aircraft with design life, maintenance, and manned systems involved, you need the extra rigor for the final design. Building and testing a prototype always teaches you more about your design and requirements than spiraling on analysis fidelity.

---

I started rambling again... Back to SWEEP!

The Weights Modules are where the physical building blocks from before come back into play. For each "type" of manufacturing item, we have to build in assumptions and methods. Some of these are more generalized, some are specific, but the key part here is that you can break these down even further, and make all the items "speak the same language." It's object-oriented.

For example: A "Fuselage" is really just a combination of many different parts. These parts could be Skin, Longerons, Minor Frames, Major Frames, and Bulkheads. These are the Primary Members of the Fuselage, and each type of member serves their own purpose (Can you describe them all? I used to ask this in interviews!). We can make assumptions for what loads and failure modes do and do not apply.

If we're smart about the architecture, we can make these interchangeable with *other* parts/assumptions. This is all really abstract, because the abstraction helps us make the system modular and functional. Pretend you make a "Panel" that assumes all cows are spherical. Later down the road you want to see what happens if cows are tetrahedrons, so you make a "PanelTet" that can be used in the same analysis routines.

Rockwell engineers pieced together a system that was modular, but essentially parameterized the structural design cycle. Simplify as much as you can, and leave the method robust to changes. Execute, and iterate. Rockwell's engineers were extremely thorough with this system. From everything we've seen so far, we essentially have to include every step for every major structural component, with as many simplifications as possible. If we didn't simplify, we couldn't run the program. If we didn't include every step, we'd oversimplify the design problem and misrepresent the physics at hand. It's a challenging field, and it's becoming obvious why a comparable solution doesn't exist.

Even with all this simplification, the procedure as a whole is still incredibly complicated. I wasn't entirely satisfied by Rockwell's flowchart, so I started drawing up my own. This time, with colors!

{{< figure src="image-03.png" alt="Side note: I'm not sure how well this image will appear in the published article, but if you can't read that, let me know and I'll fix it." caption="Side note: I'm not sure how well this image will appear in the published article, but if you can't read that, let me know and I'll fix it." >}}

That's all I have for now. Nothing else worth showing off. For the detailed assumptions and simplifications, you'd have to read through the *many* volumes from Rockwell. Together, they form this system that accurately predicts the structural weight, even when pushing the boundaries to doing something less conventional. To do even *more* unconventional things, simply add a new "building block" for what you need to change.

I'm absolutely astounded that a team of engineers built this in 1974! I only wish I could meet the creators! The shoulders of giants are just standing there, waiting for us to climb aboard. Let's honor their legacy, preserve their work, and keep building great things!

---

In the next article, I have a couple topics I'm considering:

1. Software Comparisons - What other methods are out there, and how do they compare?
2. Design Data Development Deep Dive - How did SWEEP define Mission Requirements, and estimate Airloads, Load Distributions, and Structural Internal Loads?

I'm not sure which one to pick, and I might change my mind a couple of times! 😆😅 I don't have the answers to these questions; they're things I want to discover!

Finally, I need to apologize in advance: **It's going to be a** ***while*** before the next article in this series. I really want to get some major progress on my own implementation of this system, and I want to do it right. I don't have much built so far, I have very limited time, and this is a passion project for me. I haven't given up on this, I'm just very slow!

If you read all the way to the end, thank you: 🍪. It means a lot! 🙏 Until next time!
