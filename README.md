# AWS IoT button example

This is an Internet Button reference project: when a button on the device
is pressed, a cloud backend gets a notification and performs an action.
In this particular case, AWS Lambda function sends an email to the specific
email address. But, again, the action could be anything.

[![aws iot button](https://img.youtube.com/vi/yZ8VAxJ2XpA/0.jpg)](https://www.youtube.com/watch?v=yZ8VAxJ2XpA)

## Prerequisites

- Hardware: ESP8266 NodeMCU
- Amazon AWS account
- Amazon's `aws` management tool (see https://aws.amazon.com/cli)
- `mos` management tool installed
  (see [mos installation guide](https://mongoose-os.com/software.html))

## Architecture

<p align="center">
  <img src="aws_button.png" width="75%">
</p>

The data flow is as follows:

- User presses the button
- Device sends a message to the MQTT topic `DEVICE_ID/button_pressed`
- AWS IoT receives the message and calls AWS Lambda Function
- AWS Lambda Function publishes a message to the AWS SNS (Simple Notification Service)
- AWS SNS notifies subscribers: in this case, just sends a message to a single email address
- User receives the email

## Build instructions

1. Follow the [Cloud side setup](https://mongoose-os.com/aws-iot-starter-kit/#cloud) instructions to setup AWS CLI utility and your AWS credentials
2. Follow the [Device setup](https://mongoose-os.com/aws-iot-starter-kit/#dev) instructions to setup your device and provision it with AWS IoT
3. Download [this repository as a zip file](https://github.com/mongoose-os-apps/aws-iot-button/archive/master.zip) and extract this app on your computer
4. Exit any running `mos.exe` process
5. Open a command prompt (on Windows) or terminal (on Mac/Linux) and go to the extracted app.
6. You should be able to see the `mos.yml` file by running `dir mos.yml` command (on Windows) or `ls -l mos.yml` (on Mac/Linux)
7. Run `mos config-get device.id` to find out the device ID. On Windows,
   here and further, you might need to specify the full path to the `mos.exe`
   binary: `c:\path\to\mos.exe config-get device.id`
8. Run the following command to create AWS Cloud Formation stack.
   Change `$DEVICE_ID` to your actual device ID, and `$MY_EMAIL` to your email:

```
aws cloudformation create-stack \
    --stack-name my-internet-button \
    --parameters \
        ParameterKey=TopicName,ParameterValue=$DEVICE_ID/button_pressed \
        ParameterKey=SubscriptionEmail,ParameterValue=$MY_EMAIL \
    --capabilities CAPABILITY_IAM \
    --template-body file://aws_button_template.json
```

9. Wait until the stack creation is completed (it may take a few minutes).
   Alternatively, you can use the web UI to check the status and read event
   details: https://console.aws.amazon.com/cloudformation/home
   During the stack creation, AWS will send a Subscription Confirmation email,
   so check your email and confirm the subscription by following a link.
   Run the following command to ensure that the stack creation is complete:

```
aws cloudformation wait stack-create-complete --stack-name my-internet-button
```

11. Run `mos put fs/init.js` to copy the `fs/init.js` file to your device
12. Run `mos console` to attach to the device and see device logs 
13. Reboot your device by pressing a reboot button
13. When the device is connected to the AWS IoT, push the "flash"
    button on your device.
    In the device's console, you'll see a message like this:

```
Published: yes topic: esp8266_DA84C1/button_pressed message: {"free_ram":26824,"total_ram":44520}
```

Now, check your email. It'll contain a new message:

```
Button pressed: esp8266_DA84C1/button_pressed
```

14. Now you can go to your AWS dashboard and play with your stack.
  For example, you may add more subscriptions to the SNS: other than
  sending emails, it can also call some URL, send SMS, etc. And, of course,
  you can modify your lambda function to do whatever you want in response to
  the button press.

