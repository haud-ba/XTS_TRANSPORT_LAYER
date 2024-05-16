# Introduction 
# XTS transport layer (a station based approach)

## XTS transport layer projects
### [XTS_TR07] - intended for use 

- functional basics of CA Group
- use of XTS_Utility lib
- introduction to station based approach
- configurable station placement
- station acts as sender
- configurable station design with basic transport logic
- configurable station design with variable transport logic  

        individual targeting of mover to Station
        grouping of stations for parallel or serial work flow of extern process


- function blocks with ctrl/state structs:  

        cyclic check on change of command enumeration  
        state struct enumeration with offsets for progress  


 - This project collection is intended to convey the idea of a stand alone XTS transport layer to use in heterogen environments / applications.
 The main idea is that for every process a corresponding position on the xts exists.


 - In order to reduce the amount of repetetive work when implementing a XTS into a machine, this project collection may help to put a transport layer in place
 - A transport layer shall have an interface for guiding a mover through a process station
 - A transport layer shall have an interface to manipulate a mover within a station or for a certain task
 - A transport layer shall have an interface for setting-up or clearing the CollisionAvoidance Group

### The following explanations and descriptions shall help to set up a transportation layer that suits your requirements

<div style="page-break-after: always;"></div>


## Getting Started
-  **FORMATTING: my OCD my fomatting, allspaces no tabs, indent 2**
-  **please read the code and datatypes, in most places I left comments**
-  **... MAIN(PRG) and GVL_XTS are good places to start**
-  **projects are numbered with rising level of complexity**
-  **choose wisely**  

-  ### TR_00:
    - TwinCAT project with XPU simulation and NC Axis  
-  ### TR_01: 
    - #### fb_CaGroup
		- handles Collision Avoidance Group
		- ClearGroup
		- BuildGroup
		- EnableGroup

	- #### fb_Mover
		- MC2 function blocks
		- CA function blocks

	- #### GROUP(PRG), MOVER(PRG)
		- simple examples, replaced later they will be

-  ### TR_02: 
    - ### fb_Xpu           
		- cyclic checks to ProcessingUnit
		- Mover 1 detection
		- access to local instance of Tc3_XTS_Utility function blocks; 
		- OTCID Initialization and checks added  

-  ### TR_03: 
    - ### fb_TransportUnit
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

  - I_XtsTransport_Mover:
    - Mover interface for use in fb_Station
	- CA methods used for positioning and sending of mover


-  ### TR_07: 
  - **message handling: Verbose, Info, Warn, Error**
  - e_Device:    first category where message was set
  - e_SubDevice: second category where message was set
  - iError:      Error message from function block or MoverId for Verbose msg
  - .
  - **MessageData(PRG)**
	- automated write to file, new file for each day
  - **GVL_MSG**
	- namespace for everything message related



<div style="page-break-after: always;"></div>


# Build and Test
- **each project is complete with XPU in simulation mode**


# Members
  - ## ExternControl
    - missing in this repo



  - ## XTSTransport PLC

  - designed for use with extern cyclic or non cyclic flow control
  - station based approach with individual targeting of mover
  - handshake in station with extern process flow  
    (ST_STATION_CTRL / ST_STATION_STATE)
  - individual cyclic mover interface with given set of movement functionalities   
   	(ST_MOVER_CTRL / ST_MOVER_STATE)

<div style="page-break-after: always;"></div>

  ### XtsTransport - Who's who?

  #### XTS/_Visualizations:
	  XTS_TRANSPORT: main control of transport
	  - use ST_XTS_TRANSPORT_CTRL / ST_XTS_TRANSPORT_STATE 
	    (GVL_XTS.XtsTransportCtrl / GVL_XTS.XtsTransportState)
	  - StartUp sequence for CMD_TRANSPORT_START: (wait for each PROGRESS_DONE)
		- CMD_INIT or CMD_NULL
		- CMD_GROUP_CLEAR
		- CMD_GROUP_BUILD
		- CMD_GROUP_ENABLE: (mcGroupStateNotReady)
		- CMD_MOVER_ENABLE: (mcGroupStateStandby or mcGroupStateMoving)
		- CMD_TRANSPORT_START: (now handshake with ST_STATION_CTRL / ST_STATION_STATE can start)

	  STATION_VISU:  handshake for stations
		- regular handshake sequence as buttons

	  MOVER_VISU:    Access to cyclic mover interfaces.
		- use ST_MOVER_CTRL / ST_MOVER_STATE (GVL_XTS.MoverCtrl / GVL_XTS.MoverState)

  #### MAIN:
	  you better call MAIN(), cyclic calls to everyone

  #### XTS/XTS_Parameter:
	constants are always upper case
	constants from XTS_Utility lib are mapped onto shorter names here
	use XTS_Utility lib 
	TcIoXtsEnvironmentParameterList 
	for setting up your system
	this project relys on those parameters to be correct

  #### XTS/GVL_XTS:
	AXIS_REF for mover
	global instances of function blocks and Ctrl/State structs

  #### XTS/XtsTransportUnit:
	fb_TransportUnit
	xts and mover are set to defined state
	interface to extern control for mode selection
	current state of example is that command CMD_TRANSPORT_START 
	is sending all mover to startup position 
	and adds all mover to queue of startup station 
	--> now handshake of stations can start

  #### XTS/CaGroup:
	fb_CaGroup
	handles Collision Avoidance state
	AddAll()
	RemoveAll()
	Reset()
	Enable()
	Disable()

	I_XtsTransport_Group


  #### XTS/Mover:

	fb_Mover cyclic interface 

	- for extern usage (ST_MOVER_CTRL / ST_MOVER_STATE)
	- methods are writing LastPosition and Last Gap 
	- for each mover on motion execute
	- Interface pointer for use within fb_Station and fb_TransportUnit.
	- see E_MOVER_CTRL for available functionalities

  #### XTS/XtsStation:
  **rPosStop: is added to WaitPos,**
**beware when using negative offsets (avoiding collision, no movement, no error)**

	fb_Station cyclic interface 
	- for extern usage (ST_STATION_CTRL / ST_STATION_STATE)
	- handshake for mover infeed, process (nests), outfeed
	- static offset datafield for every Mover in every station with every nest
	- additional process offset by LinkedList entry
	- Interface to LinkedList use for adding mover to target station queue 
	- after mover has left the station

  #### Planning requirements for use of fb_Station:
	- Put the Modulo turn anywhere, 
	  BUT NOT within WaitPos, StopPos, ReleaseDistance of a station. 
	  The code does not support crossing the modulo turn within a station.

	- The Use of LinkedList methods (AddTail, GetHead) 
	  requires thought about when the mover is entered into the target station.

		- 1. parallel stations for a process:
			EXAMPLE:
			P1 uses XTS_STN[1] to XTS_STN[4] 
			--> The ReleaseDistance of STN[4] shall be shortest, 
				all other stations follow accordingly.
			EXAMPLE:
			condition: STN[4].WaitPos > STN[3].WaitPos
			for n := 3 to 1
				STN[n].ReleaseDistance := 
				  STN[4].WaitPos 
				- STN[n].WaitPos 
				+ STN[4].StopPos[furthest pos out] 
				+ STN[4].ReleaseDistance

		- 2. using stations sparsely:
			in this case it is easiest to always handshake 
			the stations and use the forwarding command if 
			a station shall be skipped: STATION_MOVER_SEND.

		- 3. deactivating stations:
			make sure the queue is empty before deactivating, 
			since the waiting mover will hold up all the others
			in case of required deactivation while movers are in the queue:
			- handshake mover with 
			  E_STATION_CTRL.STATION_MOVER_SEND to new target station
			- do not send any new mover to the station in question
			- disable station
			- preceeding stations continue workflow 
			  with changed ST_STATION_CTRL.nTargetStation

  #### know thyself
	- all coordinates are modulo values, from station to station only forward, 
	  within station limits backward movement by use of negative nest offset 
	  or use of ST_MOVER_CTRL. 
	  IF move backwards you have to make sure that there is room for it 
	  --> distance between PosWait and PosStop

	- station is defined by PosWait, PosStop[], and ReleaseDistance

	TYPE ST_STATION_PARAMETER :
	STRUCT
	  sText             : STRING(80); // only description
	  // start of station, a sending station is using this value to send mover to
	  rPosWait          : REAL;       

	  // distance from Mover.ActPos has to travel in order 
	  // for station to go back to checking list entries
	  rReleaseDistance  : REAL;       

	  rGap              : REAL;
	  rVelo             : REAL;
	  rAccDec           : REAL;
	  rJerk             : REAL;

	  // how many nests (stop positions) mover has to stop at (1 = default)
	  nConfiguredStopCount  : USINT := 1; // 1-8 --> NestMask = BYTE

	  // mover stop position in station, relative to rPosWait!!
	  rPosStop          : ARRAY[1..8] OF LREAL;
	END_STRUCT
	END_TYPE


  #### XTS/XPU
      fb_Xpu:
        - one Track, one Part, (InfoServer not yet)
        - cyclic plausibility checks to ProcessingUnit and MotorModules
		- Mover 1 detection after Init
        - connects local function blocks to XTS_Utility lib
        - collects motor module info data

<div style="page-break-after: always;"></div>


# Distribute
**If you distribute you do also have to ensure support in case of questions from your recipients**

