
# Introduction 
# XTS transport layer (a station based approach)

## XTS transport layer projects

### V 4.00.07

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




# Members
  - ## APPLICATION (see XTS_DEMO_APPLICATION project)
    - introduces a grouping layer for:
		- collecting information from range of XtsStations
		- writing commands to range of XtsStations
		- fb_StationCollector: base class for grouping layer
		- fb_ProcessCollector: inherits base class fb_StationCollector, requires cyclic call of cycle method
		- control mechanics are similar to XtsStations
			- ST_PROCESS_CTRL
			- ST_PROCESS_STATE

	- introduces fb_Instance as base class for individual process procedures
		- control mechanics as ususal
		- ST_INSTANCE_CTRL
		- ST_INSTANCE_STATE

	- process instances are controlled by top level class fb_Application

	- using interfaces to connect fb_Application with the process instances and ProcessCollectors

	- fb_Application
		- control mechanics as usual
		- change of command is detected
		- commands:
			  // fb_Application:
			  // - procedures for getting fb_Instance classes and XTS_TRANSPORT_LAYER to work
			  //
			  //  - Cmd/state behaviour as in fb_TransportUnit
			  //  - E_INSTANCE_CMD  / E_INSTANCE_STATE + E_PROGRESS
			  //      INIT          /   INIT          : initialize processes and XtsStations       
			  //      DISABLE       /   DISABLED      : RemoveAllAxis from CA Group, disable CA Group, disable all movers
			  //      ENABLE        /   ENABLED       : AddAllAxis to CA Group, Enable CA Group, Enable all movers
			  //      WORK          /   WORKING       : CMD_TRANSPORT_START, enable handshakes, start writing list entries to BUFFER_INFEED
			  //      PROCEED       /   PROCEEDING    : CMD_TRANSPORT_RESTART, enable handshakes, continue writing list entries to BUFFER_INFEED
			  //      FINISH        /   FINISH        : stop writing list entries to BUFFER_INFEED, handshake processes until outfeeds are empty
			  //      CLEAR         /   CLEARED       : clear all instances of fb_Instance, clear all XtsStations
			  //-----------------------------------------------------------------------------------------------



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
		- provides interfaces to Tc3_XTS_Utility:
			- GetEnvironment	:	Tc3_XTS_Utility.I_TcIoXtsEnvironment
			- GetParameterSet(i):	Tc3_XTS_Utility.I_TcIoXtsParameterSet
			- GetXpuMover(i)	:	Tc3_XTS_Utility.I_TcIoXtsXpuMover

<div style="page-break-after: always;"></div>


# Distribute
**If you distribute you do also have to ensure support in case of questions from your recipients**

