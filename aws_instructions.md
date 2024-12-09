# AWS Instructions

Amazon Web Services provides a powerful platform for web hosting and running a wide variety of heavy-lift-compute tasks. With great power, however, comes responsibility: be careful when starting up instances, to make sure you aren't starting one that will charge you a bunch of money, and be sure to pause your instances when you aren't using them. Terminate the instance when you are done with it.

## 1. Create an account with a free trial
First, if you don't already have an account, create one with a free trial at https://aws.amazon.com/free/. You will need a credit card, but you can use only free services so you don't get charged anything. Amazon customer service is also pretty forgiving if you mess up - but best to avoid anything that will get you charged any money.

## 2. Navigate to launch instance
Go to the AWS Management Console and find the link for EC2, either from the landing page or the Service menu.

<img width="1054" alt="Screenshot 2024-12-09 at 11 01 00 AM" src="https://github.com/user-attachments/assets/169859e8-e165-4980-8b05-ca531866e10d">

Click the orange Launch Instance button

<img width="864" alt="Screenshot 2024-12-09 at 11 01 17 AM" src="https://github.com/user-attachments/assets/62c1beea-800a-4999-90ed-b60cda8c39fb">

## 3. Configure the instance
You can mostly use the default settings. However, make sure to provide a descriptive name, select `ubuntu` from the AMIs list, pick the free-tier-eligible `t2.micro` instance type, and check to allow HTTP and HTTPS traffic (this will let you access your web server from your own computer's browser). Increase the default disk space to 30 GiB so you can accomodate the full human reference genome and annotations. Click **Launch instance** after double checking your configuration.

You may also want to select Proceed without key pair and just use the in-browser terminal connection to complete your work.

<img width="912" alt="Screenshot 2024-12-09 at 11 01 43 AM" src="https://github.com/user-attachments/assets/6fd02e5d-57c7-4617-aadd-11b20c57c261">
<img width="761" alt="Screenshot 2024-12-09 at 11 02 03 AM" src="https://github.com/user-attachments/assets/04665049-1067-439f-a7ce-1a93d53c9ef1">
<img width="676" alt="Screenshot 2024-12-09 at 11 02 21 AM" src="https://github.com/user-attachments/assets/ec37556e-126a-4cdb-bf31-bc5b7118b260">
<img width="675" alt="Screenshot 2024-12-09 at 11 02 36 AM" src="https://github.com/user-attachments/assets/75b881bd-bef9-45c2-98ed-2456588f8214">


## 4. Connect to the instance
You should land on a page that lists your active instances. 

<img width="1015" alt="Screenshot 2024-12-09 at 11 02 53 AM" src="https://github.com/user-attachments/assets/965900f4-07b2-4bb1-b84e-13a065c0a53f">

Click through to the one you just started. You should now see a status page like this:

<img width="1012" alt="Screenshot 2024-12-09 at 11 03 08 AM" src="https://github.com/user-attachments/assets/745ba605-adb4-4191-a7a5-c56db05f9672">

Click the connect button. That will take you through to a terminal session wherein you can follow the setup instructions in README.md for this repository.

## 5. IP address for your web server

In the home page of EC2, you can find the link to a list of your active instances.

<img width="1011" alt="Screenshot 2024-12-09 at 11 03 27 AM" src="https://github.com/user-attachments/assets/8992cea5-ece8-49bb-92d2-61f0e639eb69">

Click through to your instance. This will take you to the summary page. The public IP address is highlighted below. You can access your apache web server, once you set it up, by simply going to `http://ipaddress` - no ports required.

<img width="1016" alt="Screenshot 2024-12-09 at 11 03 40 AM" src="https://github.com/user-attachments/assets/e52d0d95-271a-40c6-8826-5b6ebe46a24d">

## 6. Stopping, starting, or terminating your instance

On your instance list, click the check box next to your instance and use the Instance state dropdown to select an option. Stop will suspend the instance whereas Terminate will permanently end it. Terminate when you are fully done with your AWS job.

<img width="1016" alt="Screenshot 2024-12-09 at 11 03 51 AM" src="https://github.com/user-attachments/assets/c081a64e-ad58-4c66-ad22-44773648acef">
