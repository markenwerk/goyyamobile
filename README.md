# PHP Goyya Mobile simple API

[![Build Status](https://travis-ci.org/chroma-x/php-goyyamobile.svg?branch=master)](https://travis-ci.org/chroma-x/php-goyyamobile)
[![Test Coverage](https://codeclimate.com/github/chroma-x/php-goyyamobile/badges/coverage.svg)](https://codeclimate.com/github/chroma-x/php-goyyamobile/coverage)
[![Dependency Status](https://www.versioneye.com/user/projects/571f7845fcd19a0045442349/badge.svg)](https://www.versioneye.com/user/projects/571f7845fcd19a0045442349)
[![SensioLabs Insight](https://img.shields.io/sensiolabs/i/2a6b072a-0373-46e6-b6f4-10fc92cc91fb.svg)](https://insight.sensiolabs.com/projects/2a6b072a-0373-46e6-b6f4-10fc92cc91fb)
[![Code Climate](https://codeclimate.com/github/chroma-x/php-goyyamobile/badges/gpa.svg)](https://codeclimate.com/github/chroma-x/php-goyyamobile)
[![Latest Stable Version](https://poser.pugx.org/chroma-x/goyyamobile/v/stable)](https://packagist.org/packages/chroma-x/goyyamobile)
[![Total Downloads](https://poser.pugx.org/chroma-x/goyyamobile/downloads)](https://packagist.org/packages/chroma-x/goyyamobile)
[![License](https://poser.pugx.org/chroma-x/goyyamobile/license)](https://packagist.org/packages/chroma-x/goyyamobile)

Simple API abstraction for sending single short messages via [Goyya Mobile](https://www.goyya.com).


## Installation

````{json}
{
   	"require": {
        "chroma-x/goyyamobile": "~4.0"
    }
}
````

## Usage

### Autoloading and namesapce

````{php}  
require_once('path/to/vendor/autoload.php');
````

---

### Sending a single short message

#### API Client authentication

Authentication against the Goyya Mobile webservice requires a valid account ID and password or a valid auth token. 

##### API Client authentication by account credentials

````{php}
$shortMessage = new ChromaX\GoyyaMobile\Message();
$shortMessage
	->setAccountId('GOYYA_ACCOUNT_ID')
	->setAccountPassword('GOYYA_ACCOUNT_PASSWORD');
````

##### API Client authentication by auth token

````{php}
$shortMessage = new ChromaX\GoyyaMobile\Message();
$shortMessage->setAuthToken('GOYYA_AUTH_TOKEN');
````

#### Debug mode

If you enable the debug mode your messages will get submitted to your Goyya Mobile provider, but not transmitted to the receiver.

> By default the debug mode is disabled.

````{php}
$shortMessage->setDebugMode(true);
````

#### Preparing the message meta data

Settings up the meta data requires the following properties. 

- The receivers mobile number in international format, f.e. `+4915112345678`
- The senders name or mobile number in international format. Sender information should contain characters [a-z,A-Z,0-9] without whitepace only. Mobile numbers are allowed up to a length of 16 digits whith leading `00`. Sender names are allowed up to a length of 11 bytes.   
- If you want to delay the submission of the short message to a specific time, you can set the desired time by calling `setSubmissionDate(YOUR_DESIRED_TIMESTAMP)`method and enable the delayed submission by calling `setDelayedSubmission(true)`.  
- If your account is not bound to a specific plan, you can choose the plan you want to use (and pay) message wise. Call the `setSubmissionPlan` method with one of the following class constants as argument.  
  - `ChromaX\GoyyaMobile\Message::PLAN_BASIC`
  - `ChromaX\GoyyaMobile\Message::PLAN_ECONOMY`
  - `ChromaX\GoyyaMobile\Message::PLAN_QUALITY`

> **Attention:** Check your plan to make sure you are allowed to send short messages using a name instead of a mobile number.   
  By default the delayed submission if disabled.  
  The default plan is `ChromaX\GoyyaMobile\Message::PLAN_BASIC`.

````{php}
$shortMessage
	->setReceiver('RECEIVER_MOBILE_NUMBER')
	->setSender('SENDER_NAME_OR_MOBILE_NUMBER')
	->setDelayedSubmission(true)
	->setSubmissionDate(strtotime('+6 hours'))
	->setSubmissionPlan(ChromaX\GoyyaMobile\Message::PLAN_QUALITY);
````

#### Setting up the message content

The message content is set up by configuring the message type and content. 

- The message type defines how the content is handled. Call the `setMessageType` method with one of the following class constants to control how the messages content should get delivered. 
  - `ChromaX\GoyyaMobile\Message::MESSAGE_TYPE_TEXT_SMS` allows sending 160 bytes. Setting content above this range throws an `ChromaX\GoyyaMobile\Exception\InvalidArgumentException`. 
  - `ChromaX\GoyyaMobile\Message::MESSAGE_TYPE_OVERLONG_SMS` allows sending more than 160 bytes. If neccessary the content is submitted in multiple messages.  
  - `ChromaX\GoyyaMobile\Message::MESSAGE_TYPE_UTF8_SMS` allows sending more than 160 bytes of unicode characters.
- The message content is handled as defined by `setMessageType`.  

> The default message type is `ChromaX\GoyyaMobile\Message::MESSAGE_TYPE_TEXT_SMS`.  
Check the [GSM 7-bit default alphabet](https://en.wikipedia.org/wiki/GSM_03.38#GSM_7-bit_default_alphabet_and_extension_table_of_3GPP_TS_23.038_.2F_GSM_03.38) to make sure your content will be displayed as expected at the receivers phone. 

````{php}
$shortMessage
	->setMessageType(ChromaX\GoyyaMobile\Message::MESSAGE_TYPE_OVERLONG_SMS)
	->setMessage('Curabitur blandit tempus porttitor. ÄÖÜß~');
````

#### Submitting the message

The message gets submitted by calling the `submit` method. 

````{php}
$shortMessage->submit();
````

#### Getting information about the submission

After a successful submission the Goyya Mobile message ID and the effective number of messages submitted are available. 

````{php}
$messageId = $shortMessage->getMessageId();
$messageCount = $shortMessage->getMessageCount();
````

## Exception handling

Goyya Mobile simple API provides different types of exceptions. 

- `\InvalidArgumentException` is thrown on calling a setter with an invalid argument. 
- `ChromaX\CommonException` sub class exceptions get thrown if requesting the webservice of your Goyya Mobile provider fails for any reason. 

You can find more information about [PHP Common Exceptions at Github](https://github.com/chroma-x/php-common-exceptions).

All exceptions have a specific code to allow you to handle the exceptions properly. 

| Exception                                                           | Code | Description |
| :------------------------------------------------------------------ | ---: | :---------- |
| \InvalidArgumentException                                           |   10 | Receiver is no valid international mobile number; starts not with `+` or `00` |
| \InvalidArgumentException                                           |   11 | Receiver is no valid international mobile number; does contain non digit characters |
| \InvalidArgumentException                                           |   20 | Sender is not valid; does contain non [a-z,A-Z,0-9] characters |
| \InvalidArgumentException                                           |   21 | Sender is not a valid mobile number; it contains digits only but is longer than 16 bytes |
| \InvalidArgumentException                                           |   22 | Sender is not a valid name; it contains alphanumeric characters but is longer than 11 bytes |
| \InvalidArgumentException                                           |   30 | Message content is not valid; it is longer than 160 bytes but the message type is set to `ChromaX\GoyyaMobile\Message::MESSAGE_TYPE_TEXT_SMS` |
| ChromaX\CommonException\NetworkException\Base\NetworkException   |   40 | HTTP request failed |
| ChromaX\CommonException\ApiException\InvalidResponseException    |   41 | Response HTTP status code is not in the `2xx` range |
| ChromaX\CommonException\ApiException\Base\ApiException           |   42 | The Goyya Mobile provider webservice responded with an error |
| ChromaX\CommonException\ApiException\UnexpectedResponseException |   43 | The Goyya Mobile provider webservice responded with an unexpected and therefore not parsable response body |

## Full example

````{php}
require_once('path/to/vendor/autoload.php');

use ChromaX\CommonException;

try {
	$shortMessage = new ChromaX\GoyyaMobile\Message();
	$shortMessage
		->setAccountId('GOYYA_ACCOUNT_ID')
		->setAccountPassword('GOYYA_ACCOUNT_PASSWORD')
		->setDebugMode(true)
		->setReceiver('RECEIVER_MOBILE_NUMBER')
		->setSender('SENDER_NAME_OR_MOBILE_NUMBER')
		->setDelayedSubmission(true)
		->setSubmissionDate(strtotime('+6 hours'))
		->setSubmissionPlan(ChromaX\GoyyaMobile\Message::PLAN_QUALITY)
		->setMessageType(ChromaX\GoyyaMobile\Message::MESSAGE_TYPE_OVERLONG_SMS)
		->setMessage('Curabitur blandit tempus porttitor. ÄÖÜß~')
		->submit();
	echo 'Message ID ' . $shortMessage->getMessageId() . PHP_EOL;
	echo 'Message count ' . $shortMessage->getMessageCount() . PHP_EOL;
} catch (\InvalidArgumentException $exception) {
	echo $exception->getMessage() . PHP_EOL;
	echo $exception->getCode() . PHP_EOL;
} catch (CommonException\NetworkException\Base\NetworkException $exception) {
	echo $exception->getMessage() . PHP_EOL;
	echo $exception->getCode() . PHP_EOL;
} catch (CommonException\ApiException\Base\ApiException $exception) {
	echo $exception->getMessage() . PHP_EOL;
	echo $exception->getCode() . PHP_EOL;
}
````

## Contribution

Contributing to our projects is always very appreciated.  
**But: please follow the contribution guidelines written down in the [CONTRIBUTING.md](https://github.com/chroma-x/php-goyyamobile/blob/master/CONTRIBUTING.md) document.**

## TODOs

- ~~Decorate the code base with some unit tests.~~
- ~~Publish contribution guidelines.~~
- ~~Add TokenAuth based authentication instead of using account ID and password as credentials.~~
- Extend the basic API by some other useful methods like removing pending messages and sending bulk messages.

## License

PHP Goyya Mobile simple API is under the MIT license.
