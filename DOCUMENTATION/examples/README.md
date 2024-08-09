# Introduction 
# XTS transport layer examples

### [examples]
 
### [XTS_DEMO_11]: 
- APPLICATION(PRG), simple procedure of how to startup and work Xts stations

### [XTS_DEMO_12] & [XTS_DEMO_22]:
#### FB_Process:
- experimental, works for the examples
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


# Distribute
**If you distribute you do also have to ensure support in case of questions from your recipients**

