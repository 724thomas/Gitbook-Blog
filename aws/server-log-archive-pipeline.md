---
description: 서버 로그 아카이빙 파이프라인
---

# Server Log Archive Pipeline

## 파이프라인:

<figure><img src="../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>



***



<figure><img src="../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

### 1. AWS CloudWatch

AWS 리소스와 AWS에서 실시간으로 실행중인 애플리케이션을 모니터링하는 서비스를 제공해주는 AWS CloudWatch를 사용중입니다. 지표를 감시해 알림을 보내거나, 임계값을 위반한 경우 모니터링 중인 리소스를 자동으로 변경하는 경보를 생성할 수 있습니다.(예: 인스턴스 중지, Auto Scaling 등).

현재 저희 서비스에서는 대시보드를 사용하고 있지는 않지만, 로그를 수집하는데 사용하고 있습니다. 애플리케이션에서 발생하는 로그를 모두 CloudWatch에 저장하고 있는데, 해당 로그는 추후 문제가 생기거나 분석을 위해 사용될 예정입니다.

이러한 수집하고 있는 로그 데이터의 목적은 지금 당장 필요하지도 않을 뿐더러, 나중에도 자주 사용되지 않기때문에, 비용을 줄이기 위해 오래된 데이터는 다른 저장소로 옮기는 작업을 하기로 했습니다.

CloudWatch에서 어디론가 데이터를 옮긴다고 가정했을때, 기존 데이터는 CloudWatch에 남아있을 필요가 없습니다. 저희는 13일전 로그 데이터들만 일일 단위로 저장을 할 예정이라서, 2주가 지난 데이터는 자동 삭제 조치를 시켰습니다.





***



<figure><img src="../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

### 2. AWS S3

로그를 아카이빙을 하기 위한 저장소들의 후보로 S3와 Glacier가 있었지만, 현 시점에서 가장 적합한 저장소로 S3를 사용하기로 했습니다.

#### S3와 Glacier 차이:

**Glacier**

* S3 대비 가격이 1/3 정도된다. 이는 굉장히 저렴한 편으로, 말 그대로 데이터 **빙하** 를 위한 서비스입니다.
* 마음대로 검색이 불가능합니다. 완전히 저장을 목적으로 하는 서비스 이므로, 검색을 하기 위해서는 매일 주어지는 저장 용량의 5%에 대한 무료 검색이 가능하고(초과시 과금), 삭제도 저장 후 3개월 이상이 되어야합니다

**S3**

* 반면 S3는 가격이 Glacier에 3배입니다.
* 검색을 빠르게 할 수 있으며 데이터 복구와 다운로드가 거의 즉각적으로 가능합니다.

저희는 현재 출시를 앞두고 있는 상황에서 초기 발생 오류들에 대한 빠른 대처가 필요하기때문에 지금 당장 Glacier가 필요하다고 판단하지 않았습니다. 추가로, 가격이 3배 차이가 나더라도, 로그 저장 공간이 많이 필요하지 않을 걸로 예상하였습니다. 추후에 아주 아주 가끔 봐야하는 데이터들에 대해서 아카이빙을 할때에는 Glacier를 사용하겠지만, 지금은 필요가 없다고 판단하였습니다.

결정된 S3 저장소에 log를 저장하는 bucket을 생성하였습니다.





***



<figure><img src="../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

### 3. AWS Lambda

AWS 람다는 파이썬/자바스크립트로 코드를 작성할 수 있습니다. 저는 편의를 위해서 익숙한 python을 사용했습니다. 다른 lambda 함수들은 js로 작성되었지만, 코드가 복잡하지 않기 때문에 파이썬으로 진행했습니다.

람다를 통해 CloudWatch Logs에서 S3로 저장하는 과정에서, IAM을 통해서 권한 설정이 필요했습니다.

<figure><img src="../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

CloudWatch Logs에 Read, Write,

S3에 Write 권한을 부여해줬습니다.

스크립트:

```python
from datetime import datetime, timedelta
import boto3
import json
import os

# Function to calculate the time range for 13 days ago
# Logs in CloudWatch are deleted 2 weeks after
def get_thirteen_days_ago_time_range():
    target_date = datetime.now() - timedelta(days=13)
    start_time = datetime(target_date.year, target_date.month, target_date.day)
    end_time = start_time + timedelta(days=1)
    return int(start_time.timestamp() * 1000), int(end_time.timestamp() * 1000)

# Function to upload a file from Lambda's ephemeral storage to S3
def upload_to_s3(bucket, key, file_path):
    s3 = boto3.client('s3')
    with open(file_path, 'rb') as file:
        s3.upload_fileobj(file, Bucket=bucket, Key=key) 
    os.remove(file_path)  # Clean up the file from /tmp after uploading to S3

def lambda_handler(event, context):
    logs = boto3.client('logs')

    
    start_time, end_time = get_thirteen_days_ago_time_range() # Get the time range for 13 days ago
    target_date_str = (datetime.now() - timedelta(days=13)).strftime('%y%m%d')
    bucket_name = 'some-bucket-name'
    
    log_group_prefixes = ['/aws/lambda/', '/ecs/'] 
    for prefix in log_group_prefixes: # Loop through specified log group prefixes
        response = logs.describe_log_groups(logGroupNamePrefix=prefix) # Retrieve log groups that match the prefix
        
        for log_group in response['logGroups']: # Process each log group
            log_group_name = log_group['logGroupName']
            filter_response = logs.filter_log_events( # Retrieve log events for the log group within the time range
                logGroupName=log_group_name,
                startTime=start_time,
                endTime=end_time
            )
            
            tmp_file_path = f"/tmp/{log_group_name.replace('/', '_')}_{target_date_str}.txt" # Create a path for a temporary file in Lambda's ephemeral storage
            with open(tmp_file_path, 'w') as tmp_file: # creates file automatically if not exists
                for event in filter_response['events']: 
                    tmp_file.write(event['message'] + '\n')

            s3_key = f"{log_group_name.replace('/', '_')}/{target_date_str}.txt" # Construct the S3 key (file path in S3 bucket)
            upload_to_s3(bucket_name, s3_key, tmp_file_path) # Upload the file from ephemeral storage to S3

    return {
        'statusCode': 200,
        'body': json.dumps('Log data uploaded to S3 for all groups')
    }

```

해당 스크립트를 Deploy를 해주고, Test를 통해 잘 작동하는지 확인을 해봤습니다.



<figure><img src="../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

> Memory와 Ephemeral Storage

Lambda 함수의 메모리 사용량과 Ephemeral Storage(임시 저장소) 용량은 다른 개념입니다. Lambda 함수의 메모리 설정은 함수 실행에 사용되는 RAM의 양을 결정합니다. 반면, Ephemeral Storage는 함수가 실행 중에 임시 파일을 저장하는 데 사용되는 디스크 공간입니다.

메모리 사용량이 설정된 한계치에 근접하고 있다면, 메모리를 늘려야 할 수도 있습니다. 그러나 메모리 사용량이 현재 100MB 정도이고 설정된 메모리 한계가 128MB라면, 메모리가 부족하다고 판단하기에는 조금 이른 상황일 수 있습니다. 메모리 사용량은 함수 실행 중의 최대 사용량을 나타내며, 이는 실행마다 변할 수 있습니다.

#### **메모리 사용량이 높은 경우 해결 방법:**

1. **메모리 설정 증가:** Lambda 함수의 메모리 설정을 늘려 함수에 더 많은 RAM을 할당합니다. 이는 함수의 CPU 성능에도 영향을 줄 수 있습니다.
2. **코드 최적화:** 메모리 사용을 줄이기 위해 코드를 최적화합니다. 예를 들어, 대량의 데이터를 처리할 때 메모리에 한 번에 많은 양의 데이터를 로드하지 않도록 주의합니다.

#### **Ephemeral Storage 확장:**

* Ephemeral Storage는 함수가 실행 중에 임시 파일을 저장하는 데 사용됩니다. 로그 데이터를 파일로 저장하거나 큰 파일을 처리하는 경우에 필요할 수 있습니다.
* 만약 함수가 임시 파일을 많이 생성하거나 큰 파일을 다루는 경우, Ephemeral Storage의 크기를 늘릴 수 있습니다.

#### **결론:**

* 메모리 사용량이 설정값에 근접한다면 메모리를 늘리는 것을 고려해야 합니다. 함수의 메모리 사용량을 모니터링하여 필요에 따라 메모리 설정을 조정하세요.
* 임시 파일 저장이 문제가 되지 않는 한, Ephemeral Storage를 늘릴 필요는 없을 수 있습니다. Ephemeral Storage는 주로 임시 파일 저장에 관련된 문제를 해결하는 데 사용됩니다.



***



<figure><img src="../.gitbook/assets/image (168).png" alt=""><figcaption></figcaption></figure>

### 4. AWS EventBridge

Aws EventBridge는 **이벤트**를 사용하여 애플리케이션 구성 요소를 서로 연결하는 서버리스 서비스입니다. 확장 가능한 이벤트 기반 애플리케이션을 쉽게 구축할 수 있습니다.

저는 AWS EventBridge에서 Schedules를 사용하였습니다. Schedules는 Cron같은 기능을 제공하여, 특정 시간에 이벤트를 발생시켜줍니다.

<figure><img src="../.gitbook/assets/image (169).png" alt=""><figcaption></figcaption></figure>

Rate-based schedule을 사용하여 매일마다(해당 사진은 7일입니다만..) 작동하게 만들었습니다. TimeZone까지 설정할 수 있습니다.

Target으로는 AWS Lambda Invoke를 통해, 작성해두었던 Lambda 함수를 설정하였습니다.





***



## AWS Lambda 실행에 대한 고찰.

하루에는 수 많은 로그 파일이 생성될 수 있습니다. 각 로그 파일은 크기에 따라 나눠지기도 하기때문에, S3에 저장하게 될때는 하나로 합치는(해당 날짜 일단위) 작업을 추가적으로 해야합니다.

이를 위해 단순히 로그 파일들을 불러와서 하나의 큰 파일을 생성할 수 있지만, 로그 파일의 크기(갯수)가 가용 RAM을 초과하게되면 Out of Memory를 겪게됩니다.

예를 들어, 오늘 발생한 로그파일들이 1MB 크기의 400개가 있고, Lambda 함수에 할당된 RAM이 256MB일때, 로그파일들을 올리게 되면 RAM의 최대 크기 256MB를 초과하게 됨으로써 Out of Memory가 발생합니다.

이를 방지하기 위해서는 여러 방법이 있습니다.

#### 1. RAM 크기 조정

단순히 Lambda의 Configuration에 들어가서 Memory를 늘려주는 것이다. 최대 10240MB까지 늘려줄 수 있습니다. _**하지만**_ 만약 총 로그 파일들의 크기가 10240MB가 넘어간다면…?

#### 2. ~~Append를 사용하여 끊어서 저장.~~ S3에 저장된 객체는 Immutable…

위 방법으로 문제가 발생했을시 고려할 수 있는 두번째 방법입니다. 최종 로그 파일을 계속 RAM에 올려두지 않고, S3에 저장해놓고, 그 뒤에 append를 사용하여 각 파일들만 RAM에 올려서 뒤에 붙이는 방법을 사용할 수 있다. _**하지만**_ 하나의 로그파일 크기가 10240MB가 넘어간다면…?

#### 3. 합치지 않고 나눠진채로 저장.

위 방법들로는 해결이 안되는 상황이라면, 나눠져있는 로그 파일들을 단순히 S3로 옮기는, 단순히 목적만 달성하는 작업을 할 수 있습니다. 이 작업의 목적은 분석이 아닌, 비용이 더 싼 저장소로 옮기는 작업이기 때문에, 이 방법도 (최후의) 해결책이 될 수 있습니다.

#### 4. Ephemeral Storage를 활용

Lambda는 기본적으로 메모리와 Ephemeral Storage를 사용할 수 있습니다. Ephemeral Storage는 Lambda가 실행되면서 사용할 수 있는 임시 공간입니다. S3는 immutable이기때문에, Lambda의 ephemeral Storage에 이어 붙이는 방식을 사용합니다. 각 로그파일을 한줄씩 memory에 올리고, tmp 파일에 append하는 식으로 작업을 하고, 마지막으로 tmp파일을 S3로 저장하는 방식입니다.

**(최종적으로 S3에 저장할때 tmp파일이 메모리에 올라가게 되지 않을까 걱정을 했는데, AWS SDKs의 boto3를 사용하면, 내부적으로 파일을 chunk로 나누어 보내게 되고, 최종적으로 S3에서 하나의 object로 합치는 작업을 한다고 합니다.)**



### 결론.

해결책으로 저는 4번을 선택했습니다. 현 상황에서 가장 합리적인 방법이라고 생각이 들기때문입니다. 미래에 로그 파일이 커질때, memory가 아닌 ephemeral storage용량을 늘리면 되고, 10GB가 넘어가는 경우에는 그때 1번의 RAM 크기도 같이 조정하려고합니다. 만약, 20GB가 넘어간다면, 그때는 합치지 않고 나눠진채로 저장하는 방식을 사용해도 좋을 것 같습니다.
