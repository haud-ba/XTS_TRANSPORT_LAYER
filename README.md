
# XTS_TRANSPORT_LAYER: 
## A Station-Based Approach

## Introduction
This project collection provides a standalone, hard real-time XTS transport layer designed for heterogeneous industrial environments.
The Core Philosophy: For every physical process, a corresponding logical position on the XTS track exists.
To eliminate repetitive architectural overhead when integrating an XTS into a machine, this framework provides a tested foundation.

**WHY no closed library?** - Nobody learns anything by using a black box. This framework is open because the logic is meant to be verified, not just consumed.
**WHY here and now?** - I am deploying XTS sub-systems and machines, what you get is what I use, it took a while to get things together and document all of it in flowcharts
**WHY OOP?** - it's most helpful for what I do. And no, it is not an inheritance model from hell. You'll find an interface driven compositorial approach in the sources.


**It enforces:** 
- An interface for guiding a mover through a Process Station.
- An interface for manipulating a mover within a station for specific tasks.
- An interface for initializing and managing the Collision Avoidance (CA) Group.

- [DOCUMENTATION] For architectural deep dive here: https://github.com/haud-ba/XTS_TRANSPORT_LAYER/tree/main/DOCUMENTATION/Documents

- [EXAMPLES] For deployable machine templates, see: https://github.com/HAUDX/XTS_TRANSPORT_EXAMPLES 

- Use the 'Discussions' tab for questions and commentary.

**Scope of the XTS Transport Layer:**
- This framework abstracts the TwinCAT Tc3_XTS_Utility and MC3 libraries into a modular, station-based topology.

**Key Capabilities:** 
- Configurable Station Placement: Map logical stations to physical track coordinates.
- The Station as a Sender: Stations do not pull; they push. A station actively routes a mover to its next destination.
- Dynamic Transport Logic: **Individual, dynamic targeting of movers to specific stations.**
- Grouping of stations for parallel or serial workflows of external processes.
Function Blocks with Ctrl/State Structs:Cyclic execution based on command enumeration changes.
State struct enumerations with defined offsets for deterministic progress tracking (PROGRESS_DONE).

## Getting Started & Formatting Rules 
- FORMATTING: My OCD, my formatting. All spaces, no tabs. Indent 2.
**Read the code and the datatypes. I have left comments in the critical paths.**
 Start your analysis in MAIN(PRG) and GVL_XTS.
- Custom integration and tools for grouping of stations you find in the APPLICATION section of the base project
- Examples available in dedicated repository (see link above)

 
## The Class Hierarchy
 1. Collision Avoidance Class **(fb_CaGroup):** Handles the absolute state of the Collision Avoidance Group.ClearGroup, BuildGroup, EnableGroup
 2. Mover Motion Control Class **(fb_Mover):** Wraps the MC2 and CA function blocks into a cyclic, deterministic interface.
 3. XTS Processing Unit Class **(fb_Xpu):** Executes cyclic plausibility checks to the Processing Unit. Handles **Mover 1 detection** and OTCID initialization.
    Provides direct interfaces to the Tc3_XTS_Utility library.
 4. XTS Transport Class **(fb_TransportUnit):** The master interface to external control. 
    It orchestrates the fb_Xpu, fb_CaGroup, and fb_Mover. Operation modes are strictly checked via the E_XTS_TRANSPORT_CTRL enumeration (e.g., CMD_INIT, CMD_GROUP_BUILD, CMD_TRANSPORT_START).
    - Example of State Change Logic:

```
    _eCmd              := _Ctrl.Cmd;
    
    // check for command change
    // get state for cmd
    IF (_eCmd <> _eCmdOld)
    THEN
      _eState          := Cmd(_eCmd);
      _eCmdOld         := _eCmd;
    END_IF 
```

 5. The XTS Station Class **(fb_StationBase):** This is the abstract core of the routing logic. 
    This abstract base class is extened by **(fb_StationProcess)** and **(fb_StationGearInPos)** 
    A mover is handled strictly by handshakes, and targets can be dynamically updated during operations. 
    - Station Definition: Stations are defined statically in ST_STATION_PARAMETER.
    - The Interfaces: Control / State: The external interface for the IT/Process layer to command the station workflow.
      - ST_STATION_CTRL: Commands where the mover goes next (nTargetStation), which nests it stops at (nMask), and any vision-based offsets (rOffset).
      - ST_STATION_STATE: Reports the current nest mask, the active Mover ID, and the queue count.
      - I_Station_LinkedList: Every station possesses its own linked list.
        - AddTailValue: A sending station drops a mover at the tail of the target station's list.
        - GetHead: A station pulls the next Mover ID and optional data from the top of its own list.
        
    - Planning Requirements: **Know Thyself** 
      - Rule 1: The Modulo Boundary: Put the Modulo turn anywhere, BUT NOT within the WaitPos, StopPos, or ReleaseDistance of a station. 
        The code does not support a mover crossing the modulo boundary while actively executing station logic.
      
      - Rule 2: Forward Only. From station to station, movers only move forward. Within a station's limits, 
        backward movement is possible via negative nest offsets or ST_MOVER_CTRL, but you must guarantee there is physical room for it.
      
      - Rule 3: Station Topology (ST_STATION_PARAMETER). A station's physical reality is defined by four metrics: 
        - rPosWait: The exact coordinate a sending station routes the mover to.
        - rPosStop[1..8]: Up to 8 process positions, calculated relative to rPosWait.
        - rReleaseDistance: The distance the mover must travel before the station returns to checking its linked list.
        - nConfiguredStopCount: The maximum number of stops allowed in this station.
      
      - Advanced Topology: Parallel Stations. If a process requires parallel stations (e.g., 4 identical testers), 
        assign them a common rPosWait.
         - Example: GVL_XTS.Station[1] to [4] all share rPosWait := 100
            Differentiate them using the relative rPosStop[1] (e.g., 100, 200, 300, 400).
         - The ReleaseDistance of the furthest station must be the shortest, staggering backward to topologically sort the list in the receiving Station.
         - The Nest Mask (nMask). The default nest count is 1. If a mover requires multiple stops within a target station, 
            you must actively command the ST_STATION_CTRL.nMask before the mover leaves the sending station.
            
         - nConfiguredStopCount defines the physical limit.
         - nMask commands the active execution.

## Messaging & ScopeMessaging (GVL_MSG)
- The architecture features an automated chronological narrative. 
- It records Verbose, Info, Warn, and Error events, categorized by e_Device and e_SubDevice.
- Scope Included TwinCAT Scope projects are pre-configured to record the exact signals of Stations and Processes.
- State of the messaging: use of a ASCII file directly on the filesystem.
  - fb_MessageDataCtrl may be used as blueprint.
    - plug in your adapter to a database directly in this messaging module.
- messaging classes available to route process, application, OEE messages into their own files

## TRAINING [TR3056 Beckhoff Training]
 - Come to Nuremberg. We can talk architecture, debate topology, and code in person for days.
 - **TR3056** Beckhoff Training Nuremberg

