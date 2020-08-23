# Test tasks
## Task 1
CFN template is asg.yaml file.

## Task 2
To reduce the number of 50x errors while scaling up in CFN template I added alarm for average CPU usage of instances within ASG which is checked every 60 seconds for the threshold of 30%. When the average usage is higher than 30% the alarm is raised. Then scale up policy is triggered which has 60 seconds cooldown. It increases amount of instances by 80%. That way CPU usage spikes are detected early and the number of instances is increased greatly since there could be a lot of new users.
After if the usage isn't high there is an alarm which is checked every 5 minutes. It check if the usage is less than 20%. The scale down policy has 10 minutes cooldown and terminates 20% of instances. That way we make sure that CPU usage is stabilized and we can shutdown small part of instances. If there is a new spike of CPU usage in the near feature we won't spend time on launching instances again and the application will be available.
CPU usage can be changed to any other metric from AWS Cloudwatch. Metrics for the application weren't specified in the description.

## Task 3
To store PHP sessions in the Redis cluster via ElastiCache php.ini file should be configured. The path to the file is different on every OS.
During the AMI creation for the instance I suggest replacing needed php.ini file with the template where we pass `CLUSTER_ENDPOINT` environment variable for the Redis cluster:
```
session.save_handler = rediscluster
session.save_path = "seed[]=${CLUSTERENDPOINT}"
session.gc_maxlifetime = 1296000
```
That could be done with `envvsubst` command
`envsubst '${CLUSTER_ENDPOINT}' < /usr/local/lib/templates/php.ini > /path/php.ini`

Template file is added also during the AMI creation with the bash script which executes the command above. Also the script should restart the web-server after changing php.ini file.

In CFN template I added the execution of the script during the instance startup with the userdata and cfn-init. Also cluster endpoint is passed as a `CLUSTER_ENDPOINT` variable from `ClusterEndpoint` parameter to the script. Path to the executed script in the CFN template is `/usr/local/sbin/customize`.
