+++
title = "CDK でアカウントセキュリティ設定"
date = "2026-03-14"
description = "デフォルトにしておいてほしい設定"
[taxonomies]
tags = ["cdk", "aws", "typescript"]
+++

デフォルトがセキュアじゃない。必要になったら開けるからデフォルトはセキュアにしておいてくれよ・・・と思わなくもない。加えて、この辺の設定、CDKだとカスタムリソースでないと設定できないものばっか。

設定するのは以下。一般的なものなのでコード丸ごと備忘として貼っておこうかと。

- EBS パブリックアクセスをブロック
- EBS 暗号化をデフォルトで有効化
- S3 パブリックアクセスをブロック

```typescript
import * as ec2 from "aws-cdk-lib/aws-ec2";
import * as iam from "aws-cdk-lib/aws-iam";
import * as cdk from "aws-cdk-lib/core";
import * as cr from "aws-cdk-lib/custom-resources";
import type { Construct } from "constructs";

export class AccountSettingStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);
    const accountId = cdk.Stack.of(this).account;
    new ec2.CfnSnapshotBlockPublicAccess(this, "EbsBlockPublicAccess", {
      state: "block-all-sharing",
    });
    new cr.AwsCustomResource(this, "EnableEbsEncryptionByDefault", {
      onCreate: {
        service: "EC2",
        action: "enableEbsEncryptionByDefault",
        physicalResourceId: cr.PhysicalResourceId.of("EnableEbsEncryptionByDefault"),
      },
      onDelete: { service: "EC2", action: "disableEbsEncryptionByDefault" },
      policy: cr.AwsCustomResourcePolicy.fromStatements([
        new iam.PolicyStatement({
          actions: ["ec2:EnableEbsEncryptionByDefault", "ec2:DisableEbsEncryptionByDefault"],
          resources: ["*"],
        }),
      ]),
    });

    new cr.AwsCustomResource(this, "S3AccountPublicAccessBlock", {
      onCreate: {
        service: "S3Control",
        action: "putPublicAccessBlock",
        parameters: {
          AccountId: accountId,
          PublicAccessBlockConfiguration: {
            BlockPublicAcls: true,
            BlockPublicPolicy: true,
            IgnorePublicAcls: true,
            RestrictPublicBuckets: true,
          },
        },
        physicalResourceId: cr.PhysicalResourceId.of("S3AccountPublicAccessBlock"),
      },
      onDelete: {
        service: "S3Control",
        action: "deletePublicAccessBlock",
        parameters: {
          AccountId: accountId,
        },
      },
      policy: cr.AwsCustomResourcePolicy.fromStatements([
        new iam.PolicyStatement({
          effect: iam.Effect.ALLOW,
          actions: ["s3:PutAccountPublicAccessBlock", "s3:DeleteAccountPublicAccessBlock"],
          resources: ["*"],
        }),
      ]),
    });
  }
}


```
