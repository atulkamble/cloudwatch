// EC2 - Cloudwatch 

1. Launch instance >> ssh 
2. instance-id copy example: i-0f1093ca8e968d2a8
3. cloudwatch >> dashboard >> create >> instance-id paste >> CPU-Utilization >> dashboard plot
4. behavior - stop instance

sudo yum update -y
sudo yum install stress-ng -y
sudo yum install stress -y
stress --version
stress-ng --version
stress
stress-ng
stress-ng --cpu 2 --timeout 60s
stress --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 10s

5. create alarm >> add email 
6. SNS Topic - and subscription list will be maintained in background, crosscheck SNS Service.
7. confirm email 
8. 
stress --cpu 8 --io 4 --vm 2 --vm-bytes 128M --timeout 10s
stress --cpu 4 --io 4 --vm 2 --vm-bytes 128M --timeout 20s
stress --cpu 2 --io 4 --vm 2 --vm-bytes 128M --timeout 30s

8. check dashbaord / alarm dashboard
9. you will recieve and email
