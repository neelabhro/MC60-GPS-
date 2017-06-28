#################################################################
compilation/burning of programing:
1. edit makefile :
2. run msdos.lnk
3. type: make clean
4. type: make new
5. open qflash and burn the program. toggle flash switch on board to start.
#################################################################

guide to gps.c:

###################################################################
flow of control:
1. program enters proc_main_task
2. registers and open uart for debug purpose
3. registers and start timer to capture gps data and send to udp  
   server .
4. Timer starts and upon overflow , Callback_Timer(u32 timerId, void* param) is called and the  switch case structure performs upon m_udp_state values.
After a designated time, the timer restarts which is known as overflow, after which the callback_timer function re-calls the original function which then again performs the said processes.

###################################################################

lines 54-70:
all the necessary header files

lines 73-90:
defines APP_DEBUG() function 

lines 96-113:
enum flags for enum_UDPSTATE 

lines 131-145:
APN and server parameters 
note: specify apn name in line 131 , udp server ip in line 142

lines 16-166:
structure containing pointers to gprs related callbacks

lines 150-158:
socket and gprs callbacks prototype

lines 166-173:
structure containing pointers to sockets related callbacks

line 178:
uart callback prototype

line 183:
timer callback prototype

line 189:
prototype for gps function (retrieves gps in RNC format)

line 194-223:
proc_main_task: 	
		1.registers/opens debug uart port
		2.registers and starts timer for gps retrieval and sennding
		3. while(true) loop executes till it is note done initialising RIL layer .

line 225-244:
uart callback handler definition:
		used to read data through uart port. 

		internally calls :
		static s32 ReadSerialPort(Enum_SerialPort port,u8* pBuffer,u32 bufLen);

line 246-271:
this fucntion actually reads uart data and copies to *pBuffer .

line 273-291:
function to detect errors with sockets connections .

line 293-530:
timer callback function. this is the main function which periodically collects gps data and sends data to udp socket.

timeout can be controlled in line 126.

flow of control(everything happens in switch/case structure):
	1. gets simstate . if sim is ready, sets "m_udp_state" flag to "STATE_NW_QUERY_STATE" .

	2. gets sim network state. if all good, sets "m_udp_state" flag to "STATE_GPRS_REGISTER" .

	3. registers gprs callback functions and then sets "m_udp_state" flag to "STATE_GPRS_CONFIG" .

	4. configures and sets apn name and server ip from apn and server parameters . then it, sets "m_udp_state" flag to "STATE_GPRS_ACTIVATE" .

	5. activates the gprs connection and then it, sets "m_udp_state" flag to "STATE_GPRS_GET_DNSADDRESS" .

	6. gets dns address . then it, sets "m_udp_state" flag to "STATE_GPRS_GET_LOCALIP" .

	similiarly, for lines 388, 404, 419, 455, 472, 485, 519,
	the code respectively does the following:
	7. converts ip address to correct format .
	8. registers socket callback  functions.
	9.  creates socket connection .
	10. sends data to udp . note: this function calls G_p_s() function.
	11. deactivates gprs(optional. not in use) .

line 680-714:
gps function to gather gps data

