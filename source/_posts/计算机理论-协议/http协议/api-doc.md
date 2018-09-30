---
title: 接口文档编写规范
tags:
  - HTTP协议
categories:
  - HTTP协议
abbrlink: 2736957875
date: 2018-01-15 20:13:15
---
# 接口
## 描述
## 请求参数
名称	类型	是否必需	描述(取值范围)

## 返回参数
### 数据类型

# 示例
## 请求参数

## 返回实例

## 错误码
| 错误代码 | 错误信息 | HTTP 状态码 | 说明 |
| :--- | :--- | :--- | :--- |
| Account.Arrearage | Your account has an outstanding payment. | 400 | 账号已经欠费。 |
| EncryptedOption.Conflict | Encryption value of disk and snapshot conflict. | 400 | 磁盘的加密属性和快照的加密属性不一致。 |
| IncorrectVSwitchStatus | The current status of virtual switch does not support this operation. | 400 | 指定的VSwitch状态不正确。 |
| InstanceDiskCategoryLimitExceed | The specified DataDisk.n.Size beyond the permitted range, or the capacity of snapshot exceeds the size limit of the specified disk category. | 400 | 指定的磁盘大小超过了该类型磁盘上限。 |
| InstanceDiskNumber.LimitExceed | The total number of specified disk in an instance exceeds. | 400 | 镜像中包含的数据盘和数据盘参数合并后，数据盘的总数超出限制。 |
| InvalidDataDiskSize.ValueNotSupported | The specified DataDisk.n.Size beyond the permitted range, or the capacity of snapshot exceeds the size limit of the specified disk category. | 400 | 指定的DataDisk.n.Size不合法（超出范围）。 |
| InvalidDescription.Malformed | The specified parameter Description is not valid. | 400 | 指定的Description格式不合法。 |
| InvalidDiskCategory.ValueNotSupported | The specified parameter DiskCategory is not valid. | 400 | 指定的DiskCategory不合法。 |
| InvalidDiskDescription.Malformed | The specified parameter SystemDisk.DiskDescription or DataDisk.n.Description is not valid. | 400 | 指定的参数SystemDisk.DiskDescription或参数DataDisk.n.Description不合法。 |
| InvalidDiskName.Malformed | The specified parameter SystemDisk.DiskName or DataDisk.n.DiskName is not valid. | 400 | 指定的参数SystemDisk.DiskName或DataDisk.n.DiskName不合法。 |
| InvalidHostName.Malformed | The specified parameter HostName is not valid. | 400 | 指定的HostName格式不合法。 |
| InvalidInstanceName.Malformed | The specified parameter InstanceName is not valid. | 400 | 指定的InstanceName格式不合法。 |
| InvalidInstanceType.ValueNotSupported | The specified InstanceType beyond the permitted range. | 400 | 指定的InstanceType不合法（超出可选范围）。 |
| InvalidInstanceType.ValueUnauthorized | The specified InstanceType is not authorized. | 400 | 指定的InstanceType未授权使用。 |
| InvalidInternetChargeType.ValueNotSupported | The specified InternetChargeType is not valid. | 400 | 指定的 InternetChargeType 不存在。 |
| InvalidIoOptimizedValue.ValueNotSupported | IoOptimized value not supported. | 400 | 指定的IoOptimized参数不支持。 |
| InvalidNetworkType.Mismatch | Specified parameter InternetMaxBandwidthIn or InternetMaxBandwidthOut conflict with instance network type. | 400 | 指定的InternetMaxBandwidthIn或InternetMaxBandwidthOut与实例网络类型不符合。 |
| InvalidNetworkType.Mismatch | Specified parameter InternetChargeType conflict with instance network type. | 400 | 指定的InternetChargeType与实例网络类型不符合。 |
| InvalidParameter | The specified parameter InternetMaxBandwidthOut is not valid. | 400 | 指定的InternetMaxBandwidthOut不合法（不是数字或超出范围）。 |
| InvalidParameter | The specified instance bandwidth is not valid. | 400 | 指定的带宽值不合法。 |
| InvalidParameter | The specified parameter Amount is not valid. | 400 | 指定的Amount参数不合法。\(范围：\[1,100\]\) |
| InvalidParameter.Bandwidth | The specified parameter Bandwidth is not valid. | 400 | 指定的带宽值不合法。 |
| InvalidParameter.Conflict | The specified image does not support the specified instance type. | 400 | 指定的InstanceType上不允许使用该指定的镜像。 |
| InvalidParameter.Encrypted.KmsNotEnabled | The encrypted disk need enable KMS. | 400 | 账户未开通KMS服务（需用户主动开通KMS服务）。 |
| InvalidParameter.EncryptedIllegal | The value of parameter Encrypted is illegal. | 400 | 传入的参数Encrypted非法。 |
| InvalidParameter.EncryptedNotSupported | Encrypted disk is not support in this region. | 400 | 所选择的region不支持加密特性。 |
| InvalidParameter.EncryptedNotSupported | Corresponding data disk category does not support encryption. | 400 | 对应的磁盘category不支持加密。 |
| InvalidParameter.Mismatch | Specified security group and virtual switch are not in the same VPC. | 400 | 指定安全组与VSwitch不属于同一个VPC。 |
| InvalidParameter.Mismatch | Specified virtual switch is not in the specified zone. | 400 | 指定的VSwitch不在指定Zone。 |
| InvalidPassword.Malformed | The specified parameter Password is not valid. | 400 | 指定的Password格式不合法。 |
| InvalidSnapshotId.BasedSnapshotTooOld | The specified snapshot is created before 2013-07-15. | 400 | 使用了2013-07-15之前创建的快照。 |
| InvalidSpotAliUid | The specified UID is not authorized to use SPOT instance. | 400 | 该账号不能使用spot instance。 |
| InvalidSpotAuthorized | The specified Spot param is unauthorized. | 400 | 该账号没有授权创建竞价实例。 |
| InvalidSpotPrepaid | The specified Spot type is not support PrePay Instance. | 400 | 竞价实例不支持预付费。 |
| InvalidSpotPriceLimit | The specified SpotPriceLimitis not valid. | 400 | SpotPriceLimit参数不合法。 |
| InvalidSpotPriceLimit.LowerThanPublicPrice | The specified parameter spotPriceLimit can’t be lower than current public price. | 400 | 出价低于当前系统公允价格。 |
| InvalidSpotStrategy | The specified SpotStrategy is not valid. | 400 | SpotStrategy参数不合法。 |
| InvalidSystemDiskCategory.ValueNotSupported | The specified parameter SystemDisk.Category is not valid. | 400 | 指定的参数SystemDisk.Category不合法。 |
| InvalidUserData.NotSupported | The specified parameter UserData only support the vpc and IoOptimized Instance. | 400 | UserData只能使用在VPC和I/O优化实例上。 |
| InvalidUserData.SizeExceeded | The specified parameter UserData exceeds the size. | 400 | 指定的UserData过长。 |
| InvalidHpcClusterId.NotFound | The specified HpcClusterId is not found. | 400 | 指定的`HpcClusterId`不存在。 |
| InvalidHpcClusterId.Creating | The specified HpcClusterId is creating. | 400 | 指定的`HpcClusterId`正在创建中。 |
| InvalidHpcClusterId.Unnecessary | The specified HpcClusterId is unnecessary. | 400 | 只有部分实例规格`InstanceType`支持指定[E-HPC](https://www.alibabacloud.com/help/zh/doc-detail/57677.htm)集群 ID。 |
| InvalidVSwitchId.Necessary | The HpcClusterId is necessary. | 400 | 该实例规格`InstanceType`需要指定[E-HPC](https://www.alibabacloud.com/help/zh/doc-detail/57677.htm)集群 ID，您需要传入`HpcClusterId`。 |
| MissingParameter | The input parameter VSwitchId that is mandatory for processing this request is not supplied. | 400 | 缺少必填参数VSwitchId。 |
| QuotaExceed.AfterpayInstance | The maximum number of Pay-As-You-Go instances is exceeded. | 400 | 用户的按量付费实例个数达到上限。 |
| QuotaExceeded | Living instances quota exceeded in this VPC. | 400 | VPC中实例数量超限。 |
| ResourceNotAvailable | Resource you requested is not available in this region or zone. | 400 | 指定Region或Zone内该资源不可用。 |
| CategoryNotSupported | The specified zone does not offer the specified disk category. | 403 | 该可用区无权创建指定种类的磁盘。 |
| DeleteWithInstance.Conflict | The specified disk is not a portable disk and cannot be set to DeleteWithInstance attribute. | 403 | 该磁盘不支持挂载与卸载。 |
| DependencyViolation.WindowsInstance | The instance creating is window, cannot use ssh key pair to login. | 403 | Windows 实例不能使用SSH密钥对。 |
| Forbidden | User not authorized to operate on the specified resource. | 403 | 用户没有权限操作。 |
| ImageNotSubscribed | The specified image has not be subscribed. | 403 | 没有订阅镜像市场的镜像。 |
| ImageNotSupportInstanceType | The specified image don’t support the InstanceType instance. | 403 | 指定镜像不支持该实例类型。 |
| ImageRemovedInMarket | The specified market image is not available, or the specified custom image includes product code because it is based on an image subscribed from marketplace, and that image in marketplace including exact the same product code has been removed. | 403 | 镜像市场的镜像已下架，或者自定义镜像中包含的product code对应的镜像市场镜像已经下架。 |
| InstanceDiskCategoryLimitExceed | The total size of specified disk category in an instance exceeds. | 403 | 指定的磁盘种类超过了单实例的最大容量。 |
| InstanceDiskNumLimitExceed | The number of specified disk in an instance exceeds. | 403 | 指定实例已经达到可挂载磁盘的最大值。 |
| InvalidDiskCategory.Mismatch | The specified disk categories combination is not supported. | 403 | 指定的磁盘类型组合不支持。 |
| InvalidDiskCategory.NotSupported | The specified disk category is not support the specified instance type. | 403 | 指定的磁盘类型不支持该实例类型。 |
| InvalidDiskSize.TooSmall | Specified disk size is less than the size of snapshot. | 403 | 指定的磁盘小于指定快照大小。 |
| InvalidInstanceType.ZoneNotSupported | The specified zone does not support this InstanceType. | 403 | 指定Zone不支持该实例类型。 |
| InvalidNetworkType.MismatchRamRole | Ram role cannot be attached to instances of Classic network type. | 403 | 实例RAM角色不能被用于经典网络。 |
| InvalidPayMethod | The specified billing method is not valid. | 403 | 指定的付费类型不存在。 |
| InvalidResourceType.NotSupported | This resource type is not supported; please try other resource types. | 403 | 创建实例的配置暂无可用区支持，请选择其他配置创建。 |
| InvalidSnapshotId.NotDataDiskSnapshot | The specified snapshot is system disk snapshot. | 403 | 系统盘快照不能创建数据盘。 |
| InvalidSnapshotId.NotReady | The specified snapshot has not completed yet. | 403 | 快照没有完成。 |
| InvalidSystemDiskCategory.ValueUnauthorized | The disk category is not authorized. | 403 | 磁盘种类未被授权使用。 |
| InvalidUser.PassRoleForbidden | The RAM user does not have the privilege to pass a role. | 403 | RAM用户不具有PassRole的权限。 |
| InvalidUserData.Forbidden | User not authorized to input the parameter UserData, please apply for permission UserData. | 403 | 用户没有权限使用UserData。 |
| InvalidVSwitchId.NotFound | The VSwitchId provided does not exist in our records. | 403 | 指定的VSwitchId不存在。 |
| IoOptimized.NotSupported | The specified image is not support IoOptimized Instance. | 403 | 指定的镜像不支持I/O优化实例。 |
| IoOptimized.NotSupported | Vpc is not support IoOptimized instance. | 403 | VPC不支持I/O优化实例。 |
| OperationDenied | The specified snapshot is not allowed to create disk. | 403 | 特定磁盘的快照不能创建磁盘或者快照不能创建磁盘。 |
| OperationDenied | The creation of Instance to the specified Zone is not allowed. | 403 | 该可用区无权创建实例或者Zone和Region不匹配。 |
| OperationDenied | The specified Image is disabled or is deleted. | 403 | 指定的镜像找不到。 |
| OperationDenied | Sales of this resource are temporarily suspended in the specified region; please try again later. | 403 | Region暂时停售按量付费（Pay-As-You-Go）实例。 |
| OperationDenied | The capacity of snapshot exceeds the size limit of the specified disk category or the specified category is not authorized. | 403 | 指定的DataDisk.n.Size不合法（超出范围）或者磁盘种类未被授权使用。 |
| OperationDenied | The type of the disk does not support the operation. | 403 | 指定磁盘类型不支持该操作。 |
| OperationDenied.NoStock | Sales of this resource are temporarily suspended in the specified region; please try again later. | 403 | 库存不足，请尝试其它系列或者其它可用区/地域的实例。 |
| QuotaExceed.BuyImage | The specified image is from the image market, You have not bought it or your quota has been exceeded. | 403 | 指定镜像没有购买或超过限制。 |
| QuotaExceed.PortableCloudDisk | The quota of portable cloud disk exceeds. | 403 | 可挂载的云磁盘数量已经达到上限（最多16块）。 |
| RegionUnauthorized | There is no authority to create instance in the specified region. | 403 | 用户无权使用该Region。 |
| SecurityGroupInstanceLimitExceed | The maximum number of instances in a security group is exceeded. | 403 | 该安全组内的Instance数量已经达到上限。 |
| Zone.NotOnSale | The specified zone is not available for purchase. | 403 | 创建实例的可用区已经关闭售卖，请更换其他可用区/地域。（VPC类型实例的交换机会限制购买可用区）。 |
| Zone.NotOpen | The specified zone is not granted to you to buy resources yet. | 403 | 创建实例的可用区没有对该用户开放售卖。 |
| ZoneId.NotFound | The specified zone does not exists. | 403 | 指定Zone不存在。 |
| DependencyViolation.IoOptimized | The specified InstanceType must be IoOptimized instance. | 404 | 指定的实例规格必须是I/O优化实例。 |
| HOSTNAME\_ILLEGAL | hostname is not valid. | 404 | 指定的HostName参数不合法。 |
| InvalidDataDiskSnapshotId.NotFound | The specified parameter DataDisk.n.SnapshotId is not valid. | 404 | 指定的DataDisk.n.SnapshotId没找到。 |
| InvalidImageId.NotFound | The specified ImageId does not exist. | 404 | 指定的镜像不存在。 |
| InvalidInstanceChargeType.NotFound | The InstanceChargeType does not exist in our records. | 404 | 指定的InstanceChargeType不存在。 |
| InvalidKeyPairName.NotFound | The specified KeyPairName does not exist in our records. | 404 | 指定的KeyPairName不存在。 |
| InvalidRamRole.NotFound | The specified RamRoleName does not exist. | 404 | 指定的RamRoleName不存在。 |
| InvalidRegionId.NotFound | The specified RegionId does not exist. | 404 | 指定的RegionId不存在。 |
| InvalidSecurityGroupId | The specified SecurityGroupId is invalid or does not exist. | 404 | 指定的SecurityGroupId不合法或不存在（实际情况也可能是该用户无权使用此SecurityGroup）。 |
| InvalidSystemDiskSize | The specified parameter SystemDisk.Size is invalid. | 404 | 指定的SystemDisk.Size不合法。 |
| InvalidSystemDiskSize.LessThanImageSize | The specified parameter SystemDisk.Size is less than the image size. | 404 | 指定的SystemDisk.Size小于镜像大小。 |
| InvalidSystemDiskSize.LessThanMinSize | The specified parameter SystemDisk.Size is less than the min size. | 404 | 指定的SystemDisk.Size小于磁盘大小下限。 |
| InvalidSystemDiskSize.MoreThanMaxSize | The specified parameter SystemDisk.Size is more than the max size. | 404 | 指定的SystemDisk.Size大于磁盘大小上限。 |
| InvalidVSwitchId.NotFound | Specified virtual switch does not exist. | 404 | 指定的VSwitch不存在。 |
| InvalidZoneId.NotFound | The specified ZoneId does not exist. | 404 | 指定Zone不存在。 |
| IoOptimized.NotSupported | The specified InstanceType is not support IoOptimized instance. | 404 | 指定的实例类型不支持 I/O 优化实例。 |
| OperationDenied | Another Instance is being created. | 404 | 正在创建另外的实例。 |
| PaymentMethodNotFound | No billing method has been registered on the account. | 404 | 该账号下没有付款方式。 |
| InternalError | The request processing has failed due to some unknown error,exception or failure. | 500 | 内部错误。 |