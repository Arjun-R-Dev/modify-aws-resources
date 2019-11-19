import time
import boto3
import configparser
import os
import sys
from boto3 import Session

def ec2mod(rds,ec2,ec3):
    #ec2 = boto3.resource('ec2')
    #ec3 = boto3.client('ec2')
    print ("Printing list of available instances\n\ns/n\tInstance_id\t\tCurrent instance_type\n-----------------------------------------")
    i=0
    for instance in ec2.instances.all():
        i= i + 1
        print(  "{0}\t{1}\t{2}\n".format(i,instance.id, instance.instance_type))
    instance = input('Enter the instance ID to modify:   ')
    if instance :
        print (("You have selected: \'%s\' ") % (instance))
        typeof = input(("\nUpgrade/Downgrade options for the instance \'%s\' are: \n\n Instance_type\tvCore\tRAM \n----------------------------------\n t2.micro\t1CPU\t1GB RAM \n t2.small\t1CPU\t2GB RAM\n t2.medium\t2CPU\t4GB RAM \n t2.large\t2CPU\t8GB RAM\n \nEnter required instance type: ") % (instance))
        if typeof:
            ec3.stop_instances(InstanceIds=[instance])
            waiter=ec3.get_waiter('instance_stopped')
            print (("EC2 instance \'%s\' is being modified") % (instance))
            waiter.wait(InstanceIds=[instance])
            # Change the instance type
            ec3.modify_instance_attribute(InstanceId=instance, Attribute='instanceType', Value = typeof )
            ec3.start_instances(InstanceIds=[instance])
            print ("Instance has been modified")
            fetch(rds,ec2,ec3)
        else:
            print("You did not specify any instance type\n")
            ec2mod(rds,ec2,ec3)
    else :
        print ("You have not selected any instance.\n\n")
        fetch(rds,ec2,ec3)

def rdsmod(rds,ec2,ec3):
    print ("\nPrinting list of available instances\n\nInstance_name\tInstance_type\n------------------------------------------")
    #rds = boto3.clienti('rds')
    dbs = rds.describe_db_instances()
    for db in dbs['DBInstances']:
        print (("%s \t\t %s \n") % (db['DBInstanceIdentifier'],db['DBInstanceClass']))
    instance = input('Enter the Instance name to modify:    ')
    if instance :
        typeof = input(('\nUpgrade/Downgrade options for the instance \'%s\' are: \n\n Instance_type\tvCore\tRAM \n----------------------------------\n db.t2.micro\t1CPU\t1GB RAM \n db.t2.small\t1CPU\t2GB RAM \n db.t2.medium\t2CPU\t4GB RAM \n db.t2.large\t2CPU\t8GB RAM \n db.t2.xlarge\t4CPU\t16GB RAM \n\n Enter required instance type:  ') % (instance))
        if typeof:
            rds.modify_db_instance(DBInstanceIdentifier=instance,DBInstanceClass=typeof, ApplyImmediately=True)
            print (("RDS instance \'%s\' is being modified") % (instance))
            for db in dbs['DBInstances']:
                if db['DBInstanceIdentifier'] == instance:
                    a = db['DBInstanceClass']
            while a != typeof:
                time.sleep(5)
                dbs = rds.describe_db_instances()
                for db in dbs['DBInstances']:
                    if db['DBInstanceIdentifier'] == instance:
                        a = db['DBInstanceClass']
                        print (("RDS modification to \'%s\' is completed now!") % (typeof))
                        fetch(rds,ec2,ec3)
        else:
            print("You did not select any instance type!!")
            rdsmod(rds,ec2,ec3)
    else:
        print ("You did not select any instance\n\n")
        fetch(rds,ec2,ec3)

def fetch(rds,ec2,ec3):
    dbs = rds.describe_db_instances()
    print ("\nRDS\n------\n")
    i=0
    for db in dbs['DBInstances']:
        i = i+1
        print (("%s %s \t %s\n") % (i,db['DBInstanceIdentifier'],db['DBInstanceClass']))
    print("EC2\n------\n")
    for instance in ec2.instances.all():
        i= i + 1
        print(  "{0}: {1}\t{2}\n".format(i,instance.id, instance.instance_type))
    service = input ("Select the service you want to modify \n\n\t1\tEC2 \n\t2\tRDS\n\t3\tEXIT \n\nEnter your selection here : ")
    os.system('cls')
    if service == 'EC2' or service == '1' or service == 'ec2':
        print ("You have selected to modify EC2 Instance")
        ec2mod(rds,ec2,ec3)
    elif service =='RDS' or service == '2' or service == 'rds':
        print ("You have selected to modify RDS Instance")
        rdsmod(rds,ec2,ec3)
    elif service == 'exit' or service == '3' or service == 'EXIT':
        print ("\nExiting the program")
        sys.exit()
    else :
        print ("\n\nYou made an invalid selection!!!\n\n")
        fetch(rds,ec2,ec3)

def main():
    print("\nPlease provide the AWS credentials\n")
    keyid=str(input('Enter access key ID [AKIA....]:    ') or "")
    secretkey = str(input("Enter Secret Key \t\t:    "))
    reg=str(input('Enter the region [ap-south-1]\t:    ') or "")
    sess = Session(aws_access_key_id=keyid,aws_secret_access_key=secretkey,region_name=reg)
    print("Saving credentials for the current session...")
    time.sleep(2)
    os.system('cls')
    print("################################################################################")
    print (("Generating the list of instances running in \'%s\':") % (reg))
    rds = sess.client('rds')
    dbs = rds.describe_db_instances()
    ec2 = sess.resource('ec2')
    ec3 = sess.client('ec2')
    fetch(rds,ec2,ec3)
if  __name__ =='__main__':main()
