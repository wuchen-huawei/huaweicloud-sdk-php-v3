[English](./README.md) | 简体中文


# 华为云开发者 PHP 软件开发工具包 （PHP SDK）

欢迎使用华为云 PHP SDK。

华为云 PHP SDK让您无需关心请求细节即可快速使用云服务器、虚拟私有云等多个华为云服务。

这里将向您介绍如何获取并使用华为云 PHP SDK。

## 在线示例

[API Explorer](https://apiexplorer.developer.huaweicloud.com/apiexplorer/overview) 提供API检索及平台调试，支持全量快速检索、可视化调试、帮助文档查看、在线咨询。

## 现在开始

- 要使用华为云 PHP SDK，您需要拥有云账号以及该账号对应的Access Key（AK）和Secret Access Key（SK）。 请在华为云控制台“我的凭证-访问密钥”页面上创建和查看您的AKSK。更多信息请查看[访问密钥](https://support.huaweicloud.com/usermanual-ca/zh-cn_topic_0046606340.html)。

- 华为云 PHP SDK 支持 PHP7.1 及以上版本。

## SDK 获取和安装

华为云 PHP SDK 支持 PHP7.1及以上版本。执行 ``PHP --version`` 检查当前PHP的版本信息。

无论您要使用哪个产品/服务的开发工具包，都必须安装 `huawei-cloud/huawei-cloud-sdk-core` 。以使用统一身份认证服务 IAM SDK 为例，您需要安装 `huawei-cloud/huawei-cloud-sdk-core` 和 `huawei-cloud/huawei-cloud-sdk-iam` ：

- 使用 composer 从仓库安装

    推荐使用 [Composer](http://getcomposer.org/) 安装SDK ，Composer是PHP的依赖管理工具，允许你在项目中声明依赖关系，并安装这些依赖：

    ```powershell
    # 安装 Composer
    curl -sS https://getcomposer.org/installer | php
    # 安装 IAM SDK
    composer require huawei-cloud/huawei-cloud-sdk-iam:~3.0.0
    ```

    或者，你可以编辑项目中已存在的composer.json文件，添加IAM SDK作为依赖： 

    ```json
    {
      "require": {
        "huawei-cloud/huawei-cloud-sdk-iam": "3.0.0"
      }
    }
    ```

    接下来使用`composer install`命令将SDK安装到您的项目中，核心库会一并安装到项目中。

    安装完毕后，你需要引入Composer的自动加载文件：

    ```php
    require 'vendor/autoload.php';
    ```

- 使用源码安装

    下载源码，在要使用的SDK的文件中 include ` autoload.php`:

    ```bash
    # 安装核心库
    require_once('/path/to/huawei-cloud/huawei-cloud-sdk-core/vendor/autoload.php');
    
    # 安装VPC服务库
    require_once('/path/to/huawei-cloud/huawei-cloud-sdk-iam/vendor/autoload.php');
    ```
    如果下载的源码中没有`vendor`目录，请先配置源码中composer.json文件后，执行下面指令：

    ```powershell
    composer install
    ```

## 开始使用

1. 导入依赖模块:

    ```php
    use HuaweiCloud\SDK\Core\Auth\GlobalCredentials;
    use HuaweiCloud\SDK\Core\Http\HttpConfig;
    use HuaweiCloud\SDK\Core\Exceptions\ConnectionException;
    use HuaweiCloud\SDK\Core\Exceptions\RequestTimeoutException;
    use HuaweiCloud\SDK\Core\Exceptions\ServiceResponseException;
    use HuaweiCloud\SDK\Iam\V3\IamClient;
    use HuaweiCloud\SDK\Iam\V3\Model\DeletePermanentAccessKeyRequest;
    use HuaweiCloud\SDK\Iam\V3\Model\CreateCredentialOption;
    use HuaweiCloud\SDK\Iam\V3\Model\CreatePermanentAccessKeyRequestBody;
    use HuaweiCloud\SDK\Iam\V3\Model\CreatePermanentAccessKeyRequest;
    use HuaweiCloud\SDK\Iam\V3\Model\ListPermanentAccessKeysRequest;
    use HuaweiCloud\SDK\Iam\V3\Model\ShowPermanentAccessKeyRequest;
    use HuaweiCloud\SDK\Iam\V3\Model\UpdatePermanentAccessKeyRequest;
    use HuaweiCloud\SDK\Iam\V3\Model\UpdateCredentialOption;
    use HuaweiCloud\SDK\Iam\V3\Model\UpdatePermanentAccessKeyRequestBody;
    ```

2. 配置客户端属性

    2.1 默认配置

    ```php
    # 使用默认配置
    $config = HttpConfig::getDefaultConfig();
    ```

    2.2 代理配置(可选)

    ```PHP
    # 使用代理服务器（可选）
    $config->setProxyProtocol('http');
    $config->setProxyHost('proxy.huawei.com');
    $config->setProxyPort(8080);
    $config->setProxyUser('username');
    $config->setProxyPassword('password');
    ```

    2.3 连接配置(可选)

    ```php
    # 配置连接超时（可选），支持统一指定超时时长timeout=timeout，或分别指定超时时长timeout=(connect timeout, read timeout)
    $config->setTimeout(3);
    ```

    2.4 SSL配置(可选)

    ```php
    # 配置跳过服务端证书验证（可选）
    $config->setIgnoreSslVerification(true);
    # 配置服务器端CA证书，用于SDK验证服务端证书合法性
    $config->setCertFile($yourCertFile);
    ```

3. 初始化认证信息

    **说明**：
    华为云服务存在两种部署方式，Region级服务和Global级服务。Global级服务当前仅支持IAM, TMS, EPS。

    Region级服务仅需要提供 projectId。Global级服务需要提供domainId。

    - `ak` 华为云账号 Access Key 。
    - `sk` 华为云账号 Secret Access Key 。
    - `project_id` 云服务所在项目 ID ，根据你想操作的项目所属区域选择对应的项目 ID 。
    - `domain_id` 华为云账号ID 。
    - `security_token` 采用临时AK/SK认证场景下的安全票据。

    3.1 使用永久AK/SK

    ```php
    # Region级服务
    $credentials = new BasicCredentials($ak,$sk,$projectId);
       
    # Global级服务
    $credentials = new GlobalCredentials($ak,$sk,$domainId);
    ```

    3.2 使用临时AK/SK

    首选需要获得临时AK、SK和SecurityToken，获取可以从永久AK/SK获得，或者通过委托授权首选获得。

    通过永久AK/SK获得可以参考文档：https://support.huaweicloud.com/api-iam/iam_04_0002.html, 对应IAM SDK 中的createTemporaryAccessKeyByToken方法。

    通过委托授权获得可以参考文档：https://support.huaweicloud.com/api-iam/iam_04_0101.html, 对应IAM SDK 中的createTemporaryAccessKeyByAgency方法。

    ```php
    # Region级服务
    $credentials = BasicCredentials(ak, sk, projectId).withSecurityToken(securityToken);
    # Global级服务
    $credentials = GlobalCredentials(ak, sk, domainId).withSecurityToken(securityToken);
    ```

4. 初始化客户端:

    ```php
    # 初始化指定云服务的客户端 {Service}Client ，以初始化 IamClient 为例
    $iamClient = IamClient::newBuilder(new IamClient)
        ->withHttpConfig($config)
        ->withEndpoint($endpoint)
        ->withCredentials(null)
        ->withStreamLogger($stream = 'php://stdout',$logLevel =Logger::INFO) // 日志打印至控制台
        ->withFileLogger($logPath='./test_log.txt', $logLevel = Logger::INFO) // 日志打印至文件
        ->build();
    ```

    **说明:**

    - `$endpoint` 华为云各服务应用区域和各服务的终端节点，详情请查看[地区和终端节点](https://developer.huaweicloud.com/endpoint)。
    - `withFileLogger`支持如下配置：
      - `$logPath`: 日志文件路径。
      - `$logLevel`: 日志级别，默认INFO。
      - `$logMaxFiles`: 日志文件个数，默认为5个。
    - `withStreamLogger`支持如下配置：
      - `$stream`: 流对象，默认'php://stdout'。
      - `$logLevel`: 日志级别，默认INFO。

    打开日志开关后，每次请求将打印访问日志，格式如下：`"{httpMethod} {uri}" {httpStatusCode} {responseContentLength} {requestId}`

    ```php
    [2020-10-16 03:10:29][INFO] "GET https://iam.cn-north-7.ulanqab.huawei.com/v3.0/OS-CREDENTIAL/credentials/W8VHHFEFPIJV6TFOUOQO"  200 244 7a68399eb8ed63fc91018426a7c4b8a0
    ```

5. 发送请求并查看响应.

    ```php
    # 初始化请求，以调用接口 listPermanentAccessKeys 为例
    $request = new ListPermanentAccessKeysRequest(array(userId=>"***"));
    $response = $iamClient->listPermanentAccessKeys($request);
    echo respones;
    ```

6. 异常处理

    | 一级分类 | 一级分类说明 | 二级分类 | 二级分类说明 |
    | :---- | :---- | :---- | :---- |
    | ConnectionException | 连接类异常 | HostUnreachableException | 网络不可达、被拒绝 |
    | | | SslHandShakeException | SSL认证异常 |
    | RequestTimeoutException | 响应超时异常 | CallTimeoutException | 单次请求，服务器处理超时未返回 |
    | | | RetryOutageException | 在重试策略消耗完成已后，仍无有效的响应 |
    | ServiceResponseException | 服务器响应异常 | ServerResponseException | 服务端内部错误，Http响应码：[500,] |
    | | | ClientRequestException | 请求参数不合法，Http响应码：[400， 500) |

    ```php
    # 异常处理
    try {
        $request = new ListPermanentAccessKeysRequest(array(userId=>"***"));
        $response = $iamClient->listPermanentAccessKeys($request);
    } catch (ConnectionException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (RequestTimeoutException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (ServiceResponseException $e) {
        echo "\n";
        echo $e->getHttpStatusCode(). "\n";
        echo $e->getErrorCode() . "\n";
        echo $e->getErrorMsg() . "\n";
    }
    ```

7. 异步场景

    ```php
    # 初始化异步客户端，以初始化 VpcAsyncClient 为例
    $iamClient = IamAsyncClient::newBuilder(new IamAsyncClient)
        ->withHttpConfig($config)
        ->withEndpoint($endpoint)
        ->withCredentials($credentials)
        ->build();
    
    # 发送异步请求
    $request = new ShowPermanentAccessKeyRequest(array('accessKey' => "W8VHHFEFPIJV6TFOUOQO"));
    $promise = $iamClient->showPermanentAccessKeyAsync($request);
    
    # 获取异步请求结果
    $response = $promise->wait();
    ```


## 代码实例

- 使用如下代码完成Credential管理中的增删改查接口，实际使用中请将 `IamClient` 替换为您使用的产品/服务相应的 `{Service}Client`。
- 调用前请根据实际情况替换如下变量：`{your ak string}`、 `{your sk string}`、 `{your endpoint}` 以及 `{your project id}`。

```php
<?php

require_once ".\\vendor\autoload.php";

use HuaweiCloud\SDK\Core\Auth\GlobalCredentials;
use HuaweiCloud\SDK\Core\Http\HttpConfig;
use HuaweiCloud\SDK\Core\Exceptions\ConnectionException;
use HuaweiCloud\SDK\Core\Exceptions\RequestTimeoutException;
use HuaweiCloud\SDK\Core\Exceptions\ServiceResponseException;
use HuaweiCloud\SDK\Iam\V3\IamClient;
use HuaweiCloud\SDK\Iam\V3\Model\DeletePermanentAccessKeyRequest;
use HuaweiCloud\SDK\Iam\V3\Model\CreateCredentialOption;
use HuaweiCloud\SDK\Iam\V3\Model\CreatePermanentAccessKeyRequestBody;
use HuaweiCloud\SDK\Iam\V3\Model\CreatePermanentAccessKeyRequest;
use HuaweiCloud\SDK\Iam\V3\Model\ListPermanentAccessKeysRequest;
use HuaweiCloud\SDK\Iam\V3\Model\ShowPermanentAccessKeyRequest;
use HuaweiCloud\SDK\Iam\V3\Model\UpdatePermanentAccessKeyRequest;
use HuaweiCloud\SDK\Iam\V3\Model\UpdateCredentialOption;
use HuaweiCloud\SDK\Iam\V3\Model\UpdatePermanentAccessKeyRequestBody;
use Monolog\Logger;

$ak = "DQEFICZJNKNN51HSSSS";
$sk = "is5WSYQ9eFgpzZt4l4a2I2XkPr9wsxdeld7LqxVf";
$endpoint = "https://iam.cn-north-7.ulanqab.huawei.com";
$domainId = "07692345re00d2490f88c01973e3e680";

$config = HttpConfig::getDefaultConfig();
$config->setIgnoreSslVerification(true);
$credentials = new GlobalCredentials($ak,$sk,$domainId);

$iamClient = IamClient::newBuilder(new IamClient)
    ->withHttpConfig($config)
    ->withEndpoint($endpoint)
    ->withCredentials(null)
    ->withStreamLogger($stream = 'php://stdout',$logLevel =Logger::INFO)
    ->withFileLogger($logPath='./test_log.txt', $logLevel = Logger::INFO)
    ->build();

function test_createPermanentAccessKey($iamClient)
{
    $createCredentialOption = new CreateCredentialOption(array('userId'=>'09e26bb67c00d58edc10c0190eb2518b',
        'description'=>'testCreateCredentialOption'));
    $createPermanentAccessKeyRequestBody = new CreatePermanentAccessKeyRequestBody(array('credential'=>$createCredentialOption));
    $createPermanentAccessKeyRequest = new CreatePermanentAccessKeyRequest(array('body'=>$createPermanentAccessKeyRequestBody));
    try {
        $response = $iamClient->createPermanentAccessKey($createPermanentAccessKeyRequest);
    } catch (ConnectionException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (RequestTimeoutException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (ServiceResponseException $e) {
        echo "\n";
        echo $e->getHttpStatusCode(). "\n";
        echo $e->getErrorCode() . "\n";
        echo $e->getErrorMsg() . "\n";
    }
    echo "\n";
    echo $response;
    return $response;
}

function test_keystoneCreateUserTokenByPassword($iamClient)
{
    $createCredentialOption = new CreateCredentialOption(array('userId'=>'09e26bb67c00d5891f10cedc0eb2518b',
        'description'=>'testCreateCredentialOption'));
    $createPermanentAccessKeyRequestBody = new CreatePermanentAccessKeyRequestBody(array('credential'=>$createCredentialOption));
    $createPermanentAccessKeyRequest = new CreatePermanentAccessKeyRequest(array('body'=>$createPermanentAccessKeyRequestBody));
    try {
        $response = $iamClient->keystoneCreateUserTokenByPassword($createPermanentAccessKeyRequest);
    } catch (ConnectionException $e) {
        $msg = $e->getMessage();
        echo '\n'. $msg .'\n';
    } catch (RequestTimeoutException $e) {
        $msg = $e->getMessage();
        echo '\n'. $msg .'\n';
    } catch (ServiceResponseException $e) {
        echo $e->getHttpStatusCode(). '\n';
        echo $e->getErrorCode() . '\n';
        echo $e->getErrorMsg() . '\n';
    }
    echo "\n";
    echo $response;
    return $response;
}

function test_deletePermanentAccessKey($iamClient)
{

    $deletePermanentAccessKeyRequest = new DeletePermanentAccessKeyRequest(array('accessKey'=>"PZQ07SVERTNGVIWP8TQK"));
    try {
        $response = $iamClient->deletePermanentAccessKey($deletePermanentAccessKeyRequest);
    } catch (ConnectionException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (RequestTimeoutException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (ServiceResponseException $e) {
        echo "\n";
        echo $e->getHttpStatusCode(). "\n";
        echo $e->getErrorCode() . "\n";
        echo $e->getErrorMsg() . "\n";
    }
    echo "\n";
    echo $response;
    return $response;
}

function test_listPermanentAccessKeys($iamClient)
{

    $listPermanentAccessKeysRequest = new ListPermanentAccessKeysRequest(array(userId=>"09e26bb67c0EDC891f10c0190eb2518b"));
    try {
        $response = $iamClient->listPermanentAccessKeys($listPermanentAccessKeysRequest);
    } catch (ConnectionException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (RequestTimeoutException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (ServiceResponseException $e) {
        echo "\n";
        echo $e->getHttpStatusCode(). "\n";
        echo $e->getErrorCode() . "\n";
        echo $e->getErrorMsg() . "\n";
    }
    echo "\n";
    echo $response;
    return $response;
}

function test_showPermanentAccessKey($iamClient)
{
    $showPermanentAccessKeyRequest = new 					  ShowPermanentAccessKeyRequest(array('accessKey'=>"W8VHHFEFPIERRTFOUOQO"));
    try {
        $response = $iamClient->showPermanentAccessKey($showPermanentAccessKeyRequest);
    } catch (ConnectionException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (RequestTimeoutException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (ServiceResponseException $e) {
        echo "\n";
        echo $e->getHttpStatusCode(). "\n";
        echo $e->getErrorCode() . "\n";
        echo $e->getErrorMsg() . "\n";
    }
    echo "\n";
    echo $response;
    echo "\n";
    echo $response->getCredential()->getUserId();
    echo "\n";
    echo $response->getCredential()->getAccess();
    echo "\n";
    echo $response->getCredential()->getStatus();
    echo "\n";
    echo $response->getCredential()->getCreateTime();
    return $response;
}

function test_updatePermanentAccessKey($iamClient)
{
    $updateCredentialOption = new UpdateCredentialOption(array('status'=>'inactive',
        'description'=>'updatePermanentAccessKey'));
    $updatePermanentAccessKeyRequestBody = new UpdatePermanentAccessKeyRequestBody(array('credential'=>$updateCredentialOption));
    $updatePermanentAccessKeyRequest = new UpdatePermanentAccessKeyRequest(array('accessKey'=>'W8VHHFEFPWERFTFOUOQO',
        'body'=>$updatePermanentAccessKeyRequestBody));
    try {
        $response = $iamClient->updatePermanentAccessKey($updatePermanentAccessKeyRequest);
    } catch (ConnectionException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (RequestTimeoutException $e) {
        $msg = $e->getMessage();
        echo "\n". $msg ."\n";
    } catch (ServiceResponseException $e) {
        echo "\n";
        echo $e->getHttpStatusCode(). "\n";
        echo $e->getErrorCode() . "\n";
        echo $e->getErrorMsg() . "\n";

    }
    echo "\n";
    echo $response;
    return $response;
}

$response = test_showPermanentAccessKey($iamClient);
$response = test_updatePermanentAccessKey($iamClient);
$response = test_showPermanentAccessKey($iamClient);
```
