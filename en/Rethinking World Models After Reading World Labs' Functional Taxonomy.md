# Rethinking World Models After Reading World Labs' Functional Taxonomy

[中文](https://github.com/JCat007/Jacky-s-Thinking/blob/main/zh/%E8%AF%BB%E5%AE%8C%20World%20Labs%20%E7%9A%84%E4%B8%96%E7%95%8C%E6%A8%A1%E5%9E%8B%E5%8A%9F%E8%83%BD%E5%88%86%E7%B1%BB%E6%B3%95%EF%BC%8C%E6%88%91%E9%87%8D%E6%96%B0%E7%90%86%E8%A7%A3%E4%BA%86%E4%B8%96%E7%95%8C%E6%A8%A1%E5%9E%8B.md)

A close reading of World Labs' article [A Functional Taxonomy of World Models](https://www.worldlabs.ai/blog/taxonomy-of-world-models) has reshaped my understanding of World Models. The piece clarifies one of the most fundamental yet commonly confused questions in the field: what exactly is a World Model?

Over the past two years, “World Model” has become one of the most pervasive concepts in AI. Video generation platforms, robotics companies, and autonomous vehicle firms all claim to be building World Models. In the field of Physical AI, World Models are explicitly regarded as the core capability of next-generation artificial intelligence.

This raises a critical question: are all these systems truly pursuing the same goal?

World Labs offers a compelling perspective. Instead of focusing on individual model architectures, the article puts forward a defining insight: a World Model is not a single standalone model, but a complete set of capabilities.

## 1. The Capability Stack of World Models

Departing from conventional classification methods based on network structures and technical paradigms, World Labs redefines World Models from a functional perspective. It argues that a complete World Model system consists of three core, complementary capabilities that together form a closed loop for intelligent systems to perceive, simulate, and interact with the world.

### (1) Renderer

The Renderer answers the question: What does the world look like?

It generates and reconstructs sensory observations, covering image synthesis, video generation, novel view synthesis, 3D reconstruction, and more. Its core purpose is to express the world.

### (2) Simulator

The Simulator answers the question: How will the world evolve next?

It learns the temporal evolution of world states, including motion patterns, physical constraints, and causal relationships. Its core purpose is to predict the world.

### (3) Planner

The Planner answers the question: Given a defined goal, what actions should I take?

It leverages the Simulator's future predictions to derive optimal action sequences for goal achievement. Its core purpose is to act on the world.

We used to ask, “Which model qualifies as a true World Model?” Today, we should reframe the question: “Does an intelligent system possess these essential capabilities?”

```Plain
World Model
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
    Renderer        Simulator        Planner
   Express World     Predict World     Act on World

 Generate /         Predict /        Decide /
 Reconstruct        Simulate          Act
```

Figure 1: World Labs' functional modular perspective on World Models

After clarifying this capability stack, another key question emerges: if Renderers, Simulators, and Planners serve unified intelligent tasks with distinct roles, what enables their efficient collaboration?

Take the classic robotic cup-grasping scenario as an example. The three capabilities have clear divisions of labor but cannot operate independently with disjoint world perceptions. They must share a unified, consistent understanding of the world to avoid disconnections between perception, simulation, and decision-making. This leads to the core puzzle: what is the unified world representation that underpins the collaboration of the three capabilities?

I found the missing piece of this puzzle in Yann LeCun's research on JEPA and latent world representation.

## 2. LeCun's Missing Puzzle Piece: Latent World Representation

In recent years, Yann LeCun has repeatedly emphasized the core concept of Latent World Representation. His core viewpoint can be summarized succinctly: AI should not fixate on predicting raw pixels, but on learning and forecasting the latent internal representations of the world.

Though abstract at first glance, this concept becomes intuitive with practical scenarios. When a camera captures an image of a cup on a table, the raw output is merely RGB pixel data, which is superficial visual information. For a robot's decision-making, simulation, and action execution, pixels are irrelevant. What truly matters is its deep understanding of the physical world: the cup's spatial position and orientation, whether it contains liquid, its graspability, how it might shift or fall when touched, and its physical responses to external forces.

This distilled understanding, stripped of redundant pixel noise and focused on essential physical properties and causal dynamics, is the latent world representation. Renderers, Simulators, and Planners all operate based on this unified latent representation, executing their respective core functions of world expression, state prediction, and action decision.

```Plain
Reality
                       │
                       ▼
                 Observation
             (Camera / Sensor)
                       │
                       ▼
      Internal World Representation
             (Latent State)
                       │
      ┌────────────────┼────────────────┐
      │                │                │
      ▼                ▼                ▼
 Renderer         Simulator        Planner
 Express          Predict           Decide
```

Figure 2: Personal mental model of World Models

The three core capabilities rely on a shared latent world representation to perform world expression, state prediction, and action decision respectively. Combining this perspective with World Labs' framework clarifies the essence of World Models entirely. World Labs answers what capabilities a World Model requires, while LeCun answers what underlying world representation these capabilities are built upon.

The closed-loop operation of a World Model can be fully illustrated through the robotic cup-grasping task:

First, the robot collects raw observational data of the real world via cameras and sensors, obtaining superficial visual and physical observations.

Next, the system performs a critical transformation: it abstracts and refines fragmented raw observations into a unified latent world representation. This structured state includes key information such as the cup's position and pose, table boundaries, robotic arm coordinates, and object graspability, while filtering out redundant pixel noise.

The Simulator then initiates predictive simulation. Based on the current latent world state, it simulates diverse action scenarios: whether the cup will slide or shift if the robotic arm approaches from different angles, whether high-speed movement will cause collisions, and whether adjusting the grasping pose can improve success rates. Crucially, the Simulator does not predict future pixel frames; it forecasts dynamic changes and causal outcomes of real-world states, forming the core of intelligent anticipation.

Subsequently, the Planner leverages the Simulator's multi-dimensional prediction results to select the most robust and successful grasping path, generating precise action commands to drive the robotic arm's execution.

After the action is completed, the real-world state changes. Sensors recollect observational data, the system updates its latent world representation, and a new cycle of perception, simulation, and decision-making begins.

```Plain
┌──────────────────┐
               │     Reality      │
               └────────┬─────────┘
                        │
                        ▼
                  Observation
                 (Camera / Sensor)
                        │
                        ▼
          Internal World Representation
                  (Latent State)
                        │
        ┌───────────────┴───────────────┐
        ▼                               ▼
   Simulator                       Renderer
 Predict Future                 Express World
        │
        ▼
     Planner
 Decide Action
        │
        ▼
      Action
        │
        └──────────────────────────────┐
                                       │
                                       ▼
                                    Reality
```

Figure 3: The closed World Model loop for robotic cup-grasping

A noteworthy detail is that the Renderer is not necessarily involved in the closed control loop of robotic grasping tasks, yet it remains indispensable. It is responsible for externalizing internal world states: generating future video footage, visualizing robotic prediction results, and converting abstract internal latent states into human-understandable images or 3D scenes.

Therefore, Renderers, Simulators, and Planners are not competitive alternatives. Centered on the unified world representation, they undertake distinct and complementary responsibilities.

## 3. The Growing Importance of Representation

In past years, discussions on World Models have predominantly centered on prediction capabilities: predicting the next video frame, generating longer video sequences, and forecasting future states over several seconds.

However, since all predictive tasks are built upon world representation, the upper bound of an intelligent system's capability is determined not merely by prediction accuracy, but fundamentally by World Representation.

The World Representation defines a system's cognitive boundary: which physical laws it can identify, which noise it can filter out, which future states it can simulate, and which rational actions it can plan.

From this perspective, the evolution of World Models in recent years is essentially the continuous advancement of world representation paradigms. The field has progressed from processing raw pixels directly, to extracting shallow visual features, to recognizing discrete objects, and finally to modern structured, latent high-level world representations. Prediction remains vital, but it is increasingly built upon high-quality latent world representations.

```Plain
World Model
                         │
          ┌──────────────┴──────────────┐
          ▼                             ▼

      Function                  Representation

Answers:                       Answers:

What capabilities are needed?    How should the world be represented?

Renderer                    Latent Space
Simulator                   World State
Planner                     World Representation

World Labs                   Yann LeCun
```

Figure 4: Two core perspectives for understanding World Models: function vs. representation

## Conclusion: A Forward Prediction

The AI industry's core focus once centered on how to predict the world more accurately. At the current technical inflection point, I believe the next pivotal research direction will shift: how AI can build more accurate, generalizable, and physically consistent world representations.

This will likely be the key to breaking through the capability bottlenecks of future World Models.
