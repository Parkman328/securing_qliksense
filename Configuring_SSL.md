![logo](./media/image1.png)

# **Configuring SSL with SSLforFree / Let's Encrypt**

## **Partner Engineering**

<br>  
<br>
<br>
<br>

John Park  
Principal Solution Architect  
john.park@qlik.com

**Version: 1.2**  
**Initial Release Date: 7-May-20**
**Revisions** | **Notes** | **Date** | **Version**
------------------ | ----------- | --------- | -----------
Initial Draft | 20-May-2020 | John Park | 0.1 |

# Table of Contents

---

[**Summary**](#summary)

[**Part 1 - Checklist - PreConfiguration**](#part-1)

[**Part 2 - Create Certificate**](#part-2)

[**Part 3 - Import Certificate**](#part-3)

[**Part 4 - Remove Conflicts**](#part-4)

[**Part 5 - Configure Proxy**](#part-5)

[**Part 6 - Veification**](#part-6)

[**Part 7 - Additional Information**](#part-7)

## **Summary**

This document was created to help create SSL certificates for Qlik and allow HTTPS to work with Qlik Sense Server

HTTP connections are unsecure and from security sense is worst option for a external webserver deployment. A CA (Certificate Authority) called Let's Encrypt will issue free 90 day certificates and this guide will step you through setting up your Qlik Sense Server with Certificate from Let's Encrypt.

Also without proper Certificate and Qlik Mobile / Many of the SAML services will not work properly.

These are reference sites you can look at as suppliment to this site.

- <https://letsencrypt.org/>
- <https://certbot.eff.org/>
- <https://www.sslforfree.com/>
- <https://help.qlik.com/en-US/sense>
- <https://www.ssl.com/how-to/create-a-pfx-p12-certificate-file-using-openssl/>
- <https://ptarmiganlabs.com/blog/2017/10/01/free-ssl-certificates-for-qlik-sense/>

Environment

- Test Domain - test.johnpark.io
- Version - Qlik Sense Feburary 2020
- OS - Windowd 2016 Server Data Center

## **Part 1**

### Checklist - PreConfiguration

1. Make sure Qlik Sense is working and all functionality is checked on the server.
2. You should have full control over a domain or subdomain.
3. Your server should have port 80 and 443 exposed to outside.
4. Admin privilages to install IIS on Windows.
5. Admin privilages to configure and work with Certificates.
6. Remote Desktop Privilages.
7. Permission to Change Settings on Server.

**_Before you start backup your server (Certificates, Settings, Configurations)._**  
**Point Domain from (Google, GoDaddy, etc to server IP)**  
**_At this point nothing will work !!!!_**

Turn Windows Firewall / Other Firewall Off or allow it to accept 80/443 coonections

Directions to turn firewall off <https://www.faqforge.com/windows-server-2016/turn-off-firewall-windows-server-2016/>

Go to QMC and enable proxy to handle HTTP and HTTPS port (80 and 443) under proxy setting.

![Enable HTTP](./media/http-enabled.jpeg)

Go to QMC and whitelist correct Host/IP/Machine Names.  
In our example we are using AWS Server.

![While](./media/whitelist.jpeg)

#### At This point you should be able to access your Qlik Sense server from Public domain using http protocol.(If you cannot access server from public domain stop here and ask for help)

## Now if you try to get into HTTPS protocol you should see following

Warning sign saying your site Certificate is different than others  
![Https-error](./media/https-error.jpeg)

This is normal behavior since Qlik is not an CA and will issue selfsigned certificates
![Https-error](./media/https-error2.jpeg)

- Please note we are getting this error because domain name used to access this https site is not same as domain name certificate was issued for.
- Qlik Sense is setup to use default certificate which is different than our domain name.  
  If you want to learn more aobut wonderful world of certificates and security please google. HTTPS is more secure and encrypted and will not work with proper certificate issued by CA(Certificate Authority). Next step we will manipulate the server and get proper certificate installed.

## **Part 2**

### Create Cerfiticate

For Let's Encrypt CA(Certificate Authority) to issue you certificate you have to prove to them you have complete control over certificate. The flow of how Certificates and CA work together is documented well here - <https://www.ssl.com/faqs/what-is-a-certificate-authority/>

There are options to get certificate.

1. Pay companies like Symantec(Verisign), GoDaddy, DigiCert to issue you out a paid certificate.
2. Use Let'Encrypt - Free Certificate via validation.

For this document we will Option2 since Option1 and 2 are similar in most stepsa and Option 2 will allow all developers to test SSL for 90 Days free (Please note Let's Encrypt forced you to Revalidate every 90 days)

There are many implementation of Let's Encrypt
For this documentation we will use [SSlforFree](https://www.sslforfree.com/).  
We will discuss implementing Let'Encrypt or CertBot in later documents.

On your server stop Qlik Services(You need to only stop Proxy but I suggest stopping everything)

## Install IIS

- Open Server Manager and click Manage > Add Roles and Features. Click Next.
- Select Role-based or feature-based installation and click Next.
- Select the appropriate server. The local server is selected by default. Click Next.
- Enable Web Server (IIS) and click Next.
- No additional features are necessary to install the Web Adaptor, so click Next.
- On the Web Server Role (IIS) dialog box, click Next.
- On the Select role services dialog box, verify that the web server components listed below are enabled. Click Next.
- Verify that your settings are correct and click Install.
- When the installation completes, click Close to exit the wizard.

## Bind Your Domain port 80 via IIS

- Open IIS Manager via Control Panel->Systems and Security->Administrative Tools

![IIS-menu](./media/IIS-menu.jpeg)

- Inspect the IIS Website

![IIS-Overview](./media/IIS-Overview.jpeg)

- Access Bindings and change your Domain

![http-binding](./media/Http-Binding.jpeg)

- Now Access http version of your site adn IIS splash menu will show

You can validate by clicking on "Browse" on IIS menu as well accessing domain from local host outside your server.
Once done you should see default IIS website
![default-IIS](./media/default-IIS.jpeg)

## Register Domain on SSLforFree.com/ZeroSSL

- Navigate your browser to https://www.sslforfree.com/ (Easier done from Server)
- "Create Free SSL Certificate" Box type your domain

![SSLforFree](./media/SSLforFree.jpeg)

- Follow directions on SSLforFree and use HTTP File Upload.

![ConfigureSSLForFree](./media/ConfiguringSSLforFree.jpeg)

_You will need to register as the owner for Certificate_

- Using IIS manager click "Browse" to go to root folder and create the path and upload the file
  (Note: on Windows to create a folder with "." you must type "." at end of the name if ".well-known" you need to type ".well-known.")

![IISAuthentication](./media/IISAuthentication.jpeg)

_Download the Validation file generated by SSLforFree and place in directory
Create the path needed by SSLforFree and place file._

- Now you validate the path with automated system and it validate and prepare Certificate.
  ![SSL Verification](./media/SSLVerification.jpeg)

- Save the Certificate on the server.

![D](./media/DownloadCertificates.jpeg)

Now you have your Certificate !!!
_It Should be a zip file with CA Bundle, Certificate, and Private Key_

Stop IIS Website by removing Biding and Deleting Site. In Win2016 Server Service names are _Windows Process Activation Service and the World Wide Web Publishing Service_. Make sure in Services those processes will auto start.
**_(Personal Preference is removing IIS)_**

You can start Qlik Sense Server Processes again.

## **Part 3**

### Import Certificates

Now we need to create an Create Qlik Replicate Connection for Teradata with appropriate settings.

**Configure Teradata as a source in Connection Manager for Attunity.**

Click "Manage Endpoint Connections" and following window should popup.

**_Figure A.3.0._**  
Confirm Target Connection  
![Confirm Target Connections](./media/image12.jpeg)

Select "Target" as role and "Teradata" as Type.

Enter Credentials for _Teradata Server_, _Username_, _Password_

**Do not Click "Browse" on _"Default Database"_ now (We will Do that after Internal Parameters are set).**

Click on **“Advanced”** on Top Menu Bar and it should bring Internal Parameter Window

Add Internal Driver for Teradata as Override. If you type keyword "driver" in the prompt is should allow you to click on the variable and add a value.

**_Figure A.3.1._**  
Add Internal Parameters  
![Add Internal Parameters](./media/image13.jpeg)

**_Following Setting were used for testing._**

- Parameter: provider
- Value: Teradata Database ODBC Driver 17.00

View setting summary to make sure **"provider"** variable is used to override drivers setting for Target End Point.

**_Figure A.3.2._**  
View Setting Summary  
![View Setting Summary](./media/image14.jpeg)

Qlik Replicate uses TPT Operator with Attribute which can be tweaked based on environment requirements.

**_Figure A.3.3._**  
View TPT Settings  
![View Settings](./media/image15.jpeg)

**\*Press **"Browse"** button select Default database for Replicate and **"Test Connection"** button and Verify Connectivity.\***

**_Figure A.3.3._**  
Finalize Target Endpoint Settings  
![Target Endpoint](./media/image11.jpeg)

## **Part 4**

### Testing and Additional Information

If you fail in any of these steps please use the ODBC Manager built in with your os and test connections to your Teradata Server.

For Specific Qlik Replicate functions please follow replicate guidelines to setup and test CDC Tasks.

Additional information can be found in help documentation in [**Summary**](#summary).

For Testing Method Please contact your Qlik or Teradata Representatives for support from Professional services teams.

Othen Information can be found in [Qlik Community Technology Partners](https://community.qlik.com/t5/Technology-Partners-Ecosystem/ct-p/qlik-ecosystem "Qlik Technology Partner Eco System") in Technology Partner Sections where users can post questions and get them answered
