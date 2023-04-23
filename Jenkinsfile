pipeline {
  agent any
  parameters {
    string(name: 'vm_name', defaultValue: 'myvmforansible')
    string(name: 'vm_subnet', defaultValue: 'subnet-063cfea52feac4e49')
    string(name: 'vm_type', defaultValue: 't2.micro')
    string(name: 'vm_role', defaultValue: 'my_ec2_role')
    string(name: 'vm_sg_id', defaultValue: 'sg-00890e9f0d99216cc')
    string(name: 'vm_key_name', defaultValue: '31march')
}

environment {
AWS_REGION = "eu-central-1"
AWS_DEFAULT_REGION = "eu-central-1"
AMI_ID = "ami-08f54b258788948e1"  
}  

 stages {
   stage( 'SCM checkout') {
     steps {
      git branch: 'main', url: 'https://github.com/amitats/janrepo.git'
     }
   }


 stage ('Checkout Ansible Roles') {
     steps {
      script {
     sh "mkdir -p bottlerocket-linux-v1/bulld-artifaces"
     sh "mkdir -p bottlerocket-linux-v1/roles/odp-ansible-common-utils"
     dir {"bottlerocket-linux-v1/roles/odp-ansible-common-utils"} {
      git branch: 'main', url: "git@github.com:amitats/myjenkinsrepo.git"
     }

    sh "mkdir -p bottlerocket-linux-v1/roles/odp-ansible-endgame"
    dir {"bottlerocket-linux-v1/roles/odp-ansible-endgame"} {
    git branch: 'main', url: "git@github.com:amitats/jenkinstestrepo1.git"
    sh "mkdir -p files"
   }
  }
 }
}

 stage( 'Launching EC2 Instance') {
    steps {
      script {
        withCredentials([[
        $class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: "8f13d382-812b-4ce2-96e6-19a56ae858c8",
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
    ]]) {
         sh """ #!/bin/bash 
            rm -fr output.txt
            aws ec2 run-instances --region ${AWS_REGION} --count 1 \
                 --image-id ${AMI_ID} \
                 --instance-type ${params.vm_type} --key-name ${params.vm_key_name} \
                 --security-group-ids ${params.vm_sg_id} --subnet-id ${params.vm_subnet} \
                 --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=${params.vm_name}}]" \
                 --iam-instance-profile Name=${params.vm_role} \
                 --user-data file://bottlerocket-linux-v1/bottlerocketdata.sh
                 sleep 60
                 aws ec2 describe-instances  --filters "Name=tag:Name,Values=myvmforansible" --query 'Reservations[*].Instances[?!contains(State.Name, `terminated`)].[InstanceId]' --output text > output.txt 
             """
     }    
    }
  }
 }
   stage( 'Executing ansible playbook') {
    steps {
      script {
        withCredentials([[
        $class: 'AmazonWebServicesCredentialsBinding',
        credentialsId: "8f13d382-812b-4ce2-96e6-19a56ae858c8",
        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
    ]]) {
        sh  """
        rm -fr ip.txt
        rm -fr dev.inv
        aws ec2 describe-instances \
        --instance-ids `cat output.txt` --query 'Reservations[*].Instances[*].PrivateIpAddress' --output text > ip.txt
        echo "[webservers]" >> dev.inv
        echo "`cat ip.txt` ansible_user=ec2-user" >> dev.inv
        """
        ansiblePlaybook credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible2', inventory: 'dev.inv', playbook: 'myroles.yml'
     }    
    }
  }
 }
 
   
 }
}