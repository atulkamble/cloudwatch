# AWS CloudWatch

## 1. Introduction to Amazon CloudWatch

Amazon Web Services Amazon CloudWatch is an AWS **monitoring and observability service** used to monitor AWS resources, applications, and workloads.

CloudWatch collects and works with:

* **Metrics** — numerical measurements such as CPU utilization.
* **Logs** — application, operating-system, and AWS service logs.
* **Alarms** — notifications or automated actions based on metric thresholds.
* **Dashboards** — graphical visualization of metrics.
* **Events** — changes or activities that can trigger automated actions. For new event-driven configurations, AWS uses **Amazon EventBridge**; CloudWatch Events is the predecessor.
* **Traces/Application signals** — observability information for distributed applications.

[Amazon CloudWatch documentation](https://docs.aws.amazon.com/cloudwatch/?utm_source=chatgpt.com)

---

# 2. Why CloudWatch Is Important

CloudWatch helps administrators and DevOps engineers answer questions such as:

```text
Is my EC2 instance overloaded?
Is CPU usage too high?
Is my application generating errors?
How much traffic is reaching my application?
Is an AWS resource unhealthy?
Should an administrator be notified?
Should Auto Scaling add more instances?
```

Basic monitoring architecture:

```text
AWS Resources / Applications
          |
          |
          v
+-------------------------+
|    Amazon CloudWatch    |
+-------------------------+
     |       |       |
     v       v       v
  Metrics   Logs   Events
     |
     v
   Alarms
     |
     +--------------------+
     |                    |
     v                    v
 Amazon SNS         Automated Action
     |              (EC2 / Auto Scaling)
     v
Email / Notification
```

---

# 3. Points to Remember

1. CloudWatch is primarily a **monitoring and observability** service.
2. Many AWS services automatically publish metrics to CloudWatch.
3. A **Metric** represents a measurement over time.
4. Metrics are organized using **Namespaces**.
5. **Dimensions** identify characteristics of a metric, such as an EC2 Instance ID.
6. CloudWatch **Alarms** watch metrics and respond when configured conditions are met.
7. CloudWatch **Logs** stores and analyzes log data.
8. CloudWatch **Dashboards** visualize metrics.
9. **SNS** is commonly combined with CloudWatch alarms for notifications.
10. EC2 publishes several infrastructure metrics automatically.
11. Detailed EC2 monitoring provides **1-minute metrics**; basic monitoring generally uses **5-minute periods**.
12. EC2 memory and disk-space utilization are **not standard EC2 metrics**; the CloudWatch Agent can collect them.
13. S3 publishes storage metrics and supports additional request metrics when configured.
14. CloudWatch Events functionality has largely evolved into **Amazon EventBridge**.
15. CloudWatch can be integrated with services such as EC2, Lambda, Auto Scaling, SNS, and EventBridge.

---

# 4. Important CloudWatch Components

| Component        | Purpose                          | Example          |
| ---------------- | -------------------------------- | ---------------- |
| Metrics          | Numerical measurements           | CPUUtilization   |
| Alarms           | Monitor metric conditions        | CPU > 80%        |
| Logs             | Store/analyze logs               | Application logs |
| Dashboards       | Visual monitoring                | EC2 dashboard    |
| CloudWatch Agent | Collect OS/application telemetry | Memory usage     |
| EventBridge      | Event-driven automation          | EC2 state change |
| Logs Insights    | Query log data                   | Search errors    |

---

# 5. CloudWatch Metric Architecture

Example EC2 metric:

```text
Namespace
AWS/EC2
   |
   +-- Metric
       CPUUtilization
          |
          +-- Dimension
              InstanceId=i-0123456789
                  |
                  +-- Datapoints
                      |
                      +-- 20%
                      +-- 45%
                      +-- 75%
                      +-- 90%
```

A useful way to remember this is:

```text
Namespace
   |
Metric
   |
Dimension
   |
Datapoints
```

Example:

```text
AWS/EC2
   |
CPUUtilization
   |
InstanceId
   |
CPU = 75%
```

---

# 6. Practical 1 — Monitor EC2 CPU Using CloudWatch

## Architecture

```text
+-------------+
| EC2 Instance|
+------+------+
       |
       | CPUUtilization
       v
+------------------+
|    CloudWatch    |
|     Metrics      |
+--------+---------+
         |
         v
+------------------+
| CloudWatch Alarm |
|    CPU > 80%     |
+--------+---------+
         |
         v
+------------------+
|    Amazon SNS    |
+--------+---------+
         |
         v
+------------------+
| Email / Admin    |
+------------------+
```

## Steps in AWS Console

```text
Step 1:
Login to AWS Console

Step 2:
Go to EC2

Step 3:
Create or select an EC2 instance

Step 4:
Go to CloudWatch

Step 5:
Select Metrics

Step 6:
Select All metrics

Step 7:
Select EC2

Step 8:
Select Per-Instance Metrics

Step 9:
Search for your Instance ID

Step 10:
Select CPUUtilization

Step 11:
Observe the CPU utilization graph
```

---

# 7. Check EC2 Metrics Using AWS CLI

List available EC2 metrics:

```bash
aws cloudwatch list-metrics \
--namespace AWS/EC2
```

Filter metrics for CPU utilization:

```bash
aws cloudwatch list-metrics \
--namespace AWS/EC2 \
--metric-name CPUUtilization
```

Check the instance ID:

```bash
aws ec2 describe-instances \
--query "Reservations[].Instances[].InstanceId" \
--output table
```

Example metric retrieval:

```bash
aws cloudwatch get-metric-statistics \
--namespace AWS/EC2 \
--metric-name CPUUtilization \
--dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
--statistics Average \
--period 300 \
--start-time 2026-09-22T06:00:00Z \
--end-time 2026-09-22T07:00:00Z
```

---

# 8. Practical 2 — Generate CPU Load

Connect to the EC2 instance:

```bash
ssh -i mykey.pem ec2-user@PUBLIC-IP
```

For Amazon Linux:

```bash
sudo dnf install stress-ng -y
```

Generate CPU load:

```bash
stress-ng --cpu 2 --timeout 300s
```

Check CPU:

```bash
top
```

Now observe:

```text
EC2
 |
CPU Load
 |
 v
CloudWatch Metric
 |
CPUUtilization increases
```

---

# 9. Create a CloudWatch Alarm

Example requirement:

```text
CPUUtilization > 80%
```

Console steps:

```text
Step 1:
CloudWatch

Step 2:
Alarms

Step 3:
All alarms

Step 4:
Create alarm

Step 5:
Select metric

Step 6:
EC2

Step 7:
Per-Instance Metrics

Step 8:
Select CPUUtilization

Step 9:
Select Statistic = Average

Step 10:
Configure threshold

CPUUtilization
Greater than
80

Step 11:
Configure notification

Step 12:
Select/Create SNS Topic

Step 13:
Enter email address

Step 14:
Confirm SNS subscription from email

Step 15:
Give alarm a name

Example:
EC2-High-CPU-Alarm

Step 16:
Create alarm
```

---

# 10. Create Alarm Using AWS CLI

Create SNS topic:

```bash
aws sns create-topic \
--name cloudwatch-alerts
```

Subscribe an email address:

```bash
aws sns subscribe \
--topic-arn arn:aws:sns:us-east-1:123456789012:cloudwatch-alerts \
--protocol email \
--notification-endpoint admin@example.com
```

Confirm the subscription from the email.

Create alarm:

```bash
aws cloudwatch put-metric-alarm \
--alarm-name EC2-High-CPU \
--alarm-description "Alert when CPU exceeds 80 percent" \
--namespace AWS/EC2 \
--metric-name CPUUtilization \
--dimensions Name=InstanceId,Value=i-0123456789abcdef0 \
--statistic Average \
--period 300 \
--evaluation-periods 1 \
--threshold 80 \
--comparison-operator GreaterThanThreshold \
--alarm-actions arn:aws:sns:us-east-1:123456789012:cloudwatch-alerts
```

Check alarms:

```bash
aws cloudwatch describe-alarms
```

---

# 11. CloudWatch Alarm States

There are three important alarm states:

```text
              Metric
                |
                v
       +------------------+
       | CloudWatch Alarm |
       +--------+---------+
                |
       +--------+--------+
       |        |        |
       v        v        v
      OK      ALARM  INSUFFICIENT_DATA
```

**OK** — metric is within the configured threshold.

**ALARM** — metric has breached the configured threshold.

**INSUFFICIENT_DATA** — CloudWatch does not currently have enough data to determine the state.

---

# 12. Practical 3 — Monitor Amazon S3

S3 integrates with CloudWatch for storage and request monitoring.

Common S3 metrics include:

```text
BucketSizeBytes
NumberOfObjects
AllRequests
GetRequests
PutRequests
DeleteRequests
4xxErrors
5xxErrors
```

Storage metrics and request metrics have different collection/configuration behavior; request metrics generally need to be enabled.

## Console Steps

```text
Step 1:
Create an S3 bucket

Step 2:
Upload some objects

Step 3:
Go to CloudWatch

Step 4:
Select Metrics

Step 5:
Select All metrics

Step 6:
Select S3

Step 7:
Select Storage Metrics

Step 8:
Select the required bucket

Step 9:
Monitor:
BucketSizeBytes
NumberOfObjects
```

CLI:

```bash
aws cloudwatch list-metrics \
--namespace AWS/S3
```

---

# 13. EC2 vs S3 CloudWatch Metrics

| EC2               | S3              |
| ----------------- | --------------- |
| CPUUtilization    | BucketSizeBytes |
| NetworkIn         | NumberOfObjects |
| NetworkOut        | AllRequests*    |
| DiskReadBytes     | GetRequests*    |
| DiskWriteBytes    | PutRequests*    |
| StatusCheckFailed | 4xxErrors*      |

`*` Request metrics require appropriate S3 request-metrics configuration.

---

# 14. CloudWatch Agent

Standard EC2 monitoring does not automatically provide every operating-system metric.

For example:

```text
CPUUtilization        -> Standard EC2 metric
NetworkIn             -> Standard EC2 metric
NetworkOut            -> Standard EC2 metric

Memory utilization    -> CloudWatch Agent
Disk space usage      -> CloudWatch Agent
Application logs      -> CloudWatch Agent / application integration
```

Architecture:

```text
+---------------------------+
|        EC2 Instance       |
|                           |
| Application               |
| Linux / Windows           |
|                           |
| +-----------------------+ |
| | CloudWatch Agent      | |
| +----------+------------+ |
+------------|--------------+
             |
      +------+------+
      |             |
      v             v
CloudWatch       CloudWatch
 Metrics           Logs
```

---

# 15. CloudWatch Logs

CloudWatch Logs architecture:

```text
Application / EC2
        |
        v
CloudWatch Agent
        |
        v
+--------------------+
|     Log Group      |
+---------+----------+
          |
          v
+--------------------+
|     Log Stream     |
+---------+----------+
          |
          v
       Log Events
```

Remember:

```text
Log Group
   |
   +-- Log Stream
          |
          +-- Log Events
```

Example:

```text
/myapplication
      |
      +-- ec2-server-01
      |       |
      |       +-- ERROR Database connection failed
      |
      +-- ec2-server-02
              |
              +-- INFO Application started
```

---

# 16. CloudWatch Events

An important update for students:

**Amazon EventBridge is the recommended service for event-driven rules.** Amazon EventBridge builds on and extends the earlier CloudWatch Events functionality.

Example:

```text
EC2 Instance
     |
     | State Change
     v
Amazon EventBridge
     |
     | Rule
     v
+----------------+
|     Target     |
+-------+--------+
        |
   +----+----+
   |         |
   v         v
Lambda      SNS
```

Example requirement:

```text
When EC2 instance changes to STOPPED
             |
             v
       EventBridge
             |
             v
          SNS Topic
             |
             v
            Email
```

[Amazon EventBridge documentation](https://docs.aws.amazon.com/eventbridge/?utm_source=chatgpt.com)

---

# 17. Practical 4 — Create an EC2 State-Change Event

## Console Steps

```text
Step 1:
Open Amazon EventBridge

Step 2:
Go to Rules

Step 3:
Create Rule

Step 4:
Rule Name:
EC2-State-Change

Step 5:
Select:
Rule with an event pattern

Step 6:
Event Source:
AWS Events

Step 7:
AWS Service:
EC2

Step 8:
Event Type:
EC2 Instance State-change Notification

Step 9:
Select required states

Example:
stopped

Step 10:
Select Target

Example:
SNS Topic

Step 11:
Select your SNS topic

Step 12:
Create Rule

Step 13:
Stop the EC2 instance

Step 14:
EventBridge detects the state change

Step 15:
SNS sends the notification
```

Example event pattern:

```json
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["stopped"]
  }
}
```

---

# 18. EventBridge CLI Commands

Create a rule:

```bash
aws events put-rule \
--name EC2StoppedRule \
--event-pattern '{"source":["aws.ec2"],"detail-type":["EC2 Instance State-change Notification"],"detail":{"state":["stopped"]}}'
```

Check rules:

```bash
aws events list-rules
```

Describe rule:

```bash
aws events describe-rule \
--name EC2StoppedRule
```

Delete rule after removing its targets:

```bash
aws events remove-targets \
--rule EC2StoppedRule \
--ids "1"
```

```bash
aws events delete-rule \
--name EC2StoppedRule
```

---

# 19. Metrics vs Logs vs Alarms vs Events

| Feature   | Meaning                    | Example                  |
| --------- | -------------------------- | ------------------------ |
| Metric    | Numerical measurement      | CPU = 85%                |
| Log       | Detailed record/message    | ERROR: DB failed         |
| Alarm     | Evaluates metric condition | CPU > 80%                |
| Event     | Something happened         | EC2 stopped              |
| Dashboard | Visualization              | EC2 monitoring dashboard |

Easy memory trick:

```text
Metric = How much?

Log = What happened inside?

Alarm = Is there a problem?

Event = What happened?

Dashboard = Show everything visually
```

---

# 20. Complete CloudWatch Architecture

```text
                    AWS CLOUD
                        |
        +---------------+---------------+
        |               |               |
        v               v               v
      EC2              S3            Lambda
        |               |               |
        +---------------+---------------+
                        |
                        v
              +-------------------+
              | Amazon CloudWatch |
              +---------+---------+
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
       Metrics         Logs       Dashboards
          |
          v
       Alarms
          |
          v
        SNS
          |
          v
    Email / Admin


EC2 / AWS Service
        |
        | State Change/Event
        v
+-------------------+
| Amazon EventBridge|
+---------+---------+
          |
        Rule
          |
    +-----+------+
    |            |
    v            v
 Lambda         SNS
                 |
                 v
               Email
```

## Final Points to Remember

```text
CloudWatch = Monitoring + Observability

Metrics = Numerical measurements

Logs = Application/System records

Alarm = Monitor a metric against a condition

Dashboard = Visualize monitoring information

SNS = Send notifications

CloudWatch Agent = OS metrics + logs

EC2 CPU = Available automatically

EC2 Memory = Install/configure CloudWatch Agent

S3 = Storage metrics + optional request metrics

CloudWatch Events = Older event capability

EventBridge = Current AWS event-routing service

EC2 -> Metrics -> CloudWatch -> Alarm -> SNS -> Email

EC2 State Change -> EventBridge -> Rule -> SNS/Lambda
```

For a basic student lab, the best flow to practice is **EC2 → CloudWatch CPUUtilization → Alarm → SNS → Email**, followed by **EC2 state change → EventBridge → SNS**.
