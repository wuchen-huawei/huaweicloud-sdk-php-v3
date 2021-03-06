English | [简体中文](./README_CN.md)

# HuaweiCloud PHP Software Development Kit (PHP SDK)

The HuaweiCloud PHP SDK allows you to easily work with Huawei Cloud services such as Elastic Compute Service (ECS) and Virtual Private Cloud(VPC) without the need to handle API related tasks.

This document introduces how to obtain and use HuaweiCloud PHP SDK.

## Getting Started

- To use HuaweiCloud PHP SDK, you must have Huawei Cloud account as well as the Access Key and Secret key of the HuaweiCloud account.

    The accessKey is required when initializing `{Service}Client`. You can create an AccessKey in the Huawei Cloud console. For more information, see [My Credentials](https://support.huaweicloud.com/en-us/usermanual-ca/en-us_topic_0046606340.html).

- HuaweiCloud PHP SDK requires PHP 7.2 or later.


## Install PHP SDK

HuaweiCloud PHP SDK supports PHP 7.2 or later. Run ``PHP --version`` to check the version of PHP.

You must install `huawei-cloud/huawei-cloud-sdk-core` library no matter which product/service development kit you need to use. Take using IAM SDK for example, you need to install `huawei-cloud/huawei-cloud-sdk-core` library and `huawei-cloud/huawei-cloud-sdk-iam` library: 

- Use composer

    The recommended way to install SDK is with [Composer](https://getcomposer.org/). Composer is a dependency management tool for PHP that allows you to declare the dependencies your project needs and installs them into your project. 

    ```bash
    # Install Composer
    curl -sS https://getcomposer.org/installer | php
     
    # Install the IAM management library
    composer require huawei-cloud/huawei-cloud-sdk-iam:~3.0.0
    ```
    Alternatively, you can specify IAM SDK as a dependency in your project's existing composer.json file: 

    ```powershell
    {
      "require": {
        "huawei-cloud/huawei-cloud-sdk-iam": "3.0.0"
      }
    }
    ```

    Next, use the `composer install` command to install the SDK into your project, and the `huawei-cloud/huawei-cloud-sdk-core` library will be installed as a dependency.

    After installing, you need to require Composer's autoloader: 

    ```php
    require 'vendor/autoload.php';
    ```

- Install from source

    Download the source code and include `autoload.php` in the SDK file to be used.

    ```bash
    # Install the Core library
    require_once('/path/to/huawei-cloud/huawei-cloud-sdk-core/vendor/autoload.php');
    
    # Install the IAM management library
    require_once('/path/to/huawei-cloud/huawei-cloud-sdk-iam/vendor/autoload.php');
    ```

## Use PHP SDK

1. Import the required modules as follows:

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

2. Config `{Service}Client` Configurations

    2.1 Use default configuration

    ```php
    #  Use default configuration
    $config = HttpConfig::getDefaultConfig();
    ```

    2.2 Proxy(Optional)

    ```php
    # Use Proxy if needed
    $config->setProxyProtocol('http');
    $config->setProxyHost('proxy.huawei.com');
    $config->setProxyPort(8080);
    $config->setProxyUser('username');
    $config->setProxyPassword('password');
    ```

    2.3 Connection(Optional)

    ```php
    # seconds to wait for the server to send data before giving up, as a float, or (connect timeout, read timeout)
    $config->setTimeout(3);
    ```

    2.4 SSL Certification(Optional)

    ```php
    # Skip ssl certifaction checking while using https protocol if needed
    $config->setIgnoreSslVerification(true);
    # Server ca certification if needed
    $config->setCertFile($yourCertFile);
    ```

3. Initialize Credentials

    **Notice:**
    There are two types of HUAWEI CLOUD services, regional services and global services. 
    Global services currently only support IAM, TMS, EPS.

    For Regional services' authentication, projectId is required. 
    For global services' authentication, domainId is required. 

    - `ak` is the access key ID for your account.
    - `sk` is the secret access key for your account.
    - `project_id` is the ID of your project depending on your region which you want to operate.
    - `domain_id` is the account ID of HUAWEI CLOUD.
    - `security_token` is the security token when using temporary AK/SK.

    3.1 Use permanent AK/SK

    ```php
    # Region services
    $credentials = new BasicCredentials($ak,$sk,$projectId);
       
    # Global services
    $credentials = new GlobalCredentials($ak,$sk,$domainId);
    ```

    3.2 Use temporary AK/SK

    It's preferred to obtain temporary access key, security key and security token first, which could be obtained through permanent access key and security key or through an agency.

    Obtaining a temporary access key token through permanent access key and security key, you could refer to document: https://support.huaweicloud.com/en-us/api-iam/iam_04_0002.html . The API mentioned in the document above corresponds to the method of createTemporaryAccessKeyByToken in IAM SDK.

    Obtaining a temporary access key and security token through an agency, you could refer to document: https://support.huaweicloud.com/en-us/api-iam/iam_04_0101.html . The API mentioned in the document above corresponds to the method of createTemporaryAccessKeyByAgency in IAM SDK.

    ```php
    # Region services
    $credentials = BasicCredentials(ak, sk, projectId).withSecurityToken(securityToken);
       
    # Global services
    $credentials = GlobalCredentials(ak, sk, domainId).withSecurityToken(securityToken);
    ```

4. Initialize the `{Service}Client` instance:

    ```php
    # Initialize specified service client instance, take VpcClient for example
    $iamClient = IamClient::newBuilder(new IamClient)
        ->withHttpConfig($config)
        ->withEndpoint($endpoint)
        ->withCredentials(null)
        ->withStreamLogger($stream = 'php://stdout',$logLevel =Logger::INFO)  // Write log files
        ->withFileLogger($logPath='./test_log.txt', $logLevel = Logger::INFO)  // Write log to console
        ->build();
    ```

    **where:**

    - $`endpoint`: service specific endpoints, see [Regions and Endpoints](https://developer.huaweicloud.com/intl/en-us/endpoint).
    - `withFileLogger`:
        - `$logPath`: log file path.
        - `$logLevel`: log level, default is INFO.
        - `$logMaxFiles`: count of log file, default is 5.
    - `withStreamLogger`:
        - `$stream`: stream object, default is sys.stdout.
        - `$logLevel`: log level, default is INFO.

    After enabled log, the SDK will print the access log by default, every request will be recorded in console like: `"{httpMethod} {uri}" {httpStatusCode} {responseContentLength} {requestId}`

    ```php
    [2020-10-16 03:10:29][INFO] "GET https://iam.cn-north-7.ulanqab.huawei.com/v3.0/OS-CREDENTIAL/credentials/W8VHHFEFPIJV6TFOUOQO"  200 244 7a68399eb8ed63fc91018426a7c4b8a0
    ```

5. Send a request and print response.

    ```php
    # Initialize a request and print response, take interface of listPermanentAccessKeys for example
    $request = new ListPermanentAccessKeysRequest(array(userId=>"***"));
    $response = $iamClient->listPermanentAccessKeys($request);
    echo respones;
    ```

6. Exceptions

    | Level 1 | Notice | Level 2 | Notice |
    | :---- | :---- | :---- | :---- |
    | ConnectionException | Connection error | HostUnreachableException | Host is not reachable |
    | | | SslHandShakeException | SSL certification error |
    | RequestTimeoutException | Request timeout | CallTimeoutException | timeout for single request |
    | | | RetryOutageException | no response after retrying |
    | ServiceResponseException | service response error | ServerResponseException | server inner error, http status code: [500,] |
    | | | ClientRequestException | invalid request, http status code: [400? 500) |

    ```php
    # handle exceptions
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

7. Asynchronous Requests

    ```php
    # Initialize asynchronous client, take IAMAsyncClient for example
    $iamClient = IamAsyncClient::newBuilder(new IamAsyncClient)
        ->withHttpConfig($config)
        ->withEndpoint($endpoint)
        ->withCredentials($credentials)
        ->build();
    
    # send asynchronous request
    $request = new ShowPermanentAccessKeyRequest(array('accessKey' => "W8VHHFEFPIJV6TFOUOQO"));
    $promise = $iamClient->showPermanentAccessKeyAsync($request);
    
    # get asynchronous response
    $response = $promise->wait();
    ```


## Code example

- The following example shows how to query a list of IAM in a specific region, you need to substitute your real `{Service}Client` for `IamClient` in actual use.
- Substitute the values for `{your ak string}`, `{your sk string}`, `{your endpoint string}` and `{your project id}`.



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
