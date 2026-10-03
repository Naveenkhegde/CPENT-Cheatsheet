OT
===
🔙 [MENU README](../README.md#note)


# OT
```
SCADA <--Modbus--> Controller  <====> Field / Plant
(Measure, Sense, Adjust, Control)  (RTU, DCS, PLC)  (Machines, Equipment, Facilities)

Measure (Read) / Adjust (Write) : Analog signals
Sense / Control                 : Discrete / Digital (Non-analog) signals
```

# tcpdump
```bash
# In Modbus communication, TCP port 502 is typically used
tcpdump tcp port 502 -vv -w modbus.cap
tcpdump -i ens3 port 502 -A -w OT_Dump.pcap
```



# Analyze
Setting
```
1. Transmission Control Protocol
2. "Right Click"
3. Protocal Preference
4. "Cancel"  Aynalze TCP sequence numbers
```
NIC Vendor and MAC Address
```
1. View
2. Name Resolution
3. "Enable" Resolve Network Address
```
Statstic
```
1. Endpoints : Network adapters, IP count
2. Conversation
3. Protocol Hierachy
```
Transaction Idntifier Filiter
```bash
mbtcp.trans_id=247
```

# Modbus
Modbus/TCP
```
1. Transaction Identifier : 2 bytes, identifies transaction pairing
2. Protocol Identifier    : 2 bytes, indicates Modbus protocol version (0 = Modbus/TCP)
3. Length                 : 2 bytes, indicates length of following bytes
4. Unit Identifier        : 1 byte, identifies Modbus slave device or unit
```
Modbus
```
1. Function Code : 1 byte, indicates the type of Modbus operation.
                   Used to distinguish reading data, writing data, diagnostics, etc.
                   Normally, the function code in the request matches the response; if mismatched (e.g., +0x80), it indicates an error/exception code.
2. Data Field    : (Click Function Code) Variable length depending on the Modbus operation;
                   contains payload data, such as register addresses, counts, or read/write values.
```


# ModbusAnalog
> Try to Analyze modbusAnalog
> And Try to answaer question in ModbusAnalogAns.txt
