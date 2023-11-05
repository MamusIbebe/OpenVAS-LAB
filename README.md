# OpenVAS-LAB
The aim of the OpenVAS vulnerability lab project is to provide a comprehensive OpenVAS vunnerability Scan. These are two  main steps in the Vulnerability Management Lifecycle. We will be using OpenVAS(greenhouse) to scan Cloud VMs hosted on MS AZURE in order run credentialed scans to discover vulnerabilities, remediate some of the vulnerabilities, then perform a rescan to verify remediation. 

# Objective
1. Create our Free Azure Account
  a) create an openVAS machine and ssh -username for the machine @ipaddress. 
2. Prepare Vulnerability Management Scanner
3. Create Client Virtual Machine and Make it vulnerable
4. Perform Unauthenticated scan
5. Make Configurations for Authenticated Scans (Within VM)
6. Make Configurations for Authenticated Scans (OpenVAS)
7. Perform Authenticated Scan against our Vulnerable Windows
8. Remediate Vulnerabilies.
9. Verify Remediation.

# Basic Steps
   • Computer with the Internet
   • Azure Account – Pay as you go was much easier for me.
Create our Free Azure Account
   •https://azure.microsoft.com/en-us/free/ and log into  https://portal.azure.com/#home
![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/f68db302-fd01-47ef-a6e4-b80539fb18c1)


# Prepare and Configure Vulnerability Management Scanner
a. https://portal.azure.com → Go to the Marketplace → “OpenVAS secured and supported by HOSSTED”
b. “Start with a pre-set configuration” → Pick the weakest one
c. “Continue to Create VM”         
d. Resource Group: Vulnerability-Management
                  -VM Name: OpenVAS (Take note of the region and Vnet–consider East US 2)
                  
                  ...Authentication: Username → Labwork
                  
                  ...Monitoring: Disable Boot Diagnostic (don’t need it)
                  
                  ...Create → Create
                  
• After the VM has been created, SSH into it With PowerShell (windows)  using the credentials you created earlier by typing ssh Labwork@ipaddress

![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/d9604ce1-673a-45b1-acd7-4af6f8377542)


 ![Powershell configuration](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/fdc57f9f-68fb-45d6-beea-70c04364719a)

        ◦ It should show the web app URL and default username and password at this point, attempt to go to the URL in the browser and login with the username and password. If it doesn’t work, try admin/admin:

![greenbone openVAS scanner](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/5993b702-fdc8-422f-b49e-a0e0f16922cc)

# Create Client Virtual Machine and Make it Vulnerable
1. Go to  https://portal.azure.com/#home
2. Search for Virtual Machines and create a new Virtual Machine
- Resource Group: Vulnerability-Management...
- VM Name: Windows-vm
- Region: Same as the OpenVAS VM (East US 2)
- Virtual Network: Same as OpenVAS (this is important)
- Image: Windows 10 Pro
- Size: Any size with 2 vCPUs
- Username: Labwork/********
- Networking: Same Vnet as OpenVAS
- Create the VM

  ![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/b457a2e4-20d0-4f38-8186-aaaca47f2fab)

3. After the VM has been created, ensure you can RDP into it with the credentials you created.
4. After logging in, make the VM vulnerable: by disabling the firewall and download bunch of old and depracted software like old firefox/internet explorer,vlc music player9old version) http://www.oldversion.com/

 ![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/f67840e2-43ca-4e4f-9c48-24e109301061)

6. Restart the VM

# Configure and OpenVAS to Perform First Unauthenticated Scan against our Vulnerable VM
1. Login to OpenVAS → Assets → Hosts → New Host
2. Add the Client VM PRIVATE IP Address
3. Create a New Target from the Host, name it “Azure-Scan-vms”.
4. Take note of the credentials. We will add SMB credentials later.
5. Create a new Task
   - Name & Comment: “Scan - Azure-scan-VMs”
   - Scan Targets → “Azure-scan-VMs”
   - Save the Task
6. Start” the “Scan - Azure Vulnerable VMs” Task

![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/fa195f95-3e34-4feb-a3c8-2fefa52e36cf)


-Take note of Tabs, specifically the “Results” tab. Even though we installed a super old version of Firefox, note that it does not show up here.

-Note that this is because we aren’t running a credentialed scan so the scanner could not discover it. We will configure credential scans next

# Make Configurations for Credentialed Scans (Within VM)

1. Disable Windows Firewall just in case you forgot.
2. Disable User Account Control
3. Enable Remote Registry
4. Set Registry Key
   - Launch Registry Editor (regedit.exe) in “Run as administrator” mode and grant Admin Approval, if requested
   - Navigate to HKEY_LOCAL_MACHINE hive
   - Open SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System key
   - Create a new DWORD (32-bit) value with the following properties:  Name: LocalAccountTokenFilterPolicy  Value: 1
   - Close Registry Editor
   - Restart the VM

# Make Configurations for Credentialed Scans (OpenVAS)
1. Go to Configuration → Credentials → New Credential
   - Name / Comment → “Azure-scan Credentials”
   - Allow Insecure Use: Yes
   - Username: Labwork
   - Password: ***********
   - Save
2. Go to Configuration → Targets → CLONE the Target we made before
3. NEW Name / Comment: “Azure-scan VMs - Credentialed Scan”
4. Ensure the Private IP is still accurate
5. Credentials → SMB → Select the Credentials we just made: Azure-scan VM Credentials
6. Save

# Execute Credentialed Scan against our Vulnerable Windows VM
1. Within Greenbone / OpenVAS, go to Scans → Tasks
2. CLONE the “Scan - Azure Vulnerable VMs” Task, then Edit it:
   - Name / Comment → “Scan - Azure-scan-VMs - Credentialed”
   - Targets: Azure-scan-VMs - Credentialed Scan
   - Save
3. Click the Play button to launch the new Credentialed Scan, wait for it to finish
   - It will take longer than the last one. Wait for it to finish
4. After the credentialed scan finishes, you can immediately see the difference in findings


![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/84786c23-e8e8-4fc0-88dc-8fca27b1185f)




# Detailed result of the credentialed scanning

![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/db321dcf-600f-4cd5-8c8d-6cbb6b8d2692)

# Upon Further Investigation ( severity of adobe reader prone to memory corruption)



![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/6a2c2291-dfce-449c-b445-3c100506be5f)



# Remediate Vulnerabilities

1. Log back into your Win10-Vulnerable VM.
2. Uninstall Adobe Reader, VLC Player, and Firefox in my case.
3. Restart the VM.
# Verify Remediations
1. Re-initiate the “Scan - Azure Vulnerable VMs - Credentialed” scan and observe the results.

# Result of Remediation
Pay attention to second scan after uninstalling or updating old softwares.
![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/86619c03-eba2-4c2b-b499-d87140875d2b)

# More detailed Result and after Remediation.

![image](https://github.com/MamusIbebe/OpenVAS-LAB/assets/149246488/4c2e3e1e-c1e4-485e-bfab-e957bf862c33)

If you compare the result before the VM was remediated, you can tell the difference, even though there are still more remediation to be done. The critical severerity of the vulnerability went down.

# Conclusion
OpenVAS vulnerability lab project offers a valuable opportunity for individuals and organizations to enhance their cybersecurity capabilities. By delving into the world of OpenVAS, participants have gained insights into the essential processes of vulnerability assessment, scanning, and remediation.

Throughout this project, we've explored the OpenVAS tool, its architecture, and its practical applications. We've learned how to identify and prioritize vulnerabilities, interpret scan results, and take proactive steps to fortify our digital defenses. This knowledge is crucial in today's interconnected world, where cyber threats continue to evolve and pose significant risks to our digital assets.

