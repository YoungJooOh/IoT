# IoT 클라우드 기말고사 프로젝트

## 식기세척기
### 식기세척기의 문을 열고 식기를 넣고 다시 닫으면 식기를 세척한다. 내부 고장시에 사용자 또는 관리자에게 알려준다.

## 기능설명
- 어플리케이션에서 등록된 식기세척기를 조회할 수 있다.
- 식기세척기의 내부 상태를 모니터링하기 위한 기계 작동 상태 및 내부 환경 정보를 DB에 저장한다.
- 내부 기계가 고장이 나면 사용자에게 SNS서비스를 제공한다.

## 구조도
![캡처](https://user-images.githubusercontent.com/31908591/102013176-df400880-3d91-11eb-9676-6d92ccf918d2.JPG)

## 람다 함수
DynamoDBLambdaJavaProject - 새로운 식기세척기 등록시 DB에 저장한다
ListingDeviceLambdaJavaProject - 현재 등록된 식기세척기들의 정보를 불러온다
LogDeviceLambdaJavaProject - 디바이스 내부 상태 로그값을 불러온다
MonitoringLambda - 내부 고장(워터펌프)가 고장이 나면 SNS기능을 이용하여 사용자의 이메일로 알려준다
RecordingDeviceDataJavaProject2 - 워터펌프가 고장나거나 수리가 된 경우에 해당 로그 값만을 DB에 저장한다.
RecordingDeviceDataLambdaJavaProject - 전체 로그값을 DB에 저장한다
UpdateDeviceLambdaJavaProject - 앱에서 디바이스를 제어할 수 있다. 

```javascript

package com.amazonaws.lambda.demo;

import com.amazonaws.services.dynamodbv2.AmazonDynamoDB;
import com.amazonaws.services.dynamodbv2.AmazonDynamoDBClientBuilder;
import com.amazonaws.services.dynamodbv2.document.DynamoDB;
import com.amazonaws.services.dynamodbv2.document.Item;
import com.amazonaws.services.dynamodbv2.document.PutItemOutcome;
import com.amazonaws.services.dynamodbv2.document.spec.PutItemSpec;
import com.amazonaws.services.dynamodbv2.model.ConditionalCheckFailedException;
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;

public class PuttingPersonHandler implements RequestHandler<Person, String> {
    private DynamoDB dynamoDb;
    private String TABLE_NAME = "DishWasher";
    private String REGION = "ap-northeast-2";

    @Override
    public String handleRequest(Person input, Context context) {
        this.initDynamoDbClient();

        putData(input);
        return "Saved Successfully!!";
    }

    private void initDynamoDbClient() {
        AmazonDynamoDB client = AmazonDynamoDBClientBuilder.standard()
                .withRegion(REGION).build();
         this.dynamoDb = new DynamoDB(client);
    }

    private PutItemOutcome putData(Person person) 
              throws ConditionalCheckFailedException {
                return this.dynamoDb.getTable(TABLE_NAME)
                  .putItem(
                    new PutItemSpec().withItem(new Item()
                            .withPrimaryKey("id",person.id)
                            .withString("Region", person.Region)
                            .withString("lastName", person.lastName)));
            }
}

class Person {
    public String Region;
    public String lastName;
    public int id;
}
'''
