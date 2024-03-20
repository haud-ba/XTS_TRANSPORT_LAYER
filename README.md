# Introduction 
# XTS transport layer (a station based approach)
# --------------------------------------------------------------------
## XTS transport layer projects
- functional basics of CA Group
- use of XTS_Utility lib
- introduction to station based approach
- configurable station placement
- station acts as sender
- configurable station design with basic transport logic
- configurable station design with variable transport logic  
  
        individual targeting of mover to Station
        grouping of stations for multiplicated or sequenced work flow  
  
  
- function blocks with ctrl/state structs:  
  
        cyclic check on change of command enumeration  
        state struct enumeration with offsets for progress  
  
  
 - This project collection is intended to convey the idea of a stand alone XTS transport layer to use in heterogen environments / applications.
 The main idea is that for every process a corresponding position on the xts is required.
 For every process a mover has to be made available
 Every process works a mover.
 - In order to reduce the amount of repetetive work when implementing a XTS into a machine, this project collection may help to put a transport layer in place
 - A transport layer shall have an interface for guiding a mover through a process station
 - A transport layer shall have an interface to manipulate a mover within a station or for a certain task
 - A transport layer shall have an interface for setting-up or clearing the CollisionAvoidance Group
   
### The following explanations and descriptions shall help to set up a transportation layer that suits your requirements
  
<div style="page-break-after: always;"></div>


## Getting Started
-  **projects are numbered with rising level of complexity**
-  **choose your level of entry**  

-  ### TR_00:
    - TwinCAT project with XPU simulation and NC Axis  
-  ### TR_01: 
    - fb_Xpu           handles CA Group, global control struct to trigger error reset, building group and enabling mover  
-  ### TR_02: 
    - fb_Xpu           Tc3_XTS_Utility; OTCID Initialization and checks added  
  
-  ### TR_03: 
    - Introduction of a operating modes interface: fb_UnitControl
    - fb_UnitControl   
      - interface to extern control
      - interface to fb_Xpu
      - Operation Modes (change of mode is checked):  

      - **E_UNIT_CTRL :**  
      
              -   UNIT_NO_CMD,  
              -   UNIT_INIT     :=    10, // clear axis, clear group and not enable  
              -   UNIT_IDLE     :=    30, // do nothing, clear axis and group
              -   UNIT_HOME     :=    50, // home
              -   UNIT_AUTO     :=    80, // auto start pos //TODO: implement recovery procedure
              -   UNIT_RESET    :=    90  // clear all  

  
<div style="page-break-after: always;"></div>

  
-  ### TR_04: 
    - fb_Mover: interface to extern control; wraps MC function blocks
      - command and state interface for access to mover functionalities (e_Mover_Order)  
        - **ST_MOVER_CTRL**:  struct for extern control to write  
        - **ST_MOVER_STATE**: struct for extern control to read  
  
      - cyclic info data for mover
        - **ST_MOVER_INFO**:  struct with mover position and additional information

              -  bMoverEnabled            : BIT;    // Axis[i].Status.ControlLoopClosed
              - bMoverError               : BIT;    // Axis[i].Status.Error
              - bMoverCoupled             : BIT;    // Axis[i].Status.Coupled OR _fbGearInPosCa[i].InSync
              - bMoverStandstill          : BIT;    // Axis[i].Status.NotMoving
              - bMoverAvoidingCollision   : BIT;    // Axis[i].NcToPlc.StateDWord2.0

              - nMasterId       : USINT;          // mirror NC AxisId of extern Master axis
              - rModuloActPos   : REAL;           // TO_REAL(Axis[i].NcToPlc.ModuloActPos)
              - rAbsActPos      : REAL;           // TO_REAL(Axis[i].NcToPlc.ActPos)
              - rAbsTargetPos   : REAL;           // TO_REAL(fb_Xts._rLastSetPos[i])
              - rModuloTargetPos: REAL;           // TO_REAL(Tc2_Math.LMOD(_rLastSetPos[i], _RailLength))

<div style="page-break-after: always;"></div>

- ### TR_05:  
  - **Introduction of fb_Station: mover is handled by handshakes, targets can be set during operations**
  - **Stations are defined in ST_STATION_PARAMETER**
    - Station Parameter description see below (Who's who)
      
  - fb_Station       
    - interface to extern control; infeed from linked list entry, process handshake, sending mover to target and adding tail at linked list of target station  
    - command and state interface for access to station workflow  
    
          - ST_STATION_CTRL:  
            - eCmd            : E_STATION_CTRL;  
            - nMask           : BYTE;   // nest mask for sending mover to next station  
            - nTargetStation  : USINT;  // where to next? [index of station in global array]  
            - rOffset         : REAL;   // optional offset for mover in target station when leaving sending station;  
                                        // use case: position correction of workpiece on wpc by vision  
      
          - ST_STATION_STATE :
            - eState          : E_STATION_STATE;
            - nMask           : BYTE;   // current nest mask of mover in station
            - nMoverId        : USINT;  // mover ID in station
            - nQueue          : USINT;  // queue count for station

  - I_Station_LinkedList:
    - Linked list implementation for sending mover to target, every station has its own list
      - AddTailValue: 
        - sending station adds mover at tail of target station
      - GetHead:
        - station gets mover ID and optional data from top of its list
      - Count:        
        - how many mover were entered into list

  - I_StationMover:
    - Mover interface for use in fb_Station
      - GetMoverInfo:   
        - reads to ST_MOVER_INFO
      - Halt:
        - MC function block for setting mover passive in CA Group in standstill at WorkPosition
      - MoverAbsolute:
        - CA function block for pulling in mover from WaitPos to WorkPos, target and standstill is checked
      - SendToPosition: 
        - CA function block for sending mover to target station, mover has to move ReleaseDistance for success
      - RailLength:
        - used for calculating abs pos from modulo pos and crossing of modulo turn
  
<div style="page-break-after: always;"></div>

  
- ### TR_06: 
  - #### fb_UnitControl   UNIT_HOME added - all mover are commanded to home position  
  - Commands detected on change
  - state contains Command and offset for progress  
  
- ### TR_07: 
  - #### MAPPING          
    - Unions for mapping PLC to any cyclic runtime 
    - non cyclic acess via ADS can be set in MAPPING(PRG)
      
      
- ### TR_08: 
  - #### Extern Control PLC  
    - mapped structs in PLC 
    - two stations handling as example for process handshaking sequence
    
- ### TR_09: 
  - #### fb_MessageSystem 
    - template for a message system with categories  
    
- ### TR_09: 
  - #### fb_Configuration 
    - Station Parameters are read from file

<div style="page-break-after: always;"></div>

# Build and Test
- **each project is complete with XPU in simulation mode**
- TR_09 has file based configuration for 2 Stations
- copy xml configuration from project folder (see TC tree view) to \c$\TwinCAT\XPU_PLC_StationConfig\


# Members
  - ## ExternControl
    - simple example project for controlling XtsBase transport layer

    - fb_Process.Cycle is handshaking mover (TON for process)


  - ## XTSBase PLC

  - heavy use of pointers

  - designed for use with extern cyclic or non cyclic flow control
  - use of mapping in this example for connecting to second PLC (runtime)
  - station based approach with individual targeting of mover
  - handshake in station with extern process flow (ST_STATION_CTRL / ST_STATION_STATE)
  - individual cyclic mover interface with given set of movement functionalities (ST_MOVER_CTRL / ST_MOVER_STATE)
    
<div style="page-break-after: always;"></div>

  ### XtsBase - Who's who?

  #### GVL_XTS:
      constants are always upper case
      constants from XTS_Utility lib are mapped onto shorter names here
      use XTS_Utility lib TcIoXtsEnvironmentParameterList for setting up your system
        this project relys on those parameters to be correct
      AXIS_REF for mover
      instances of function blocks and Ctrl/State structs

  #### Communication:
      wherever you may roam
      MAPPING(PRG)
      data is mapped onto unions for connecting to any cyclic runtime

  #### Configuration:
      fb_Configuration
      station parameter struct is read from file, manual write available
      see GVL_XTS for file path

  #### Control:
      fb_Unit_Control
      xts and mover are set to defined state
      interface to extern control for mode selection
      current state of example is that command UNIT_HOME is sending all mover to startup position 
      and adds all mover to queue of startup station --> now handshake of stations can start

  #### MessageSystem:
      fb_MessageSystem
      not connected in project, just a template for further use

  #### Mover:
      fb_Mover cyclic interface for extern usage (ST_MOVER_CTRL / ST_MOVER_STATE)
      methods and e_Mover_Order are writing LastPosition and Last Gap for each mover on motion execute
      Interface pointer for use within fb_Station
      see E_MOVER_ORDER for available functionalities

  #### Station:
      fb_Station cyclic interface for extern usage (ST_STATION_CTRL / ST_STATION_STATE)
      handshake for mover infeed, process (nests), outfeed
      static offset datafield for every Mover in every station with every nest
      additional dynamic offset by LinkedList entry (used on first infeed)
      Interface to LinkedList use for adding mover to target station queue after mover has left the station

      know thyself
        - all coordinates are modulo values, from station to station only forward, 
          within station limits backward movement by use of negative nest offset 
          or use of ST_MOVER_CTRL. 
          IF move backwards you have to make sure that there is room for it 
          --> distance between WaitPos and WorkPos

        - station is defined by WaitPos, WorkPos, Nests and Release distance
          - Text:                     free to use for description in visu etc.
          - WaitPos:                  the sending station is using this value as target when releasing mover
          - WorkPos:                  position after infeed, process handshake is done here
          - Release:                  distance mover has to move in order for station to be empty again and check next mover
          - Gap, Velo, AccDec, Jerk:  used for infeed and outfeed
          - ConfiguredNestCount:      how many nests have to be worked on this mover
          - NestOffset:               is added to WorkPos, 
                                      beware when using negative offsets 
                                      (avoiding collision, no movement, no error)

  #### XPU
      fb_Xpu:
        - one Track, one Part
        - setting up of CA group for all mover
        - plausibility checks
        - connects to XTS_Utility lib
        - cyclic call of AXIS_REF
        - collects motor module info data

<div style="page-break-after: always;"></div>


# Distribute
**If you distribute you do also have to ensure support in case of questions from your recipients**

