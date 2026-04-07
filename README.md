<h1>Intune: Device Enrollment and Endpoint Management</h1>
<p>
Building off the initial setup of user identities in Microsoft 365, this project focuses on securing the physical hardware those users interact with using Microsoft Intune.

I wanted to simulate the automated provisioning and securing of a new corporate laptop. I did this by joining a virtual machine to Entra ID, configuring security compliance baselines, and deploying configuration profiles.

I also touched on application provisioning silently pushing third-party software (Firefox) onto a new corporate laptop.

</p>

<h2>Environments and Technologies Used</h2>

* **Infrastructure:** Microsoft Intune Admin Center, Microsoft Entra Admin Center, Microsoft Azure (Windows 10 VM)
* **Core Concepts:** Mobile Device Management (MDM), Entra ID Join, Device Compliance, Configuration Profiles, Zero-Touch Deployment, Microsoft Store Repository Integration, Remote Troubleshooting (`dsregcmd`), Intune Management Extension


<h2>The Setup</h2>
<p>
  
Before enrolling the device, I had to ensure the cloud environment was ready to accept and manage it. Out of the box, Entra ID handles identity, but it doesn't automatically pass device management over to Intune. The first time I ran through this lab I ran into this issue where my device would join successfully to Entra but would be missing from Intune.

To fix this, I navigated to the `Microsoft Entra Admin Center > Mobility (MDM and MAM)` and clicked on Microsoft Intune. I configured the MDM user scope to "`All`". This enabled MDM Auto-Enrollment, ensuring that as soon as a user signs into a new device with their corporate credentials, Intune immediately takes over management of that hardware.

<img width="920" height="424" alt="image" src="https://github.com/user-attachments/assets/965c57c8-20f2-448f-86f1-eaccd559ae14" />


</p>


<h2>My Goals</h2>

The scenario here was deploying a new device for an employee. 

My goals were as follows:

* Enroll the device: Connect the hardware to the corporate Entra ID and Intune environments.

* Enforce Configuration Profiles: Lock down the device by blocking user access to the Windows Settings app and enforcing a screen lock timeout.

* Define Compliance Policies: Create a baseline that requires a password and BitLocker device encryption.

* Deploy Security Features: Push a silent BitLocker encryption profile to automatically secure the drive and bring the device into compliance without user intervention.

* Zero-Touch App Deployment: Push a required third-party application (Mozilla Firefox) silently to the device so it is ready for the user immediately.

Below I will go through the configuration and deployment of each requireme

Below I will go through the configuration and deployment of each requirement, verifying it works and explaining the objective behind the configuration.

<h2>Deployment and Configuration Steps</h2>

<h3>1. Device Enrollment (Entra ID Join)</h3>

To simulate a user receiving a new laptop, I booted up the Windows Virtual Machine and navigated to `Settings > Accounts > Access work or school`.

<img width="1861" height="925" alt="Image" src="https://github.com/user-attachments/assets/8b53b5a8-414c-458b-ba83-6260825d8394" />

<img width="1911" height="1027" alt="Image" src="https://github.com/user-attachments/assets/6196f6c2-488c-4efa-998a-a456579f42aa" />

<img width="1912" height="1033" alt="Image" src="https://github.com/user-attachments/assets/635185d6-9976-4a41-a647-b0d99ef548ac" />

Instead of just adding an email, I selected Join this device to Azure Active Directory (Entra ID) and signed in using John’s corporate credentials. After a reboot, I was able to log directly into the Windows lock screen using his Microsoft 365 email and password.

<img width="1915" height="1033" alt="Image" src="https://github.com/user-attachments/assets/e6c150e3-f406-4bba-86f7-3a4069e7b182" />

<img width="1914" height="1027" alt="Image" src="https://github.com/user-attachments/assets/3289f435-3302-4473-91df-fcd0038ec106" />

To verify the handoff to Intune was successful, I went to the Intune Admin Center > Devices > All devices and confirmed the VM populated in the management console.

<img width="1876" height="924" alt="Image" src="https://github.com/user-attachments/assets/e65326a5-715b-42a2-9d69-b28e9d470973" />


<h3>2. Deploying Configuration Profiles (Device Restrictions)</h3>

Next, I needed to lock down the device to prevent the user from making unauthorized system changes.

In the Intune Admin Center, I navigated to Devices > Configuration and created a new Windows 10/11 Template policy for Device restrictions.

<img width="1870" height="928" alt="Image" src="https://github.com/user-attachments/assets/76bcb789-c44c-431a-9312-26979cf01bce" />

<img width="1864" height="922" alt="Image" src="https://github.com/user-attachments/assets/94d361c8-8ace-470c-b395-2f090213c43f" />

<img width="1878" height="928" alt="Image" src="https://github.com/user-attachments/assets/7b872579-486a-4ba5-abc6-b9d9495a8bb4" />

I configured two specific settings:

Password: Set the maximum minutes of inactivity until the screen locks to 15 minutes.

Control Panel and Settings: Set the Settings app to Block.

<img width="1858" height="922" alt="Image" src="https://github.com/user-attachments/assets/cb04d0fc-a45e-4f14-aa64-fa8cba1144b6" />

<img width="1861" height="853" alt="Image" src="https://github.com/user-attachments/assets/d3bc270e-df12-4e27-8dba-7690e0006c22" />

<img width="1872" height="874" alt="Image" src="https://github.com/user-attachments/assets/50533bd4-ca80-4bf5-884e-61f6efedfd63" />

I assigned this profile to all users. To test it, I went back to the VM, forced a manual Intune sync, and attempted to open the Settings app. It immediately closed and displayed an "organization blocked" prompt, confirming the policy applied successfully.

<img width="1873" height="916" alt="Image" src="https://github.com/user-attachments/assets/c5010b3e-c22a-49f1-9d8f-00285c298c24" />

<img width="1861" height="919" alt="Image" src="https://github.com/user-attachments/assets/e9c720af-ff04-4475-8049-834313305c3b" />


<h3>3. Establishing Compliance Policies</h3>

A device might be managed, but I needed Intune to report on whether it was secure.

Under Devices > Compliance, I created a Windows 10/11 compliance policy. Under the System Security tab, I required a password to unlock the device and required Encryption of data storage on device (BitLocker).

<img width="1879" height="924" alt="Image" src="https://github.com/user-attachments/assets/7a36935c-7051-48de-903f-5aefd745bcb7" />

<img width="1876" height="928" alt="Image" src="https://github.com/user-attachments/assets/76227703-a427-48f1-a59c-709ab3601747" />

<img width="1861" height="922" alt="Image" src="https://github.com/user-attachments/assets/98ee7eb2-bf6f-4d2c-8141-393bcda509e6" />

<img width="1864" height="924" alt="Image" src="https://github.com/user-attachments/assets/7b0216d7-eb27-415f-81d7-c46b2999e072" />

<img width="1878" height="928" alt="Image" src="https://github.com/user-attachments/assets/4f2c0b17-8818-4101-b42c-f17cac93b2d5" />

After forcing another sync on the VM, I checked the Intune portal. The device was flagged as Not Compliant because the virtual hard drive was not encrypted. This proved the reporting mechanism was working exactly as intended.

<img width="1878" height="922" alt="Image" src="https://github.com/user-attachments/assets/49b4d7f7-82c1-4b8d-bb1a-09a40ce82d73" />

<img width="1866" height="928" alt="Image" src="https://github.com/user-attachments/assets/440b44be-3a40-44b2-ab65-052b408e6414" />

<img width="1902" height="1011" alt="Image" src="https://github.com/user-attachments/assets/cfd3f5dc-fbfd-4c9e-ba45-0b1c28952091" />



<h3>4. Silent Application Sourcing and Assignment</h3>

Next, I moved on to software provisioning. I navigated to the Microsoft Intune Admin Center > Apps > All apps and selected Add.

<img width="1869" height="921" alt="Image" src="https://github.com/user-attachments/assets/d5884ff1-fc48-452e-ba0e-47c53bf9945a" />

<img width="1866" height="919" alt="Image" src="https://github.com/user-attachments/assets/ec7fcfe8-2957-440e-8072-4d6aaac0a9d3" />

<img width="1870" height="925" alt="Image" src="https://github.com/user-attachments/assets/6da7aaf8-641f-4e9c-949b-7223f4ef4fd2" />

<img width="1861" height="919" alt="Image" src="https://github.com/user-attachments/assets/ed5fd5bd-3c66-4644-9f53-79f93caf87e5" />

Instead of manually uploading an MSI installation file, I utilized the Microsoft Store app (new) repository to pull the official Mozilla Firefox package. On the Assignments tab, I bypassed the "Available" group (which puts the app in the Company Portal for voluntary download) and added all users to the Required assignment group. This forces a silent background installation.

<img width="1870" height="919" alt="image" src="https://github.com/user-attachments/assets/eb96dd1d-777c-4214-96ed-193fee2f58ab" />



<h3>6. Verifying the Zero-Touch Deployment</h3>

Because the deployment was configured as "Required," the user does not see an installation wizard or a loading bar.

To expedite the installation for the lab, I forced another manual sync on the Windows VM via Settings > Accounts > Access work or school. After a few minutes, Firefox successfully populated in the Windows Start Menu. I verified this success administratively by checking the Firefox app dashboard in the Intune portal, which reported a successful Device install status.

<img width="1863" height="928" alt="Image" src="https://github.com/user-attachments/assets/036e8ed5-84ca-4184-be06-c8abcae24159" />
<img width="1915" height="1033" alt="Image" src="https://github.com/user-attachments/assets/e43d636a-217b-425f-b5df-5cbc0577a58f" />

<h2>Takeaways</h2>

<p>
Cloud identities and physical hardware go hand in hand and we will continue to rely on this connection even more. Intune is a vital tool for companies to manage endpoints remotely, enforce corporate security standards, and troubleshoot cloud-join connectivity issues.

I learned the critical difference between Compliance Policies (which simply audit and report on a device's security state) and Configuration Profiles (which actually enforce changes and restrict user actions). Using zero-touch deployment to automatically push required applications onto devices was cool to see happen in real-time and represents a huge time savings.

I also learned how to use the `dsregcmd /status` command in the Windows Command Prompt to verify a device's Entra ID join state and MDM URL routing. Understanding how to force manual syncs on the client side, rather than waiting for Intune's default 8-hour check-in window, proved essential for testing and validating deployments efficiently.

I have a blog post to go along with my miniseries of M365 labs which I have linked [here](https://techwithsimeon.io/microsoft-365-administration).
</p>
