[toc]
# 将arduino uno的数据上传到云平台

## 解决方案
加一块`esp8266`的单片机，连接wifi，在通过`mqtt协议`，既可以将传感器的数据上传到云平台，并且接收云端上传的数据。

ps，该份代码，本来是将`配送车的传感器信息`连接到云平台，并能反馈控制。但是由于esp8266供电(一连接uno，显示屏就暗了，好解决)和esp8266自动断电能重新WiFi连接的问题，还没有在配送车上启动。功能已经调试完成

## adruino uno方面代码
接收与esp8266/蓝牙的信号，同时有显示屏，继电器，温度传感器模组，具体实现可以看代码
```
/*
 *连接esp8266与uno的连接
 *将温度数据发送到云端，并能接收云端的指令
*/
/*------------库函数*----------------------*/
//液晶显示模块
#include <Wire.h> 
#include "LiquidCrystal_I2C.h"
// 此处0x27改成刚才检测到的地址即可。
LiquidCrystal_I2C lcd(0x27,16,4);
//蓝牙模块
#include <SoftwareSerial.h> 
// Pin10为RX，接HC05的TXD
// Pin11为TX，接HC05的RXD
SoftwareSerial BT(10, 11); 
/*--------------变量设置*----------------*/
int lock= 7;//电磁锁
float read_val;//读取数据
char  c_length[10];
char temp[] = "T";
String comdata = "";
int mark = 0;
/*-------------结构体数据--------------*/
struct {
  float temptrue ;//温度数据
  bool isReceive ;
  }send_data;
/*------------延时设置----------------------*/
unsigned long previousMillis = 0;
// 此处为1000毫秒，也即1秒钟，
const long interval = 1000;
/*-------------------函数头-------------------*/
void first_show(); //首次显示内容
void box_show(); //开箱显示
void tep_show(); //温度显示
void tep_send();//温度数据发送
/*---------------初始化----------------------*/
void setup(){
  lcd.init();//初始化
  lcd.backlight(); //开启lcd背光灯
  send_data.isReceive = false ;//发送数据标志位
  pinMode(lock, OUTPUT);
  pinMode(13, OUTPUT);
  Serial.begin(115200);
  BT.begin(38400);
  }
/*-----------------------主循环-----------------------*/
void loop(){
  unsigned long currentMillis = millis();
  read_val=analogRead(A0);//Connect LM35 on Analog 0
  send_data.temptrue = (float) read_val * (5/10.24); //转化为温度
  /*--------------间隔发送数据------------------*/
  if (currentMillis - previousMillis >= interval) {
    previousMillis = currentMillis;
    tep_send();
    }
  /*------------------蓝牙端控制-----------------------*/
  while(BT.available()){
    char c = BT.read();
    Serial.println("BT is ready!");
    if(c=='1'){
      box_show();
       }
     if(c=='2') {
      tep_show();
                }
     if(c=='3'){
       re_show();       
            }
      }
      /*----------------esp8266连接测试-----------------------*/
      //不断循环检测串口缓存，一个个读入字符串，
    while (Serial.available() > 0&&  Serial.available() <10 )
      {
        comdata += char(Serial.read());
        delay(2);
        
        mark = 1;
      }
      if(mark == 1){
        Serial.println(comdata);
        param_data();
        }
       first_show();
      }
     
/*-------------------具体函数实现部分-------------------*/
 //初始化显示
void first_show(){
      lcd.setCursor(0, 0); 
      lcd.print("The ROS car");
//      lcd.setCursor(0, 1); 
//      lcd.print("   ---2020.10.3");
  }
//开锁以及lcd显示
void box_show(){
      lcd.init();  // initialize the lcd 
      lcd.setCursor(0, 0);  
      lcd.print("open box");
      digitalWrite(lock,HIGH);
      delay(500);
      digitalWrite(lock,LOW);
      delay(3000);}
//显示温度
void tep_show(){
        lcd.init();
        lcd.setCursor(0, 0);
        lcd.print("Tep:");   
        lcd.print(send_data.temptrue);
        lcd.print("C");
        delay(3000);}
//重启屏幕
void re_show(){
      lcd.init();  // initialize the lcd 
      lcd.setCursor(0, 0);  
      lcd.print("Restart lcd");
      digitalWrite(13,HIGH);
      delay(3000);
  }
  //发送温度数据
void tep_send(){

  //float ->char
  dtostrf(send_data.temptrue,2,2,c_length);
  char temp[] = "T";
  //字符串拼接
  strcat(temp,c_length);
  Serial.println(temp); 
    }
void param_data(){
  for(int i = 0; i < comdata.length() ; i++){
        if(comdata[i] == '$'){
         send_data.isReceive = true;
          }
          /*-------------三种模式---------------*/
          //开锁以及lcd显示
         if(comdata[i] == '1'){
            if(send_data.isReceive)
            {  
            box_show();
             send_data.isReceive = false;
             mark =0;
             comdata = ""; 
              }
            }
            //显示温度
            if(comdata[i] == '2'){
            if(send_data.isReceive)
            {  
             tep_show();
             send_data.isReceive = false;
             mark =0;
             comdata = ""; 
              }
            }
            //重启屏幕
            if(comdata[i] == '3'){
            if(send_data.isReceive)
            {  
             re_show();
             send_data.isReceive = false;
             mark =0;
             comdata = ""; 
              }
            }
        }
        send_data.isReceive = false;
        mark =0;
        comdata = ""; 
       }
```

## esp8266 方面代码

由于用了阿里云提供的官方库，并且在提供的cpp里面进行了修改，主要是**字符串的发送与解析**，代码非常冗余。
主要是不会跳出**esp8266主函数**里面的循环，只能对sdk的内容进行修改。
### 主函数
```
/*-----------库函数----------------------*/
//wifi模块
#include <ESP8266WiFi.h>
static WiFiClient espClient;
// 引入阿里云 IoT SDK
#include "AliyunIoTSDK.h"
//软引脚设置
//#include<SoftwareSerial.h>
//D8 -> TX ,D7 ->RX
//SoftwareSerial mySerial(15, 13);  // RX, TX
/*--------- 设置产品和设备的信息，从阿里云设备信息里查看-----------*/
#define PRODUCT_KEY "xxxx"
#define DEVICE_NAME "esp8266"
#define DEVICE_SECRET "xxxxxxx"
#define REGION_ID "cn-shanghai"

/*------------wifi 设置-----------------------*/
#define WIFI_SSID "xxxx"
#define WIFI_PASSWD "xxxxx"

void setup()
{
    Serial.begin(115200);
   // mySerial.begin(115200);
    
    // 初始化 wifi
    wifiInit(WIFI_SSID, WIFI_PASSWD);
    
    // 初始化 iot，需传入 wifi 的 client，和设备产品信息
    AliyunIoTSDK::begin(espClient, PRODUCT_KEY, DEVICE_NAME, DEVICE_SECRET, REGION_ID);
    
    // 绑定一个设备属性回调，当远程修改此属性，会触发 powerCallback
    // PowerSwitch 是在设备产品中定义的物联网模型的 id
    AliyunIoTSDK::bindData("BtSwitch", BtSwitch);
    

}

void loop()
{
    AliyunIoTSDK::loop();
    //Serial.println("18");
//    if(Serial.available()){
//     float c = Serial.read();
//     Serial.println(c);
//    AliyunIoTSDK::send("TargetTemperature", c);
//      }
      
}

// 初始化 wifi 连接
void wifiInit(const char *ssid, const char *passphrase)
{
    WiFi.mode(WIFI_STA);
    WiFi.begin(ssid, passphrase);
    while (WiFi.status() != WL_CONNECTED)
    {
        delay(1000);
        Serial.println("WiFi not Connect");
    }
    Serial.println("Connected to AP");
}

// 开关修改的回调函数
void BtSwitch(JsonVariant p)
{
    int PowerSwitch = p["BtSwitch"];
    if(PowerSwitch == 1){
      //字符串发送
      Serial.write("$1");
      }
    else if(PowerSwitch == 2){
      Serial.write("$2");
      }
     else if(PowerSwitch == 3){
      Serial.write("$3");
      }
}
```

### 阿里云sdk
#### cpp部分
```
#include "AliyunIoTSDK.h"
#include <PubSubClient.h>
#include <SHA256.h>

#define CHECK_INTERVAL 1000
#define MESSAGE_BUFFER_SIZE 10
#define RETRY_CRASH_COUNT 5

static const char *deviceName = NULL;
static const char *productKey = NULL;
static const char *deviceSecret = NULL;
static const char *region = NULL;
static String txtMsg = "";
char char_one ;  
int flag_i =0 ;
String param_c ;
char test_str[100];
struct DeviceProperty
{
    String key;
    String value;
};

DeviceProperty PropertyMessageBuffer[MESSAGE_BUFFER_SIZE];

#define MQTT_PORT 1883

#define SHA256HMAC_SIZE 32
#define DATA_CALLBACK_SIZE 20

#define ALINK_BODY_FORMAT "{\"id\":\"123\",\"version\":\"1.0\",\"method\":\"thing.event.property.post\",\"params\":%s}"
#define ALINK_EVENT_BODY_FORMAT "{\"id\": \"123\",\"version\": \"1.0\",\"params\": %s,\"method\": \"thing.event.%s.post\"}"

static unsigned long lastMs = 0;
static int retry_count = 0;

static PubSubClient *client = NULL;

char AliyunIoTSDK::clientId[256] = "";
char AliyunIoTSDK::mqttUsername[100] = "";
char AliyunIoTSDK::mqttPwd[256] = "";
char AliyunIoTSDK::domain[150] = "";

char AliyunIoTSDK::ALINK_TOPIC_PROP_POST[150] = "";
char AliyunIoTSDK::ALINK_TOPIC_PROP_SET[150] = "";
char AliyunIoTSDK::ALINK_TOPIC_EVENT[150] = "";

static String hmac256(const String &signcontent, const String &ds)
{
    byte hashCode[SHA256HMAC_SIZE];
    SHA256 sha256;

    const char *key = ds.c_str();
    size_t keySize = ds.length();

    sha256.resetHMAC(key, keySize);
    sha256.update((const byte *)signcontent.c_str(), signcontent.length());
    sha256.finalizeHMAC(key, keySize, hashCode, sizeof(hashCode));

    String sign = "";
    for (byte i = 0; i < SHA256HMAC_SIZE; ++i)
    {
        sign += "0123456789ABCDEF"[hashCode[i] >> 4];
        sign += "0123456789ABCDEF"[hashCode[i] & 0xf];
    }

    return sign;
}

static void parmPass(JsonVariant parm)
{
    //    const char *method = parm["method"];
    for (int i = 0; i < DATA_CALLBACK_SIZE; i++)
    {
        if (poniter_array[i].key)
        {
            bool hasKey = parm["params"].containsKey(poniter_array[i].key);
            if (hasKey)
            {
                poniter_array[i].fp(parm["params"]);
            }
        }
    }
}
// 所有云服务的回调都会首先进入这里，例如属性下发
static void callback(char *topic, byte *payload, unsigned int length)
{
   // Serial.print("Message arrived [");
   // Serial.print(topic);
   // Serial.print("] ");
    payload[length] = '\0';
   // Serial.println((char *)payload);

    if (strstr(topic, AliyunIoTSDK::ALINK_TOPIC_PROP_SET))
    {

        StaticJsonDocument<200> doc;
        DeserializationError error = deserializeJson(doc, payload); //反序列化JSON数据

        if (!error) //检查反序列化是否成功
        {
            parmPass(doc.as<JsonVariant>()); //将参数传递后打印输出
        }
    }
}

static bool mqttConnecting = false;
void(* resetFunc) (void) = 0; //制造重启命令
void AliyunIoTSDK::mqttCheckConnect()
{
    if (client != NULL && !mqttConnecting)
    {
        if (!client->connected())
        {
            client->disconnect();
            Serial.println("Connecting to MQTT Server ...");
            mqttConnecting = true;
            if (client->connect(clientId, mqttUsername, mqttPwd))
            {
                Serial.println("MQTT Connected!");
            }
            else
            {
                Serial.print("MQTT Connect err:");
                Serial.println(client->state());
                retry_count++;
                if(retry_count > RETRY_CRASH_COUNT){
                    resetFunc();
                }
            }
            mqttConnecting = false;
        }
        else
        {
            //Serial.println("state is connected");
           // read_serial_data();
            retry_count = 0;
        }
    }
}

void AliyunIoTSDK::begin(Client &espClient,
                         const char *_productKey,
                         const char *_deviceName,
                         const char *_deviceSecret,
                         const char *_region)
{

    client = new PubSubClient(espClient);
    productKey = _productKey;
    deviceName = _deviceName;
    deviceSecret = _deviceSecret;
    region = _region;
    long times = millis();
    String timestamp = String(times);

    sprintf(clientId, "%s|securemode=3,signmethod=hmacsha256,timestamp=%s|", deviceName, timestamp.c_str());

    String signcontent = "clientId";
    signcontent += deviceName;
    signcontent += "deviceName";
    signcontent += deviceName;
    signcontent += "productKey";
    signcontent += productKey;
    signcontent += "timestamp";
    signcontent += timestamp;

    String pwd = hmac256(signcontent, deviceSecret);

    strcpy(mqttPwd, pwd.c_str());

    sprintf(mqttUsername, "%s&%s", deviceName, productKey);
    sprintf(ALINK_TOPIC_PROP_POST, "/sys/%s/%s/thing/event/property/post", productKey, deviceName);
    sprintf(ALINK_TOPIC_PROP_SET, "/sys/%s/%s/thing/service/property/set", productKey, deviceName);
    sprintf(ALINK_TOPIC_EVENT, "/sys/%s/%s/thing/event", productKey, deviceName);

    sprintf(domain, "%s.iot-as-mqtt.%s.aliyuncs.com", productKey, region);
    client->setServer(domain, MQTT_PORT); /* 连接WiFi之后，连接MQTT服务器 */
    client->setCallback(callback);

    mqttCheckConnect();
}

void AliyunIoTSDK::loop()
{
    client->loop();
    read_serial_data();
     param_read_data();
    if (millis() - lastMs >= CHECK_INTERVAL)
    {
        lastMs = millis();
        mqttCheckConnect();
        send_data_lot();
        messageBufferCheck();
    }
}

void AliyunIoTSDK::sendEvent(const char *eventId, const char *param)
{
    char topicKey[156];
    sprintf(topicKey, "%s/%s/post", ALINK_TOPIC_EVENT, eventId);
    char jsonBuf[1024];
    sprintf(jsonBuf, ALINK_EVENT_BODY_FORMAT, param, eventId);
    //Serial.println(jsonBuf);
    boolean d = client->publish(topicKey, jsonBuf);
  //  Serial.print("publish:0 成功:");
    //Serial.println(d);
}
void AliyunIoTSDK::sendEvent(const char *eventId)
{
    sendEvent(eventId, "{}");
}
unsigned long lastSendMS = 0;

// 检查是否有数据需要发送
void AliyunIoTSDK::messageBufferCheck()
{
    int bufferSize = 0;
    for (int i = 0; i < MESSAGE_BUFFER_SIZE; i++)
    {
        if (PropertyMessageBuffer[i].key.length() > 0)
        {
            bufferSize++;
        }
    }
    // Serial.println("bufferSize:");
    // Serial.println(bufferSize);
    if (bufferSize > 0)
    {
        if (bufferSize >= MESSAGE_BUFFER_SIZE)
        {
            sendBuffer();
        }
        else
        {
            unsigned long nowMS = millis();
            // 3s 发送一次数据
            if (nowMS - lastSendMS > 5000)
            {
                sendBuffer();
                lastSendMS = nowMS;
            }
        }
    }
}

// 发送 buffer 数据
void AliyunIoTSDK::sendBuffer()
{
    int i;
    String buffer;
    for (i = 0; i < MESSAGE_BUFFER_SIZE; i++)
    {
        if (PropertyMessageBuffer[i].key.length() > 0)
        {
            buffer += "\"" + PropertyMessageBuffer[i].key + "\":" + PropertyMessageBuffer[i].value + ",";
            PropertyMessageBuffer[i].key = "";
            PropertyMessageBuffer[i].value = "";
        }
    }

    buffer = "{" + buffer.substring(0, buffer.length() - 1) + "}";
    send(buffer.c_str());
}

void addMessageToBuffer(char *key, String value)
{
    int i;
    for (i = 0; i < MESSAGE_BUFFER_SIZE; i++)
    {
        if (PropertyMessageBuffer[i].key.length() == 0)
        {
            PropertyMessageBuffer[i].key = key;
            PropertyMessageBuffer[i].value = value;
            break;
        }
    }
}
void AliyunIoTSDK::send(const char *param)
{

    char jsonBuf[1024];
    sprintf(jsonBuf, ALINK_BODY_FORMAT, param);
    //Serial.println(jsonBuf);
    boolean d = client->publish(ALINK_TOPIC_PROP_POST, jsonBuf);
//    Serial.print("publish:0 成功:");
//    Serial.println(d);
}
void AliyunIoTSDK::send(char *key, float number)
{
    addMessageToBuffer(key, String(number));
    messageBufferCheck();
}
void AliyunIoTSDK::send(char *key, int number)
{
    addMessageToBuffer(key, String(number));
    messageBufferCheck();
}
void AliyunIoTSDK::send(char *key, double number)
{
    addMessageToBuffer(key, String(number));
    messageBufferCheck();
}

void AliyunIoTSDK::send(char *key, char *text)
{
    addMessageToBuffer(key, "\"" + String(text) + "\"");
    messageBufferCheck();
}

int AliyunIoTSDK::bindData( char *key, poniter_fun fp)
{
    int i;
    for (i = 0; i < DATA_CALLBACK_SIZE; i++)
    {
        if (!poniter_array[i].fp)
        {
            poniter_array[i].key = key;
            poniter_array[i].fp = fp;
            return 0;
        }
    }
    return -1;
}

int AliyunIoTSDK::unbindData(char *key)
{
    int i;
    for (i = 0; i < DATA_CALLBACK_SIZE; i++)
    {
        if (!strcmp(poniter_array[i].key, key))
        {
            poniter_array[i].key = NULL;
            poniter_array[i].fp = NULL;
            return 0;
        }
    }
    return -1;
}

void AliyunIoTSDK::read_serial_data(){
    if(Serial.available()>0){      
     char_one = (char)Serial.read();
     if (char_one == '\n') {
           txtMsg +=char_one; 
           //Serial.println(txtMsg);
            
        }else {  
           txtMsg +=char_one; 
        }
    // Serial.println("ok");
    }
}
void AliyunIoTSDK::param_read_data(){
  int i =100;

  for(i=1;txtMsg[i];i++){
     param_c +=  txtMsg[i];
     test_str[i-1] = txtMsg[i];
    if (txtMsg[0] = 'T'){
       flag_i = 1;
      }
    else{
      flag_i = 0;
      } 
   if (txtMsg[i] == '\n'){
       txtMsg = ""; 
       return ;
      }

    }
    }  
 void AliyunIoTSDK::send_data_lot(){
  switch(flag_i){
    case 0:
    memset(test_str,0,sizeof(test_str));
    param_c = "";
     break;
    case 1:
     float send_f;
     char* pEnd;
     send_f = strtof(test_str,&pEnd);
     send("TargetTemperature", send_f);
     break;
      
    }
  }
```

#### 函数头部分
```
#ifndef ALIYUN_IOT_SDK_H
#define ALIYUN_IOT_SDK_H
#include <Arduino.h>
#include <ArduinoJson.h>
#include "Client.h"

typedef void (*poniter_fun)(JsonVariant ele); //定义一个函数指针

typedef struct poniter_desc
{
  char *key;
  poniter_fun fp;
} poniter_desc, *p_poniter_desc;

// 最多绑定20个回调
static poniter_desc poniter_array[20];
static p_poniter_desc p_poniter_array;

class AliyunIoTSDK
{
private:
  // mqtt 链接信息，动态生成的
  static char mqttPwd[256];
  static char clientId[256];
  static char mqttUsername[100];
  static char domain[150];
 
  // 定时检查 mqtt 链接
  static void mqttCheckConnect();

  static void messageBufferCheck();
  static void sendBuffer();
public:
  
  // 标记一些 topic 模板
  static char ALINK_TOPIC_PROP_POST[150];
  static char ALINK_TOPIC_PROP_SET[150];
  static char ALINK_TOPIC_EVENT[150];
  // 在主程序 loop 中调用，检查连接和定时发送信息
  static void loop();

  static void read_serial_data();
  static void param_read_data();
  static void send_data_lot();

  /**
   * 初始化程序
   * @param ssid wifi名
   * @param passphrase wifi密码
   */
  static void begin(Client &espClient,
                    const char *_productKey,
                    const char *_deviceName,
                    const char *_deviceSecret,
                    const char *_region);

  /**
   * 发送数据
   * @param param 字符串形式的json 数据，例如 {"${key}":"${value}"}
   */
  static void send(const char *param);
  /**
   * 发送 float 格式数据
   * @param key 数据的 key
   * @param number 数据的值
   */
  static void send(char *key, float number);
  /**
   * 发送 int 格式数据
   * @param key 数据的 key
   * @param number 数据的值
   */
  static void send(char *key, int number);
  /**
   * 发送 double 格式数据
   * @param key 数据的 key
   * @param number 数据的值
   */
  static void send(char *key, double number);
  /**
   * 发送 string 格式数据
   * @param key 数据的 key
   * @param text 数据的值
   */
  static void send(char *key, char *text);

  /**
   * 发送事件到云平台（附带数据）
   * @param eventId 事件名，在阿里云物模型中定义好的
   * @param param 字符串形式的json 数据，例如 {"${key}":"${value}"}
   */
  static void sendEvent(const char *eventId, const char *param);
  /**
   * 发送事件到云平台（空数据）
   * @param eventId 事件名，在阿里云物模型中定义好的
   */
  static void sendEvent(const char *eventId);

  /**
   * 绑定回调，所有云服务下发的数据都会进回调
   */
  // static void bind(MQTT_CALLBACK_SIGNATURE);

  /**
   * 绑定事件回调，云服务下发的特定事件会进入回调
   * @param eventId 事件名
   */
  // static void bindEvent(const char * eventId, MQTT_CALLBACK_SIGNATURE);
  /**
   * 绑定属性回调，云服务下发的数据包含此 key 会进入回调，用于监听特定数据的下发
   * @param key 物模型的key
   */
  static int bindData(char *key, poniter_fun fp);
  /**
   * 卸载某个 key 的所有回调（慎用）
   * @param key 物模型的key
   */
  static int unbindData(char *key);
};
#endif
```