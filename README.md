This repo contains the steps and files I used to build an EMR cluster with Spark and Hive (EMR Serveless) apps. See step 9 for issues I encountered to look out for if doing this tutorial.

1 - Upload files health_violations.py and food_establishment_data.csv to s3. An s3 bucket that statisfies these requirements is needed https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs.html#:~:text=Buckets%20and%20folders%20that%20you%20use%20with%20Amazon%20EMR%20have%20the%20following%20limitations%3A.

2 - Create EMR IAM role:

	aws emr create-default-roles

3 - Create Sprak Cluster:

    aws emr create-cluster \
   	--name "<Name of  Emr Cluster>" \
   	--release-label <emr-5.36.0> \
   	--applications Name=Spark \
   	--ec2-attributes KeyName=<MyEC2KeyPairname> \
   	--instance-type m5.xlarge \
   	--instance-count 3 \
   	--use-default-roles    

4 - Check cluster status:
    aws emr describe-cluster --cluster-id <clusterid>

5 - Submit work to the EMR cluster:

		aws emr list-clusters --cluster-states WAITING							
		aws emr add-steps \
		--cluster-id <myClusterId> \
		--steps Type=Spark,Name="<My Spark Application>",ActionOnFailure=CONTINUE,Args=[<s3://DOC-EXAMPLE-BUCKET/health_violations.py>,--data_source,<s3://DOC-EXAMPLE-BUCKET/food_establishment_data.csv>,--output_uri,<s3://DOC-EXAMPLE-BUCKET/MyOutputFolder>]

 If this cmd fails, use the console wth these steps:
 https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs.html#:~:text=To%20submit%20a%20Spark%20application%20as%20a%20step%20using%20the%20console								

6 - Check cluster status:

    aws emr describe-step --cluster-id <myClusterId> --step-id <s-1XXXXXXXXXXA>							

Navigate to the s3 bucket output folder provided when adding the steps to see the results of the job in a file with prefix part-.

7 - SSH into cluster:
  a- Add SSH inbound rule with you IP to the cluster SG as outlined here https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs.html#:~:text=To%20allow%20SSH%20access%20for%20trusted%20sources%20for%20the%20ElasticMapReduce%2Dmaster%20security%20group

  b- SSH into cluster manager node:

  aws emr ssh --cluster-id <j-2AL4XXXXXX5T9> --key-pair-file <~/.aws/						

    Look at the logs:

    cd /mnt/var/log/spark && ls

8 - Clean up:

  a- Terminate the cluster:

      aws emr terminate-clusters --cluster-ids <myClusterId>

      aws emr describe-cluster --cluster-id <myClusterId>		

9 - Some issues encountered:
Cluster failed (TERMINATED) right after I built it a few times due to the following:
	a- I didn't have any VPCs in the region I was trying to use.
	b- I didn't setup a key pair before building the cluster.

The cmd to submit work to the cluster (step 5) did not work at first so I used the console following these steps and figured out that I was using an s3 arn instead of an s3 URI in the CLI cmd above https://docs.aws.amazon.com/emr/latest/ManagementGuide/emr-gs.html#:~:text=To%20submit%20a%20Spark%20application%20as%20a%20step%20using%20the%20console  	

10 - Using EMR serverless (I ran out of time and did not add the steps but this is the tutorial I used https://docs.aws.amazon.com/emr/latest/EMR-Serverless-UserGuide/getting-started.html).
