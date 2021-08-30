# Getting Started

## Step 1: Log into AWS

Your mentor will provide instructions to log into the AWS console.

## Step 2: Create a DynamoDB table for catalog records

In this step, you will create a new table in [AWS DynamoDB](DynamoDB) named *Catalog*. Each table requires a Primary
Key that is used to partition data across DynamoDB servers. A table can also have a Sort Key. The
combination of Primary Key and Sort Key uniquely identifies each item in a DynamoDB table.

1. In the AWS Management Console, click **Services**, then click **DynamoDB**.

1. Click **Create table**.

1. For **Table name**, type: `Catalog`.

1. For **Primary key**, type `titleID` and leave **String** selected.

1. for **Sort key**, type `availabilityStart` and leave **String** selected.

    Your table will use default settings for indexes and provisioned capacity.

1. Click **Create table**.
    The table will be created in less than a minute.

## Step 3: Create an AWS Lambda Function to copy catalog records into DynamoDB

In this step, you will create an [AWS Lambda](Lambda) function that reads the catalog dataset into your
DynamoDB table. You will run the Lambda function once to load your initial catalog database. Feel
free to alter it and rerun as needed if you need to recreate your database.

1. On the **Services** menu, click **Lambda**.

    :warning: Do not change the Region. You must us **US East (Virginia)** for this hackathon

1. Click **Create function**.

    :information_source:  Blueprints are code templates for writing Lambda functions. For example, blueprints are
   provided for standard Lambda triggers such processing DynamoDB update streams. This lab provides
   you with a pre-written Lambda function, so you will **Author from scratch**.

1. Choose **Author from scratch**.

1. In the **Create function** window, configure:

    * Function name: `Load-Database`
    * Runtime: *Python 3.9*

        :warning: Make sure to select Python 3.9. If you select another version of Python, the lambda could
    fail.

    * Expand **▸ Change default execution role**
        * Execution role: Select **Use an existing role**
        * Existing role: Select **TeamRole**

1. Click **Create function**.

    A page will be displayed with your function configuration.

    :information_source: Take note how AWS Lambda functions can be triggered using **+ Add
    Trigger**. You will be manually invoking this lambda to initialize your database, but
    you may want to utilize triggered lambdas later on in your hackathon solution.

1. Select the **Configuration** tab.

1. Click **Edit**.

1. In the **Basic settings** window, configure:
    * Timeout: `2` min `0` sec

1. Click **Save**.

1. Select the **Code** tab.

1. Open `lambda_function.py`

1. Copy the lambda code below and paste it into the editor:

    ```python
    import csv
    import io
    import urllib.request
    import boto3

    url = 'https://raw.githubusercontent.com/kdelaney/ncat_hackathon_2021/main/catalog.tsv'

    def lambda_handler(event, context):
        print('Downloading catalog csv')
        csv_data = urllib.request.urlopen(url)
        items = csv.DictReader(io.TextIOWrapper(csv_data), delimiter='\t')

        print('Loading data into DynamoDB table: Catalog')
        dynamodb = boto3.resource('dynamodb')
        table = dynamodb.Table('Catalog')

        record_count = 0
        with table.batch_writer() as batch: 
            for item in items:
                record_count += 1
                batch.put_item(Item=item)
        return f'Successfully loaded {record_count} records into catalog database'
    ```

    :information_source: Examine the above code. It is performing the following steps:
    * Receives an Event and Context (both are ignored)
    * Downloads the dataset
    * Reads the CSV
    * Puts the data into DynamoDB using [boto3](boto3) (the AWS SDK for Python).

1. Click **Deploy**.

    :information_source: Every time you make a change, you need to **Deploy** before you can **Test**.

1. Click **Test**.

1. For **Event name**, type `TestEvent`.

    :information_source: Usually lambdas handle some sort of `Event`, such as a message from
    [SNS](SNS) or [SQS](SQS) or an update from [S3](S3) or [DynamoDB](DynamoDB).
    In this case, we just invoke the lambda once, ignoring the event.

1. Click **Create**.

1. Click **Test**.

    It will take approximately 45 seconds to load the data into your table. 

    When the lambda execution is complete, you will see the **Execution result**:

    ```console
    Test Event Name
    TestEvent

    Response
    "Successfully loaded 1172 records into catalog database"

    Function Logs
    START RequestId: 02c15768-df09-4f3a-df09-a9e02be638b7 Version: $LATEST
    Downloading catalog csv
    Loading data into DynamoDB table: Catalog
    Loaded 999 records into catalog database
    END RequestId: 02c15768-df09-4f3a-df09-a9e02be638b7
    REPORT RequestId: 02c15768-df09-4f3a-df09-a9e02be638b7	Duration: 31088.51 ms ...

    Request ID
    02c15768-df09-4f3a-df09-a9e02be638b7
    ```

1. In the AWS Management Console, click **Services**, then click **DynamoDB**.

1. Click **Tables**.

1. Click **Catalog**.

1. Click **View Items**

    :tada: Your data is now in the `Catalog` table.

## Step 4: Hack

Now you are ready to start hacking! Some questions to ponder:

* What mechanism will you use to simulate a catalog record becoming available?
* What is the best way to notify customers?
* Does your solution show customer obsession?
  * Does it spark action and ignite interest?
  * Does it serve all customers and communities?
  * Is it simple, personal, delightful?
  * Does it contribute to an immersive entertainment experience?
* What other customer problems can you solve using this data?
  What can you invent and simplify? How else can you think big?

## Resources

* [boto3](boto3)
* [S3](S3)
* [DynamoDB](DynamoDB)
* [Lambda](Lambda)
* [SNS](SNS)
* [SQS](SQS)

[boto3]: https://boto3.amazonaws.com/v1/documentation/api/latest/index.httml
[S3]: https://docs.aws.amazon.com/AmazonS3/latest/userguide/GetStartedWithS3.html
[DynamoDB]:
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GettingStartedDynamoDB.html
[Lambda]: https://docs.aws.amazon.com/lambda/latest/dg/getting-started.html
[SNS]: https://docs.aws.amazon.com/sns/latest/dg/sns-getting-started.html
[SQS]:
https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-getting-started.html

Dataset information courtesy of IMDb (http://www.imdb.com).
Used with permission.
