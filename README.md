# MailHog SMTP Testing Environment — Setup & Testing Log

## Network Communication Verification

1. **Check IP addresses on both VMs:**

   - Kali Linux:  
     ```bash
     ip a  # Lists network interfaces and their IPs
     ```
   - Windows 11:  
     ```powershell
     ipconfig /all  # Displays IP configuration details
     ```

2. **Set static IP on Windows VM:**

   - Configured Windows network adapter to static IP `10.0.2.14/24`.

3. **Test connectivity via ping:**

   - From Windows to Kali:  
     ```powershell
     ping 10.0.2.15  # Successful ping
     ```
   - From Kali to Windows:  
     ```bash
     ping 10.0.2.14  # Initially failed due to Windows Firewall blocking ICMP
     ```

4. **Fix Windows Firewall to allow ICMP (ping):**

   - Temporarily disabled firewall to confirm connectivity:  
     ```powershell
     netsh advfirewall set allprofiles state off
     ```
   - After confirming pings work, re-enabled firewall:  
     ```powershell
     netsh advfirewall set allprofiles state on
     ```
   - Allowed inbound ICMP Echo Request by enabling the firewall rule:  
     "File and Printer Sharing (Echo Request - ICMPv4-In)"  
     This allowed Kali to ping Windows successfully.

---

## msmtp Installation and Configuration

1. **Install msmtp on Kali:**

   ```bash
   sudo apt update
   sudo apt install msmtp msmtp-mta -y
   
2. **Create configuration file ~/.msmtprc with:**

    defaults
    auth           off
    tls            off
    logfile        ~/.msmtp.log

    account        mailhog
    host           127.0.0.1
    port           1025
    from          `ScaNattacker@kali.local`

    account default : mailhog

   
 3. **Secure the configuration file permissions:**

    chmod 600 ~/.msmtprc


 4. **MailHog Installation and Setup**
    Install Go programming environment (required to install MailHog):

    sudo apt install golang-go -y
    go install github.com/mailhog/MailHog@latest

    ~/go/bin/MailHog **#confirm binary location**

    **Run MailHog binary location:**
    [HTTP] Binding to address: 0.0.0.0:8025
    [SMTP] Binding to address: 0.0.0.0:1025

    **Add MailHog binary to PATH permanently:**
    echo 'export PATH=$PATH:~/go/bin' >> ~/.bashrc
    source ~/.bashrc


    **Firewall and Port Verification from Windows**
    From an elevated PowerShell prompt on Windows, verify connectivity to Kali's MailHog ports:
    Test-NetConnection -ComputerName 10.0.2.15 -Port 8025  # HTTP interface
    Test-NetConnection -ComputerName 10.0.2.15 -Port 1025  # SMTP interface


    Both tests should succeed.


    **Accessing MailHog Web Interface**
    Open browser on Windows VM and navigate to:
    http://10.0.2.15:8025

    The MailHog web UI should load, showing captured emails.


    **Sending Test Emails Using msmtp**
    Send a basic test email from Kali, from ScaNattacker@kali.local to test@example.com:

    echo -e "From: 'ScaNattacker@kali.local'\nTo: 'test@example.com'\nSubject: Test Email\n\nThis is a simple test email." | msmtp 'test@example.com'

    //
    **Email with Attachement**
    echo “this is the content of the attachment.” > attachment.txt

    **Create MIME email with attachment:**

    cat > mail.txt <<EOF
    From: 'ScaNattacker@kali.local'
    To: test@example.com
    Subject: Email with Attachment
    MIME-Version: 1.0
    Content-Type: multipart/mixed; boundary="BOUNDARY"

    --BOUNDARY
    Content-Type: text/plain; charset="UTF-8"

    Hello, this email contains an attachment.

    --BOUNDARY
    Content-Type: text/plain; name="attachment.txt"
    Content-Disposition: attachment; filename="attachment.txt"
    Content-Transfer-Encoding: base64

    $(base64 attachment.txt)

    --BOUNDARY--
    EOF



   **Sent email:**
   msmtp 'test@example.com' < mail.txt



   **Logging**
   The SMTP client (msmtp) logs all sending activity to ~/.msmtp.log. Use this log to troubleshoot any mail delivery issues.
   Aug 10 07:47:13 host=127.0.0.1 tls=off auth=off from='test@kali.local'>
   Aug 10 08:00:32 host=127.0.0.1 tls=off auth=off from='ScaNattacker@kali.local'>



   **Summary**
   This document outlines the setup and validation of an isolated SMTP testing environment using MailHog on Kali Linux with msmtp as the sending client. The environment is accessible from a Windows VM via a host-only network, enabling secure testing of email sending and payload delivery without involving external mail servers.

