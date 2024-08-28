# Introduction 
# XTS transport layer examples

## [examples]
 
### [XTS_DEMO_11]: 
- APPLICATION(PRG), simple procedure of how to startup and work Xts stations
  - wait for (MAIN.eInit = PROGRESS_DONE)
  - use manual write for bStart; --> move all movers to start position, then start handshaking stations
  - use manual write for bReset; --> movement is stopped, group cleared, station cleared

### [XTS_DEMO_12] & [XTS_DEMO_22]:
### [PROCESS]
- _Visualizations:
  - PROCESS_XTS_TRANSPORT:
	- startup:
	  - Wait for (MAIN.eInit = PROGRESS_DONE)
	  - CMD_GROUP_CLEAR: wait for XTS Transport Result: PROGRESS_DONE
	  - CMD_GROUP_BUILD: wait for XTS Transport Result: PROGRESS_DONE
	  - CMD_GROUP_ENABLE: wait for XTS Transport Result: PROGRESS_DONE
	  - CMD_MOVER_ENABLE:
	  - CMD_TRANSPORT_START:

- PROCESS(PRG):
  - everything process related is controlled here
  - .tProcess: TON timer for simulating process duration
  - .AUTO(): Action for Process handshake after CMD_TRANSPORT_START+PROGRESS_DONE
	   
#### FB_Process:
- **experimental**, **unsupported**, works for the examples...
- grouping of XTS stations for processes
- process configuration for:
	- First, Last Station
	- Xts Station, targeting to next process
- process types:
	 - NULL,             // nothing happens here
	 - SINGLE,           // single XTS Station
	 - PARALLEL,         // synced station handshakes for multiple XTS Stations, same process on all stations, all who enter are processed
	 - PARALLEL_STRICT,  // synced station handshakes for multiple XTS Stations, same process on all stations, all who are enabled must be processed
	 - SERIAL,           // synced station handshakes for multiple XTS Stations, all mover have to go through all stations in process
	 - DISTRIBUTOR       // single XTS Station, no work done here, automated distribution for LastTarget to FirstTarget

#### FB_Process and GVL_XTS.StationParameter
- try different values for .rReleaseDistance
  - E_PROCESS_TYPE.PARALLEL,
    E_PROCESS_TYPE.PARALLEL_STRICT: StationFirst must have biggest values of XtsStations within FB_Process[i], subsequent XtsStations up to LastStation have to decrease .rReleaseDistance (by 25mm or 1/4*.rGap)
  - E_PROCESS_TYPE.SINGLE,
    E_PROCESS_TYPE.SERIAL,
	E_PROCESS_TYPE.DISTRIBUTOR: .rReleaseDistance may vary from -10mm to max the distance from Mover.ActPos (at the time of Mover_OUT) to target minus 10mm

## [scope]
- TwinCAT scope project for recording the signals of Stations and Processes


# Distribute
**If you distribute you do also have to ensure support in case of questions from your recipients**

