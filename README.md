Combining Behavior Tree and Finite State Machine
===
Introduction
---
Hi! In this ReadMe I will be explaining and showing my research project for my course Gameplay Programming in Howest DAE. I have always enjoyed the workings of behavior trees and finite state machines on their own, so felt like it would be suitable for me to try and put them together. This project is also more than just that, as I combined it with a project, I was making for another course that looks like a hunting simulator.

You will first find a short conclusion with the result of this project at the top to know if this is something you are looking for in your own project. You can read a more thorough  conclusion at the end with all the explanation  on how to implement this along the way. Feel free to look at the table of contents below and read up on what you need and skip what you feel comfortable with.

**Disclaimer:** Some naming may be different in this document than my project because it is clearer that way. In any case I will give links to all my files with the steps nonetheless so you cannot get lost.

Table of Contents
---

- [**Short conclusion & Result**](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/ReadMe/README.md#1-short-conclusion--result)
- [**Information on used data structures**](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/ReadMe/README.md#2-information-on-used-data-structures)
    - Behavior Trees
    - Finite State Machine
    - Combining BT and FSM
- [**Project setup**](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/ReadMe/README.md#3-project-setup)
    - Setup the project
    - Assets I used for my project
- [**Implementation of my project**](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/ReadMe/README.md#4-implementation-of-my-project)
    - What will we make?
    - Needed CPP classes
    - Making the NPC class
        - Making the NPC Blackboard
        - Making the NPC Behavior Tree
            - Adding the BT to the NPC
            - Adding a custom task to the BT
    - Making the FSM
        - Making the States
        - Making the Conditions
        - Making the States and Transitions
        - Making the Finite State Machine class
    - Making the NPC_AIController
    - Last changes inside some files
    - How it all ties together
- [**Conclusion**](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/ReadMe/README.md#5-conclusion)
    - What could be better?
    - Final word
- [**References**](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/ReadMe/README.md#6-references)

1: Short conclusion & Result
---
**⍟ Short conclusion:**
I made my project in Unreal 5.2.1. As Unreal already has a behavior tree inside their program, I found making a state machine not necessary  for any project. I do think it could work very well for when you are making
a C++ project with an enemy/NPC that needs AI input. As far as combining both the BT and the FSM (then made in C++ by yourself), I do not think Unreal is the best place for that. I was able to find a way to do it, but I recommend using them separately  for their own use, and not together in Unreal. Nonetheless, this was a very interesting research project.

**⍟ Result:**

[link to video](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/researchProject.mp4)

Link to a video I have provided showing my BT going off and stopping once it enters my FSM. More concrete info can be found in this ReadMe.

2: Information on used data structures
---
Before we begin, I want to give some basic information on the two data structures we will be using: behavior tree and finite state machine. You will need at least a basic understanding of both to continue this project with some level of confidence. I have given a lot of information to be thorough. Feel free to continue to the next point when you feel you have understood what you need for your implementations.

*[Pictures are from references noted with ★ ]*

**⍟ Behavior Trees (BT):**
- **What is a BT?:**
    - Mathematical Model of Plan Execution – describe switching between a finite set of tasks in a modular fashion. The behavior tree represents all possible tasks that your character can take.
- **The 4 kinds of nodes in a BT:**
    - **Root Node:**
        - **Has:** no parent ; one child (ticks)
        - **Meaning:** It is the start of your BT.
    - **Composite Node:**
        - **Has:** one parent ; one or more children
        - **Kinds of composite nodes:**
            - **Selector:** It will return 'success' back to the parent node if any of the children node tasks succeed. If one child fails, it will continue to the next child. Only returns 'failure' back if all children tasks fail.
              
             <img width="379" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/4e3850c6-553b-496f-a2e8-4f7bea16dfa2">

            - **Sequence:** Returns 'failure' if any of the children fail and returns 'success' if all children pass. Depending on the program if the sequence has an order: it can add more unpredictability to an AI character in cases where there is not a clear order. Or when there is an order, sequences can be set up to play a predicted sequence for the AI. For example, in Unreal (5.2.1) sequences children activate from left to right.
              
             <img width="381" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/27a6f3f3-d25d-4a3e-b0fa-1f44d735b5fb">

            - **Parallel:** Acts similar to the sequence node. Also returns 'failure' if any of the children fail and returns 'success' if all children pass. The difference is that instead of going through each child at one time, all children nodes will be activated at the same time. This also means: if one child fails, it will stop all running tasks and send back 'failure'. Unilaterally terminating all threads could cause problems. Leaving the game inconsistent or failing to free the thread. A solution for this: termination of all threads is a request rather than a direct termination. For this to work, all tasks need to be able to receive a termination request and be able to clean up after themselves.
              
              <img width="170" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/1a305325-60ee-4e16-929c-ec148a48401c">

    - **Leaf Node:**
        - **Has:** one parent ; no child
        - **Meaning:** These nodes perform some computation and return a state value.
        - **Kinds of leaf nodes:**
            - **Condition node:** Checks whether a certain condition has been met or not.
              
              <img width="176" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/ac1f5863-3e33-4348-a2da-c987f15ce227">

            - **Action node:** Performs computations to change a state.
              
              <img width="165" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/692554ce-2298-47a0-a066-3d1307ff2c7c">

    - **Decorator Node:**
        - **Has:** one parent ; one child
        - **Meaning:** A node that can either transform the result they receive from their child node's status to: terminate the child, or repeat processing of the child. Dependent on the type of decorator node.
        
          *[Symbol on graph: ◇]*
          
          <img width="251" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/b930d8b7-ab9e-4c5e-986b-6f36ed674b94">

        - **Kind of decorator nodes:**
            - **Inverter:** Inverts or negates the result of the child node.
            - **Succeeder:** Always returns success to the parent node, irrespective of what the child node returned. Can be used for cases where you want to process a branch where failure is expected or anticipated, but you don't want to abandon processing of a sequence that that branch/node sits on.
            - **Repeater:** Reprocess the child node each time it returns a result. This makes the tree runs continuously and can also be optionally set for a number of times to after return back to the parent node.
              
              <img width="111" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/96332de8-3a71-4524-8497-5dfe96ef2cd1">

            - **Repeat until fail:** It's like a repeater, but it will repeat until the child returns 'failure'. When it returns 'failure', the decorator node sends a 'success' to the parent.


- Decorator and Leaf nodes can help in defining the flow of your code and can be used with parameters.

- **Running a BT can result in 4 statuses:**
    - **Running:** If the execution has not finished yet.
    - **Success:** If the goal is achieved.
    - **Failure:** If the goal is not achieved.
    - **Error:** Only returned when an unexpected error happened inside the tree.
 
- **Benefits of using a BT:**
    - **Maintainability:** Nodes can be made independent from each other: easy to make/delete/change nodes without having to re-do the whole BT. With this being said; nodes are able to read and/or write to variables provides in the BT. These can be processed later with contextual data and allow the behavior tree to act as a cohesive unit.
    - **Scalability:** When you have a lot of nodes, they can be decomposed into smaller sub-trees. Saving some readability.
    - **Reusability:** Due to the independence of nodes in a BT, sub-trees are independent and can be taken and used in other branches of the BT, other BT's or other projects.
    - **Failsafe:** Branches in BT can be used for an AI to have a fallback tactic in case the previous behavior(s) would fail. This makes full crashes easier to handle/fix.
 
- **Downfalls of using a BT:**
    - Tree traversal issues:
        - It will always start from the root node, never from another one.
        - If you have a big BT, it can be inefficient to keep having to go through a whole BT for one task.

- **A picture of a BT as an example:**
    - A simple BT to show how to open a door:

<img width="517" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/d782190b-86d0-4d3d-8735-c0333332d4ea">

---

**⍟ Finite State Machine (FSM):**

- **What is a FSM?:**
    - A state machine with a set number of states.
    - Example:
      
      <img width="377" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/0c8435dd-1cce-4945-97d2-466a712d3818">


- **What is a state machine?:**
    - State machines take account of both the world around them (decision trees, transitions) and their internal state. As long as there is no event or influence to change the current state, it will continue carrying out the actions of that state in a loop.
 
- **How do states work in a FSM?:**
    - In a FSM, one character can only have one state.
    - Each state can/has its own actions, behaviors and/or variables.
    - States are held together by transitions.
      
      <img width="101" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/85a7fd71-0faa-414f-b945-883d194c7f5b">


- **What are transitions?:**
    - Each transition in a FSM leads from one state to another: the target state, and has a set of conditions.
    - if the FSM/game determines that the conditions of the transitions are met, it will trigger. It triggers a change from the state it was in to the new state found from the transition.
      
      <img width="173" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/78bf828e-8663-49c5-8001-7018c8d323c9">


- **Downfalls for using a FSM:**
    - **It is final:** There can be no states/conditions added or changed inside their actions while the game is being played.
    - **Maintainability:** When adding or removing a state, it is necessary to change the conditions of all other states that have transition to the new or old one. Big changes are more susceptible to errors that may pass unnoticed.
    - **Scalability:** FSM can have infinite states and transitions. This can get exceptionally large and very unreadable very fast. This also makes a FSM very hard to debug and losing the advantage of graphical readability.
    - **Reusability:** As the conditions are inside the states, the coupling between the states is strong, being practically impossible to use the same behavior in multiple projects.

---

**⍟ Combining BT and FSM:**

- **FSM inside of a BT?:**
    - Lots of resources recommend a BT with FSM inside it. This is due to BT being good at a lot of different behavior switches and different tasks, but subtrees can become exceptionally large. This is where FSM comes in. BT ensures everything goes to the correct state and then FSM can work out what it needs to do. This makes everything very readable.

- **So rather a BT in a FSM?:**
    - Other sources do state you can still have BT in a FSM too. This way, you can make sure and know in which state you are in at all times. You can then let the state roll out a BT with the conditions it needs to switch to another state if needed.

- **What is best then?:**
    - Each can be used and each in diverse ways and forms. The consensus was that the programmer should look at the project carefully and plan out what will be needed and best suited for the tasks that needs to be done.
Either way, combining FSM and BT can make code more readable and manageable for the programming making it.

**Examples:**

I will show 2 examples of a BT in a FSM and a FSM in a BT. Source is: https://medium.com/@abdullahahmetaskin/finite-state-machine-and-behavior-tree-fusion-3fcce33566 ; which can also be found at the references at the end of this ReadMe. As there is not a lot of information online and I found this source that had clear examples, I wanted to include their work here, but please also go look at their original post.

- **FSM inside of a BT:**
    - **Three different ways of putting a FSM inside of a BT:**
        - **State Machine:** It runs the state where the condition is true. It cannot be divided by anything other than the prioritized state over this state’s priority.

        - **Sequenced State Machine:** It proceeds through the states in order from start to finish, and exits the state machine once all states have been completed.

        - **Endless Sequenced State Machine:** Similar to the sequenced state machine, it goes through the states in order from start to finish. However, when all states have been completed, it starts again from the initial state and continues indefinitely in a loop. It cannot be interrupted by any other state unless a prioritized state overrides its current state.
 
![image](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/882869da-49b2-4654-9dc6-aa670824c03c)
    
- **Example of usage:**

![1_0tZJsd6X-IvoVUsaSBjksg](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/d93c7b12-7275-4038-9d9f-9d3ddd5d1b20)


- **BT inside of a FSM:**
    - Behavior tree is a well-known design pattern for writing reusable behaviors. One of the things I did based on the sequence creation capabilities of behavior tree is the division of state machines into types.
    - However, the part of behavior tree that is actually implemented in this architecture is the selectors. Selectors create the structure that decides whether a state should be executed or not. Each state has its own enter and exit selectors, so if a condition needs to be used for the entry of two states, it does not need to be rewritten.
    - Additionally, the support for multiple selectors allows dozens of conditions to be added in a row, but it is beneficial to keep the conditions clear or concise to avoid it becoming a costly operation. Because one of the biggest disadvantages of the behavior tree pattern is that it constantly needs to check all conditions. It is not a problem at the top layers, but as the complexity increases, the cost begins to increase significantly.

![1_OBcQQ1IU8FEj-HQgA2KgLA](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/a7f7561c-c1c7-4900-856a-8a6558664c0b)




3: Project setup
---
**⍟ Setup the project:**
I will take you through my project. A lot of code will be linked and explained here, or the comments should be enough to explain it to you. It is a lot of code, so I will try to make it as concise as possible.

Open Unreal 5.2.1 and make a new blank project in C++. You can also use the third person project to already have basic movement, as I will not be going over that here. My project might look a little different in the character files, as I started from blank. I have also installed Rider as my preferred IDE. Feel free to use what you are comfortable with. I will be making my FSM in C++ inside of Unreal and using the integrated BT system.

<img width="894" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/3d45838a-e97b-4851-bc85-f68275dde698">

---

**⍟ Assets I used for my project:**
- [Landscape Pro 2.0 Auto-Generated Material](https://www.unrealengine.com/marketplace/en-US/product/landscape-pro-auto-generated-material)
- [ANIMAL VARIETY PACK](https://www.unrealengine.com/marketplace/en-US/product/17c2d7d545674204a7644c3c0c4c58ba)

4: Implementation of my project
---
**⍟ 4.1. What will we make?:**

This is a very small FSM inside of a BT setup, but can very much be expanded on.

We will be making an animal AI with perception. Inside the BT, the animal will be walking around to random points, until it sees us. If the sight perception sees us, the animal will go into the FSM and starts running through the states. You can see some information on the FSM below:

- **States:**
    - **Detecting:** Stand still and increase the perception range.
    - **Run:** Run away from the player.
    - **Wary:** Moves around slowly with a slight increase in perception range.
    - **Idle:** FSM is not used and will stand in Idle.


- **Transitions:**
    - **Can see the player:** Returns true when you sensed the player with AI perception.
    - **Ran away long enough:** Counts down a local time, return true when you reach it. This is to let the player run away for x amount of time.
    - **Slow walked for long enough:** Counts down a local time, return true when you reach it. This is to let the player walk slowly for x amount of time.
    - **Detected enough:** Counts down a local time, return true when you reach it. This is to let the player go back out of the FSM when you cannot detect anything anymore.

- **This in a schematic form is:**
  
<img width="317" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/20e460b8-0e6a-4454-a57c-f72d7fe3b77d">

---

**⍟ 4.2. Needed CPP classes:**

We will need multiple C++ classes. You can make these by going in your C++ folder in Unreal and pressing right click.

<img width="370" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/51c4c98a-6147-4fc4-b841-b1055353ef76">

- **The following classes should be made:**
    - **NPC** : inheriting from Character : for your NPC class
    - **Character** : inheriting from Character : for your player character class *(third person project might already have this)*
    - **NPC_AIController** : inheriting from AIModule.AIController : the controller for your NPC
    - **FFinalStateMachine** : inheriting from None
    - **FSMStates** : inheriting from None
    - **FSMCondition** : inheriting from None
    - **StatesAndTransitions** : inheriting from None
 
We will go through what they need at another stage.

---

**⍟ 4.3. Making the NPC class:**

! Please note this is the **NPC** class and not the Player class. I will only go over the classes that relate to the NPC and my research project. You are free to look in my code files to find implementations for other items. We will only touch the player class in C++ for sight perception, nothing else.

- **Make a blueprint class based on your NPC C++ class**:
    - You can do this by right-clicking your C++ class.
      
      ![image](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/1509eed3-453f-49c7-af85-a4dea1d7cabb)

    - Following the same assets I used, you can place a deer skeletal mesh in the NPC for visuals.
      
      <img width="321" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/81b7cd60-584f-4ec2-8227-5fa3f0b6c97b">

    - Next open the blueprint and search for "Pawn" in the details. You will find a variable that says **"AI Controller Class"**, give this your own C++ made Controller called "NPC_AIController" from step 2.
 
    ![image](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/796eb451-823f-4731-bb78-6d257b0b3c56)



We will add more later.

---

**⍟ 4.3.1 Making the NPC Blackboard:**

- In a folder for your NPC, make a new blackboard for your NPC naming it **BB_NPC** .
    - You can find this in the following branch when you right click in your empty folder:
       
    <img width="407" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/d114dbc2-a515-4ac2-b6d1-f63ac0b6a8ca">

---

**⍟ 4.3.2 Making the NPC Behavior Tree:**

- In a folder for your NPC, make a new behavior tree for your NPC naming it **BT_NPC** .
    - You can find this in the same branch as the Blackboard

- Open it and make sure the blackboard got connected to the behavior tree in the details, if it did not: give the blackboard **BB_NPC** you just made in 3.1 in the details.

    <img width="391" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/8e9a3082-da2e-413e-9b0e-5a17ff9a4707">

---
**⍟ 4.3.2.1 Adding the BT to the NPC:**

- Open your NPC C++ class.
- Add a **protected UPROPERTY that is a UBehaviorTree***
    - While being a UPROPERTY: we will be able to edit the value in the blueprint editor directly. Just make sure to give the correct info in our UPROPERTY to be able to do that.
      
      <img width="561" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/cdbfccf5-89cf-4f7a-bb5c-bc80661bd0a0">

- Then add a public function **GetbehaviorTree** and implement it in C++. This is just a standard getter to be used later.

    <img width="252" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/c15f7ee3-6995-4aa2-9dbf-bd7e5a71a581">
    <img width="283" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/65ef2815-9ba1-4056-97b8-1e6be6ec2267">

- **Before you continue don't forget to save and recompile your C++ code while Unreal is closed and open Unreal again after.**

- Then open the **BP_NPC** again and search in the details for tree. You can now give the **BT_NPC** you created to the NPC through the C++ code you created.

    <img width="319" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/312f4bc2-1f18-4352-9019-b0f913854cc0">



---

**⍟ 4.3.2.2 Adding a custom task to the BT:**

- To show the behavior tree working on its own, I made a behavior tree task. I do this exact task in C++ at another time and if you wish to make it fully in C++ you can find that here: [press link](https://www.youtube.com/watch?v=1g1nhuy-kAI&list=PLWUvrI0mg8VKTJWKqsDn_xyZfD3f7Xye-&index=5). I did not make it in C++ as I already do it at another point and wished to visually separate it. 

- Create a new folder in your NPC folder named **"Tasks"**
    - Press right click and select **"Blueprint Class"**
    - Then search for: **"BTT_ask_BlueprintBase"** and make one called **"BTT_RoamingLocation"**
      
      <img width="473" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/b9eeccb7-18b2-4221-9047-c77e2e827a02">

- After open the newly made **BTT_RoamingLocation**:
    - Add the following variables to the class:
        - Radius: [Float] : The radius in which a random location can be given back.
        - RandomLocationOut : [BlackboardKeySelector] : The blackboard-key which we will give the random location info to.

      <img width="278" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/72a86e26-335a-4e89-afa3-6492c2c2ae1c">

    - Then **add the following graph**:
        - This is mostly self-explanatory. Gets the actor's location and gives back a new location that can be reached in the radius from that given location. After puts it in the given blackboard key selector.
      
      <img width="758" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/a32b796c-ea9c-4d56-b94b-565a8a62ffc7">

- After, go back to your **BB_NPC** and press the button **"New Key"** and give it the following info:
  
  <img width="638" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/bf33e612-b36d-4cd9-8381-6bf64412291f">

- After, go to your **BT_NPC**
    - Add the following graph by dragging out arrows from the bottom:
        - Be careful to put them in the same order, as the "Selector" works from left to right.
          
          <img width="568" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/e4a48bca-53ea-47ed-825c-285eac2ef305">

    - Then click on your task **"BTT_RoamingLocation"** and give it a wanted radius and the blackboard vector key you just made.
      
      <img width="382" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/b2070c1c-cb91-4406-9889-152071b12307">

    - Then click on the **"MoveTo"** task and give it the same location vector key you gave to your custom task:
      
      <img width="385" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/052a5d9a-7d9a-4f93-910e-a1bb4f03d3ac">

    - The wait node is my own preference; you may choose the time or to not use it in general.

You can now compile and build, and the NPC should be roaming around searching for a location and walking towards it.

---

**⍟ 4.4 Making the FSM:**

There are a lot of variables that go around; I recommend making the whole of 3.3 before trying out anything as I will go file per file.

---

**⍟ 4.4.1 Making the States:**

You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/FSMStates.h)

- This is a base state class that we will inherit from later
- The variables are all things I want to change/update when a state changes. I define them in the base class to avoid duplication in code lines.
- Everything else is explained in the comments with the code.
- I put everything in the .h file and deleted the unused .cpp file that was automatically created.

---

**⍟ 4.4.2 Making the Conditions:**

You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/FSMCondition.h)

- This is a base condition class that we will inherit from later
- I have an extra ResetVars() function to reset variables if needed. Not all classes need this so its default initialized, but I want to call this function from the base class and avoid duplication where I can.
- I put everything in the .h file and deleted the unused .cpp file that was automatically created.

---

**⍟ 4.4.3 Making the States and Transitions:**

I have deleted the pre-made functions/constructor in the files as these are not needed.

- The .h file:
    - You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/StatesAndTransitions.h)
    - States:
        - Each state inherits from the base class we just made in 3.3.1.
        - Few states have extra functionality and have extra member variables for it, all explained in the .cpp files.
    - Transitions:
        - Each state inherits from the base class we just made in 3.3.2.
        - Conditions that are time based have local float timers that are reset in the reset function from the base class.

- The .cpp file:
    - You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/StatesAndTransitions.cpp)
    - All code is explained in the comments with the corresponding code. Everything should be clear when reading the code + comments.

---

**⍟ 4.4.4 Making the Finite State Machine class:**

- The .h file:
    - You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/FFinalStateMachine.h)
    - As you know how FSM work by now: there is an "AddTransition" function to add a transition where the FSM knows from which state they can go to another state if a condition is fulfilled. These will get added in the TransitionMap declared. This can be confusing so will be explained below. Besides that, there is a function to change the current state of the FSM.
    - TMultiMap TransitionMap explenation:

        ```typedef TPair<FSMCondition*, FSMStates*> FTransitionStatePair;```

        - A TPair is created as a transition pair. This puts the condition and the state it will go to after in a pair together.
        - To other C++ programmers; this is used for about the equivalent of the std::pair

        ```typedef TArray<FTransitionStatePair> FTransitions;```

        - We put the pair in a TArray so one state can have multiple transitions
            - To other C++ programmers; this is used for about the equivalent of the std::vector
     
        ```TMultiMap<FSMStates*, FTransitions> TransitionsMap;```

        - We put that TArray with another FSMState inside a TMultimap.
            - The FSMState in the transition map signals to the start state. This is the state the NPC must be in to then get the transitions for.
            - I used a multimap after considering multiple other containers due to the ability to store duplicates.

- The .cpp file:
    - You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/FFinalStateMachine.cpp)
    - When initializing the FSM, you can give the state with which it will start.
    - Other code is explained all in the comments in the cpp file.


---

**⍟ 4.5 Making the NPC_AIController:**

- I will not be talking about my AI perception code. You are free to read my comments explaining them and I will link references that I used for such work at the end if you are interested.
- The .h file:
    - You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/NPC_AIController.h)
    - Initializes everything the NPC and FSM for the NPC would need. You can look at the comments to see what is what.
- The .cpp file:
    - You can find the file with the code [here](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/NPC_AIController.cpp)
    - When the AI_NPCController possesses an NPC, set up your FSM.
        - Firstly, you make all your states and conditions.
        - Then you initialize your FSM in your possession function.
        - When you have made a FSM, you can add the transitions into that FSM.
    - More info about what happens can be found in the comments.

---
**⍟ 4.6 Last changes inside some files:**
- Inside your character C++ files:
    - For sight perception, you need to add some items, look at code below and add them.
        - [H file](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/CPP_Character.h)
        - [CPP file](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/CPP_Character.cpp)
-Inside your NPC C++ files:
    - For the sight perception, you need to add some items, look at code below and add them. These are used to be able to change the variables from the Unreal editor.
        - [H file](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/NPC.h)
        - [CPP file](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/blob/main/EndProject/Source/EndProject/NPC.cpp)
- Inside your BB_NPC:
    - Add the following variables as bools:
      
      ![image](https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/e7d27129-3040-4aea-a0a8-8cea8034f969)

- Inside your BT_NPC:
    - Add to the tree with the following behavior:
      
      <img width="795" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/a49c367d-73e7-4e6b-97aa-24b62a981ed0">

    - The blue boxes are decorators on top of the sequences that each represent a different branch to go into.

- Inside your project settings:
    - Change the basic classes to the custom ones you have made.
      
      <img width="425" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/2ed16519-534c-4e32-ac4e-d7d8bc24eba2">


---
**⍟ 4.7 How it all ties together:**

And that was all the work! Now some extra information on how it all ties together if you are unsure:

- How do the BT and the FSM work together?
    - As the NPC spawns and gets owned by its controller, it looks at the BT it has.
      
      <img width="727" alt="image" src="https://github.com/Howest-DAE-GD/gpp-researchtopic-BauwkeS/assets/124317645/cd28a4f8-d664-4b85-a48d-5355479712c3">

    - As you can see here, the selector will either go one way or the other, depending on if the boolean "CanSeePlayer"is true or false.
        - If it is false, this means the NPC will keep looking at the sequence of finding a new point to walk to and waiting.
        - If it is true, it will render the left branch of the BT inaccessible. Inside of your code, you also gave a transition to the FSM saying if you saw the player, the state will change to the run state. This has made the FSM become active and run through its states according to if the player is perceived again. Every time it is, the FSM will go back to the run state. It must get through the run, detecting and wary state before returning to the idle state. When that happens, you know you cannot see the player anymore and will put this back to false. This makes the BT restart over again to the left side.
        - Whenever the player is perceived, it will break off any remaining actions in the BT and move on to the FSM.

This is a short final explanation. If you feel you missed something, feel free to scroll back to the difficult part(s).

5: Conclusion
---
**⍟ What could be better?:**
My implementation of my FSM was not bad at all. I used my course work and the book AI for Games (by Ian Millington) as reference. It was done almost fully as intended. I think I could improve on my container structure and Addtransition function. I doesn not fully work as I intended it to be. Yes, it works perfectly fine due to being in a TMultiMap but it is not the best solution. I would come back to this to change this if I would continue to work on this.

Secondly, I still wonder if there is a better transition from the BT to the FSM and back. I do think my implementation works and is not bad and does not create a lot of unnecessary variables, but I feel like there might be a better way. This could be a good thing to continue to explore when working further on this.

**⍟ Final word:**
Although this research took me a couple of weeks and a lot of tears, I think I have learned an extreme amount. About data structures, but also about Unreal. This was my first C++ work in Unreal and I have read so many of the official documentation and pages of forums of google searches; I feel like I learned a lot. This made me realize I do enjoy working in Unreal a lot and it would definitely be something I will come back to in the future for passion projects, if not my job.

Thank you for reading this whole ReadMe. I am aware it is long, but I wanted to make sure to give a lot of info on all the aspects of my research project. 

6: References
---
**⍟ REFERENCES FOR SECTION 2 DATASTRUCTURES ⍟**
- Forums:
    - https://forum.unity.com/threads/behavior-trees-and-finite-state-machine-discussion.462903/
    - https://www.gamedev.net/forums/topic/700989-fsm-bt-htn-goap-other/
- Other:
    - ★ https://web.stanford.edu/class/cs123/lectures/CS123_lec08_HFSM_BT.pdf
    - https://medium.com/@abdullahahmetaskin/finite-state-machine-and-behavior-tree-fusion-3fcce33566
    - https://www.gamedeveloper.com/programming/behavior-trees-for-ai-how-they-work
    - https://www.gdcvault.com/play/1021848/Building-a-Better-Centaur-AI
    - https://web.archive.org/web/20140327131120/http://www4.ncsu.edu/~drwrigh3/docs/courses/csc216/fsm-notes.pdf
    - https://www.gamedeveloper.com/programming/are-behavior-trees-a-thing-of-the-past-
    - https://medium.com/@abdullahahmetaskin/finite-state-machine-and-behavior-tree-fusion-3fcce33566
    - ★ Book: Ai for Games [3rd ed.] by Ian Millington
- Extra informational ones if you want to learn more (not included in the read-me due to being too indepth):
    - https://web.archive.org/web/20170118123034/http://users.etech.haw-hamburg.de/users/Schwarz/En/Lecture/Ds/Notes/DigSys1.pdf
    - https://web.archive.org/web/20110715110405/http://www.fceia.unr.edu.ar/asist/harel01.pdf
  
    

**⍟ REFERENCES FOR CODING ⍟**

**⍟ Unreal Documentation: (specific useful ones)**
- AI based:
    - https://docs.unrealengine.com/5.2/en-US/API/Runtime/AIModule [and all branches under this one]
    - https://docs.unrealengine.com/5.3/en-US/ai-debugging-in-unreal-engine/
    - https://docs.unrealengine.com/5.3/en-US/BlueprintAPI/AI/Components/Blackboard/
    - https://docs.unrealengine.com/5.2/en-US/ai-perception-in-unreal-engine/
- Containers C++:
    - https://docs.unrealengine.com/5.0/en-US/map-containers-in-unreal-engine/
    - https://docs.unrealengine.com/4.27/en-US/API/Runtime/Core/Containers [and all branches under this one]
    - https://docs.unrealengine.com/5.0/en-US/API/Runtime/Core/Containers [and all branches under this one]
- C++:
    - https://docs.unrealengine.com/5.2/en-US/console-varaibles-cplusplus-in-unreal-engine/
    - https://docs.unrealengine.com/4.26/en-US/ProgrammingAndScripting/ClassCreation/CodeOnly/
- Other:
    - https://docs.unrealengine.com/5.3/en-US/API/Runtime/Engine/Kismet/UGameplayStatics/ [and all branches under this one]
  
**⍟ YouTube:**
- Used in project:
    - https://www.youtube.com/playlist?list=PLWUvrI0mg8VKTJWKqsDn_xyZfD3f7Xye- (some parts)
    - https://www.youtube.com/playlist?list=PLffQ7MqqczuvBdyn1QJqUbPrA6CjLseUc (some parts)
- Used only for information purposes:
    - https://www.youtube.com/watch?v=hr9ybCCPw9Y&t
    - https://www.youtube.com/watch?v=YSOAKnnS8iw
    - https://www.youtube.com/watch?v=I_zw09kfqPA
    - https://www.youtube.com/watch?v=YSOAKnnS8iw&t
    - https://www.youtube.com/watch?v=I_zw09kfqPA&t
    - https://www.youtube.com/watch?v=EZBnx-ZSoS0
    - https://www.youtube.com/watch?v=0MhL9O09ZsM&list=PLZlv_N0_O1ga0aV9jVqJgog0VWz1cLL5f&index=93&t
    - https://www.youtube.com/watch?v=OytuX_swh8M
    - https://www.youtube.com/watch?v=GdLvZacVmkA
    - https://www.youtube.com/watch?v=hx4a9SwbGdw
    - https://www.youtube.com/watch?v=-t3PbGRazKg&list=PLNwKK6OwH7eW1n49TW6-FmiZhqRn97cRy&index=2&t
    - https://www.youtube.com/watch?v=blNza5q7sp4
      
**⍟ Other:**
- Course material 2023 Gameplay Programming in Howest DAE
- https://en.cppreference.com/w/cpp/container/map
- https://www.jetbrains.com/help/rider/Unreal_Engine__Debugger.html
- Containers:
    - https://github.com/MrRobinOfficial/Guide-UnrealEngine?tab=readme-ov-file#tsortedmap
    - https://cpp.hotexamples.com/examples/-/TMap/CreateIterator/cpp-tmap-createiterator-method-examples.html#google_vignette
- Forums:
    - https://forums.unrealengine.com/t/get-the-last-element-of-a-tmap/604663/10
    - https://forums.unrealengine.com/t/crash-on-accessing-a-tmap/366137
    - https://forums.unrealengine.com/t/how-to-get-ai-controller-in-c/594116
    - https://forums.unrealengine.com/t/print-to-screen-using-c/357351
    - https://forums.unrealengine.com/t/convert-float-to-fstring-c/275580/2

