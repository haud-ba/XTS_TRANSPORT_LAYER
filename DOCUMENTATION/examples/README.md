# Introduction 
# XTS transport layer examples

## [examples]
 
### [XTS_DEMO_11]: 
- APPLICATION(PRG), simple procedure of how to startup and work Xts stations
  - wait for (MAIN.eInit = PROGRESS_DONE)
  - use manual write for bStart; --> move all movers to start position, then start handshaking stations
  - use manual write for bReset; --> movement is stopped, group cleared, station cleared

### [XTS_DEMO_GEAR_IN_POS_CA]:
- APPLICATION(PRG), simple procedure for startup and reset of XtsTransport
  - wait for (MAIN.eInit = PROGRESS_DONE)
  - use bStart and bReset for control
  
- GEAR_IN_POS_CA(PRG)
  - gear in  sequence for movers (one at a time)
  - checks SyncDistance of MasterAxis
  - sends mover away after MasterAxis moved SyncDistance
  - utilizes GVL_XTS.MoverCtrl/GVL_XTS.MoverState for control and movements of movers

### [XTS_DEMO_SIMULATION]:
### [XTS_DEMO_SIMULATION_NEW_MC]:
## start simulating your transport system before anything is built ##
## new MC Controller available ##
- MAIN.StationParameterInit:
  - configuration of XtsStations
 
- GVL_APPLICATION:
  - fb_Simulation: grouping of XtsStation range for synced handling (e.g. process multiplication), automated handshakes with TON timer for process time
  
- APPLICATION(PRG), startup and reset of XtsTransport
  - Init(): fb_Simulation configuration, initial times are set here
  - wait for (MAIN.eInit = PROGRESS_DONE)
  - use XTS_TRANSPORT visualization for manual startup (TransportInit, ClearGroup, BuildGroup, EnableGroup, EnableMovers, TransportStart)
  - bNext and bEnter for halting fb_Simulation

## [scope]
- TwinCAT scope project for recording the signals of Stations and Processes

