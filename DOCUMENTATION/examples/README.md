# Introduction 
# XTS transport layer examples

## [examples]
 
### [XTS_DEMO_11]: 
- APPLICATION(PRG), simple procedure of how to startup and work Xts stations
  - wait for (MAIN.eInit = PROGRESS_DONE)
  - use manual write for bStart; --> move all movers to start position, then start handshaking stations
  - use manual write for bReset; --> movement is stopped, group cleared, station cleared

### [XTS_GEAR_IN_POS_CA]:
- APPLICATION(PRG), simple procedure for startup and reset of XtsTransport
  - wait for (MAIN.eInit = PROGRESS_DONE)
  - use bStart and bReset for control
  
- GEAR_IN_POS_CA(PRG)
  - gear in  sequence for movers (one at a time)
  - checks SyncDistance of MasterAxis
  - sends mover away after MasterAxis moved SyncDistance
  - utilizes GVL_XTS.MoverCtrl/GVL_XTS.MoverState for control and movements of movers


## [scope]
- TwinCAT scope project for recording the signals of Stations and Processes

