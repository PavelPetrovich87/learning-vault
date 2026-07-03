# User Story Mapping: Plan to Learn Faster (Flashcards)

#flashcards #flashcards/product-discovery #flashcards/system-design 

When conducting **Product Discovery** for a new feature, why must engineers rely on behavioral telemetry (usage metrics) over direct user interviews?
?
Users are unreliable predictors of their own behavior; they give polite feedback or get distracted by "cool features." The only real validation of a successful feature is if the metric proves they ==choose to use it every day==.
<!--SR:!2026-05-07,15,290-->

In **Incremental Delivery (Henrik Kniberg's analogy)**, what is the correct iterative sequence to building a complex system (like a car), and what architectural principle does this enforce?
?
Sequence: Skateboard -> Scooter -> Bicycle -> Motorcycle -> Car.
Principle: Every iteration must be a **usable, end-to-end slice** that allows you to test the core domain assumption (transportation) and gather real user feedback.
<!--SR:!2026-05-21,21,250-->

When conducting **Product Discovery** for a new feature, why must engineers rely on behavioral telemetry (usage metrics) over direct user interviews?
?
Users are unreliable predictors of their own behavior; they give polite feedback or get distracted by "cool features." The only real validation of a successful feature is if the metric proves they ==choose to use it every day==.
<!--SR:!2026-05-07,15,290-->

In **Lean Startup**, what is the specific purpose of an **MVPe (Minimum Viable Product Experiment)**, and how does it differ from an MVP?
?
An MVPe is the absolute smallest increment built explicitly to ==learn something==. It is often *less than minimal* and intentionally not viable or scalable for the general market, used solely to test assumptions with early-adopter development partners.
<!--SR:!2026-05-20,20,250-->

When validating a high-risk workflow where backend database schema changes are slow, expensive, and block the team, what architectural shortcut can be used for the MVPe?
?
Use ==simple, in-memory databases== to back the early experimental software. Test the workflow with a small subset of early-adopter customers, and only commit to the real schema migrations after the business logic is validated.
<!--SR:!2026-05-05,7,250-->

In the **Build-Measure-Learn** loop, what is the primary indicator that an early experiment has matured into a true **Minimum Viable Product (MVP)** ready for market scale?
?
When your early development partners transition to ==routinely using the product and actively recommending it to others (becoming reference customers)==.
<!--SR:!2026-05-18,18,250-->
