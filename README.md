# Monitoring Infrastructure Using Amazon CloudWatch

## Project Overview
Monitoring applications and infrastructure is crucial for delivering reliable and consistent IT services. The requirements range from collecting statistics for long-term analysis to quickly reacting to changes and outages. Additionally, monitoring supports compliance reporting by continuously verifying that infrastructure meets organizational standards.

In this project, I utilized Amazon CloudWatch Metrics, Amazon CloudWatch Logs, Amazon CloudWatch Events, and AWS Config to monitor applications and infrastructure.

### Objectives
By completing this project, I was able to:
- Use the AWS Systems Manager Run Command to install the CloudWatch agent on Amazon Elastic Compute Cloud (Amazon EC2) instances.
- Monitor application logs using the CloudWatch agent and CloudWatch Logs.
- Monitor system metrics using the CloudWatch agent and CloudWatch Metrics.
- Create real-time notifications using CloudWatch Events.
- Track infrastructure compliance using AWS Config.

## Task 1: Installing the CloudWatch Agent
I used the CloudWatch agent to collect metrics from EC2 instances and on-premises servers. This included:
- System-level metrics from EC2 instances, such as CPU allocation, free disk space, and memory utilization.
- Monitoring of hybrid environments by collecting system metrics from on-premises servers.
- Collecting system and application logs from both Linux and Windows servers.
- Gathering custom metrics from applications using the StatsD and collectd protocols.

![image](https://github.com/user-attachments/assets/15acc197-8371-4fba-9f31-08ffda94fc59)

To install the CloudWatch agent, I leveraged AWS Systems Manager's Run Command:
1. Opened the **AWS Management Console** and navigated to **Systems Manager**.
2. Selected **Run Command** from the left panel.
![image](https://github.com/user-attachments/assets/b1eecaf8-a639-41ae-aa38-0868e4101d52)

4. Choose **Run a Command** and selected `AWS-ConfigureAWSPackage`.
![image](https://github.com/user-attachments/assets/a38f48c6-e456-48dd-98dd-6f75dd859cc2)

6. Configured the parameters:
   - **Action**: Install
   - **Name**: AmazonCloudWatchAgent
   - **Version**: latest
![image](https://github.com/user-attachments/assets/7a6721d5-bb3e-43eb-8d63-cc6c4a86abf7)

7. Selected **Choose instances manually** and checked the box for the **Web Server** instance.
![image](https://github.com/user-attachments/assets/099f4948-cbb1-4a7b-afbd-d836102061f5)

9. Executed the command and waited for a success confirmation.
![image](https://github.com/user-attachments/assets/cc0aca57-d936-4f41-837c-2f6e62f2c5ea)
![image](https://github.com/user-attachments/assets/f4c1eb06-581c-4502-98ae-dd7094a575b3)


After confirming the successful installation, I proceeded to configure the CloudWatch agent to collect system metrics and web server logs.

### Configuring CloudWatch Agent
To define the log collection and monitoring settings, I stored a configuration file in **AWS Systems Manager Parameter Store**:
1. Opened **Parameter Store** and created a new parameter.
![image](https://github.com/user-attachments/assets/2bc0beff-f559-4245-9b33-ddb0fa7a14a3)

3. Set the **Name** as `Monitor-Web-Server`.
4. Added a description: "Collect web logs and system metrics".
![image](https://github.com/user-attachments/assets/1c880f68-77cb-4060-b891-5506c3614cec)

6. Defined the configuration file to monitor and created the parameter:
   - Web server logs (access and error logs)
   - CPU, disk, memory, and swap usage

```json
{
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "log_group_name": "HttpAccessLog",
            "file_path": "/var/log/httpd/access_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          },
          {
            "log_group_name": "HttpErrorLog",
            "file_path": "/var/log/httpd/error_log",
            "log_stream_name": "{instance_id}",
            "timestamp_format": "%b %d %H:%M:%S"
          }
        ]
      }
    }
  },
  "metrics": {
    "metrics_collected": {
      "cpu": {
        "measurement": [
          "cpu_usage_idle", "cpu_usage_iowait", "cpu_usage_user", "cpu_usage_system"
        ],
        "metrics_collection_interval": 10
      },
      "mem": {
        "measurement": ["mem_used_percent"],
        "metrics_collection_interval": 10
      }
    }
  }
}
```
![image](https://github.com/user-attachments/assets/63e73b42-89cb-452e-b285-00e72c3ec79c)

2. **Configure the CloudWatch Agent to Use the New Parameter:**
   - Go to `Run Command`.
   - Click `Run Command`.
   - Select `AmazonCloudWatch-ManageAgent`.
![image](https://github.com/user-attachments/assets/b8561186-3dfc-4dc9-8044-645dcf08c735)

   - Configure parameters:
     - **Action**: Configure
     - **Mode**: EC2
     - **Optional Configuration Source**: SSM
     - **Optional Configuration Location**: Monitor-Web-Server
     - **Optional Restart**: Yes
![image](https://github.com/user-attachments/assets/af035033-54a7-4fd6-bf21-ff5df075782d)

   - Select `Choose instances manually` and choose `Web Server`.
![image](https://github.com/user-attachments/assets/2c1e2163-5763-4e78-93ea-da760a02bd1d)

   - Click `Run` and wait for the status to change to `Success`.
![image](https://github.com/user-attachments/assets/5c31c3d6-4782-4972-92c4-c4a2af450ff2)

---

## Task 3: Monitoring Application Logs Using CloudWatch Logs
CloudWatch Logs allows monitoring of applications and system logs.

![image](https://github.com/user-attachments/assets/b61e8911-e04d-4f17-98eb-34edf481d805)

### Steps:

1. **Generating some log data:**

- Copy and open the WebServerIP value
![image](https://github.com/user-attachments/assets/b2315691-6df2-4c50-a392-f641a6e6f503)
![image](https://github.com/user-attachments/assets/9af302aa-fd10-47a6-a7fc-65e9b1160ded)

- Generate log data by attempting to access a page that does not exist
![image](https://github.com/user-attachments/assets/e546a930-8887-4ed9-ba45-15ab7acff02f)


2. **Verify Logs in CloudWatch:**
   - Navigate to `CloudWatch` in AWS Management Console.
   - In the left panel, select `Logs` → `Log Groups`.
   - Check for `HttpAccessLog` and `HttpErrorLog`.
![image](https://github.com/user-attachments/assets/57086f29-8bc2-44a2-be17-09c79d462d6c)

- Choose HttpAccessLog, LogStreams and the id of the instance that is attached to
![image](https://github.com/user-attachments/assets/5dcf6df7-6059-4566-8882-bb3a640ae6eb)

We now see a line with the **/start** request with a code of 404, which means that the page was not found.
![image](https://github.com/user-attachments/assets/3e8e01e7-c6bc-458e-b555-9d80f0210ca6)

3. **Create Log Metric Filters:**
   - In `CloudWatch Logs`, choose `Create Metric Filter`.
![image](https://github.com/user-attachments/assets/8b450dc0-39eb-4587-932e-36df6f06383e)

   - Define, test and create a filter pattern to track error occurrences.
![image](https://github.com/user-attachments/assets/97a6f896-2a5d-4e03-9345-93672fbde265)
![image](https://github.com/user-attachments/assets/0968c6a1-1d46-4b3c-bb83-3db8fb102d51)
![image](https://github.com/user-attachments/assets/7c8d6c7f-a21d-4d3e-bc11-aace2f2dbb05)
![image](https://github.com/user-attachments/assets/311d574c-d39d-4810-bd14-128afcca13dd)

   - Set up an alarm to notify on high error rates by selecting the filter and creating an SNS Topic.
![image](https://github.com/user-attachments/assets/8189657a-fb5c-42c3-9875-f7975c9b3ad7)
![image](https://github.com/user-attachments/assets/14493d4b-c9c0-4f83-85dc-5687dd4a0254)
![image](https://github.com/user-attachments/assets/059e6da7-3692-4745-be4b-316edbfbdadd)
![image](https://github.com/user-attachments/assets/eeef00d3-b776-4f35-9eb6-c104686963f0)
![image](https://github.com/user-attachments/assets/b4f9f6dc-99e4-4b87-8d18-8a9f3788141d)
![image](https://github.com/user-attachments/assets/52c2c595-9b5f-4071-86f5-58f90197b68f)
![image](https://github.com/user-attachments/assets/06d44bcd-1a30-42e3-88ea-71e3e83eaf2f)
![image](https://github.com/user-attachments/assets/ccac0f40-f7e5-46eb-b8f3-1534ff0e3990)

- We test if the alarm is correctly set up by generating new log data, taking into account that it should be 5 errors or above to trigger it

 We return to the browser tab repeatedly requesting other pages that don't exist, for example, **/start2, /start3 and so on**
![image](https://github.com/user-attachments/assets/d0c3c95f-b0f3-41dd-9477-995e8c2289f0)

- Now our metric is in an **Alarm state** and sends the event to the email
![image](https://github.com/user-attachments/assets/4845a578-648f-45b2-8e89-e1ca27802cf3)
![image](https://github.com/user-attachments/assets/63dd0167-861c-44cc-beab-5544213c04db)

---

## Task 4: Monitoring System Metrics Using CloudWatch Metrics
CloudWatch Metrics allows real-time monitoring of system performance.
![image](https://github.com/user-attachments/assets/b1a1b1f8-7a5e-4d50-8799-a54763ae46cc)

### Steps:
1. **Verify Metrics Collection:**
   - Navigate to `EC2` → `Select the instance and Monitoring tab`.
Check `CPUUsage`, `Memory`, and `DiskUsage`
![image](https://github.com/user-attachments/assets/db8a84a7-265c-45bb-a764-283c19feec49)

- On CloudWatch metrics with CWAgent we can obtain more detailed information
![image](https://github.com/user-attachments/assets/57c4e0d7-238f-475e-8b15-6e8d23095f61)
![image](https://github.com/user-attachments/assets/4b488b9d-9a68-4ba3-ab19-7811e4d0d8bd)

We see the disk space metrics that the CloudWatch agent is capturing.
![image](https://github.com/user-attachments/assets/f4661725-f26e-4005-bc78-8271dd4831d0)

2. **Create a CloudWatch Alarm for Instance termination status:**
![image](https://github.com/user-attachments/assets/ddf67954-fa38-4e8a-ab45-25f5484e5d86)

    - Go to `CloudWatch` → Events, choose Rules.
![image](https://github.com/user-attachments/assets/3d2eddb6-bf46-4903-9863-199c1c3d57a4)
![image](https://github.com/user-attachments/assets/05e40f28-d102-4111-a0c5-06d67f3c5947)
![image](https://github.com/user-attachments/assets/eaf597d4-c438-4b0a-982c-da532ca0ad82)
![image](https://github.com/user-attachments/assets/50d71dd9-20e4-4729-aba3-606ae980a15f)
![image](https://github.com/user-attachments/assets/73189231-9ba0-4c5e-985a-e956b201278b)
![image](https://github.com/user-attachments/assets/c230b588-512f-4eb2-877e-732766ad88e0)

   - Configure notifications via Amazon SNS and test if the alarm is correctly set up by stopping the instance on purpose.
![image](https://github.com/user-attachments/assets/9982b823-753b-4590-a20e-52a4fc799c9e)
![image](https://github.com/user-attachments/assets/1796080f-bcde-40e3-9e7b-50a2dfc6b646)
![image](https://github.com/user-attachments/assets/42ebc6b5-efdd-41a8-aae8-59005a4d8482)

---

## Task 5: Tracking Compliance with AWS Config
AWS Config tracks infrastructure changes and compliance with policies.

### Steps:
1. **Enable AWS Config:**
   - Open `AWS Config` in the AWS Management Console.
   - Click `Set up AWS Config` and enable resource tracking.

2. **Define Compliance Rules:**
   - Select `Rules` → `Add Rule`.
![image](https://github.com/user-attachments/assets/eb0bb721-c4d5-41bf-b33a-26f87f06b27d)
![image](https://github.com/user-attachments/assets/a1005401-8da1-43d5-9002-91f296cd33bf)

![image](https://github.com/user-attachments/assets/5c2f6481-a402-4c9e-8f50-e6db6ddfe89e)

This rule now looks for resources that do not have a project tag

- Add a rule that looks for EBS volumes that are not attached to EC2 instances
![image](https://github.com/user-attachments/assets/4410ee1c-d92f-4993-85a3-ea182692750f)
![image](https://github.com/user-attachments/assets/1abab23f-640b-4de6-9fd2-4dc9ae1435c6)

- Choose each of the rules to view the result of the audits
Under Resources in scope select Compliant from the list.
The following should be among the results:

1. required-tags: A compliant EC2 instance (because the Web Server has a project tag) and many non-compliant resources that do not have a project tag

![image](https://github.com/user-attachments/assets/6de45086-2d7a-48d8-bb98-6b412db3e0c4)

2. ec2-volume-inuse-check: One compliant volume (attached to an instance) and one non-compliant volume (not attached to an instance)
![image](https://github.com/user-attachments/assets/a98712cf-d9cc-421a-915f-d3da108ca872)

---

## Summary
In this project, I:
- Installed and configured the CloudWatch agent.
- Collected system logs and metrics.
- Monitored application logs with CloudWatch Logs.
- Set up alarms and event notifications.
- Ensured compliance with AWS Config.

This setup helps in proactive monitoring and maintaining infrastructure reliability.
