# alidayu
阿里大鱼短信封装composer插件


Rely

Aliyun\Core;
你的命名空间里需要有阿里云的核心sdk，如果没有，这里推荐使用这个人写的。
lokielse/aliyun-open-api-core


参考操作SDK：
```php
<?php namespace Myapp;
use Config,Log;
use Aliyun\Core as AliCore;
use alidayu\Request\Dysmsapi\V20170525 as AliSMS;

/**
 *
 * 阿里大鱼短信操作类
 *
 * @author sscs <laosql@163.com>
 *
 * Timer 2017/7/13
 *
 */

class SmsSDK
{

    private $accessKeyId;
    private $accessKeySecret;
    private $product;
    private $domain;
    private $region;
    private $acsClient;

    /**
     * init
     */
    public function __construct()
    {
        $this->accessKeyId = config('oss.sms.accessKeyId');
        $this->accessKeySecret = config('oss.sms.accessKeySecret');
        //短信API产品名
        $this->product = config('oss.sms.product');
        //短信API产品域名
        $this->domain = config('oss.sms.domain');
        //暂时不支持多Region
        $this->region = config('oss.sms.region');

        // 设置参数
        $regionIds = AliCore\Regions\EndpointConfig::getRegionIds();
        $regionIds = array(
                $this->region
        );
        $productDomains = array(
            new AliCore\Regions\ProductDomain($this->product, $this->domain)
        );
        $endpoint       = new AliCore\Regions\Endpoint($this->region, $regionIds, $productDomains);
        $endpoints      = array( $endpoint );
        AliCore\Regions\EndpointProvider::setEndpoints($endpoints);

        //初始化访问的acsCleint
        $profile = AliCore\Profile\DefaultProfile::getProfile($this->region, $this->accessKeyId, $this->accessKeySecret);

        $this->acsClient= new AliCore\DefaultAcsClient($profile);
    }

    /**
     * 短信发送
     *
     * @param $phone         必填  短信接收号码  支持以逗号分隔的形式进行批量调用,批量上限为
     *                             20个手机号码,批量调用相对于单条调用及时性稍有延迟
     *
     * @param $sms_signature 必填  短信签名 阿里云的签名名称，汉字
     * @param $sms_code      必填  短信模板code 
     * @param $array         选填  替换模板的变量，比如 array('name'=>'张竣升') 
     * @param $out_id        选填 流水号ID 
     *
     * @return bool
     */
    public function send($phone,$sms_signature,$sms_code,$array,$out_id = '')
    {

        try {
            $request = new AliSMS\SendSmsRequest;
            $request->setPhoneNumbers($phone);
            $request->setSignName($sms_signature);
            $request->setTemplateCode($sms_code);
            if(count($array)) $request->setTemplateParam(json_encode($array));
            if($out_id) $request->setOutId($out_id);
            //发起访问请求
            $acsResponse = $this->acsClient->getAcsResponse($request);
        } catch (Exception $e) {
            Log::error('阿里大鱼短信服务错误(Try):'.$e.' -'.date('Y-m-d H:i:s',time()));
        }

        if($acsResponse->Code !== 'OK')
        {
            Log::error('阿里大鱼短信服务错误:'.$acsResponse->Message.' -'.date('Y-m-d H:i:s',time()));
            return false;
        }

        return true;
    }
}

调用：


$SmsSDK = new SmsSDK();
$phone = 15226810495;
$sms_signature = '张竣升测试';
$sms_code = 'SMS_77175036';
$array = array('name' => '张竣升','code'=>6661);
$out_id = '';
$res = $SmsSDK->send($phone,$sms_signature,$sms_code,$array,$out_id = '');
dd($res);
//true

```