<p align="center">
<img src="https://i.imgur.com/Ua7udoS.png" alt="Traffic Examination"/>
</p>

<h1>Network Security Groups (NSGs) and Inspecting Traffic Between Azure Virtual Machines</h1>
In this tutorial, we observe various network traffic to and from Azure Virtual Machines with Wireshark as well as experiment with Network Security Groups. <br />

<h2>Environments and Technologies Used</h2>

- Microsoft Azure (Virtual Machines/Compute)
- Remote Desktop
- Various Command-Line Tools
- Various Network Protocols (SSH, RDH, DNS, HTTP/S, ICMP)
- Wireshark (Protocol Analyzer)

<h2>Operating Systems Used </h2>

- Windows 10 (21H2)
- Ubuntu Server 20.04

<h2>High-Level Steps</h2>

- Create 2 virtual machines in azure
- Install Wireshark and observe multiple different traffics.

<h2>Actions and Observations</h2>

<p>
  <h2>1. Create a new resource group and 2 VMs in azure.</h2>

  <img width="944" alt="image" src="https://github.com/user-attachments/assets/ababd13e-dc72-45ef-8601-01f50e43a0e8" />

</p>
<p>
Go to resource groups and create one named Networking-Lab, once that is done go to virtual machines and create 2, one running windows 10 and one running ubuntu. Make sure for the size that they have atleast 2 vcpus and set the username and password to whatever you would like. Make sure both VMs are in the same resource group and region. For networking, press new and name it Lab-Vnet and review and create the VMs.
</p>
<br />

<p>
  <h2>2. Install Wireshark and start packet capture</h2>
  
![image](https://github.com/user-attachments/assets/a16bc745-a654-4665-a82a-728ab96eae80)

</p>
<p>
On your Windows 10 VM and go to www.wireshark.org. Download the windows 64x installer. Open file and jsut click next until it installs. Open up Wireshark and press on Ethernet and then the shark fin in the top left. This shows the network traffic happening on the backend.
</p>
<br />

<p>
  <h2>3. Filter for ICMP traffic and observe ping requests</h2>

  ![image](https://github.com/user-attachments/assets/0a0195b6-1677-4784-8f2f-c2392049da7b)

</p>
<p>
Type ICMP in the bar on top, it should turn green. This filters for ICMP traffic. Go back to azure and copy your linux VM provate IP address. No open Power shell on the Windows VM and type "ping (private IP address)" and you should see requests and replies from both the VMs. If you click on the first one and expand Ethernet II you can see the source mac address. If you type in "ipconfig /all" you should be able to see the same mac address labeled physical address. If you want you can ping everyday websites like www.google.com and see what happens.
</p>
<br />

<p>
  <h2>4.Configuring a Firewall [Network Security Group]</h2>

 <img width="947" alt="image" src="https://github.com/user-attachments/assets/a3aa4835-e055-4383-9a9d-918fe1fe1e63" />

</p>
<p>
In powershell on the Windows Vm type "ping (Ubuntu VMs private Ip address) -t" this will perpetually ping the VM. Now we are going to set up a firewall to block the pings on the Ubuntu VM so go back to azure and click the Ubuntu VM, go to networking and network settings. Under network security group click Ubuntu-vm-nsg and then settings and inbound security rules. Add a rule, Source any, Destination ane, Destination port ranges put * which means any. For protocol put ICMPv4, action deny and priority to 290. This will block any ICMP traffic which ping uses and the priority just makes sure its done first. Click add and then go back to the Windows VM, as soon as the rule is finsihed updating you should see Request timed out. In Wireshark now you should only see requests and no replies. Now re-enable ICMP traffic by going back to the inbound rules and deleting the one that we just made. You should see we are now getting replies again. To stop the ping press Ctrl-C.
</p>
<br />

<p>
  <h2>5. Observe SSH traffic</h2>

  ![image](https://github.com/user-attachments/assets/f0b183dc-c541-4195-81d9-99fadc1ba077)

</p>
<p>
In Wireshark on the Windows VM filter for SSH in the top bar. SSH is for secure shell traffic. In powershell type <username>@<private IP address> for the Ubuntu VM. For me this looks like labuser@10.0.0.5, type yes and then enter the password and you should be connected. When you type in the password nothing appears but you are typing, this is just for security reasons. If you type hostname you should see Ubuntu-Vm. As you type anything into secure shell you should see traffic happening in wireshark. If you click on a packet coming from your Windows machine and go under Transmission Control Protocol you should see the destination port being 22 because ssh uses TCp port 22 to communicate. If you type touch file.txt and then ls you can see that you created a file text on the Ubuntu machine. You can also type tcp.port == 22 into the top bar isntead of ssh and it will show you the same traffic since ssh uses TCP port 22. Now type exit to sever the connection. 
</p>
<br />

<h2>6. Observe DHCP traffic</h2>

  ![image](https://github.com/user-attachments/assets/ffcd5d6b-4f01-4567-8fae-811895b40133)

</p>
<p>
DHCP is responsible for automatically assigning devices with IP addresses. So type dhcp in the top bar in wireshark and then run power shell as adminstrator. What we want to do is assign a new IP address to the Windows Vm so open notepad and type ipconfig /release in the first line and ipconfig /renew in the second line and save it as  dhcp.bat file in Program Files. Type cd "c:\Program Files" in power shell and type ls and you should see the .bat file. Type .\dhcp.bat and press enter. This should automatically release the IP which will dissconect you from the VM and then give you a new address and automatically reconnect you. You should see in wirehark that the ip was realeased which is why the source became 0.0.0.0 and the dhcp server interacting with it by sending out a general signal to 255.255.255.255. while we gave the windows vm a new IP address it ended up getting the same one anyway.
</p>
<br />

<h2>7. Observe DNS traffic</h2>

  ![image](https://github.com/user-attachments/assets/e9a6e432-46ac-4773-83f5-01408520fc6e)


</p>
<p>
Now we are going to observe DNS traffic. DNS stands for Domain Name System which links www.google.com to its IP address. So change the filter to DNS now and restart the capture. Type nslookup disney.com into powershell and you should see disney.com's IP address. If you try looking up this address in google it won't load the website since it is not the website name even though it is the right IP address.
</p>
<br />

<h2>8. Observe RDP traffic</h2>

 ![image](https://github.com/user-attachments/assets/46596452-54b5-49ab-87ac-5c7a632c5758)


</p>
<p>
Now we are going to observe RDP traffic. RDP stands for Remote Desktop Protocol which is what we're using to connext to our VM so what traffic do you think we will see and how much? So we will type tcp.port == 3389 into the filter bar in Wire shark which will filter for RDP traffic. As you can see there is constant traffic because it is constantly streaming a picture of the desktop screen which requires constant traffic unlike SSH which sent traffic when you sent keystrokes not all the time. That is it for this one.
</p>
<br />
