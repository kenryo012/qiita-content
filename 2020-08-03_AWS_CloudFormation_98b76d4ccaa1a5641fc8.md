<!--
title:   Template error: Fn::Select cannot select nonexistent value at index 1
tags:    AWS,CloudFormation
id:      98b76d4ccaa1a5641fc8
private: false
-->
## 問題

以下のCloudFormationテンプレートでサブネットを東京リージョンに構築しようとしたところ、表題のエラーを吐きました。

```yaml
Resources:

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.2.0/24
      VpcId: !Ref CFnVPC
      AvailabilityZone: !Select [ 1, !GetAZs ]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PrivateSubnet2
```

![1.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/6ebcd95a-57b5-b87c-cbee-e8270bedd003.png)


## 原因

`Fn::GetAZs`はアカウントのデフォルトサブネットのAZを配列で返却します。
私のアカウントではデフォルトサブネットのAZを1つしか登録していませんでした。
問題のCloudFormationテンプレートで、`Fn::GetAZs`で取得したAZ配列の2番目を参照しますが、存在しないためエラーとなりました。

## 対処方法

デフォルトサブネットを作成し、AZを増やします。
東京リージョンには`ap-northeast-1a`,`ap-northeast-1c`,`ap-northeast-1d`の3つのAZが存在します。(実際は`ap-northeast-1b`もありますが、使用できるアカウントが制限されているため除外します)

アカウントが使用できるAZの確認

```
aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName'
```

![2.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/9623b09f-3ce9-dec5-2583-b2d5c4ee93a9.png)



デフォルトサブネットのAZを確認(`Fn::GetAZs`が返す値)

```
aws ec2 describe-subnets --filters "Name=default-for-az, Values=true" --query 'Subnets[].AvailabilityZone'
```

![3.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/fbbb24d7-8860-d1d5-275d-fd9ffda6278d.png)


デフォルトサブネットをAZに作成

```
aws ec2 create-default-subnet --availability-zone ap-northeast-1c
```

![4.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/51169a8f-fdf7-343b-3106-d676f0dac0bb.png)


## 確認

再度、CloudFormationテンプレートでスタック作成してみます。

![5.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/545707/91bfd3e0-a8a5-172d-f215-8985135ee736.png)


エラーなくサブネットを作成することができました。

## 参考

- [CloudFormation の Fn::GetAZs で ap-northeast-1d が返ってこなくてハマった](https://blog.manabusakai.com/2018/11/cloudformation-getazs-function/)
- [Template error: Fn::Select cannot select nonexistent value at index 1 #37
](https://github.com/widdix/aws-cf-templates/issues/37)
- [デフォルトサブネットの作成](https://docs.aws.amazon.com/ja_jp/vpc/latest/userguide/default-vpc.html#create-default-subnet)
- [東京リージョンの新しいアベイラビリティゾーン「ap-northeast-1d」がリリースされました。](https://dev.classmethod.jp/articles/new-az-ap-northeast-1d/)