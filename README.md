## RDS Snapshot Export to S3 Pipeline

This repository creates the automation necessary to export Amazon RDS snapshots to S3 for a specific database whenever a snapshot is created, 
whether created by an automated snapshot, manual, or by AWS Backup service.

## Usage

1. Install the [Amazon Cloud Development Kit](https://aws.amazon.com/cdk/) (CDK).
2. Clone this repository and `cd` into it.
3. Modify the arguments to the `RdsSnapshotExportPipelineStack` constructor in `$/bin/cdk.ts` according to your environment.
    * `dbName`: This RDS database must already exist.
    * `rdsEvents`: This should be indicate the RDS event ID and corresponsing snapshot type, where:
      * `rdsEventId` should be:
        * `RdsEventId.DB_AUTOMATED_AURORA_SNAPSHOT_CREATED` for Amazon Aurora databases
        * `RdsEventId.DB_AUTOMATED_SNAPSHOT_CREATED` for RDS automated snapshots
        * `RdsEventId.DB_MANUAL_SNAPSHOT_CREATED` for AWS Backup service or otherwise.
        * `RdsEventId.DB_BACKUP_SNAPSHOT_FINISHED_COPY` for AWS Backup service snapshots, created shortly after a prior snapshot has been taken.
      * `rdsSnapshotType` should be:
        * `RdsSnapshotType.DB_AUTOMATED_SNAPSHOT` for Automated snapshots or 
        * `RdsSnapshotType.DB_BACKUP_SNAPSHOT`for Backup service snapshots or
        * `RdsSnapshotType.DB_MANUAL_SNAPSHOT`for manual snapshots.
    * `s3BucketName`: An S3 bucket with the provided name will be created automatically for you.

    For example, the following configuration will automatically export all snapshots and snapshot-copies, created by both the Automated service 
    and by AWS Backup of an existing RDS database named `my-rds-db`, to a new S3 bucket named `my-rds-db-snapshots-export`:
    ```
    dbName: 'my-rds-db',
    rdsEvents: [
      {
        rdsEventId: RdsEventId.DB_AUTOMATED_SNAPSHOT_CREATED,
        rdsSnapshotType: RdsSnapshotType.DB_AUTOMATED_SNAPSHOT
      },
      {
        rdsEventId: RdsEventId.DB_MANUAL_SNAPSHOT_CREATED,
        rdsSnapshotType: RdsSnapshotType.DB_BACKUP_SNAPSHOT
      },
      {
        rdsEventId: RdsEventId.DB_BACKUP_SNAPSHOT_FINISHED_COPY,
        rdsSnapshotType: RdsSnapshotType.DB_BACKUP_SNAPSHOT
      }
    ],
    s3BucketName: 'my-rds-db-snapshots-export'
    ```
4. Execute the following:
    * `npm install`
    * `npm run cdk bootstrap`
    * `npm run cdk deploy`
5. Open up your `<dbName>-rds-snapshot-exporter` function in the [AWS Lambda](https://console.aws.amazon.com/lambda/home) console and configure a test event using the contents of [$/event.json](./event.json) OR [$/event.aurora.json](./event.aurora.json) as a template, depending on whether or not you're using Amazon Aurora.
    * **NOTE:** The example content is a *subset* of an SNS event notification containing the minimum valid event data necessary to successfully trigger the Lambda function's execution. You should modify the `<SNAPSHOT_NAME>` value within the `Message` key to match an existing RDS snapshot (e.g. `rds:<dbName>-YYYY-MM-DD-hh-mm`). You may also need to modify the `MessageId` if you are attempting to export the same snapshot more than once.
6. Click the **Test** button to start an export.

You can check on the progress of the export in the [Exports in Amazon S3](https://console.aws.amazon.com/rds/home#snapshots-list:tab=exporttos3) listing. When that is finished, you can use the [AWS Glue Crawler](https://console.aws.amazon.com/glue/home#catalog:tab=crawlers) that was created for you to crawl the export, then use [Amazon Athena](https://console.aws.amazon.com/athena/home#query) to perform queries on the exported snapshot.

## Cleanup

Execute `npm run cdk destroy` to delete resources pertaining to this example.

You will also need to delete the following manually:
   * The S3 bucket that was created to store the snapshot exports.
   * The [CDKToolkit CloudFormation Stack](https://console.aws.amazon.com/cloudformation/home#/stacks?filteringText=CDKToolkit) created by `npm run cdk bootstrap`.
   * The `cdktoolkit-stagingbucket-<...>` bucket.

## Demo

[![Demo](.github/demo-video.png)](https://www.youtube.com/watch?v=lyNGeDg6EII)

## License

This library is licensed under the MIT-0 License. See the [LICENSE](./LICENSE) file.
