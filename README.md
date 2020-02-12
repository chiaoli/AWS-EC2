# AWS EC2
### EC2 (Elastic Compute Cloud)
EC2 為 Amazon 所提供的線上虛擬機器，透過使用者金鑰可從本機端連線至 Amazon 線上服務的虛擬機中進行各式操作，在虛擬機的作業系統部分，可透過 Amzon 所提供的 AMI （如 Windows, Ubuntu, Red Hat等）選擇偏好的作業系統，並為這些虛擬機配置自己所需的服務。
在本節內容中主要會講述如何建立一個基礎的 VPC（Virtual Private Cloud, 虛擬私有雲端）並分別使用 AWS console 及 AWS CLI 建立並啟動 EC2 並透過使用 IAM 所建立的使用者存取金鑰連線至 EC2 之中。
### 本節目標：
* 建立 VPC 環境供 EC2 使用
* 使用 AWS console 建立並啟動 EC2
* 使用 AWS CLI 建立並啟動 EC2
* 於 EC2 上安裝 Docker、Docker-compose
* 使用 Docker 建立 nginx 容器
* 終止 EC2
## 1.建立 VPC
VPC 是 EC2 的 Networking layer，可透過 VPC 建立一個私有或公開的網路，在本節會簡略帶過 VPC 的概念，建立一個基礎且可公開存取的 VPC 供 EC2 使用。

首先點選 **服務->VPC** 來到 VPC 主畫面。
![](https://i.imgur.com/zE0Il30.png)

點選 **Launch VPC Wizard** 按鈕
![](https://i.imgur.com/mMj00Oa.png)

這邊 AWS 提供一些常見的網路架構範本，我們選擇建立單一公開子網路的 VPC
![](https://i.imgur.com/rQ6fd9C.png)

使用預設值設定並命名 EC2-tutorial 建立點選下一步即可
![](https://i.imgur.com/QpCD9oe.png)


## 2.使用 AWS console 建立並啟動 EC2
點選 **服務->EC2** 來到 EC2 主畫面
![](https://i.imgur.com/dqs0Efl.png)

在主畫面中點選 **Launch Instance** 來快速啟動一台 EC2 Instance
![](https://i.imgur.com/bmIjn6h.png)

Step 1 為選用 AMI（Amazon Machine Image），AMI 為 EC2 上所使用的系統映像檔，AWS 提供多種作業系統可供選擇，這邊我們選用 Amazon Luunx 2 AMI 為範例，需注意免費而度帳戶只能使用下方標有 **Free tier eligible** 標籤之 AMI，其他如畫面中 Amazon RDS 則是使用後需要依照其收費規則進行收費的 AMI。
![](https://i.imgur.com/8aKjzn7.png)

在這個 Step 2 畫面可看到 AWS 提供多種不同規格的資源可進行選用，未來可依照自身需求搭配不同需求的資源（如 CPU，Memory，Storage，Network Performance等），這邊我們一樣選用預設 **Free tier eligible** 標籤的方案。
![](https://i.imgur.com/0sIT0gQ.png)

接下來進行 Step 3 網路環境的設定，如下圖所示，選用剛才建立的 **EC2-tutorial** 的 VPC，需注意的是在紅匡處的 **Auto-assign Public IP** 選用 **Enable**。
![](https://i.imgur.com/pPxHJab.png)

Step 4 及 Step 5 分別為 Storage 及 Tag 的設定，這邊先用預設值設定即可，在 Step 6 的 Security Group 的設定有點像是為這台 EC2 Instance 設定防火牆規則，這邊的預設範本為啟用 SSH 的 22 Port 並設為所有人可公開存取，若需要限制可連線的來源可於 **Source** 欄位進行設定。我們使用預設值並將此 Security Group 命名為 **EC2-tutorial-sg**
![](https://i.imgur.com/E4AkcXy.png)

最後的 Step 7 則是進行前述所有步驟的 Review，點選確認後即會開始建立一台 EC2 Instance
![](https://i.imgur.com/SYzWHkG.png)

點選後會看到出現如下圖的提示要求選擇這台 EC2 Instance 所要使用的金鑰對，這邊我們選擇建立一組新的金鑰對並命名為 **ec2-tutorial-key** 並下載到本機端。
需注意這組金鑰對與 IAM 那邊所設定的金鑰對並不同，現在所設定的是專門用於 SSH 登入 EC2 時所使用的金鑰，而 IAM 所設定的是讓我們能夠透過 AWS CLI 進行所有 AWS 上所有服務的操作。
![](https://i.imgur.com/AHFZ4y3.png)

在 EC2 Instance 畫面中可看到 EC2 已成功建立並執行起來，點選 **Connect** 按鈕。
![](https://i.imgur.com/C1ON9QY.png)

點選後會出現如下畫面，AWS 提供三種登入 EC2 的方式，最常會使用的為第一及第三種；下圖為使用 SSH client 方式連線至 EC2 的說明。將前述步驟的金鑰下載下來後修改為適當的權限後即可透過指令連線至 EC2 之中。
![](https://i.imgur.com/LhrNbof.png)

另外一種為 Browser-based SSH，會在新分頁開啟一個 EC2 內的 CLI 畫面。
![](https://i.imgur.com/nzZ0FLu.png)

以上步驟就是透過 AWS console 建立一台 EC2 Instance 的流程。

## 3.使用 AWS CLI 建立並啟動 EC2
使用 AWS CLI 建立一台 EC2 Instance 需搭配在 IAM 章節所建立的**使用者存取金鑰**，可透過指令`aws configure list`查看目前使用的存取金鑰訊息，而用 AWS CLI 的方式建立也相對來說會更困難一點，我們必須事先建立好 VPC、Security Group，Instance 金鑰對等內容，而在指定 AMI 的放式也是需要知道 AMI 所對應的 ID 才行。
### 建立金鑰對
若要建立金鑰對，請使用 create-key-pair 命令搭配 `--query` 選項和 `--output text` 選項，將您的私有金鑰直接輸送到檔案中。
```
$ aws ec2 create-key-pair --key-name <KEYPAIR_NAME_HERE> --query 'KeyMaterial' --output text > <KEYPAIR_NAME_HERE>.pem
```
範例
```
$ aws ec2 create-key-pair --key-name ec2-tutorial-key --query 'KeyMaterial' --output text > ec2-tutorial-key.pem
```
顯示目前 AWS 上已存在的金鑰對
```
$ aws ec2 describe-key-pairs
```
### 建立 EC2
建立 EC2 的 AWS CLI 命令結構如下：
```
$ aws ec2 run-instances --image-id <AMI-ID_HERE> --count 1 --instance-type t2.micro --key-name <KEY_PAIR_HERE> --security-group-ids <SG-ID_HERE> --subnet-id <SUBNET-ID_HERE>
```
範例：
```
$ aws ec2 run-instances --image-id ami-011facbea5ec0363b --count 1 --instance-type t2.micro --key-name ec2-tutorial-key --security-group-ids sg-095edc699c066d2ea --subnet-id subnet-0a560ad7f6257e180
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-011facbea5ec0363b",
            "InstanceId": "i-0c0bc5c8b78661cef",
            "InstanceType": "t2.micro",
            "KeyName": "ec2-tutorial-key",
            "LaunchTime": "2020-02-12T08:25:44.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "ap-northeast-1a",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-10-0-0-26.ap-northeast-1.compute.internal",
            "PrivateIpAddress": "10.0.0.26",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0a560ad7f6257e180",
            "VpcId": "vpc-079c1e0df29ee1405",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "",
            "EbsOptimized": false,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2020-02-12T08:25:44.000Z",
                        "AttachmentId": "eni-attach-09503629bd58d9bbc",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "EC2-tutorial-sg",
                            "GroupId": "sg-095edc699c066d2ea"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "06:55:0e:c9:b7:a8",
                    "NetworkInterfaceId": "eni-04cf8dc185b8dbb95",
                    "OwnerId": "727752420676",
                    "PrivateDnsName": "ip-10-0-0-26.ap-northeast-1.compute.internal",
                    "PrivateIpAddress": "10.0.0.26",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-10-0-0-26.ap-northeast-1.compute.internal",
                            "PrivateIpAddress": "10.0.0.26"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0a560ad7f6257e180",
                    "VpcId": "vpc-079c1e0df29ee1405",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "EC2-tutorial-sg",
                    "GroupId": "sg-095edc699c066d2ea"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            }
        }
    ],
    "OwnerId": "727752420676",
    "ReservationId": "r-0c56d05c95f27559f"
}
```
成功建立後會顯示這台 EC2 Instance 相關的設定參數於畫面上，使用 AWS CLI 建立 EC2 雖然相對來說會比較不親民，但這種方式對於 EC2 的設定了解卻有很大的幫助，在未來如果有需要用 Terraform 進行 EC2 的管理會更容易。

### 其他指令：
列出目前 EC2 Instance
```
$ aws ec2 describe-instances
```

## 4.於 EC2 上安裝 Docker、Docker-compose
### 安裝 Docker
上述步驟建立好後透過 SSH client 連線至 EC2 Instance 之中。
更新 EC2 系統內的套件資料庫。
```
$ sudo yum update -y
```
安裝最新的 Docker Community Edition 套件。
```
$ sudo amazon-linux-extras install docker
```
啟動 Docker 服務。
```
$ sudo service docker start
```
將 ec2-user 新增至 docker 群組，讓我們可以在不使用 sudo 的情況下執行 Docker 命令。
```
$ sudo usermod -a -G docker ec2-user
```
登出並重新登入，以取得新的 docker 群組許可。關閉目前的 SSH 終端機視窗，即可完成此操作，並在新的 SSH 終端機視窗中重新連接至執行個體。新的 SSH 工作階段將會有適當的 docker 群組許可。

驗證 ec2-user 可以在不使用 sudo 的情況下執行 Docker 命令。
```
$ docker info
```
### 安裝 Docker-compose
透過下列指令下載目前 Docker-compose release 版本
```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
新增可執行權限給 Docker-compse binary 檔
```
$ sudo chmod +x /usr/local/bin/docker-compose
```
建立 docker-compose 指令連結
```
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```
確認 docker-compose 指令版本
```
$ docker-compose --version
```
## 5.使用 Docker 建立 nginx 容器
完成安裝 Docker 後，執行下列指令將 EC2 port 80 與 nginx 容器 port 80 進行綁定並命名容器為 webserver，Docker 會從 Docker Hub 下載 nginx 映像檔並執行。
```
$ docker run -d -p 80:80 --name webserver nginx
```

從 EC2 頁面中可得知現在執行的 EC2 Instance 的 Public IP 以及 DNS 域名。
![](https://i.imgur.com/l9S7Jgy.png)

在瀏覽器上輸入此域名，會發現瀏覽器顯示無法連線到該網站，原因為我們先前設定的 Security Group 只允許 SSH 的連線，因此我們需要針對 Security Group 進行修改。
![](https://i.imgur.com/51dQfoV.png)

點選 EC2 畫面中 Security Group 的分頁並選擇我們建立的 **EC2-tutorial-sg
** 進行修改。
![](https://i.imgur.com/W9h9fP4.png)

點選 **Add Rule** 按鈕並新增 HTTP 規則並儲存。
![](https://i.imgur.com/sN1pT0R.png)

開啟瀏覽器重新連線到該域名會發現可以成功看到在 EC2 中啟動的 nginx 容器。
![](https://i.imgur.com/xFWz1ut.png)

## 6.終止 EC2
### 使用 AWS console 終止 EC2
在 EC2 Instance 頁面中點選 **Actions->Instance State->Terminate** 關閉並終止 EC2，會將 EC2 的內容完全移除。
![](https://i.imgur.com/6yDHwm2.png)
### 使用 AWS CLI 終止 EC2
若使用 AWS CLI 需知道該 EC2 Instance 的 ID，查詢方式可透過 EC2 Instance 頁面或是 AWS CLI 指令 `aws ec2 describe-instances`或是`aws ec2 describe-instances --query "Reservations[].Instances[].InstanceId"`只顯示出 Instance ID 的方式。
![](https://i.imgur.com/RSdSa97.png)
終止命令結構如下：
```
$ aws ec2 terminate-instances --instance-ids <INSTANCE_ID_HERE>
```
範例：
```
$ aws ec2 terminate-instances --instance-ids i-016d9e02052843aaf
{
    "TerminatingInstances": [
        {
            "CurrentState": {
                "Code": 32,
                "Name": "shutting-down"
            },
            "InstanceId": "i-016d9e02052843aaf",
            "PreviousState": {
                "Code": 16,
                "Name": "running"
            }
        }
    ]
}
```

## Reference
[AWS Documentation](https://docs.aws.amazon.com/)
[AWS CLI Documentation](https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-services-ec2.html)
[Installing Docker on Amazon Linux 2
](https://docs.aws.amazon.com/zh_tw/AmazonECS/latest/developerguide/docker-basics.html)
## License
MIT license

