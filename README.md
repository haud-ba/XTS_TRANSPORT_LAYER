# Introduction 
# XTS transport layer (a station based approach)
# --------------------------------------------------------------------
## XTS transport layer projects
### [XTS_TR06] - intended for use 
 
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
 The main idea is that for every process a corresponding position on the xts.
  
  
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
    - fb_CaGroup
		- handles Collision Avoidance Group
		- ClearGroup
		- BuildGroup
		- EnableGroup
		
	- fb_Mover
		- MC2 function blocks
		- CA function blocks

	- GROUP(PRG), MOVER(PRG)
		- simple examples, will be replaced later
		
-  ### TR_02: 
    - fb_Xpu           
		- cyclic checks to ProcessingUnit
		- Mover 1 detection
		- access to local instance of Tc3_XTS_Utility function blocks; 
		- OTCID Initialization and checks added  
  
-  ### TR_03: 
    - fb_TransportUnit
      - interface to extern control
      - interface to fb_Xpu
	  - interface to fb_CaGroup
	  - Interface to fb_Mover
      - Operation Modes (change of mode is checked): 

			_eCmd                         := _Ctrl.Cmd;

			// check for command change
			// get state for cmd
			IF (_eCmd <> _eCmdOld)
			THEN
			  _eState                     := Cmd(_eCmd);
			  _eCmdOld                    := _eCmd;
			END_IF
	  
	        //////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

			{attribute 'strict'}
			{attribute 'to_string'}
			TYPE E_XTS_TRANSPORT_CTRL :
			(
			  CMD_NULL,
			  CMD_INIT                  := 10,  // Xpu, Group, Mover initialization: clear errors, clear group, check Xpu data, check mover detection
			  CMD_IDLE,

			  CMD_MOVER_ENABLE          := 20,  // all mover are enabled, checked for ControlLoopClosed
			  CMD_MOVER_DISABLE,                // all mover are disabled, no axis in CaGroup: no error, any axis in CaGroup: group error

			  CMD_GROUP_CLEAR           := 30,  // halt all movers, clear errors, remove all movers from group, clear group
			  CMD_GROUP_BUILD,                  // add all movers to group
			  CMD_GROUP_ENABLE,                 // group with movers is enabled

			  CMD_TRANSPORT_START       := 40,  // move all mover[i] to start position
			  CMD_TRANSPORT_RESTART             // move all mover[i] to LastPosition[i]

			)UINT;
			END_TYPE
  
<div style="page-break-after: always;"></div>

  
-  ### TR_06: 
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


# Build and Test
- **each project is complete with XPU in simulation mode**
- TR_09 has file based configuration for 2 Stations
- copy xml configuration from project folder (see TC tree view) to \c$\TwinCAT\XPU_PLC_StationConfig\


# Members
  - ## ExternControl
    - missing in this repo



  - ## XTSBase PLC

  - heavy use of pointers

  - designed for use with extern cyclic or non cyclic flow control
  - use of mapping in this example for connecting to second PLC (runtime)
  - station based approach with individual targeting of mover
  - handshake in station with extern process flow (ST_STATION_CTRL / ST_STATION_STATE)
  - individual cyclic mover interface with given set of movement functionalities (ST_MOVER_CTRL / ST_MOVER_STATE)
    
<div style="page-break-after: always;"></div>

  ### XtsBase - Who's who?

  #### MAIN:
	  you better call MAIN(), cyclic calls to everyone

  #### XTS_Parameter:
      constants are always upper case
      constants from XTS_Utility lib are mapped onto shorter names here
      use XTS_Utility lib TcIoXtsEnvironmentParameterList for setting up your system
        this project relys on those parameters to be correct

  #### GVL_XTS:
      AXIS_REF for mover
      global instances of function blocks and Ctrl/State structs

  #### XtsTransportUnit:
      fb_TransportUnit
      xts and mover are set to defined state
      interface to extern control for mode selection
      current state of example is that command UNIT_HOME is sending all mover to startup position 
      and adds all mover to queue of startup station --> now handshake of stations can start

  #### CaGroup:
	  fb_CaGroup
	  handles Collision Avoidance state
	  AddAll()
	  RemoveAll()
	  Reset()
	  Enable()
	  Disable()
	  
	  I_XtsTransport_Group
	  

  #### Mover:
      fb_Mover cyclic interface for extern usage (ST_MOVER_CTRL / ST_MOVER_STATE)
      methods and e_Mover_Order are writing LastPosition and Last Gap for each mover on motion execute
      Interface pointer for use within fb_Station
      see E_MOVER_CTRL for available functionalities

  #### XtsStation:
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

