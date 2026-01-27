# Terminal Logs Guide - ECG Application

## How to Operate Buttons to Get Proper Logs in Terminal

This guide explains the proper sequence of operations to see all command logs in your terminal.

---

## 📋 Available Buttons in ECG Test Page

1. **Start Button** - Starts ECG data acquisition
2. **Stop Button** - Stops ECG data acquisition  
3. **Ports Button** - Configure COM port and baud rate
4. **Generate Report Button** - Generate ECG PDF report
5. **Twelve Leads Button** - Switch to 12-lead view
6. **Six Leads Button** - Switch to 6-lead view
7. **Back Button** - Return to previous screen

---

## 🔄 Proper Operation Sequence for Complete Logs

### Step 1: Launch Application
```bash
python ECGdiv/src/main.py
```

**Expected Logs:**
- Application initialization
- Module imports
- Dashboard configuration
- User login prompts

---

### Step 2: Login
- Enter your credentials in the login dialog
- Click "Login" button

**Expected Logs:**
```
✅ User [username] logged in successfully
✅ Dashboard initialized
```

---

### Step 3: Navigate to ECG Test Page
- From Dashboard, navigate to ECG Test/Recording page
- The page will initialize

**Expected Logs:**
```
ECG Test Page imported successfully
ECG Test Page initialized
```

---

### Step 4: Configure Port (Optional)
- Click **"Ports"** button
- Select COM port (e.g., COM8)
- Set baud rate (usually 115200)
- Click OK

**Expected Logs:**
```
Port configuration updated: COM8, 115200
```

---

### Step 5: Click "Start" Button ⭐ MAIN ACTION

When you click the **Start** button, the following sequence happens automatically:

#### 5.1 Port Scanning
```
======================================================================
🔍 PORT SCANNING: Scanning all COM ports for ECG device...
======================================================================
📋 Found X COM port(s): COM1, COM8, ...
🔌 Testing port: COM1
   📤 Sending START command to COM1...
   ❌ Port COM1 did not respond correctly
🔌 Testing port: COM8
   📤 Sending START command to COM8...
   📥 Response: Code=0x20, Type=ack, OpCode=0x10
   ✅ Port COM8 responded with ACK!
✅ PORT DETECTED: COM8
```

#### 5.2 START Command
```
======================================================================
🚀 START COMMAND: Initiating communication with device
======================================================================
SOFTWARE → DEVICE: START Command Packet
Raw Hex: E800111000000000000000000000000000000000008E
📤 Software sending START command to device...
✅ Command packet written to serial port (22 bytes)
⏳ Waiting for device response (timeout: 2.0s)...
DEVICE → SOFTWARE: START ACK Response Packet
📥 Device responded with packet (22 bytes)
✅ START COMMAND: Success! Device acknowledged START command
```

#### 5.3 VERSION Command (Automatic) ⭐
```
============================================================
🔍 VERSION COMMAND: Requesting device version...
============================================================
📤 STOP → E8 00 11 11 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 8E
📥 STOP ACK ← [ACK packet hex]
✅ STOP confirmed
📤 VERSION → E8 01 11 14 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 8E
📥 VERSION ACK ← [ACK packet hex]
📥 VERSION DATA ← [DATA packet hex]
============================================================
✅ VERSION COMMAND SUCCESS
============================================================
✅ DEVICE VERSION: TTECGM00V_010101
============================================================
```

#### 5.4 Data Acquisition Starts
```
✅ Packet-based ECG device started on port COM8
📡 Ready to receive data packets...
Serial connection established successfully!
[Packet #100, Counter: X] Received valid packet with 12 leads
[Packet #200, Counter: X] Received valid packet with 12 leads
...
```

---

### Step 6: Monitor Data (Optional)
- Watch the terminal for packet reception logs
- ECG waveforms will appear on screen
- Heart rate and other metrics will update

**Expected Logs:**
```
[Packet #100, Counter: 21] Received valid packet with 12 leads
HEARTBEAT: 72 BPM
✅ [Windows] Hardware sampling rate detected: 233.6 Hz
```

---

### Step 7: Click "Stop" Button

When you click the **Stop** button:

#### 7.1 STOP Command
```
======================================================================
🛑 STOP COMMAND: Requesting device to stop
======================================================================
SOFTWARE → DEVICE: STOP Command Packet
Raw Hex: E801111100000000000000000000000000000000008E
📤 Software sending STOP command to device...
✅ Command packet written to serial port (22 bytes)
⏳ Waiting for device response (timeout: 2.0s)...
DEVICE → SOFTWARE: STOP ACK Response Packet
📥 Device responded with packet (22 bytes)
✅ STOP COMMAND: Success! Device acknowledged STOP command
```

#### 7.2 Statistics
```
Stopping packet-based ECG data acquisition...
Total data packets received: 2048
Packet loss summary: 33/2081 packets lost (1.59% loss)
No sequence gaps detected - perfect packet continuity!
```

---

## 📊 Complete Log Sequence Summary

### When "Start" Button is Clicked:
1. ✅ Port scanning (all COM ports)
2. ✅ START command sent
3. ✅ START ACK received
4. ✅ **VERSION command sent automatically** ⭐
5. ✅ STOP command sent (to put device in IDLE)
6. ✅ STOP ACK received
7. ✅ VERSION command sent
8. ✅ VERSION ACK received
9. ✅ VERSION DATA received
10. ✅ **Version string decoded and logged** ⭐
11. ✅ Data acquisition begins
12. ✅ Packet reception logs

### When "Stop" Button is Clicked:
1. ✅ STOP command sent
2. ✅ STOP ACK received
3. ✅ Statistics logged
4. ✅ Acquisition stopped

---

## 🎯 Key Points

1. **Version Command is Automatic**: The version command is automatically called right after the device starts. You don't need to click any special button.

2. **Watch Terminal During Start**: The most important logs appear when you click "Start" - watch the terminal at that moment.

3. **Version Logs Appear After START**: Look for the VERSION COMMAND section in logs right after START command succeeds.

4. **Filtering Works**: The code automatically filters out ECG streaming frames (0x20) to get clean ACK/DATA responses.

5. **Device Must Be Connected**: Make sure your ECG device is connected to the COM port before clicking Start.

---

## 🔍 What to Look For in Logs

### ✅ Success Indicators:
- `✅ PORT DETECTED: COM8`
- `✅ START COMMAND: Success!`
- `✅ STOP confirmed`
- `✅ VERSION ACK validated`
- `✅ DEVICE VERSION: [version string]`
- `✅ Packet-based ECG device started`

### ❌ Error Indicators:
- `❌ Port did not respond correctly`
- `❌ VERSION COMMAND: Error occurred`
- `Timeout waiting for frame`
- `Expected ACK 0x21, got 0x[XX]`

---

## 💡 Tips for Best Logs

1. **Clear Terminal First**: Clear your terminal before starting to see clean logs
2. **Watch During Start**: Keep terminal visible when clicking Start button
3. **Check Connection**: Ensure device is properly connected before starting
4. **Wait for Completion**: Let each command complete before clicking next button
5. **Read Error Messages**: If errors occur, read the full error message in terminal

---

## 📝 Example Complete Session Log

```
Starting ECG Monitor Application
User ptr logged in successfully
ECG Test Page initialized

[User clicks START button]

======================================================================
🔍 PORT SCANNING: Scanning all COM ports for ECG device...
======================================================================
📋 Found 2 COM port(s): COM1, COM8
🔌 Testing port: COM1
   ❌ Port COM1 did not respond correctly
🔌 Testing port: COM8
   ✅ Port COM8 responded with ACK!
✅ PORT DETECTED: COM8

======================================================================
🚀 START COMMAND: Initiating communication with device
======================================================================
📤 Software sending START command to device...
✅ START COMMAND: Success! Device acknowledged START command

============================================================
🔍 VERSION COMMAND: Requesting device version...
============================================================
📤 STOP → E8 00 11 11 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 8E
📥 STOP ACK ← E8 XX 11 21 00 11 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 8E
✅ STOP confirmed
📤 VERSION → E8 01 11 14 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 8E
📥 VERSION ACK ← E8 XX 11 21 00 14 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 8E
📥 VERSION DATA ← E8 XX 11 24 00 54 54 45 43 47 4D 30 30 56 5F 30 31 30 31 30 31 8E
============================================================
✅ VERSION COMMAND SUCCESS
============================================================
✅ DEVICE VERSION: TTECGM00V_010101
============================================================

✅ Packet-based ECG device started on port COM8
📡 Ready to receive data packets...
[Packet #100, Counter: 21] Received valid packet with 12 leads
HEARTBEAT: 72 BPM

[User clicks STOP button]

======================================================================
🛑 STOP COMMAND: Requesting device to stop
======================================================================
✅ STOP COMMAND: Success! Device acknowledged STOP command
Total data packets received: 2048
```

---

## 🚀 Quick Start Checklist

- [ ] Application launched
- [ ] User logged in
- [ ] ECG Test Page opened
- [ ] COM port configured (or auto-detected)
- [ ] **START button clicked** ⭐ (Version logs appear here)
- [ ] Monitor terminal for version string
- [ ] STOP button clicked when done
- [ ] Check terminal for complete logs

---

**Note**: The version command runs automatically when you click Start. No additional button clicks needed!


