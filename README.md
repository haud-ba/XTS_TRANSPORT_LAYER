# Introduction 
# XTS transport layer (a station based approach)

### [XTS Transport Layer]
  - XtsTransport PLC intended for use

### [DOCUMENTATION] 
  - pdf you should read,  a very simple example and a basic scope project


### Scope of the XTS Transport Layer:
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

-  ### Collision Avoidance class: 
    - #### fb_CaGroup
		- handles Collision Avoidance Group
		- ClearGroup
		- BuildGroup
		- EnableGroup
	
-  ### Mover Motion Control class:
	- #### fb_Mover
		- MC2 function blocks
		- CA function blocks

		
-  ### XTS Processing Unit class: 
    - ### fb_Xpu           
		- cyclic checks to ProcessingUnit
		- Mover 1 detection
		- access to local instance of Tc3_XTS_Utility function blocks; 
		- OTCID Initialization and checks added
		- provides interfaces from XTS_Utility lib
  
-  ### XTS Transport class: 
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

  
-  ### XTS Station class: 
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


-  ### Messaging: 
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


# Members
  ## APPLICATION (ExternControl)
   - example for handshaking of XtsStations
   - **copy/paste at your own risk**


  ## XTS Transport PLC
  - transport layer project
  - designed for use with extern cyclic or non cyclic flow control
  - station based approach with individual targeting of mover
  - handshake in station with extern process flow  
    (ST_STATION_CTRL / ST_STATION_STATE)
  - individual cyclic mover interface with given set of movement functionalities   
   	(ST_MOVER_CTRL / ST_MOVER_STATE)
    
<div style="page-break-after: always;"></div>

  ## PLC/XtsTransport/XtsTransport Project: - Who's who?

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
	
	for setting up your system; this project relys on those parameters to be correct

  #### XTS/GVL_XTS:
	AXIS_REF for mover
	GROUP_REF for Collision Avoidance Group
	
	global instances of function blocks and Ctrl/State structs

  #### XTS/XtsTransportUnit:
	fb_TransportUnit
	xts and mover are set to defined state
	interface to extern control for mode selection
	current state of example is that command CMD_TRANSPORT_START 
	is sending all mover to startup position 
	and adds all mover to queue of startup station when CaGroup is in standstill
	--> now handshake of stations can start

  #### XTS/CaGroup:
	fb_CaGroup
	handles Collision Avoidance state
	AddAll()
	RemoveAll()
	Reset()
	Enable()
	Disable()

	I_XtsTransport_Group - InterfacePointer used by fb_TransportUnit
	  

  #### XTS/Mover:

	fb_Mover cyclic interface 

	- for extern usage (ST_MOVER_CTRL / ST_MOVER_STATE)
	- methods are writing LastPosition and Last Gap 
	- for each mover on motion execute
	- Interface pointer for use within fb_Station and fb_TransportUnit.
	- see E_MOVER_CTRL for available functionalities
	 
	- I_XtsTransportMover - InterfacePointer, used in fb_CaGroup, fb_Station, fb_TransportUnit
	

  #### XTS/XtsStation:
  **rPosStop: is added to WaitPos,**
**beware when using negative offsets (avoiding collision, no movement, no error)**

	fb_Station.MoveData():
	// build WorkPos from parameter, 
	// static station data and information on mover
	
	_stInfeed.rPos := _stParameter[_nStationId].rPosWait
		       +  _stParameter[_nStationId].rPosStop[_nNest]
		       +  _stMoverData.rOffset
		       +  _rMoverOffset[_nStationId][_stMoverData.nMoverId][_nNest];


	fb_Station.MoverOut():
	// build data for sending mover to target station

	// who am I sending?
	    _stMoverDataSend.nMoverId       := _stStationState.nMoverId;

	// get optional information about nests to work
	    _stMoverDataSend.nMask          := _stStationCtrl.nMask;

	// get optional offset
	    _stMoverDataSend.rOffset        := _stStationCtrl.rOffset;

	// get target station
	    _stMoverDataSend.nTargetStation := _stStationCtrl.nTargetStation;

	// get position of target
	    _stOutfeed.rPos := _stParameter[_stMoverDataSend.nTargetStation].rPosWait;



	fb_Station.Cycle():
	
	- for extern usage (ST_STATION_CTRL / ST_STATION_STATE)
	- handshake for mover infeed, process (nests), outfeed
	- static offset datafield for every Mover in every station with every nest
	- additional process offset by LinkedList entry
	- Interface to LinkedList use for adding mover to target station queue 
	  after mover has left the station

  #### Planning requirements for use of fb_Station:
- Put the Modulo turn anywhere, 
  **BUT NOT** within WaitPos, StopPos, ReleaseDistance of a station. 
  The code does not support crossing the modulo turn within a station.

- Since the project is designed for stations to send movers to a flexible target, with flexible nest positions,
  the control struct of a station you have to use to forward/command those parameters together with the mover ID
	-	 ST_STATION_CTRL.nMask: commands the nest count and nest position of the mover in target station 
	-	 ST_STATION_CTRL.nTargetStation: index of station in GVL_XTS.StationParameter[]

- The Use of LinkedList methods (AddTail, GetHead) 
  requires thought about when the mover is entered into the target station.
  
	- 1. parallel stations for a process, with common rPosWait:
	
	    EXAMPLE: 

		**Process uses GVL_XTS.Station[1] to GVL_XTS.Station[4]**

        **ST_STATION_PARAMETER (GVL_XTS.StationParameter)**

				
				--> [1].rPosWait := 100
				    [2].rPosWait := 100
				    [3].rPosWait := 100
				    [4].rPosWait := 100

		Define how many rPosStop(nests) the stations may have (configured count)
		
				--> [1].nConfiguredStopCount := 1 (default)
				    [2].nConfiguredStopCount := 1
				    [3].nConfiguredStopCount := 1
				    [4].nConfiguredStopCount := 1

		Define the process position(s) relative to rPosWait
		
				--> [1].rPosStop[1] := 100
				    [2].rPosStop[1] := 200
				    [3].rPosStop[1] := 300
				    [4].rPosStop[1] := 400
			
		The ReleaseDistance of STN[4] shall be shortest, all other stations follow accordingly.
		
				--> [1].rReleaseDistance := 40
				    [2].rReleaseDistance := 30
				    [3].rReleaseDistance := 20
				    [4].rReleaseDistance := 10
  
  
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
	- One Station --> One Mover.
	- all coordinates are modulo values
	- from station to station only forward
	- within station limits backward movement by use of negative nest offset 
	  or use of ST_MOVER_CTRL.
	- IF move backwards you have to make sure that there is room for it 
	  - check PosStop[]
	  - each PosStop[] is relative to PosWait

	- station location is defined by:
		 - PosWait
		 - PosStop[1..8]
		 - ReleaseDistance and 
		 - ConfiguredStopCount
 
 
		  TYPE ST_STATION_PARAMETER :
			STRUCT
			  // Only description
			  sText             : STRING(80);
			  
			  // start of station
			  // a sending station is using this value to send mover to
			  rPosWait          : REAL;       

			  // distance from Mover.ActPos has to travel in order 
			  // for station to go back to checking list entries
			  rReleaseDistance  : REAL;       

			  // CA and dyn constraints for infeed and outfeed
			  rGap              : REAL;
			  rVelo             : REAL;
			  rAccDec           : REAL;
			  rJerk             : REAL;

			  // how many nests (stop positions) mover has to stop at 
			  // (1 = default)
			  nConfiguredStopCount  : USINT := 1; // 1-8 --> NestMask = BYTE

			  // mover stop position in station
			  // relative to rPosWait!!
			  rPosStop          : ARRAY[1..8] OF LREAL;
			  
			END_STRUCT
		  END_TYPE

	- The default nest count is 1, so a mover with only one stop in station does not need ST_STATION_CTRL.nMask
	
	- **IF your mover has more than one stop in its target station, you need to set ST_STATION_CTRL.nMask:**

		- ST_STATION_PARAMETER.nConfiguredStopCount: static configuration of possible stop positions within station
			- static parameter is the limit to how many nests are possible
		- ST_STATION_CTRL.nMask: active nest count and position of mover in target station, you have to set this value for every mover leaving the sending station.
			- active parameter is only working on configured nests

  #### XTS/XPU
      fb_Xpu:
        - one Track, one Part, (InfoServer not yet)
        - cyclic plausibility checks to ProcessingUnit and MotorModules
		- Mover 1 detection after Init
        - connects local function blocks to XTS_Utility lib
        - collects motor module info data
		- provides interfaces to Tc3_XTS_Utility:
			- GetEnvironment	:	Tc3_XTS_Utility.I_TcIoXtsEnvironment
			- GetParameterSet(i):	Tc3_XTS_Utility.I_TcIoXtsParameterSet
			- GetXpuMover(i)	:	Tc3_XTS_Utility.I_TcIoXtsXpuMover

<div style="page-break-after: always;"></div>


# Distribute
**If you distribute you do also have to ensure support in case of questions from your recipients**

