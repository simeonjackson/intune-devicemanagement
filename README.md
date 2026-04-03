<h1>Intune: Device Enrollment and Endpoint Management</h1>
<p>
Building off the initial setup of user identities in Microsoft 365, this project focuses on securing the physical hardware those users interact with using Microsoft Intune.

I wanted to simulate the automated provisioning and securing of a new corporate laptop. I did this by joining a virtual machine to Entra ID, configuring security compliance baselines, and deploying configuration profiles.

I also touched on application provisioning silently pushing third-party software (Firefox) onto a new corporate laptop.

</p>

<h2>Environments and Technologies Used</h2>

* **Infrastructure:** Microsoft Intune Admin Center, Microsoft Entra Admin Center, Microsoft Azure (Windows 10 VM)
* **Core Concepts:** Mobile Device Management (MDM), Entra ID Join, Device Compliance, Configuration Profiles, Silent BitLocker Encryption, Zero-Touch Deployment, Microsoft Store Repository Integration, Remote Troubleshooting (`dsregcmd`), Intune Management Extension


<h2>The Setup</h2>
<p>
  
Before enrolling the device, I had to ensure the cloud environment was ready to accept and manage it. Out of the box, Entra ID handles identity, but it doesn't automatically pass device management over to Intune.

To fix this, I navigated to the `Microsoft Entra Admin Center > Settings > Mobility (MDM and MAM)` and configured the MDM user scope to "`All`". This enabled MDM Auto-Enrollment, ensuring that as soon as a user signs into a new device with their corporate credentials, Intune immediately takes over management of that hardware.

Additionally, to manage the virtual machine remotely, I had to bypass a common Remote Desktop Protocol (RDP) hurdle with cloud-joined devices by disabling Network Level Authentication (NLA) on the VM and modifying my .rdp file to support Entra ID credential pass-through.
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

To simulate a user receiving a new laptop, I booted up the Windows Virtual Machine and navigated to Settings > Accounts > Access work or school.

Instead of just adding an email, I selected Join this device to Azure Active Directory (Entra ID) and signed in using John’s corporate credentials. After a reboot, I was able to log directly into the Windows lock screen using his Microsoft 365 email and password.

To verify the handoff to Intune was successful, I went to the Intune Admin Center > Devices > All devices and confirmed the VM populated in the management console.

[Insert Screenshot: The Windows VM listed as "Managed by Intune" in the All Devices dashboard]


<h3>2. Deploying Configuration Profiles (Device Restrictions)</h3>

Next, I needed to lock down the device to prevent the user from making unauthorized system changes.

In the Intune Admin Center, I navigated to Devices > Configuration and created a new Windows 10/11 Template policy for Device restrictions.
I configured two specific settings:

Password: Set the maximum minutes of inactivity until the screen locks to 15 minutes.

Control Panel and Settings: Set the Settings app to Block.

I assigned this profile to all users. To test it, I went back to the VM, forced a manual Intune sync, and attempted to open the Settings app. It immediately closed and displayed an "organization blocked" prompt, confirming the policy applied successfully.

[Insert Screenshot: The error message on the Windows VM when attempting to open the blocked Settings app]


<h3>3. Establishing Compliance Policies</h3>

A device might be managed, but I needed Intune to report on whether it was secure.

Under Devices > Compliance, I created a Windows 10/11 compliance policy. Under the System Security tab, I required a password to unlock the device and required Encryption of data storage on device (BitLocker).

After forcing another sync on the VM, I checked the Intune portal. The device was flagged as Not Compliant because the virtual hard drive was not encrypted. This proved the reporting mechanism was working exactly as intended.

[Insert Screenshot: Intune dashboard showing the device marked as "Not Compliant" with the BitLocker requirement failing]


<h3>4. Remediating Compliance via Silent BitLocker Encryption</h3>

To fix the non-compliant status, I needed to encrypt the drive remotely without relying on the end-user to do it manually.

Under Endpoint security > Disk encryption, I created a new BitLocker profile. I configured the settings to silently encrypt the OS drive, specifically setting it to hide third-party prompts and block the standard BitLocker setup wizard from appearing on the user's screen.

Once assigned and synced, Intune silently triggered the virtual TPM on the VM and began encrypting the drive in the background. After the encryption finished, the device checked back in with Intune and its status successfully flipped from "Not Compliant" to Compliant.

[Insert Screenshot: The Windows VM showing "BitLocker is On" alongside the updated "Compliant" status in the Intune portal]

<h3>5. Silent Application Sourcing and Assignment</h3>

With the device secured, I moved on to software provisioning. I navigated to the Microsoft Intune Admin Center > Apps > All apps and selected Add.

Instead of manually uploading an MSI installation file, I utilized the Microsoft Store app (new) repository to pull the official Mozilla Firefox package. On the Assignments tab, I bypassed the "Available" group (which puts the app in the Company Portal for voluntary download) and added all users to the Required assignment group. This forces a silent background installation.

[Insert Screenshot: The Intune App assignment page showing Google Chrome assigned as "Required" for All Users]


<h3>6. Verifying the Zero-Touch Deployment</h3>

Because the deployment was configured as "Required," the user does not see an installation wizard or a loading bar.

To expedite the installation for the lab, I forced another manual sync on the Windows VM via Settings > Accounts > Access work or school. After a few minutes, Firefox successfully populated in the Windows Start Menu. I verified this success administratively by checking the Firefox app dashboard in the Intune portal, which reported a successful Device install status.

[Insert Screenshot: A split screen showing Google Chrome in the VM's Start Menu, alongside the Intune portal showing a green checkmark for Device Install Status]

<h2>Takeaways</h2>

<p>
Cloud identities and physical hardware go hand in hand and we will continue to rely on this connection even more. Intune is a vital tool for companies to manage endpoints remotely, enforce corporate security standards, and troubleshoot cloud-join connectivity issues.

I learned the critical difference between Compliance Policies (which simply audit and report on a device's security state) and Configuration Profiles (which actually enforce changes and restrict user actions). Using zero-touch deployment to automatically push required applications onto devices was cool to see happen in real-time and represents a huge time savings.

I also learned how to use the `dsregcmd /status` command in the Windows Command Prompt to verify a device's Entra ID join state and MDM URL routing. Understanding how to force manual syncs on the client side, rather than waiting for Intune's default 8-hour check-in window, proved essential for testing and validating deployments efficiently.

I have a blog post to go along with my miniseries of M365 labs which I have linked [here](https://github.com/simeonjackson/ad-troubleshooting).
</p>
