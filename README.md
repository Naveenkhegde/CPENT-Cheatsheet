CPENT 
===

🔙 [HOME](../../HOME.md)

# Certificate

![](./Certificate.png)

---

# Exam Reflections
- **Introduction**
At the end of 2023, I enrolled in the CPENT course at Ucom. Because I was preparing to take OSCP a year later, I decided to take CPENT first to practice and get a feel for the exam environment. After researching it, although the scope is broad, there is a consistent exam pattern. In addition, many penetration testers and network administrators around me hold this certification, so I prepared with LPT (Licensed Penetration Tester Master) as my goal. I thought to myself: if I can't pass this, I shouldn't even think about OSCP.

- **Classes / Training**
Those who have taken Ucom courses or are in the cybersecurity community probably know which instructor teaches it. The instructor was excellent; listen carefully and complete every exercise and selected iLab at least once. The instructor's notes are also very useful. Because iLab access lasts 180 days, you can take some time after class to practice hands-on again. Personally, I procrastinated for six months before taking the exam—many classmates who took the class with me had already passed by the time I took it. Remember that the exam must be taken within one year, so make sure it doesn't expire.

- **Exam**
The exam can be taken as two 12-hour sessions or one 24-hour session. Those who have passed usually recommend the two 12-hour option: on one hand, mental stress is much lower; on the other hand, in the gap between the two exam sessions, you can review, research, or seek guidance on questions you were unsure about. Therefore, I also chose the two 12-hour option. I booked an empty room in northern Taiwan specifically for the exam. Once the countdown on the exam dashboard finishes, click in and follow the steps to complete the proctoring setup. I used Kali Linux as my primary attack machine and connected to the exam lab environment via OpenVPN.

   **Common Questions:**   
   ```
   1. How do you choose the exam time slot?
      Basically, you need to schedule the exam at least three days in advance. Exam slots usually start around 12:00 - 13:00 (noon) and add 12 hours from there, so it basically runs from noon to midnight.
   2. Can you listen to music during the exam?
      The proctor I had did not allow it, but I heard that some proctors do. You can ask through the chat window.
   3. Can you eat while taking the exam?
      After asking the proctor, they said as long as you don't leave the camera view, you don't even need to ask. So I devoured burgers right in front of the camera.
   4. Can you take bathroom breaks?
      Yes, just inform the proctor, but don't stay away for too long, and remember to notify them when you return to your seat.
   5. Can you use dual monitors?
      Officially no. However, some people have reported turning off their laptop display and projecting to a single external monitor—exercise your own discretion. I obediently stuck to my small laptop screen.
   ```
- **Exam Tips**
  1. Take screenshots as you go, otherwise you will definitely forget.
  2. You don't always have to escalate privileges the standard way; sometimes kernel exploits can achieve the desired result.
  3. If possible, adjust the order of your wordlist/dictionary, otherwise brute-forcing will take a very long time.
  4. Some people might get leaked solutions online, but you MUST adapt to the actual machine state.
     A solution is only one methodology. I encountered a case where I had to find the required info on a completely different IP than what was in the guide.
     So don't blindly copy commands from cheat sheets; explore the network environment properly, otherwise your steps won't make sense.
  5. You can end the exam early. In my second 12-hour session, I submitted after only 4 hours, as long as you've verified that all necessary information and screenshots are saved.
  6. Remember to click "Save Answer" after finishing each question set.

- **Results & Reporting**
Within 7 days after the exam, you need to submit your penetration testing report. There is a template available for download on the dashboard. Keep in mind it must be within 7 days, otherwise you'll need to pay an extra $100 USD extension fee, which is not worth it. Here is a pitfall I encountered: on the 5th day when I pressed the submit button to upload my report, the screen just spun without any response, and didn't confirm that the report uploaded successfully. In this situation, do not hesitate to contact customer support immediately. I sent emails and bombarded their live ChatBox, because their response time isn't very fast, and no one wants to risk paying extra fees if it exceeds 7 days. Later, the support team told me to email the report directly to them. Remember to follow up on your report status; once it is properly uploaded to the dashboard, you will clearly see that your report is under review. Always track it carefully to protect your rights.

![](./CPENT_UpdateReportFail.gif)

---


# Exam
1. Materials
   - ASPEN course code activation
   - Digital courseware / e-book (1-year validity)
   - Received via email:
     ilabs Training (Access) Key
   - iLab lab environment (180 days)
     https://eccouncil.learnondemand.net/
2. Exam Registration
   - Course code activated
   - Exam Dashboard will be activated (1-year validity)
   - Must complete exam within 30 days of scheduling; missing it forfeits retake opportunities
   - If an emergency occurs and you miss it, contact training center for retake voucher options
3. Exam Scheduling & Rules
   - Passport required for ID verification
   - Can only be scheduled on weekdays (business days)
   - Exam format selection (cannot be changed once chosen):
     - 12 + 12 hours (interval between sessions can be anytime within 30 days)
     - 24 hours
   - PenTest Report (must be submitted within 7 days)
   - CPENT: 70%, LPT (Master): 90%
   - 500 points * 5 sections = 2500 total points
   - Launch Exam -> OpenVPN -> Exam Range
   - Exam timer window must remain open during the exam
   - Single monitor only; Mac virtual desktops are not allowed; the proctor meeting must be able to record everything
4. Exam Content (5 Sections)
   - Practical / Multiple Choice Questions
     - Approx. 55 questions in total, ~8-12 questions per section
     - Information to find includes: versions, keys, passwords, flags
   - Total score: 2500
   - Question Types / Sections:
     - AD (500)
     - Binary exploitation + IOT (500)
     - CTF (500)
     - OT (500)
     - Pivot & Double Pivot (500)
5. Certification Renewal
   - ECE: $120 USD / 3 years
   - Annual Fee: $250 USD / year
     Tip: You can maintain an Aspen subscription or basic membership to keep the annual fee at $80 USD / year

---



# NOTE
Categorized and organized notes. Click links directly and use the back button to navigate between pages without needing to search through directories.
1. Penetration Test
   - [Scan](1.Penetration%20Test/Penetration%20Test%20-%20Scan%20.md)
     - IP Scan
     - Port Scan
     - Service Scan
   - [Enumeration & Service](1.Penetration%20Test/Penetration%20Test%20-%20Enumeration%20&%20Service.md)
     - SNMP
     - NetBIOS & NetBT
     - CIFS / SMB
     - RDP
     - SSH
     - Windows Access
     - OWASPBWA
   - [Privilege Escalation](1.Penetration%20Test/Penetration%20Test%20-%20Privilege%20Escalation.md)
     - MS17-010
     - Sudo
     - PwnKit
     - DirtyCow
     - Egress Busting (Remote Control)
     - Persistent
2. [OT](./2.OT/OT.md)
   - OT
   - tcpdump
   - Analyze
   - Modbus
   - ModbusAnalog
3. [IOT](./3.IOT/IOT.md)
   - Enviroment
   - Find Something
   - Dlink_firmware.bin
   - D6000-V1.0.0.41_1.0.1.bin
   - encrypted.bin
   - encrypted.bin - XORTool
4. [Pivot & Double Pivot](./4.Pivot%20&%20Double%20Pivot/Pivot%20&%20Double%20Pivot.md)
   - Enviroment
   - 4-Ways
     - SSH port forwarding (SSH Tunneling)
     - Port forwarding
     - Meterpreter
     - Chisel
5. [PWN](./5.PWN/PWN.md)
   - Foreword
   - Execuable
   - Memory
   - Assembly
   - StackFrame
   - Function
   - PEDA
   - Challenge
6. AD Pentest
   - [AD](./6.AD%20Pentest/AD.md)
     - Tree 、 Forest
     - Manager
     - AD
     - Penetration
       - [Recon](./6.AD%20Pentest/Recon.md)
       - [Kerberos](./6.AD%20Pentest/Kerberos.md)
         - Export Kerberos Tickets
         - Pass The Ticket
         - Golden Ticket Attack (TGT)
         - Kerberoasting
         - Zerologon
         - Sliver Ticket Attack (TGST)
7. [WEB RCE](./7.Web%20to%20RCE/Web%20to%20RCE.md)
   - SHELLSHOCK
   - LFI to RCE
   - PHP_SESSION_UPLOAD_PROGRESS
   - WordPress - LFI

